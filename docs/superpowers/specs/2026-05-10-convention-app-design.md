# Convention App — Design Spec
**Date:** 2026-05-10  
**Project:** Blobs Network — BlobCon Convention Feature  
**Scope:** Standalone single-HTML-file app with Firebase backend

---

## Overview

A standalone convention app for the Blobs social network platform. Organizers host a named convention (e.g. "BlobCon 2025") and attendees can browse the session schedule, discover other attendees, plan meetups, explore sponsor booths, and chat in real time. An admin mode (password-protected) lets the organizer edit all convention content directly in the browser.

---

## Architecture

- **Delivery:** Single self-contained HTML file (`convention.html`)
- **Persistent data:** Firebase Firestore (collections: `convention`, `sessions`, `attendees`, `booths`, `meetups`)
- **Real-time chat:** Firebase Realtime Database (`/chat` node)
- **Admin auth:** Password `1234321`, stored in `sessionStorage` — no server-side auth
- **Seed data:** On first load, if Firestore collections are empty, dummy BlobCon 2025 data is written automatically

---

## Navigation

**Home Hub → Section Pages**

Users land on the Convention Home. Five large cards navigate to full-screen section views. A back button on each section returns to the hub.

Sections:
1. Sessions (📅)
2. Attendees (👥)
3. Meetups (🤝)
4. Booths (🏪)
5. Group Chat (💬)

---

## Screens

### Convention Home
Displays:
- Cover banner: convention name, tagline, presenter line, dates, location, attendee count
- Five section cards (icon, label, subtitle stat)
- 🔐 Admin button (top-right corner of banner)

Admin-editable fields: name, tagline, presenter line, dates, location, cover gradient/image URL, attendee count label.

---

### Sessions
- Sessions grouped by day (Day 1 / Day 2 / Day 3)
- Each session card: title, speaker, time, room, track tag (color-coded), description
- Firestore collection: `sessions` — fields: `title`, `speaker`, `time`, `day`, `room`, `track`, `description`

**Admin controls:** Add session button (opens inline form), edit/delete icon on each card.

---

### Attendee Directory
- Searchable grid of attendee cards
- Each card: avatar (initial or photo URL), name, company, role
- Search filters by name or company (client-side)
- Firestore collection: `attendees` — fields: `name`, `company`, `role`, `avatarUrl`

**Admin controls:** Add attendee button, edit/delete on each card.

---

### Meetup Planner
- List of all meetups (pending + confirmed)
- Each meetup: proposer name, participants, time slot, location/link, status badge
- "Propose Meetup" button opens a form: participant names (free text), date/time, location or video link, notes
- Meetups stored in Firestore collection `meetups` — fields: `proposer`, `participants`, `datetime`, `location`, `notes`, `status` (`pending`/`confirmed`)
- Any user can propose; status starts as `pending`; anyone can click "Confirm" to accept

**Admin controls:** Delete any meetup.

---

### Virtual Booths
- Grid of booth cards: company logo/initial, company name, tagline, short description, website link button
- Firestore collection: `booths` — fields: `company`, `tagline`, `description`, `logoUrl`, `website`

**Admin controls:** Add booth button, edit/delete on each card.

---

### Group Chat
- Firebase Realtime Database at `/chat`
- Messages display: display name, message text, timestamp
- On first message, user is prompted to enter a display name (stored in `localStorage`)
- Messages load in real time via `onValue` listener
- No pagination for MVP — last 200 messages shown

**Admin controls:** None (chat is open to all).

---

## Admin Mode

- Triggered by clicking 🔐 Admin button on the home banner
- Password prompt: `1234321`
- On success: `sessionStorage.setItem('adminMode', 'true')`
- Admin mode persists for the browser session, resets on tab close
- When active: edit/add/delete controls appear on all sections; home banner fields become editable inline
- Visual indicator: subtle "ADMIN" badge in the top nav while active

---

## Dummy Seed Data (BlobCon 2025)

Seeded automatically on first load when Firestore collections are empty.

**Convention details:**
- Name: BlobCon 2025
- Tagline: The Future of Social Connectivity
- Presenter: Blobs Network
- Dates: June 12–14, 2025
- Location: San Francisco, CA

**Sessions (8):**
- Day 1: Opening Keynote, Workshop: Blob Architecture, Panel: The Anti-Polarization Stack
- Day 2: Talk: Token Economies in Social Networks, Workshop: Metaverse Integration, Fireside: Building for Trust
- Day 3: Talk: AI Moderation at Scale, Closing Keynote

**Attendees (12):** Mix of roles (engineers, designers, PMs, founders) with names and companies

**Booths (5):** Sponsor companies with taglines and descriptions

---

## Visual Style

- Dark theme (`#0f172a` background, `#1e293b` cards)
- Accent: indigo/violet gradient (`#4f46e5` → `#7c3aed`)
- Modern card-based UI — glassmorphism effects, smooth transitions
- Inspired by Luma/Partiful event pages
- Typography: Inter or system-ui, bold headings, clean hierarchy
- Animations: section transitions slide in, cards have hover lift effect

---

## Out of Scope (MVP)

- User authentication (no login system)
- Per-session RSVP or seat limits
- Push notifications
- Metaverse/3D virtual space
- Ticket purchasing
- Multiple simultaneous conventions
