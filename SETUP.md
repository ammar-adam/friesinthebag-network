# Fries in the Bag — Setup Guide

Get the app running locally and connect the backend (Supabase + OAuth) so leaderboards and Gmail/Outlook work.

---

## 1. Run locally

No build step. Open the repo and serve the folder:

```bash
cd fries-in-bag
npx serve .
# or
python -m http.server 8080
```

Then open `http://localhost:3000` (or 8080). The app works with **only LinkedIn CSV**; scores will be limited until you add credentials below.

---

## 2. Environment / configuration

All config is **inline in `index.html`** (no `.env` by default). Open `index.html` and find this block near the top of the `<script>`:

```javascript
const SUPABASE_URL = 'YOUR_SUPABASE_URL';
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
const GOOGLE_CLIENT_ID = 'YOUR_GOOGLE_CLIENT_ID';
const AZURE_CLIENT_ID = 'YOUR_AZURE_CLIENT_ID';
```

Replace the placeholders with your real values (see below). For production you can switch to reading from `window.__ENV__` or a small config file if you prefer.

---

## 3. Supabase (leaderboards)

Leaderboards are the only server-side feature; everything else runs in the browser.

### Create a project

1. Go to [supabase.com](https://supabase.com) and create a project.
2. In the dashboard: **Settings → API**. Copy:
   - **Project URL** → `SUPABASE_URL`
   - **anon public** key → `SUPABASE_ANON_KEY`

### Create the table

In Supabase: **SQL Editor → New query**. Run:

```sql
create table if not exists leaderboard (
  id uuid primary key default gen_random_uuid(),
  display_name text,
  school text,
  company text,
  potential_score numeric,
  actual_score numeric,
  tier_label text,
  created_at timestamptz default now(),
  opted_in boolean default true
);

-- Allow anonymous read/insert for the app (no auth required for this MVP)
alter table leaderboard enable row level security;

create policy "Allow public read leaderboard"
  on leaderboard for select
  using (opted_in = true);

create policy "Allow anon insert leaderboard"
  on leaderboard for insert
  with check (true);
```

### Optional: use Auth later

The app does **not** use Supabase Auth for the MVP (no sign-up required). If you add auth later:

- Create a `users` or `profiles` table and link `leaderboard` to `auth.uid()`.
- Tighten RLS so users can only insert/update their own row.

---

## 4. Google (Gmail + Calendar + Contacts)

Used for the “actual” score and to pull email/calendar metadata.

### Create OAuth credentials

1. [Google Cloud Console](https://console.cloud.google.com/) → create or select a project.
2. **APIs & Services → Library**: enable **Gmail API**, **Google People API** (Contacts), **Google Calendar API**.
3. **APIs & Services → Credentials → Create credentials → OAuth client ID**.
4. Application type: **Web application**.
5. **Authorized JavaScript origins**:
   - `http://localhost:3000` (or your dev URL)
   - `https://yourdomain.com` (production)
6. **Authorized redirect URIs**: add the same origins (e.g. `http://localhost:3000/`).  
   For Google’s OAuth 2.0 token client used in the app, the redirect is often handled by the library; if you use a different flow, add the exact redirect URI.
7. Copy the **Client ID** → `GOOGLE_CLIENT_ID` in `index.html`.

No client secret is needed for the token client flow used in the app.

---

## 5. Microsoft (Outlook + Calendar + Contacts)

Used as an alternative to Gmail for the “actual” score.

### Create an app registration

1. [Azure Portal](https://portal.azure.com) → **Microsoft Entra ID** (or Azure Active Directory) → **App registrations → New registration**.
2. Name: e.g. “Fries in the Bag”. Supported account types: **Accounts in any organizational directory and personal Microsoft accounts**.
3. Redirect URI: **Single-page application (SPA)** and add:
   - `http://localhost:3000` (or your dev URL)
   - `https://yourdomain.com` (production)
4. Register. Copy **Application (client) ID** → `AZURE_CLIENT_ID` in `index.html`.

### API permissions

1. **App registrations → your app → API permissions → Add a permission**.
2. **Microsoft Graph** → Delegated:
   - `Mail.Read`
   - `Calendars.Read`
   - `Contacts.Read`
   - `User.Read`
3. Grant admin consent if required by your org (not needed for personal accounts).

No client secret is required for the MSAL.js public client (popup) flow used in the app.

---

## 6. Summary checklist

| Item | Where to get it | Put in `index.html` |
|------|-----------------|----------------------|
| Supabase URL | Supabase → Settings → API → Project URL | `SUPABASE_URL` |
| Supabase anon key | Supabase → Settings → API → anon public | `SUPABASE_ANON_KEY` |
| Google Client ID | Google Cloud Console → Credentials → OAuth 2.0 Client ID | `GOOGLE_CLIENT_ID` |
| Azure Client ID | Azure Portal → App registrations → your app → Application (client) ID | `AZURE_CLIENT_ID` |

- **LinkedIn**: No config. Users upload their CSV export.
- **Optional sources** (WhatsApp, Telegram, Twitter/X, iCloud, Slack): File upload only; no API keys.

---

## 7. Deploy (e.g. Vercel)

1. Push the repo to GitHub (only `index.html`, `README.md`, `SETUP.md`, `package.json` / `package-lock.json` if you use them; no secrets).
2. In Vercel: **Import** the repo, deploy as a **static** site (no build command).
3. Set **Root Directory** to the folder that contains `index.html` if needed.
4. Add your production URLs to:
   - Google OAuth **Authorized JavaScript origins** and **Authorized redirect URIs**
   - Azure app **Redirect URI** (SPA).

After deploy, open the live URL and test:

- Upload LinkedIn CSV → “Calculate with what I have” or “show me the damage”.
- Connect Gmail or Outlook (with credentials set) and run again.
- Submit to leaderboard (with Supabase set) and check the **leaderboard** table in Supabase.

---

## 8. Troubleshooting

- **“Google OAuth not loaded”**: Ensure `https://accounts.google.com/gsi/client` is not blocked and `GOOGLE_CLIENT_ID` is set.
- **“Outlook sign-in failed”**: Check `AZURE_CLIENT_ID` and that the SPA redirect URI in Azure matches your current origin (e.g. `http://localhost:3000`).
- **Leaderboard insert fails**: In Supabase, confirm the table and RLS policies exist and the anon key is correct. Check the browser Network tab for the failing request and Supabase logs.
