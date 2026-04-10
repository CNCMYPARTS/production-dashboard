# Production Job Dashboard

A live, browser-based job tracking dashboard that reads and writes to Google Sheets as its database.

---

## Quick start (5 steps)

### Step 1 — Create your Google Sheet

1. Go to [sheets.google.com](https://sheets.google.com) and create a new spreadsheet.
2. Rename the first tab to exactly: **`Jobs`**
3. Add these column headers in **Row 1** (copy exactly):

| A | B | C | D | E | F | G | H |
|---|---|---|---|---|---|---|---|
| ID | Name | Status | Priority | DueDate | Assigned | Notes | Files |

4. Note the **Sheet ID** from the URL:
   ```
   https://docs.google.com/spreadsheets/d/  THIS_PART_IS_YOUR_SHEET_ID  /edit
   ```

---

### Step 2 — Enable the Google Sheets API

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a new project (or use an existing one)
3. Go to **APIs & Services → Library**
4. Search for **"Google Sheets API"** → Click it → Click **Enable**

---

### Step 3 — Create an API Key

1. In Google Cloud Console go to **APIs & Services → Credentials**
2. Click **+ Create Credentials → API Key**
3. Copy the key — it looks like: `AIzaSyABC123...`
4. (Recommended) Click **Edit API Key** and under "API restrictions" select "Google Sheets API" to limit its scope

> **Important:** Make your Google Sheet **publicly viewable** (Share → Anyone with the link → Viewer)  
> for read operations to work with just an API key.  
> For write operations (add/edit/delete), see the OAuth note below.

---

### Step 4 — Add your credentials to the dashboard

**Option A — Enter in the app (easiest)**  
Open the dashboard URL → fill in Sheet ID and API Key in the yellow banner → click Connect.  
These are saved in your browser's localStorage.

**Option B — Hard-code for team deploys**  
Open `index.html` and find this line near the top of the `<script>`:
```js
const CONFIG = {
  SHEET_ID: window.__SHEET_ID__ || localStorage.getItem("sheet_id") || "",
  API_KEY:  window.__API_KEY__  || localStorage.getItem("api_key")  || "",
```
Replace the empty strings with your actual values:
```js
  SHEET_ID: "your-sheet-id-here",
  API_KEY:  "your-api-key-here",
```

---

### Step 5 — Deploy to Vercel

**Via Vercel CLI (fastest):**
```bash
npm i -g vercel
cd production-dashboard
vercel
```
Follow the prompts. Your dashboard will be live at a URL like `your-project.vercel.app`.

**Via GitHub (recommended for teams):**
1. Push this folder to a GitHub repo
2. Go to [vercel.com](https://vercel.com) → New Project → Import your repo
3. Vercel auto-detects `vercel.json` — just click Deploy
4. Share the URL with your team

---

## Google Sheet write permissions (for add/edit/delete)

The Google Sheets API requires **OAuth 2.0** (not just an API key) for write operations.

**Simplest path for a small team:** Use [Sheetson](https://sheetson.com) or [Sheety](https://sheety.co) as a thin wrapper — they handle auth and give you a REST API on top of your Sheet. Replace the `appendToSheets`, `updateInSheets`, and `deleteFromSheets` functions in `index.html` with their endpoints.

**For full control:** Set up a small backend (e.g. a Vercel serverless function in `/api/jobs.js`) that holds a service account key and proxies write calls — keeps credentials server-side and secure.

---

## File structure

```
production-dashboard/
├── index.html      ← The entire dashboard (single file)
├── vercel.json     ← Vercel deployment config
└── README.md       ← This file
```

---

## Customising the dashboard

| What to change | Where in index.html |
|---|---|
| Company name / title | `<title>` tag and `<h1>` in topbar |
| Column names in Sheet | `CONFIG.RANGE` and the `appendToSheets` row array |
| Status options | The `<select>` dropdowns in the modal + `statusClass()` function |
| Priority levels | Same as above + `priorityClass()` |
| Auto-refresh interval | Add `setInterval(loadFromSheets, 60000)` after the `DOMContentLoaded` block |

---

## Auto-refresh

To have the dashboard poll for updates every 60 seconds, add this inside the `DOMContentLoaded` listener in `index.html`:

```js
setInterval(() => {
  if (CONFIG.SHEET_ID && CONFIG.API_KEY) loadFromSheets();
}, 60000);
```
