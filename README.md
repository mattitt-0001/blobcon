# BlobCon 2025 — Convention App

A standalone single-HTML-file convention app for the Blobs Network platform. Built with Supabase as the shared backend — everyone who opens the URL sees the same data in real time.

---

## Live URL

**https://mattitt-0001.github.io/blobcon/convention.html**

---

## What It Does

- **Sessions** — Browse the full 3-day schedule (14 talks/workshops), grouped by day and sorted by time
- **Attendees** — Searchable directory of all registered attendees
- **Meetups** — Anyone can propose a meetup; admin can confirm or delete
- **Booths** — Sponsor/exhibitor directory with website links
- **Group Chat** — Real-time chat (polls every 3 seconds), shared across all users

---

## Admin Mode

Click the **🔐 Admin** button on the home banner.

**Password:** `1234321`

Admin mode lets you:
- Add / edit / delete sessions, attendees, booths
- Confirm or delete meetups
- Edit the convention banner (name, tagline, dates, location)

Admin mode lasts for the browser session (resets when you close the tab).

---

## Backend — Supabase

All data is stored in Supabase (Postgres database). Every user who opens the app reads from and writes to the same database.

| What | Where |
|---|---|
| Dashboard | https://supabase.com/dashboard/project/gnrcebzljstaulwlkubu |
| Table Editor | https://supabase.com/dashboard/project/gnrcebzljstaulwlkubu/editor |
| SQL Editor | https://supabase.com/dashboard/project/gnrcebzljstaulwlkubu/sql |
| Project URL | `https://gnrcebzljstaulwlkubu.supabase.co` |

**Tables:** `sessions`, `attendees`, `booths`, `meetups`, `messages`, `config`

The anon key is embedded in `convention.html`. Row-Level Security is enabled with public read/write policies (appropriate for an open convention app with a single admin password).

### Re-create tables from scratch (SQL Editor)

```sql
create table config (key text primary key, value jsonb);
create table sessions (id text primary key, day int, time text, title text, speaker text, room text, track text, description text);
create table attendees (id text primary key, name text, company text, role text, avatar_url text);
create table booths (id text primary key, company text, tagline text, description text, logo_url text, website text);
create table meetups (id text primary key, proposer text, participants text, datetime text, location text, notes text, status text);
create table messages (id text primary key, author text, text text, timestamp bigint);

alter table config enable row level security;
alter table sessions enable row level security;
alter table attendees enable row level security;
alter table booths enable row level security;
alter table meetups enable row level security;
alter table messages enable row level security;

create policy "public all" on config for all using (true) with check (true);
create policy "public all" on sessions for all using (true) with check (true);
create policy "public all" on attendees for all using (true) with check (true);
create policy "public all" on booths for all using (true) with check (true);
create policy "public all" on meetups for all using (true) with check (true);
create policy "public all" on messages for all using (true) with check (true);
```

---

## GitHub Pages — Hosting

Hosted on GitHub Pages. The repo is: **https://github.com/mattitt-0001/blobcon**

### To push changes after editing `convention.html`:

```bash
cd /Users/maticohen/Desktop/Blobs
git add convention.html
git commit -m "Update convention app"
git push https://mattitt-0001:YOUR_TOKEN@github.com/mattitt-0001/blobcon.git main
```

Get a new token at: https://github.com/settings/tokens → Generate new token → check **repo** scope.

GitHub Pages auto-deploys within ~60 seconds of a push.

---

## Local Development

Run a local server:
```bash
cd /Users/maticohen/Desktop/Blobs
python3 -m http.server 8765
```

Then open: http://localhost:8765/convention.html

---

## Architecture

- **Single file:** `convention.html` — all HTML, CSS, and JS in one file. No build tools, no npm, no dependencies.
- **Backend:** Supabase REST API called directly with `fetch()` — no SDK needed.
- **Chat:** localStorage stores your display name; messages poll Supabase every 3 seconds for new entries.
- **Admin auth:** Password checked client-side, session stored in `sessionStorage` (not localStorage — resets on tab close).
- **Seed data:** On first load, if the `sessions` table is empty, all BlobCon 2025 dummy data is inserted automatically.

### History of architecture decisions
1. Started with **Firebase** (Firestore + Realtime Database) — blocked by security rules, couldn't get writes working
2. Switched to **localStorage** — worked but data was per-browser only, not shared
3. Switched to **Supabase** — shared database, works from any browser, no complex setup
