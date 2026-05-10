# Convention App Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a standalone single-HTML-file convention app with Firebase backend featuring session schedule, attendee directory, meetup planner, virtual booths, real-time group chat, and password-protected admin mode.

**Architecture:** Single `convention.html` file with inline CSS and JS. Firebase Firestore stores all structured data (sessions, attendees, booths, meetups, convention config). Firebase Realtime Database powers real-time group chat. Admin mode unlocked by password `1234321` stored in `sessionStorage`. Dummy BlobCon 2025 data auto-seeded into Firestore on first load.

**Tech Stack:** Vanilla JS (ES6+), Firebase SDK v10 (CDN), CSS custom properties + animations, Firebase Firestore, Firebase Realtime Database

**Verification method:** Since there is no test runner for a single HTML file, each task ends with an explicit browser verification checklist. Open `convention.html` directly in Chrome (file://) or serve it locally with `npx serve .`.

---

## File Map

| File | Responsibility |
|------|---------------|
| `convention.html` | Entire application — shell, styles, all screen HTML templates, all JS logic |

All tasks modify `convention.html`. Task 1 creates it.

---

### Task 1: HTML Shell + Firebase Config + CSS Foundation

**Files:**
- Create: `convention.html`

- [ ] **Step 1: Create the file with this exact content**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>BlobCon 2025</title>
  <style>
    :root {
      --bg: #0f172a;
      --surface: #1e293b;
      --surface2: #334155;
      --border: #334155;
      --text: #f1f5f9;
      --text-muted: #94a3b8;
      --accent: #6366f1;
      --accent2: #7c3aed;
      --green: #22c55e;
      --red: #ef4444;
      --radius: 12px;
      --font: -apple-system, BlinkMacSystemFont, 'Inter', 'Segoe UI', sans-serif;
    }
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      background: var(--bg);
      color: var(--text);
      font-family: var(--font);
      min-height: 100vh;
      overflow-x: hidden;
    }
    #app { max-width: 480px; margin: 0 auto; min-height: 100vh; position: relative; }

    /* Screens */
    .screen { display: none; min-height: 100vh; flex-direction: column; }
    .screen.active { display: flex; }

    /* Utility */
    .btn {
      display: inline-flex; align-items: center; justify-content: center;
      gap: 6px; padding: 10px 18px; border-radius: 8px; border: none;
      font-size: 14px; font-weight: 600; cursor: pointer; transition: opacity .15s, transform .1s;
    }
    .btn:active { transform: scale(0.97); }
    .btn-primary { background: var(--accent); color: white; }
    .btn-primary:hover { opacity: 0.9; }
    .btn-ghost { background: transparent; color: var(--text-muted); border: 1px solid var(--border); }
    .btn-ghost:hover { border-color: var(--accent); color: var(--accent); }
    .btn-danger { background: var(--red); color: white; }
    .btn-sm { padding: 6px 12px; font-size: 12px; }

    input, textarea, select {
      background: var(--surface2); border: 1px solid var(--border);
      color: var(--text); border-radius: 8px; padding: 10px 14px;
      font-family: var(--font); font-size: 14px; width: 100%;
      outline: none; transition: border-color .15s;
    }
    input:focus, textarea:focus, select:focus { border-color: var(--accent); }
    textarea { resize: vertical; min-height: 80px; }

    label { font-size: 12px; color: var(--text-muted); display: block; margin-bottom: 4px; }
    .form-group { margin-bottom: 14px; }

    /* Modal */
    .modal-overlay {
      display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.7);
      z-index: 100; align-items: center; justify-content: center; padding: 20px;
    }
    .modal-overlay.open { display: flex; }
    .modal {
      background: var(--surface); border-radius: var(--radius); padding: 24px;
      width: 100%; max-width: 400px; border: 1px solid var(--border);
    }
    .modal h3 { margin-bottom: 16px; }
    .modal-actions { display: flex; gap: 10px; margin-top: 16px; justify-content: flex-end; }

    /* Card */
    .card {
      background: var(--surface); border: 1px solid var(--border);
      border-radius: var(--radius); padding: 16px;
      transition: border-color .15s, transform .15s;
    }
    .card:hover { border-color: var(--accent); transform: translateY(-1px); }

    /* Toast */
    #toast {
      position: fixed; bottom: 24px; left: 50%; transform: translateX(-50%) translateY(60px);
      background: var(--surface2); color: var(--text); padding: 10px 20px;
      border-radius: 20px; font-size: 13px; z-index: 200;
      transition: transform .3s; pointer-events: none;
    }
    #toast.show { transform: translateX(-50%) translateY(0); }

    /* Spinner */
    .spinner {
      width: 32px; height: 32px; border: 3px solid var(--border);
      border-top-color: var(--accent); border-radius: 50%;
      animation: spin .7s linear infinite; margin: 40px auto;
    }
    @keyframes spin { to { transform: rotate(360deg); } }

    /* Section nav */
    .section-header {
      display: flex; align-items: center; gap: 12px;
      padding: 16px; border-bottom: 1px solid var(--border);
      position: sticky; top: 0; background: var(--bg); z-index: 10;
    }
    .back-btn {
      background: none; border: none; color: var(--text-muted);
      cursor: pointer; font-size: 20px; padding: 4px; line-height: 1;
    }
    .back-btn:hover { color: var(--text); }
    .section-title { font-size: 17px; font-weight: 700; }

    /* Admin badge */
    #admin-badge {
      display: none; position: fixed; top: 12px; left: 50%;
      transform: translateX(-50%); background: var(--accent);
      color: white; font-size: 10px; font-weight: 700;
      padding: 3px 10px; border-radius: 20px; letter-spacing: 1px;
      z-index: 50; pointer-events: none;
    }
    #admin-badge.visible { display: block; }
  </style>
</head>
<body>
<div id="app">

  <!-- Screens injected per task -->
  <div id="screen-home" class="screen active">
    <p style="color:var(--text-muted);padding:40px;text-align:center;">Loading…</p>
  </div>

</div>

<!-- Toast -->
<div id="toast"></div>
<!-- Admin badge -->
<div id="admin-badge">ADMIN MODE</div>
<!-- Modal container -->
<div id="modal-container"></div>

<!-- Firebase SDK v10 (compat mode for simplicity) -->
<script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-database-compat.js"></script>

<script>
// ─── Firebase Config ─────────────────────────────────────────────
const firebaseConfig = {
  apiKey: "AIzaSyCqyCcs2R2e7AegGjvFAwG98wlamtbHvZY",
  authDomain: "bard-frontend.firebaseapp.com",
  projectId: "bard-frontend",
  storageBucket: "bard-frontend.firebasestorage.app",
  messagingSenderId: "175205271074",
  appId: "1:175205271074:web:2b7bd4d34d33bf38e6ec7b",
  databaseURL: "https://bard-frontend-default-rtdb.firebaseio.com"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();
const rtdb = firebase.database();

// ─── Globals ──────────────────────────────────────────────────────
const ADMIN_PASSWORD = '1234321';
function isAdmin() { return sessionStorage.getItem('adminMode') === 'true'; }

// ─── Toast ────────────────────────────────────────────────────────
function showToast(msg) {
  const t = document.getElementById('toast');
  t.textContent = msg;
  t.classList.add('show');
  setTimeout(() => t.classList.remove('show'), 2500);
}

// ─── Navigation ───────────────────────────────────────────────────
function showScreen(id) {
  document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
  const el = document.getElementById('screen-' + id);
  if (el) el.classList.add('active');
}

// ─── Modal helper ─────────────────────────────────────────────────
function openModal(html, onSubmit) {
  const container = document.getElementById('modal-container');
  container.innerHTML = `<div class="modal-overlay open" id="active-modal">${html}</div>`;
  if (onSubmit) {
    const form = container.querySelector('form');
    if (form) form.addEventListener('submit', e => { e.preventDefault(); onSubmit(form); });
  }
  container.querySelector('.modal-overlay').addEventListener('click', e => {
    if (e.target === e.currentTarget) closeModal();
  });
}
function closeModal() {
  document.getElementById('modal-container').innerHTML = '';
}

console.log('Firebase initialized. App shell ready.');
</script>
</body>
</html>
```

- [ ] **Step 2: Open the file in a browser**

Open `convention.html` in Chrome. You should see a dark page with "Loading…" text. No errors in the console (check DevTools → Console). Firebase will report a permission error for Firestore if not configured — that's fine for now.

- [ ] **Step 3: Commit**

```bash
cd /Users/maticohen/Desktop/Blobs
git init
git add convention.html
git commit -m "feat: add HTML shell with Firebase config and CSS foundation"
```

---

### Task 2: Convention Home Screen

**Files:**
- Modify: `convention.html` — replace `#screen-home` content and add home JS

- [ ] **Step 1: Replace the `#screen-home` div and add home CSS**

Replace this block:
```html
  <div id="screen-home" class="screen active">
    <p style="color:var(--text-muted);padding:40px;text-align:center;">Loading…</p>
  </div>
```

With:
```html
  <!-- HOME -->
  <div id="screen-home" class="screen active">
    <div id="home-banner" style="
      background: linear-gradient(135deg, #1e1b4b, #312e81, #4f46e5);
      padding: 32px 20px 24px; position: relative; flex-shrink: 0;
    ">
      <button onclick="promptAdmin()" style="
        position:absolute;top:14px;right:14px;background:rgba(255,255,255,0.12);
        border:none;color:white;font-size:12px;padding:6px 12px;border-radius:20px;cursor:pointer;
      ">🔐 Admin</button>
      <div id="home-presenter" style="color:rgba(255,255,255,0.55);font-size:11px;letter-spacing:2px;text-transform:uppercase;margin-bottom:6px;"></div>
      <div id="home-name" style="color:white;font-size:28px;font-weight:800;margin-bottom:6px;"></div>
      <div id="home-tagline" style="color:rgba(255,255,255,0.75);font-size:14px;margin-bottom:14px;"></div>
      <div style="display:flex;gap:16px;flex-wrap:wrap;">
        <span id="home-dates" style="color:rgba(255,255,255,0.65);font-size:12px;"></span>
        <span id="home-location" style="color:rgba(255,255,255,0.65);font-size:12px;"></span>
        <span id="home-count" style="color:rgba(255,255,255,0.65);font-size:12px;"></span>
      </div>
    </div>

    <div style="padding: 20px; display: grid; grid-template-columns: 1fr 1fr; gap: 14px;">
      <div class="card" onclick="showScreen('sessions')" style="cursor:pointer">
        <div style="font-size:28px;margin-bottom:10px;">📅</div>
        <div style="font-weight:700;margin-bottom:4px;">Sessions</div>
        <div id="stat-sessions" style="color:var(--text-muted);font-size:12px;">Loading…</div>
      </div>
      <div class="card" onclick="showScreen('attendees')" style="cursor:pointer">
        <div style="font-size:28px;margin-bottom:10px;">👥</div>
        <div style="font-weight:700;margin-bottom:4px;">Attendees</div>
        <div id="stat-attendees" style="color:var(--text-muted);font-size:12px;">Loading…</div>
      </div>
      <div class="card" onclick="showScreen('meetups')" style="cursor:pointer">
        <div style="font-size:28px;margin-bottom:10px;">🤝</div>
        <div style="font-weight:700;margin-bottom:4px;">Meetups</div>
        <div id="stat-meetups" style="color:var(--text-muted);font-size:12px;">Loading…</div>
      </div>
      <div class="card" onclick="showScreen('booths')" style="cursor:pointer">
        <div style="font-size:28px;margin-bottom:10px;">🏪</div>
        <div style="font-weight:700;margin-bottom:4px;">Booths</div>
        <div id="stat-booths" style="color:var(--text-muted);font-size:12px;">Loading…</div>
      </div>
      <div class="card" onclick="showScreen('chat')" style="cursor:pointer;grid-column:1/-1">
        <div style="display:flex;align-items:center;gap:12px;">
          <div style="font-size:28px;">💬</div>
          <div>
            <div style="font-weight:700;margin-bottom:2px;">Group Chat</div>
            <div style="color:var(--text-muted);font-size:12px;">Live conversation</div>
          </div>
          <div style="margin-left:auto;width:10px;height:10px;background:var(--green);border-radius:50%;"></div>
        </div>
      </div>
    </div>
  </div>

  <!-- SESSION SCHEDULE -->
  <div id="screen-sessions" class="screen">
    <div class="section-header">
      <button class="back-btn" onclick="showScreen('home')">←</button>
      <span class="section-title">📅 Sessions</span>
      <button id="btn-add-session" class="btn btn-primary btn-sm" style="margin-left:auto;display:none" onclick="openAddSession()">+ Add</button>
    </div>
    <div id="sessions-list" style="padding:16px"><div class="spinner"></div></div>
  </div>

  <!-- ATTENDEE DIRECTORY -->
  <div id="screen-attendees" class="screen">
    <div class="section-header">
      <button class="back-btn" onclick="showScreen('home')">←</button>
      <span class="section-title">👥 Attendees</span>
      <button id="btn-add-attendee" class="btn btn-primary btn-sm" style="margin-left:auto;display:none" onclick="openAddAttendee()">+ Add</button>
    </div>
    <div style="padding:12px 16px">
      <input id="attendee-search" type="text" placeholder="Search by name or company…" oninput="filterAttendees()" />
    </div>
    <div id="attendees-list" style="padding:0 16px 16px"><div class="spinner"></div></div>
  </div>

  <!-- MEETUP PLANNER -->
  <div id="screen-meetups" class="screen">
    <div class="section-header">
      <button class="back-btn" onclick="showScreen('home')">←</button>
      <span class="section-title">🤝 Meetups</span>
      <button class="btn btn-primary btn-sm" style="margin-left:auto" onclick="openProposeMeetup()">+ Propose</button>
    </div>
    <div id="meetups-list" style="padding:16px"><div class="spinner"></div></div>
  </div>

  <!-- VIRTUAL BOOTHS -->
  <div id="screen-booths" class="screen">
    <div class="section-header">
      <button class="back-btn" onclick="showScreen('home')">←</button>
      <span class="section-title">🏪 Booths</span>
      <button id="btn-add-booth" class="btn btn-primary btn-sm" style="margin-left:auto;display:none" onclick="openAddBooth()">+ Add</button>
    </div>
    <div id="booths-list" style="padding:16px"><div class="spinner"></div></div>
  </div>

  <!-- GROUP CHAT -->
  <div id="screen-chat" class="screen">
    <div class="section-header">
      <button class="back-btn" onclick="showScreen('home')">←</button>
      <span class="section-title">💬 Group Chat</span>
    </div>
    <div id="chat-messages" style="flex:1;overflow-y:auto;padding:16px;display:flex;flex-direction:column;gap:10px;"></div>
    <div style="padding:12px 16px;border-top:1px solid var(--border);display:flex;gap:10px;">
      <input id="chat-input" type="text" placeholder="Type a message…" onkeydown="if(event.key==='Enter')sendChatMessage()" />
      <button class="btn btn-primary" onclick="sendChatMessage()">Send</button>
    </div>
  </div>
```

- [ ] **Step 2: Add the home loading JS** — add this inside the `<script>` block, just before `console.log('Firebase initialized...')`

```javascript
// ─── Home Screen ─────────────────────────────────────────────────
async function loadHome() {
  const snap = await db.collection('convention').doc('config').get();
  const c = snap.exists ? snap.data() : {};
  document.getElementById('home-presenter').textContent = c.presenter || '';
  document.getElementById('home-name').textContent = c.name || 'Convention';
  document.getElementById('home-tagline').textContent = c.tagline || '';
  document.getElementById('home-dates').textContent = c.dates ? '📅 ' + c.dates : '';
  document.getElementById('home-location').textContent = c.location ? '📍 ' + c.location : '';
  document.getElementById('home-count').textContent = c.attendeeLabel ? '👥 ' + c.attendeeLabel : '';
}

async function loadStats() {
  const [sessions, attendees, meetups, booths] = await Promise.all([
    db.collection('sessions').get(),
    db.collection('attendees').get(),
    db.collection('meetups').get(),
    db.collection('booths').get(),
  ]);
  document.getElementById('stat-sessions').textContent = sessions.size + ' sessions';
  document.getElementById('stat-attendees').textContent = attendees.size + ' registered';
  document.getElementById('stat-meetups').textContent = meetups.size + ' scheduled';
  document.getElementById('stat-booths').textContent = booths.size + ' exhibitors';
}

loadHome();
loadStats();
```

- [ ] **Step 3: Verify in browser**

Reload. You should see: dark banner with empty text fields (no data yet), five section cards in a 2-col grid, chat card spanning full width. Clicking any card navigates to that section with a spinner. Back button returns to home.

- [ ] **Step 4: Commit**

```bash
git add convention.html
git commit -m "feat: add home hub and all section screen scaffolds"
```

---

### Task 3: Admin Authentication

**Files:**
- Modify: `convention.html` — add `promptAdmin`, `activateAdmin`, `deactivateAdmin`, and `refreshAdminControls`

- [ ] **Step 1: Add admin JS** — add inside `<script>` block after `loadStats()` call

```javascript
// ─── Admin Auth ───────────────────────────────────────────────────
function promptAdmin() {
  if (isAdmin()) { deactivateAdmin(); return; }
  openModal(`
    <div class="modal">
      <h3>🔐 Admin Login</h3>
      <form id="admin-form">
        <div class="form-group">
          <label>Password</label>
          <input type="password" name="password" autofocus placeholder="Enter admin password" />
        </div>
        <div class="modal-actions">
          <button type="button" class="btn btn-ghost" onclick="closeModal()">Cancel</button>
          <button type="submit" class="btn btn-primary">Unlock</button>
        </div>
      </form>
    </div>
  `, form => {
    const pw = form.password.value;
    if (pw === ADMIN_PASSWORD) {
      sessionStorage.setItem('adminMode', 'true');
      closeModal();
      refreshAdminControls();
      showToast('Admin mode enabled');
    } else {
      showToast('Wrong password');
    }
  });
}

function deactivateAdmin() {
  sessionStorage.removeItem('adminMode');
  refreshAdminControls();
  showToast('Admin mode disabled');
}

function refreshAdminControls() {
  const admin = isAdmin();
  document.getElementById('admin-badge').classList.toggle('visible', admin);
  // Show/hide add buttons on section headers
  ['btn-add-session', 'btn-add-attendee', 'btn-add-booth'].forEach(id => {
    const el = document.getElementById(id);
    if (el) el.style.display = admin ? 'inline-flex' : 'none';
  });
  // Re-render lists so edit/delete controls appear
  if (document.getElementById('screen-sessions').classList.contains('active')) loadSessions();
  if (document.getElementById('screen-attendees').classList.contains('active')) loadAttendees();
  if (document.getElementById('screen-booths').classList.contains('active')) loadBooths();
}
```

- [ ] **Step 2: Wire `showScreen` to call `refreshAdminControls`** — replace the existing `showScreen` function with:

```javascript
function showScreen(id) {
  document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
  const el = document.getElementById('screen-' + id);
  if (el) el.classList.add('active');
  refreshAdminControls();
  // Lazy-load section data
  if (id === 'sessions') loadSessions();
  if (id === 'attendees') loadAttendees();
  if (id === 'meetups') loadMeetups();
  if (id === 'booths') loadBooths();
  if (id === 'chat') initChat();
}
```

- [ ] **Step 3: Verify in browser**

Click 🔐 Admin. Enter wrong password → toast "Wrong password". Enter `1234321` → "Admin mode enabled" toast + purple "ADMIN MODE" badge at top. Click 🔐 Admin again → badge disappears. Navigate to Sessions → "+ Add" button visible in admin mode only.

- [ ] **Step 4: Commit**

```bash
git add convention.html
git commit -m "feat: add admin password authentication with session persistence"
```

---

### Task 4: Firebase Seed Data

**Files:**
- Modify: `convention.html` — add `seedDummyData` function that writes to Firestore if collections are empty

- [ ] **Step 1: Add seed data JS** — add inside `<script>` block after `refreshAdminControls`

```javascript
// ─── Seed Data ────────────────────────────────────────────────────
async function seedDummyData() {
  const config = await db.collection('convention').doc('config').get();
  if (config.exists) return; // already seeded

  const batch = db.batch();

  // Convention config
  batch.set(db.collection('convention').doc('config'), {
    name: 'BlobCon 2025',
    tagline: 'The Future of Social Connectivity',
    presenter: 'BLOBS NETWORK PRESENTS',
    dates: 'June 12–14, 2025',
    location: 'San Francisco, CA',
    attendeeLabel: '342 Attendees'
  });

  // Sessions
  const sessions = [
    { day: 1, time: '9:00 AM', title: 'Opening Keynote: The Anti-Polarization Stack', speaker: 'Alex Rivera', room: 'Main Stage', track: 'Keynote', description: 'Kicking off BlobCon with a deep dive into how Blobs Network is reimagining social connectivity.' },
    { day: 1, time: '11:00 AM', title: 'Workshop: Blob Architecture in Practice', speaker: 'Mia Chen', room: 'Workshop A', track: 'Engineering', description: 'Hands-on session covering the technical design of micro-communities and blob federation.' },
    { day: 1, time: '2:00 PM', title: 'Panel: Building for Trust in Social Networks', speaker: 'Rivera, Chen, & Obi', room: 'Main Stage', track: 'Community', description: 'A candid conversation about fake accounts, moderation, and earning user trust at scale.' },
    { day: 2, time: '10:00 AM', title: 'Talk: Token Economies & Incentive Design', speaker: 'Sam Obi', room: 'Hall B', track: 'Product', description: 'How token rewards can shift social network behavior from outrage to authentic engagement.' },
    { day: 2, time: '12:00 PM', title: 'Workshop: Metaverse Integration with Blobs', speaker: 'Priya Nair', room: 'Workshop B', track: 'Engineering', description: 'Build your first 3D blob neighborhood — avatars, spatial audio, and digital swag.' },
    { day: 2, time: '3:00 PM', title: 'Fireside: Lessons from 0 to 1M Users', speaker: 'Jordan Lee', room: 'Main Stage', track: 'Growth', description: 'Jordan shares unfiltered lessons from launching a social platform in a market dominated by giants.' },
    { day: 3, time: '10:00 AM', title: 'Talk: AI Moderation at Scale', speaker: 'Fatima Al-Zahra', room: 'Hall B', track: 'AI', description: 'Practical approaches to using LLMs for content moderation without killing organic discussion.' },
    { day: 3, time: '2:00 PM', title: 'Closing Keynote: Where We Go From Here', speaker: 'Alex Rivera', room: 'Main Stage', track: 'Keynote', description: 'Vision for the next chapter of Blobs Network and the future of decentralized social platforms.' },
  ];
  sessions.forEach(s => batch.set(db.collection('sessions').doc(), s));

  // Attendees
  const attendees = [
    { name: 'Alex Rivera', company: 'Blobs Network', role: 'CEO & Co-founder', avatarUrl: '' },
    { name: 'Mia Chen', company: 'Blobs Network', role: 'Engineering Lead', avatarUrl: '' },
    { name: 'Sam Obi', company: 'TokenForge', role: 'Product Director', avatarUrl: '' },
    { name: 'Priya Nair', company: 'MetaLayer', role: 'CTO', avatarUrl: '' },
    { name: 'Jordan Lee', company: 'Emerge Ventures', role: 'Partner', avatarUrl: '' },
    { name: 'Fatima Al-Zahra', company: 'SafeNet AI', role: 'Research Lead', avatarUrl: '' },
    { name: 'Carlos Mendes', company: 'Freelance', role: 'UI/UX Designer', avatarUrl: '' },
    { name: 'Yuki Tanaka', company: 'Horizon Labs', role: 'Backend Engineer', avatarUrl: '' },
    { name: 'Dana Park', company: 'Social Good Co', role: 'Head of Trust & Safety', avatarUrl: '' },
    { name: 'Eli Shapiro', company: 'ProtocolX', role: 'Developer Advocate', avatarUrl: '' },
    { name: 'Amara Diallo', company: 'CommunityOS', role: 'Founder', avatarUrl: '' },
    { name: 'Ben Walsh', company: 'Blobs Network', role: 'Growth Manager', avatarUrl: '' },
  ];
  attendees.forEach(a => batch.set(db.collection('attendees').doc(), a));

  // Booths
  const booths = [
    { company: 'TokenForge', tagline: 'Incentivize everything', description: 'Token infrastructure for social platforms — rewards, governance, and digital ownership at scale.', logoUrl: '', website: 'https://example.com' },
    { company: 'MetaLayer', tagline: 'Your world, your rules', description: '3D virtual space SDK — drop a metaverse into any web app in under an hour.', logoUrl: '', website: 'https://example.com' },
    { company: 'SafeNet AI', tagline: 'Moderation that gets it right', description: 'AI content moderation API trained on nuanced community guidelines, not just keyword lists.', logoUrl: '', website: 'https://example.com' },
    { company: 'ProtocolX', tagline: 'Open social infrastructure', description: 'Decentralized identity and social graph protocol — bring your followers anywhere.', logoUrl: '', website: 'https://example.com' },
    { company: 'CommunityOS', tagline: 'Communities deserve better tools', description: 'The operating system for online communities — analytics, moderation, and member management.', logoUrl: '', website: 'https://example.com' },
  ];
  booths.forEach(b => batch.set(db.collection('booths').doc(), b));

  await batch.commit();
  console.log('Seed data written to Firestore.');
}

// Run seed then load home
seedDummyData().then(() => {
  loadHome();
  loadStats();
});
```

- [ ] **Step 2: Remove the standalone `loadHome(); loadStats();` calls** that were added in Task 2 (they're now called after seed inside `.then()`)

Find and delete these two lines that appear earlier in the script:
```javascript
loadHome();
loadStats();
```

- [ ] **Step 3: Verify in browser**

Reload. The banner should now show "BlobCon 2025 / The Future of Social Connectivity / June 12–14…". Section cards show real counts: "8 sessions", "12 registered", etc. Open Firebase Console → Firestore → confirm collections `convention`, `sessions`, `attendees`, `booths` exist with documents.

- [ ] **Step 4: Commit**

```bash
git add convention.html
git commit -m "feat: seed BlobCon 2025 dummy data into Firestore on first load"
```

---

### Task 5: Sessions Screen

**Files:**
- Modify: `convention.html` — add `loadSessions`, `openAddSession`, `openEditSession`, `deleteSession`

- [ ] **Step 1: Add sessions CSS** — add inside `<style>` tag

```css
.track-badge {
  display: inline-block; font-size: 10px; font-weight: 700;
  padding: 2px 8px; border-radius: 20px; text-transform: uppercase; letter-spacing: .5px;
}
.track-Keynote { background: #7c3aed22; color: #a78bfa; }
.track-Engineering { background: #0ea5e922; color: #38bdf8; }
.track-Community { background: #22c55e22; color: #4ade80; }
.track-Product { background: #f59e0b22; color: #fbbf24; }
.track-Growth { background: #f97316 22; color: #fb923c; }
.track-AI { background: #ec489922; color: #f472b6; }

.day-group { margin-bottom: 24px; }
.day-label { font-size: 11px; font-weight: 700; color: var(--text-muted); text-transform: uppercase; letter-spacing: 1px; margin-bottom: 10px; }
.session-card { margin-bottom: 10px; }
.session-card h4 { font-size: 15px; margin-bottom: 4px; }
.session-meta { display: flex; gap: 10px; align-items: center; font-size: 12px; color: var(--text-muted); margin-bottom: 6px; }
.session-desc { font-size: 13px; color: var(--text-muted); line-height: 1.5; }
.admin-row { display: flex; gap: 8px; margin-top: 10px; }
```

- [ ] **Step 2: Add sessions JS** — add inside `<script>` block

```javascript
// ─── Sessions ──────────────────────────────────────────────────────
let allSessions = [];

async function loadSessions() {
  const container = document.getElementById('sessions-list');
  container.innerHTML = '<div class="spinner"></div>';
  const snap = await db.collection('sessions').orderBy('day').orderBy('time').get();
  allSessions = snap.docs.map(d => ({ id: d.id, ...d.data() }));

  if (allSessions.length === 0) {
    container.innerHTML = '<p style="color:var(--text-muted);text-align:center;padding:40px">No sessions yet.</p>';
    return;
  }

  const byDay = {};
  allSessions.forEach(s => {
    if (!byDay[s.day]) byDay[s.day] = [];
    byDay[s.day].push(s);
  });

  container.innerHTML = Object.keys(byDay).sort().map(day => `
    <div class="day-group">
      <div class="day-label">Day ${day}</div>
      ${byDay[day].map(s => `
        <div class="card session-card">
          <div class="session-meta">
            <span>${s.time}</span>
            <span>📍 ${s.room}</span>
            <span class="track-badge track-${s.track}">${s.track}</span>
          </div>
          <h4>${s.title}</h4>
          <div style="font-size:13px;color:var(--accent);margin-bottom:6px;">🎤 ${s.speaker}</div>
          <div class="session-desc">${s.description}</div>
          ${isAdmin() ? `
            <div class="admin-row">
              <button class="btn btn-ghost btn-sm" onclick="openEditSession('${s.id}')">✏️ Edit</button>
              <button class="btn btn-danger btn-sm" onclick="deleteSession('${s.id}')">🗑 Delete</button>
            </div>` : ''}
        </div>
      `).join('')}
    </div>
  `).join('');
}

function sessionFormHtml(s = {}) {
  return `
    <div class="modal">
      <h3>${s.id ? 'Edit' : 'Add'} Session</h3>
      <form id="session-form">
        <div class="form-group"><label>Title</label><input name="title" value="${s.title || ''}" required /></div>
        <div class="form-group"><label>Speaker</label><input name="speaker" value="${s.speaker || ''}" required /></div>
        <div class="form-group"><label>Day (1/2/3)</label><input name="day" type="number" min="1" max="3" value="${s.day || 1}" required /></div>
        <div class="form-group"><label>Time (e.g. 9:00 AM)</label><input name="time" value="${s.time || ''}" required /></div>
        <div class="form-group"><label>Room</label><input name="room" value="${s.room || ''}" /></div>
        <div class="form-group"><label>Track</label>
          <select name="track">
            ${['Keynote','Engineering','Community','Product','Growth','AI'].map(t =>
              `<option value="${t}" ${s.track === t ? 'selected' : ''}>${t}</option>`
            ).join('')}
          </select>
        </div>
        <div class="form-group"><label>Description</label><textarea name="description">${s.description || ''}</textarea></div>
        <div class="modal-actions">
          <button type="button" class="btn btn-ghost" onclick="closeModal()">Cancel</button>
          <button type="submit" class="btn btn-primary">Save</button>
        </div>
      </form>
    </div>`;
}

function openAddSession() {
  openModal(sessionFormHtml(), async form => {
    const data = {
      title: form.title.value, speaker: form.speaker.value,
      day: parseInt(form.day.value), time: form.time.value,
      room: form.room.value, track: form.track.value, description: form.description.value
    };
    await db.collection('sessions').add(data);
    closeModal(); loadSessions(); loadStats(); showToast('Session added');
  });
}

function openEditSession(id) {
  const s = allSessions.find(x => x.id === id);
  if (!s) return;
  openModal(sessionFormHtml(s), async form => {
    const data = {
      title: form.title.value, speaker: form.speaker.value,
      day: parseInt(form.day.value), time: form.time.value,
      room: form.room.value, track: form.track.value, description: form.description.value
    };
    await db.collection('sessions').doc(id).update(data);
    closeModal(); loadSessions(); showToast('Session updated');
  });
}

async function deleteSession(id) {
  if (!confirm('Delete this session?')) return;
  await db.collection('sessions').doc(id).delete();
  loadSessions(); loadStats(); showToast('Session deleted');
}
```

- [ ] **Step 3: Verify in browser**

Navigate to Sessions. Should show 8 sessions grouped under Day 1, Day 2, Day 3, each with time, room, track badge, speaker name, description. In admin mode: Edit and Delete buttons appear on each card. Add button in header opens form; saving adds a new session. Edit pre-fills form. Delete prompts confirmation.

- [ ] **Step 4: Commit**

```bash
git add convention.html
git commit -m "feat: add sessions screen with admin CRUD"
```

---

### Task 6: Attendee Directory

**Files:**
- Modify: `convention.html` — add `loadAttendees`, `filterAttendees`, `openAddAttendee`, `openEditAttendee`, `deleteAttendee`

- [ ] **Step 1: Add attendee CSS** — add inside `<style>` tag

```css
.attendees-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
.attendee-card { text-align: center; }
.avatar {
  width: 52px; height: 52px; border-radius: 50%;
  background: linear-gradient(135deg, var(--accent), var(--accent2));
  display: flex; align-items: center; justify-content: center;
  font-size: 20px; font-weight: 700; color: white;
  margin: 0 auto 10px;
}
.attendee-name { font-size: 14px; font-weight: 700; margin-bottom: 2px; }
.attendee-role { font-size: 11px; color: var(--text-muted); }
.attendee-company { font-size: 11px; color: var(--accent); margin-bottom: 6px; }
```

- [ ] **Step 2: Add attendees JS** — add inside `<script>` block

```javascript
// ─── Attendees ────────────────────────────────────────────────────
let allAttendees = [];

async function loadAttendees() {
  const container = document.getElementById('attendees-list');
  container.innerHTML = '<div class="spinner"></div>';
  const snap = await db.collection('attendees').orderBy('name').get();
  allAttendees = snap.docs.map(d => ({ id: d.id, ...d.data() }));
  renderAttendees(allAttendees);
}

function renderAttendees(list) {
  const container = document.getElementById('attendees-list');
  if (list.length === 0) {
    container.innerHTML = '<p style="color:var(--text-muted);text-align:center;padding:40px">No attendees found.</p>';
    return;
  }
  container.innerHTML = `<div class="attendees-grid">${list.map(a => `
    <div class="card attendee-card">
      <div class="avatar">${a.name.charAt(0)}</div>
      <div class="attendee-name">${a.name}</div>
      <div class="attendee-company">${a.company}</div>
      <div class="attendee-role">${a.role}</div>
      ${isAdmin() ? `
        <div class="admin-row" style="justify-content:center;margin-top:8px">
          <button class="btn btn-ghost btn-sm" onclick="openEditAttendee('${a.id}')">✏️</button>
          <button class="btn btn-danger btn-sm" onclick="deleteAttendee('${a.id}')">🗑</button>
        </div>` : ''}
    </div>
  `).join('')}</div>`;
}

function filterAttendees() {
  const q = document.getElementById('attendee-search').value.toLowerCase();
  renderAttendees(allAttendees.filter(a =>
    a.name.toLowerCase().includes(q) || a.company.toLowerCase().includes(q)
  ));
}

function attendeeFormHtml(a = {}) {
  return `
    <div class="modal">
      <h3>${a.id ? 'Edit' : 'Add'} Attendee</h3>
      <form id="attendee-form">
        <div class="form-group"><label>Name</label><input name="name" value="${a.name || ''}" required /></div>
        <div class="form-group"><label>Company</label><input name="company" value="${a.company || ''}" /></div>
        <div class="form-group"><label>Role</label><input name="role" value="${a.role || ''}" /></div>
        <div class="modal-actions">
          <button type="button" class="btn btn-ghost" onclick="closeModal()">Cancel</button>
          <button type="submit" class="btn btn-primary">Save</button>
        </div>
      </form>
    </div>`;
}

function openAddAttendee() {
  openModal(attendeeFormHtml(), async form => {
    const data = { name: form.name.value, company: form.company.value, role: form.role.value, avatarUrl: '' };
    await db.collection('attendees').add(data);
    closeModal(); loadAttendees(); loadStats(); showToast('Attendee added');
  });
}

function openEditAttendee(id) {
  const a = allAttendees.find(x => x.id === id);
  if (!a) return;
  openModal(attendeeFormHtml(a), async form => {
    await db.collection('attendees').doc(id).update({ name: form.name.value, company: form.company.value, role: form.role.value });
    closeModal(); loadAttendees(); showToast('Attendee updated');
  });
}

async function deleteAttendee(id) {
  if (!confirm('Remove this attendee?')) return;
  await db.collection('attendees').doc(id).delete();
  loadAttendees(); loadStats(); showToast('Attendee removed');
}
```

- [ ] **Step 3: Verify in browser**

Navigate to Attendees. Should show a 2-col grid with 12 attendee cards, each with initial avatar, name, company, role. Search field filters in real time. Admin mode shows edit/delete icons. Add attendee form works.

- [ ] **Step 4: Commit**

```bash
git add convention.html
git commit -m "feat: add attendee directory with search and admin CRUD"
```

---

### Task 7: Meetup Planner

**Files:**
- Modify: `convention.html` — add `loadMeetups`, `openProposeMeetup`, `confirmMeetup`, `deleteMeetup`

- [ ] **Step 1: Add meetup CSS** — add inside `<style>` tag

```css
.meetup-card h4 { font-size: 15px; margin-bottom: 6px; }
.meetup-meta { font-size: 12px; color: var(--text-muted); margin-bottom: 6px; }
.status-badge {
  display: inline-block; font-size: 10px; font-weight: 700;
  padding: 2px 8px; border-radius: 20px;
}
.status-pending { background: #f59e0b22; color: #fbbf24; }
.status-confirmed { background: #22c55e22; color: #4ade80; }
```

- [ ] **Step 2: Add meetups JS** — add inside `<script>` block

```javascript
// ─── Meetups ──────────────────────────────────────────────────────
async function loadMeetups() {
  const container = document.getElementById('meetups-list');
  container.innerHTML = '<div class="spinner"></div>';
  const snap = await db.collection('meetups').orderBy('datetime').get();
  const meetups = snap.docs.map(d => ({ id: d.id, ...d.data() }));

  if (meetups.length === 0) {
    container.innerHTML = '<p style="color:var(--text-muted);text-align:center;padding:40px">No meetups yet. Propose one!</p>';
    return;
  }

  container.innerHTML = meetups.map(m => `
    <div class="card meetup-card" style="margin-bottom:12px">
      <div style="display:flex;align-items:center;gap:8px;margin-bottom:6px">
        <span class="status-badge status-${m.status}">${m.status}</span>
        <span style="font-size:11px;color:var(--text-muted)">by ${m.proposer}</span>
      </div>
      <h4>👥 ${m.participants}</h4>
      <div class="meetup-meta">🕐 ${m.datetime} &nbsp;|&nbsp; 📍 ${m.location}</div>
      ${m.notes ? `<div style="font-size:13px;color:var(--text-muted)">${m.notes}</div>` : ''}
      <div class="admin-row" style="margin-top:10px">
        ${m.status === 'pending' ? `<button class="btn btn-primary btn-sm" onclick="confirmMeetup('${m.id}')">✓ Confirm</button>` : ''}
        ${isAdmin() ? `<button class="btn btn-danger btn-sm" onclick="deleteMeetup('${m.id}')">🗑 Delete</button>` : ''}
      </div>
    </div>
  `).join('');
}

function openProposeMeetup() {
  openModal(`
    <div class="modal">
      <h3>🤝 Propose a Meetup</h3>
      <form id="meetup-form">
        <div class="form-group"><label>Your Name</label><input name="proposer" required placeholder="Your name" /></div>
        <div class="form-group"><label>Participants (names, comma-separated)</label><input name="participants" required placeholder="e.g. Alex, Mia, Sam" /></div>
        <div class="form-group"><label>Date & Time</label><input name="datetime" required placeholder="e.g. June 13, 2:00 PM" /></div>
        <div class="form-group"><label>Location or Link</label><input name="location" required placeholder="e.g. Lobby, or https://meet.example.com" /></div>
        <div class="form-group"><label>Notes (optional)</label><textarea name="notes" placeholder="What's this meetup about?"></textarea></div>
        <div class="modal-actions">
          <button type="button" class="btn btn-ghost" onclick="closeModal()">Cancel</button>
          <button type="submit" class="btn btn-primary">Propose</button>
        </div>
      </form>
    </div>
  `, async form => {
    await db.collection('meetups').add({
      proposer: form.proposer.value,
      participants: form.participants.value,
      datetime: form.datetime.value,
      location: form.location.value,
      notes: form.notes.value,
      status: 'pending'
    });
    closeModal(); loadMeetups(); loadStats(); showToast('Meetup proposed!');
  });
}

async function confirmMeetup(id) {
  await db.collection('meetups').doc(id).update({ status: 'confirmed' });
  loadMeetups(); showToast('Meetup confirmed!');
}

async function deleteMeetup(id) {
  if (!confirm('Delete this meetup?')) return;
  await db.collection('meetups').doc(id).delete();
  loadMeetups(); loadStats(); showToast('Meetup deleted');
}
```

- [ ] **Step 3: Verify in browser**

Navigate to Meetups. Initially shows "No meetups yet." Click "+ Propose" → fill form → meetup appears with "pending" badge. Click "✓ Confirm" → badge changes to "confirmed". Admin mode shows delete button.

- [ ] **Step 4: Commit**

```bash
git add convention.html
git commit -m "feat: add meetup planner with propose/confirm flow"
```

---

### Task 8: Virtual Booths

**Files:**
- Modify: `convention.html` — add `loadBooths`, `openAddBooth`, `openEditBooth`, `deleteBooth`

- [ ] **Step 1: Add booths CSS** — add inside `<style>` tag

```css
.booth-card { margin-bottom: 14px; }
.booth-logo {
  width: 48px; height: 48px; border-radius: 10px;
  background: linear-gradient(135deg, var(--accent), var(--accent2));
  display: flex; align-items: center; justify-content: center;
  font-size: 20px; font-weight: 800; color: white; flex-shrink: 0;
}
.booth-header { display: flex; align-items: center; gap: 12px; margin-bottom: 8px; }
.booth-company { font-size: 16px; font-weight: 700; }
.booth-tagline { font-size: 12px; color: var(--accent); margin-bottom: 2px; }
.booth-desc { font-size: 13px; color: var(--text-muted); line-height: 1.5; margin-bottom: 10px; }
```

- [ ] **Step 2: Add booths JS** — add inside `<script>` block

```javascript
// ─── Booths ───────────────────────────────────────────────────────
let allBooths = [];

async function loadBooths() {
  const container = document.getElementById('booths-list');
  container.innerHTML = '<div class="spinner"></div>';
  const snap = await db.collection('booths').orderBy('company').get();
  allBooths = snap.docs.map(d => ({ id: d.id, ...d.data() }));

  if (allBooths.length === 0) {
    container.innerHTML = '<p style="color:var(--text-muted);text-align:center;padding:40px">No booths yet.</p>';
    return;
  }

  container.innerHTML = allBooths.map(b => `
    <div class="card booth-card">
      <div class="booth-header">
        <div class="booth-logo">${b.company.charAt(0)}</div>
        <div>
          <div class="booth-company">${b.company}</div>
          <div class="booth-tagline">${b.tagline}</div>
        </div>
      </div>
      <div class="booth-desc">${b.description}</div>
      <div style="display:flex;gap:10px;align-items:center">
        <a href="${b.website}" target="_blank" class="btn btn-ghost btn-sm">🌐 Visit Website</a>
        ${isAdmin() ? `
          <button class="btn btn-ghost btn-sm" onclick="openEditBooth('${b.id}')">✏️ Edit</button>
          <button class="btn btn-danger btn-sm" onclick="deleteBooth('${b.id}')">🗑</button>` : ''}
      </div>
    </div>
  `).join('');
}

function boothFormHtml(b = {}) {
  return `
    <div class="modal">
      <h3>${b.id ? 'Edit' : 'Add'} Booth</h3>
      <form id="booth-form">
        <div class="form-group"><label>Company Name</label><input name="company" value="${b.company || ''}" required /></div>
        <div class="form-group"><label>Tagline</label><input name="tagline" value="${b.tagline || ''}" /></div>
        <div class="form-group"><label>Description</label><textarea name="description">${b.description || ''}</textarea></div>
        <div class="form-group"><label>Website URL</label><input name="website" type="url" value="${b.website || ''}" placeholder="https://…" /></div>
        <div class="modal-actions">
          <button type="button" class="btn btn-ghost" onclick="closeModal()">Cancel</button>
          <button type="submit" class="btn btn-primary">Save</button>
        </div>
      </form>
    </div>`;
}

function openAddBooth() {
  openModal(boothFormHtml(), async form => {
    await db.collection('booths').add({ company: form.company.value, tagline: form.tagline.value, description: form.description.value, website: form.website.value, logoUrl: '' });
    closeModal(); loadBooths(); loadStats(); showToast('Booth added');
  });
}

function openEditBooth(id) {
  const b = allBooths.find(x => x.id === id);
  if (!b) return;
  openModal(boothFormHtml(b), async form => {
    await db.collection('booths').doc(id).update({ company: form.company.value, tagline: form.tagline.value, description: form.description.value, website: form.website.value });
    closeModal(); loadBooths(); showToast('Booth updated');
  });
}

async function deleteBooth(id) {
  if (!confirm('Delete this booth?')) return;
  await db.collection('booths').doc(id).delete();
  loadBooths(); loadStats(); showToast('Booth deleted');
}
```

- [ ] **Step 3: Verify in browser**

Navigate to Booths. Shows 5 sponsor cards each with logo initial, company, tagline, description, "Visit Website" link. Admin mode: Edit/Delete buttons visible. Add booth form works.

- [ ] **Step 4: Commit**

```bash
git add convention.html
git commit -m "feat: add virtual booths screen with admin CRUD"
```

---

### Task 9: Group Chat

**Files:**
- Modify: `convention.html` — add `initChat`, `sendChatMessage`, `renderMessages`

- [ ] **Step 1: Add chat CSS** — add inside `<style>` tag

```css
.chat-message { display: flex; flex-direction: column; gap: 2px; }
.chat-bubble {
  background: var(--surface); border-radius: 12px 12px 12px 4px;
  padding: 10px 14px; max-width: 85%; align-self: flex-start;
  border: 1px solid var(--border);
}
.chat-author { font-size: 11px; color: var(--accent); font-weight: 700; margin-bottom: 2px; }
.chat-text { font-size: 14px; line-height: 1.4; }
.chat-time { font-size: 10px; color: var(--text-muted); }
```

- [ ] **Step 2: Add chat JS** — add inside `<script>` block

```javascript
// ─── Group Chat ───────────────────────────────────────────────────
let chatInitialized = false;

function initChat() {
  if (chatInitialized) return;
  chatInitialized = true;

  const chatRef = rtdb.ref('/blobcon2025/chat');
  chatRef.limitToLast(200).on('value', snap => {
    const messages = [];
    snap.forEach(child => messages.push({ id: child.key, ...child.val() }));
    renderMessages(messages);
  });
}

function renderMessages(messages) {
  const container = document.getElementById('chat-messages');
  if (messages.length === 0) {
    container.innerHTML = '<p style="color:var(--text-muted);text-align:center;padding:40px">No messages yet. Say hello!</p>';
    return;
  }
  container.innerHTML = messages.map(m => `
    <div class="chat-message">
      <div class="chat-author">${m.author}</div>
      <div class="chat-bubble">
        <div class="chat-text">${escapeHtml(m.text)}</div>
      </div>
      <div class="chat-time">${new Date(m.timestamp).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}</div>
    </div>
  `).join('');
  container.scrollTop = container.scrollHeight;
}

async function sendChatMessage() {
  const input = document.getElementById('chat-input');
  const text = input.value.trim();
  if (!text) return;

  let author = localStorage.getItem('chatDisplayName');
  if (!author) {
    author = prompt('Enter your display name for the chat:');
    if (!author) return;
    localStorage.setItem('chatDisplayName', author.trim());
  }

  input.value = '';
  await rtdb.ref('/blobcon2025/chat').push({
    author: author.trim(),
    text,
    timestamp: Date.now()
  });
}

function escapeHtml(str) {
  return str.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
}
```

- [ ] **Step 3: Verify in browser**

Navigate to Group Chat. On first message, prompts for display name (stored in localStorage on confirm). Messages appear in real time. Open two browser tabs — message sent in one appears instantly in the other. Messages scroll to bottom automatically.

- [ ] **Step 4: Commit**

```bash
git add convention.html
git commit -m "feat: add real-time group chat via Firebase Realtime Database"
```

---

### Task 10: Admin — Edit Home Banner

**Files:**
- Modify: `convention.html` — add `openEditBanner` function and edit button visible in admin mode

- [ ] **Step 1: Add "Edit Banner" button to home screen** — inside `#home-banner`, add after the 🔐 Admin button:

```html
<button id="btn-edit-banner" onclick="openEditBanner()" style="
  display:none;position:absolute;bottom:14px;right:14px;
  background:rgba(255,255,255,0.15);border:none;color:white;
  font-size:12px;padding:6px 12px;border-radius:20px;cursor:pointer;
">✏️ Edit Banner</button>
```

- [ ] **Step 2: Update `refreshAdminControls`** — add the banner button to the list of toggled elements by adding inside the existing `refreshAdminControls` function body, after the `forEach` call:

```javascript
const bannerBtn = document.getElementById('btn-edit-banner');
if (bannerBtn) bannerBtn.style.display = admin ? 'block' : 'none';
if (document.getElementById('screen-home').classList.contains('active') && admin) loadHome();
```

- [ ] **Step 3: Add `openEditBanner` JS** — add inside `<script>` block

```javascript
// ─── Edit Banner ─────────────────────────────────────────────────
async function openEditBanner() {
  const snap = await db.collection('convention').doc('config').get();
  const c = snap.exists ? snap.data() : {};
  openModal(`
    <div class="modal">
      <h3>✏️ Edit Convention Details</h3>
      <form id="banner-form">
        <div class="form-group"><label>Convention Name</label><input name="name" value="${c.name || ''}" required /></div>
        <div class="form-group"><label>Tagline</label><input name="tagline" value="${c.tagline || ''}" /></div>
        <div class="form-group"><label>Presenter Line (e.g. "COMPANY PRESENTS")</label><input name="presenter" value="${c.presenter || ''}" /></div>
        <div class="form-group"><label>Dates</label><input name="dates" value="${c.dates || ''}" /></div>
        <div class="form-group"><label>Location</label><input name="location" value="${c.location || ''}" /></div>
        <div class="form-group"><label>Attendee Label (e.g. "342 Attendees")</label><input name="attendeeLabel" value="${c.attendeeLabel || ''}" /></div>
        <div class="modal-actions">
          <button type="button" class="btn btn-ghost" onclick="closeModal()">Cancel</button>
          <button type="submit" class="btn btn-primary">Save</button>
        </div>
      </form>
    </div>
  `, async form => {
    await db.collection('convention').doc('config').set({
      name: form.name.value, tagline: form.tagline.value,
      presenter: form.presenter.value, dates: form.dates.value,
      location: form.location.value, attendeeLabel: form.attendeeLabel.value
    });
    closeModal(); loadHome(); showToast('Convention details updated');
  });
}
```

- [ ] **Step 4: Verify in browser**

In admin mode, home banner shows "✏️ Edit Banner" button. Clicking it opens a form pre-filled with current values. Saving updates the banner in real time. Changes persist after page reload.

- [ ] **Step 5: Commit**

```bash
git add convention.html
git commit -m "feat: add admin banner editor for convention details"
```

---

### Task 11: Visual Polish

**Files:**
- Modify: `convention.html` — upgrade CSS with animations, glassmorphism, modern hover effects

- [ ] **Step 1: Replace the CSS `:root` and base styles with this enhanced version** — find the `<style>` tag and prepend (before `:root`) the Google Fonts import, then update the root variables and add animation utilities:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800;900&display=swap" rel="stylesheet">
```

Add this to `<head>` just before the `<style>` tag.

- [ ] **Step 2: Update `:root` font variable** — change:

```css
--font: -apple-system, BlinkMacSystemFont, 'Inter', 'Segoe UI', sans-serif;
```

to:

```css
--font: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
```

- [ ] **Step 3: Add screen transition animations** — add inside `<style>` tag:

```css
/* Screen transitions */
.screen { animation: none; }
.screen.active { animation: fadeSlideIn .25s ease; }
@keyframes fadeSlideIn {
  from { opacity: 0; transform: translateY(12px); }
  to   { opacity: 1; transform: translateY(0); }
}

/* Card hover glow */
.card {
  transition: border-color .2s, transform .2s, box-shadow .2s;
}
.card:hover {
  border-color: var(--accent);
  transform: translateY(-2px);
  box-shadow: 0 8px 32px rgba(99,102,241,0.15);
}

/* Banner shimmer overlay */
#home-banner::after {
  content: '';
  position: absolute; inset: 0;
  background: linear-gradient(180deg, transparent 60%, rgba(15,23,42,0.4) 100%);
  pointer-events: none;
}

/* Glass modal */
.modal {
  background: rgba(30,41,59,0.95);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(99,102,241,0.3);
  box-shadow: 0 20px 60px rgba(0,0,0,0.5);
}

/* Input focus ring */
input:focus, textarea:focus, select:focus {
  border-color: var(--accent);
  box-shadow: 0 0 0 3px rgba(99,102,241,0.15);
}

/* Smooth button press */
.btn { transition: opacity .15s, transform .1s, box-shadow .15s; }
.btn-primary:hover { box-shadow: 0 4px 20px rgba(99,102,241,0.4); }

/* Section header blur */
.section-header {
  background: rgba(15,23,42,0.85);
  backdrop-filter: blur(12px);
  border-bottom: 1px solid rgba(51,65,85,0.8);
}

/* Chat bubble animation */
.chat-message { animation: bubbleIn .2s ease; }
@keyframes bubbleIn {
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}

/* Avatar gradient variety */
.avatar:nth-child(3n+1) { background: linear-gradient(135deg, #6366f1, #7c3aed); }
.avatar:nth-child(3n+2) { background: linear-gradient(135deg, #0ea5e9, #6366f1); }
.avatar:nth-child(3n)   { background: linear-gradient(135deg, #ec4899, #7c3aed); }

/* Section card icon pulse on hover */
.card:hover > div:first-child { transform: scale(1.1); transition: transform .2s; }

/* Scrollbar */
::-webkit-scrollbar { width: 4px; }
::-webkit-scrollbar-track { background: transparent; }
::-webkit-scrollbar-thumb { background: var(--border); border-radius: 2px; }
```

- [ ] **Step 4: Verify in browser**

Reload. Screens fade in on navigation. Cards have smooth hover glow. Modal has glass blur effect. Inter font renders throughout. Chat messages animate in. Scrollbar is thin and minimal.

- [ ] **Step 5: Commit**

```bash
git add convention.html
git commit -m "feat: add visual polish — animations, glassmorphism, Inter font, hover effects"
```

---

## Self-Review Checklist

### Spec Coverage

| Spec Requirement | Task |
|---|---|
| Single HTML file with Firebase Firestore | Task 1 |
| Firebase Realtime Database for chat | Task 9 |
| Admin password `1234321` + sessionStorage | Task 3 |
| Dummy BlobCon 2025 seed data | Task 4 |
| Home Hub + section card navigation | Task 2 |
| Session schedule with day grouping + admin CRUD | Task 5 |
| Attendee directory with search + admin CRUD | Task 6 |
| Meetup planner with propose/confirm + admin CRUD | Task 7 |
| Virtual booths with website link + admin CRUD | Task 8 |
| Admin edit of convention banner | Task 10 |
| Modern visual style (Luma/Partiful-inspired) | Task 11 |

All spec requirements covered. ✓

### Notes for Implementer

- The Firebase config in Task 1 reuses the same project as the existing `BLOB_Network_Prototype (6).html`. If Firestore security rules block writes, update them in the Firebase Console to allow reads/writes for the `convention`, `sessions`, `attendees`, `booths`, `meetups` collections. A permissive dev rule: `allow read, write: if true;`
- The `databaseURL` in the config (`https://bard-frontend-default-rtdb.firebaseio.com`) must match the actual Realtime Database URL in the Firebase project. Verify this in Firebase Console → Realtime Database before Task 9.
- Tasks can be executed in order — each task builds on the previous one. Do not skip tasks.
