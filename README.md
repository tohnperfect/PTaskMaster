# TaskMaster

A mobile-first task manager that uses **your own Google Sheets as the database**. Sign in with Google, and every task you create is written to a spreadsheet in *your* Drive — which means you can open it, edit it, back it up, and share it with collaborators like any other Google file.

No server to deploy. No database to manage. No third-party backend holding your data.

> 🌸 **Origin story.** This app was built during my Thai New Year (Songkran) holiday in April 2026, while traveling around Japan 🇯🇵. It was developed **entirely with [Claude](https://claude.ai) via vibe coding** — no manual coding, just conversation, iteration, and feel. The whole thing is one `index.html` file shaped by prompts between temple visits and train rides.

---

## ✨ Highlights

- **Zero-infrastructure** — single static `index.html`. Host anywhere (GitHub Pages, Netlify, a USB stick, `file://`).
- **Your data, your Sheets** — each workspace is one Google Spreadsheet in your Drive. You stay the owner.
- **Multi-workspace** — run separate boards for research, teaching, startup, personal, etc.
- **Team sharing** — invite collaborators by email; Drive handles the permissions. Removing access is one click.
- **Eisenhower matrix built in** — every task is classified as Big/Small × Urgent/Not Urgent, with priority (1–10) and monetary value (THB).
- **Google Calendar integration** — push any task with a deadline straight to your calendar.
- **Analytics** — completion rates, throughput by day/week/month/year, value booked vs. pending, quadrant breakdown.
- **Offline-friendly session** — access token is cached locally so you don't re-authenticate on every reload.

---

## 🧩 Tech stack

| Layer | Choice |
|---|---|
| Frontend | Vanilla JavaScript, no build step, no framework |
| Auth | Google Identity Services (OAuth 2.0, implicit token flow) |
| Storage | Google Sheets API v4 |
| Workspace discovery & sharing | Google Drive API v3 |
| Calendar push | Google Calendar API v3 |
| Styling | Hand-written CSS, DM Serif Display + Plus Jakarta Sans |
| Target viewport | Mobile-first, max-width 480px |

Everything lives in one file — about 770 lines of HTML/CSS/JS.

---

## 🚀 Quick start

### 1. Clone

```bash
git clone https://github.com/<your-username>/taskmaster.git
cd taskmaster
```

### 2. Set up Google Cloud

You need an OAuth Client ID so Google lets the app talk to your Sheets/Drive/Calendar.

1. Go to the [Google Cloud Console](https://console.cloud.google.com/) and create a new project (or reuse one).
2. Enable these APIs under **APIs & Services → Library**:
   - Google Sheets API
   - Google Drive API
   - Google Calendar API
3. Configure the **OAuth consent screen** (External is fine for personal use; add yourself as a test user).
4. Under **Credentials → Create Credentials → OAuth client ID**, choose **Web application** and add the URL(s) you'll serve the app from to **Authorized JavaScript origins**. For example:
   - `http://localhost:8080`
   - `https://<your-username>.github.io`
5. Copy the generated **Client ID**.

### 3. Wire in your Client ID

Open `index.html` and replace the value of `CLIENT_ID` near the top of the `<script>` block:

```js
const CLIENT_ID = 'YOUR_CLIENT_ID_HERE.apps.googleusercontent.com';
```

### 4. Serve it

Any static server will do:

```bash
# Python
python3 -m http.server 8080

# Node
npx serve .
```

Then open `http://localhost:8080` and sign in.

### 5. (Optional) Deploy

Push to GitHub and turn on **GitHub Pages** (`Settings → Pages → Deploy from branch`). Make sure the GitHub Pages URL is in your OAuth client's authorized origins list.

---

## 🔐 OAuth scopes

TaskMaster requests these scopes on sign-in:

```
email
profile
https://www.googleapis.com/auth/spreadsheets
https://www.googleapis.com/auth/drive
https://www.googleapis.com/auth/calendar.events
```

The broad `drive` scope (rather than `drive.file`) is required so that spreadsheets **shared with you by other users** show up in your workspace list — `drive.file` only exposes files the app itself created.

---

## 🗂 How workspaces work

A workspace is just a Google Spreadsheet with:

- A name prefixed with `[TM]` (e.g. `[TM] Research Lab`)
- An `appProperties` tag `taskmaster=true` for reliable discovery
- Two sheets inside: `Tasks` and `Config`

The workspace list is populated by a Drive query that finds any spreadsheet matching either signal — that's how **Shared With Me** entries appear after a collaborator shares their file with you.

### `Tasks` sheet columns

| # | Column | Type | Notes |
|---|---|---|---|
| 1 | `id` | string | Auto-generated unique ID |
| 2 | `title` | string | Required |
| 3 | `description` | string | |
| 4 | `isBig` | bool | Eisenhower axis |
| 5 | `isUrgent` | bool | Eisenhower axis |
| 6 | `priority` | 1–10 | |
| 7 | `value` | number | Monetary value in THB |
| 8 | `deadline` | ISO datetime | Optional |
| 9 | `assignees` | JSON array | Serialized list of names |
| 10 | `group` | string | e.g. Research, Teaching |
| 11 | `done` | bool | |
| 12 | `createdAt` | ISO datetime | |
| 13 | `finishedAt` | ISO datetime | Set when `done` flips to true |
| 14 | `notes` | string | |

### `Config` sheet

Two rows — key/value — storing the list of groups and people available across the workspace:

```
groups    ["Research","Teaching","Admin","Startup"]
people    ["Peggy","Tan","Hue","Nat"]
```

Because this is plain Sheets, you can also add/remove tasks directly in Google Sheets and they'll show up the next time you open the workspace.

---

## 📱 Using the app

- **Sign in with Google** → pick or create a workspace
- **+ New Workspace** creates a new `[TM]`-tagged spreadsheet in your Drive
- **Share** (owner only) opens a modal that invites collaborators as editors and lists current access
- Inside a workspace:
  - Add/edit tasks via the floating **+** button
  - Toggle between **Active** and **Done** tabs
  - Filter by quadrant, sort by priority/deadline/value/date, group by quadrant or category
  - Tap the bar chart icon for **Analytics**
  - Tap the gear icon for **Settings** (manage groups, people, export)
  - Tap the workspace name at the top to switch workspaces

---

## 🔒 Privacy & security notes

- The app runs entirely in your browser. There is no TaskMaster server.
- Your OAuth access token is cached in `localStorage` so you don't sign in on every reload. Revoke access any time at [myaccount.google.com/permissions](https://myaccount.google.com/permissions).
- Because the `drive` scope is broad, consider publishing the app only under your own OAuth client and keeping it in *Testing* mode unless you go through Google's verification.
- Sharing a workspace shares the underlying Google Sheet. Anyone with editor access can modify tasks — same trust model as a shared document.

---

## 🛣 Roadmap ideas

- Recurring tasks
- Subtasks / checklists inside a task
- Attachments (Drive links)
- Richer analytics (velocity charts, burndown)
- PWA install + offline-first write queue
- Keyboard-first desktop layout

Contributions welcome — it's one HTML file, so the hack loop is fast.

---

## 📝 License

MIT — see `LICENSE`.

---

## 🙌 Acknowledgements

Born out of a Songkran-holiday-in-Japan experiment: what happens if you build a real, shippable productivity tool *purely* through conversation with an LLM? Turns out, you get a working multi-user task manager backed by Google Sheets, written while riding the Shinkansen.

Built with [Claude](https://claude.ai) — all code generated, iterated, and debugged through vibe coding. No IDE autocomplete, no Stack Overflow deep-dives. Just prompts and taste.

Designed as a personal productivity tool for juggling research, teaching, and startup workloads, where tasks live across many contexts and need to be shareable with collaborators without standing up yet another SaaS account.
