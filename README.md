# Fries in the Bag Network

**See how connected you actually are vs. how connected you could be.**

Single-page web app that calculates your **potential** network score (from LinkedIn) and **actual** network score (from email, calendar, messaging). All processing is client-side; only leaderboard data is sent to Supabase.

## Run locally

Open `index.html` in a browser, or serve the folder:

```bash
npx serve .
# or
python -m http.server 8080
```

## Configuration

Replace placeholders in `index.html` (inside the script block):

- **Google (Gmail):** Set `GOOGLE_CLIENT_ID` to your OAuth 2.0 Client ID from [Google Cloud Console](https://console.cloud.google.com/apis/credentials). Enable Gmail API, People API, and Google Calendar API; add authorized JavaScript origins.
- **Microsoft (Outlook):** Set `AZURE_CLIENT_ID` to your App (client) ID from [Azure Portal](https://portal.azure.com) → App registrations. Add redirect URI (e.g. `http://localhost:3000` or your deployed URL). Request API permissions: Mail.Read, Calendars.Read, Contacts.Read, User.Read.
- **Supabase:** Set `SUPABASE_URL` and `SUPABASE_ANON_KEY` from your [Supabase](https://supabase.com) project.

## Supabase leaderboard table

Run in the SQL editor:

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

-- Optional: enable read access for anon
alter table leaderboard enable row level security;
create policy "Allow public read" on leaderboard for select using (true);
create policy "Allow anon insert" on leaderboard for insert with check (true);
```

## Data sources

- **Required:** LinkedIn Connections CSV (Settings → Data Privacy → Get a copy of your data → Connections).
- **Required (one of):** Gmail or Outlook (OAuth).
- **Optional:** WhatsApp (.txt), Telegram (JSON), iCloud Contacts (.vcf) uploads.

## Deploy

Static site. Deploy the repo (e.g. `index.html` at root) to Vercel, GitHub Pages, or any static host. Set the same OAuth redirect URIs and Supabase keys for production.
