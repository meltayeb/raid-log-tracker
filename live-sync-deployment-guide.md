# RAID Log Tracker — Live Google Sheets Sync Setup

This guide connects the RAID Log Tracker app to your real Google Sheet ("RAID Log Tracker - Data Source") so that adding, editing, and deleting RAID items and actions in the app actually reads and writes that Sheet, live.

It has three parts: deploying the backend script, hosting the app somewhere with real network access, and connecting the two.

## Before you start

Two files are attached alongside this guide:

- `Code.gs` — the backend script that runs inside your Google Sheet and answers the app's requests.
- `raid-log-tracker.html` — the updated app file, with the real Google Sheets connector built in.

**Important:** a published Claude Artifact (the preview you've been using) cannot make outbound network calls, so live sync will never work from inside that preview — connecting there will always fail with an honest network error. Live sync only works once `raid-log-tracker.html` is hosted somewhere with real internet access. See Part 2 for the simplest option (GitHub Pages).

## Part 1 — Deploy the Apps Script backend

1. Open your "RAID Log Tracker - Data Source" Google Sheet.
2. Go to **Extensions → Apps Script**. This opens a script editor bound to that Sheet.
3. Delete any placeholder code in the editor (e.g. an empty `myFunction(){}`), then paste in the entire contents of the attached `Code.gs`.
4. Save the project (the disk icon, or Ctrl/Cmd+S). Give it a name like "RAID Tracker API" if prompted.
5. Set your secret token:
   - In the left sidebar, click **Project Settings** (the gear icon).
   - Scroll to **Script Properties** and click **Add script property**.
   - Property: `API_TOKEN`. Value: any long random string you make up (e.g. `raid-9f3k2m8x7q1z`) — this is what keeps strangers from writing to your Sheet, so keep it private.
   - Click **Save script properties**.
6. Deploy it as a Web App:
   - Back in the editor, click **Deploy → New deployment**.
   - Click the gear icon next to "Select type" and choose **Web app**.
   - Description: anything, e.g. "RAID Tracker API v1".
   - **Execute as:** Me (your account).
   - **Who has access:** choose **Anyone** (simplest — the token is still required for any request to succeed), or **Anyone within [your Google Workspace domain]** if your org's Workspace allows restricting to signed-in domain users. Since this is Izam team7 data, the domain-restricted option is worth using if it's available to you.
   - Click **Deploy**.
   - Google will ask you to authorize the script's access to the Sheet the first time — click through the consent screens (you'll likely see an "unverified app" warning since this is your own script; click **Advanced → Go to [project name] (unsafe)** to proceed — this is expected for scripts you write yourself).
7. Copy the **Web app URL** shown after deployment. It looks like:
   `https://script.google.com/macros/s/AKfycb.../exec`
   Keep this tab open — you'll need this URL plus your token in Part 3.

**If you ever edit Code.gs again:** click **Deploy → Manage deployments → (pencil icon) → New version → Deploy** to push the update to the same URL. Editing the script alone does not update a live deployment.

## Part 2 — Host the app somewhere with real network access

Pick whichever is easiest for you; GitHub Pages is free and simple.

**Option A: GitHub Pages**
1. Create a new GitHub repository (public or private).
2. Upload `raid-log-tracker.html` to it, renamed to `index.html`.
3. Go to the repo's **Settings → Pages**, set **Source** to your main branch, and save.
4. GitHub gives you a URL like `https://yourname.github.io/reponame/` — open that; it's now hosted with real network access.

**Option B: Any existing web server / hosting you already use** — just upload `raid-log-tracker.html` there and open its URL. Any static file host works (Netlify, Vercel, an internal company server, etc.) since the app is a single self-contained HTML file with no build step.

## Part 3 — Connect the app to the Sheet

1. Open the hosted app (from Part 2 — not the Claude Artifact preview).
2. Go to **Settings → Data Source**.
3. Click the **Google Sheets** mode card.
4. In the URL field, paste your Web App URL from Part 1 with your token appended as a query parameter, in this exact form:
   `https://script.google.com/macros/s/AKfycb.../exec?token=raid-9f3k2m8x7q1z`
   (replace with your actual URL and token).
5. Click **Connect Google Sheet**. If it succeeds, the status will show "Connected" and the app will load your live RAID items and actions straight from the Sheet.
6. From here on, adding, editing, and deleting RAID items or actions in the app writes directly back to the Sheet. Use **Sync now** any time to pull in changes someone else made directly in the Sheet.

## Troubleshooting

- **"Couldn't reach the Google Sheet backend"** — you're likely still in the Claude Artifact preview (expected to always fail there), or the Web App URL is wrong/not deployed yet.
- **"Invalid or missing token"** — the `?token=...` value doesn't match the `API_TOKEN` script property. Re-check both.
- **"Tab 'RAID' not found in this spreadsheet"**-style errors — the script expects tabs literally named `RAID`, `ACTIONS`, `PROJECTS`, `WORKSTREAMS`, matching your existing Sheet.
- **Changes not showing up for a teammate** — each person's browser needs to be pointed at the same hosted app URL and the same Web App URL/token; Local mode is per-browser and won't sync between people.
