# Stealth Mode Three — Design Assessment Studio

A self-hosted, proctored design-assessment site. Admins build briefs from real
vectorization proofs, send candidates a secure link, and the candidate works
through each proof while their camera, microphone, and screen are recorded.

No server or database required — it runs entirely in the browser and is hosted as
a static site (GitHub Pages, Netlify, Vercel, Cloudflare Pages, etc.).

---

## What's in here

```
index.html        The whole app (logo + thumbnails embedded, PDFs referenced)
proofs/           The six assessment PDFs, served to candidates
assets/logo.png   Brand logo (also embedded in the page)
.nojekyll         Tells GitHub Pages to serve files as-is
README.md         This file
```

## Deploy to GitHub Pages

1. Create a new GitHub repository and upload everything in this folder
   (keep the structure — `proofs/` must stay next to `index.html`).
2. Repo → **Settings → Pages**.
3. **Build and deployment → Source** → **Deploy from a branch**.
4. Branch **main**, folder **/ (root)** → **Save**.
5. After a minute your site is live at
   `https://<your-username>.github.io/<repo-name>/`.

Command line:
```bash
git init && git add . && git commit -m "Assessment studio"
git branch -M main
git remote add origin https://github.com/<you>/<repo>.git
git push -u origin main
```
Then enable Pages as above.

## Using it

**Admin**
1. First visit: set an admin password (resettable from the login screen or the
   “Change password” button in the top bar).
2. **Build brief:** pick proof tasks, enter the candidate's name/email and a time
   limit, then **Generate candidate link**.
3. **Preview as candidate** opens the exact candidate experience for testing.
4. Copy or email the link to your candidate.
5. **Review submissions:** load the candidate's `.sm3` file (see below).

**Candidate**
1. Opens the link, reads the welcome, continues to the system check.
2. Enables camera + mic and shares their screen.
3. Works each task: the proof PDF shows view-only beside a notes/files panel.
4. On finish they get a short reference code and download two files to send back.

## How submissions come back (no long codes)

When a candidate finishes they get:

- A **7-character reference code** (e.g. `KHDW3T2`) to quote in their email.
- A **submission file** `submission_<name>_<code>.sm3` — a small file holding
  their answers and proctoring snapshots.
- The **proctoring recording** `…​.webm` (screen + camera).

They email those files to you. In the dashboard open **Review submissions**, load
the `.sm3` file to read every answer and see the snapshots, and optionally load
the `.webm` to play the recording inline. Nothing to copy and paste.

## Optional: submit directly into a system (auto-return)

If you'd rather receive submissions automatically, set a **Submission webhook URL**
under **Advanced** when building a brief. On finish, the candidate's submission is
`POST`ed there as JSON. Two zero-cost ways to receive it:

**A. Google Sheet (via Apps Script)** — submissions land as rows in a spreadsheet.
1. Create a Google Sheet → **Extensions → Apps Script**, paste:
   ```javascript
   function doPost(e){
     const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
     const d = JSON.parse(e.postData.contents);
     sheet.appendRow([new Date(), d.sid, d.name, d.email,
       JSON.stringify(d.answers), d.proctor && d.proctor.screen]);
     return ContentService.createTextOutput("ok");
   }
   ```
2. **Deploy → New deployment → Web app**, access **Anyone**, copy the URL.
3. Paste that URL into the **Submission webhook URL** field.

**B. Formspree / similar** — create a form endpoint and paste its URL into the
same field.

(The downloadable files still work as a backup even when a webhook is set.)

## Requirements & notes

- Camera, mic, and screen sharing need a **secure context** — i.e. the `https://`
  GitHub Pages address, or opening the file directly in Chrome/Edge. They will not
  work inside an embedded preview iframe.
- Use **Chrome or Edge on desktop** for screen sharing.
- The admin password and sent-link list are stored in the admin's own browser.
- The proof PDFs are unmodified and shown view-only.
