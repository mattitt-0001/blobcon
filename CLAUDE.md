# BlobCon Convention App — Claude Context

## What This Project Is

A standalone single-HTML-file convention app for the Blobs Network platform.
File: `/Users/maticohen/Desktop/Blobs/convention.html`

Built during May 2026. The user is Mati Cohen (mati@veev.com).

---

## Live URLs

| What | URL |
|---|---|
| Live app | https://mattitt-0001.github.io/blobcon/convention.html |
| GitHub repo | https://github.com/mattitt-0001/blobcon |
| Supabase dashboard | https://supabase.com/dashboard/project/gnrcebzljstaulwlkubu |
| Supabase table editor | https://supabase.com/dashboard/project/gnrcebzljstaulwlkubu/editor |

---

## Credentials & Keys

| What | Value |
|---|---|
| Admin password (in-app) | `1234321` |
| Supabase project ref | `gnrcebzljstaulwlkubu` |
| Supabase project URL | `https://gnrcebzljstaulwlkubu.supabase.co` |
| Supabase anon key | `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImducmNlYnpsanN0YXVsd2xrdWJ1Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3Nzg0ODc1NzAsImV4cCI6MjA5NDA2MzU3MH0.y9KWyDkI4GL0bfjx0yzOrOKRZ-dPy9RUrB18SKVYmGw` |
| GitHub username | `mattitt-0001` |
| GitHub repo | `blobcon` |

---

## Architecture

- **Single file:** `convention.html` — all HTML, CSS, JS in one file. No build tools, no npm.
- **Backend:** Supabase (Postgres) accessed via REST API with plain `fetch()` — no Supabase SDK.
- **Chat:** Polls Supabase `messages` table every 3 seconds for new entries.
- **Admin auth:** Password `1234321` checked client-side → `sessionStorage.setItem('adminMode', 'true')`. Resets on tab close.
- **Seed data:** `seedData()` checks if `sessions` table is empty; if so, inserts all BlobCon 2025 dummy data.
- **Hosted:** GitHub Pages (auto-deploys ~60s after push to main branch).

## Supabase Tables

```
config     (key text PK, value jsonb)
sessions   (id text PK, day int, time text, title text, speaker text, room text, track text, description text)
attendees  (id text PK, name text, company text, role text, avatar_url text)
booths     (id text PK, company text, tagline text, description text, logo_url text, website text)
meetups    (id text PK, proposer text, participants text, datetime text, location text, notes text, status text)
messages   (id text PK, author text, text text, timestamp bigint)
```

All tables have RLS enabled with `for all using (true) with check (true)` — fully public read/write.

## Key DB Helper Functions (in convention.html)

```javascript
dbGet(table, qs)      // GET /rest/v1/{table}?select=*{qs}
dbInsert(table, data) // POST — insert one row or array of rows
dbPatch(table, id, data) // PATCH ?id=eq.{id}
dbDelete(table, id)   // DELETE ?id=eq.{id}
dbUpsert(table, data) // POST with Prefer: resolution=merge-duplicates
```

---

## Local Dev

```bash
cd /Users/maticohen/Desktop/Blobs
python3 -m http.server 8765
# open http://localhost:8765/convention.html
```

## Push to GitHub

```bash
cd /Users/maticohen/Desktop/Blobs
git add convention.html README.md CLAUDE.md
git commit -m "your message"
git push https://mattitt-0001:YOUR_TOKEN@github.com/mattitt-0001/blobcon.git main
```

Get a new GitHub token: https://github.com/settings/tokens → Generate new → check **repo** scope.

---

## Architecture History

1. **Firebase** (original plan) — Firestore + Realtime Database, CDN SDK. Failed: security rules blocked all writes despite `allow read, write: if true`. Never resolved.
2. **localStorage** (fallback) — Replaced Firebase entirely. Worked perfectly but data was per-browser only — not shared across users.
3. **Supabase** (current) — Replaced localStorage. Plain fetch() to Supabase REST API. Shared database, works from any browser, GitHub Pages compatible.

---

## Seed Data Summary (BlobCon 2025)

- **14 sessions** across 3 days (Keynote, Engineering, Community, Product, Growth, AI tracks)
- **12 attendees** (Alex Rivera, Mia Chen, Sam Obi, Priya Nair, etc.)
- **5 booths** (TokenForge, MetaLayer, SafeNet AI, ProtocolX, CommunityOS)
- Convention: June 12–14 2025, San Francisco, CA

---

## Design Spec & Plan

- Spec: `docs/superpowers/specs/2026-05-10-convention-app-design.md`
- Plan: `docs/superpowers/plans/2026-05-10-convention-app.md`
