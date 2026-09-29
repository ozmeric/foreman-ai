# Foreman AI — demo deployment (Cabinet and Closet Express) (your accounts)

Runs entirely in your Google, Cloudflare and GitHub accounts. About 45 minutes.

**What's real in the demo:** Claude writing, email replies to leads (sent from your Gmail), "lead needs you" alert emails, the website form, photo uploads, the hourly autopilot.
**What's simulated:** posting to Instagram/Facebook/Google and text messages (they show in the app, nothing is sent).

## Files

| File | Goes to |
|---|---|
| `Code.gs`, `appsscript.json` | Apps Script (bound to a Google Sheet) |
| `worker.js` | Cloudflare Worker |
| `index.html`, `manifest.json`, `sw.js`, `icon-192.png`, `icon-512.png`, `lead-form.html` (+ your `buildflow-logo.png`) | GitHub Pages repo |

## 1. Google Sheet + Apps Script

1. Create a Google Sheet named **Foreman Demo**.
2. **Extensions → Apps Script.** Delete the sample code and paste `Code.gs`.
3. **Project Settings (gear) →** check *Show "appsscript.json" manifest file*. Open `appsscript.json` in the editor and replace it with the one provided (sets time zone to America/New_York).
4. **Project Settings → Script Properties → Add:**
   - `APP_PIN` = a 4–6 digit PIN for the demo
   - `ANTHROPIC_KEY` = your Claude API key (console.anthropic.com → API keys)
   - `NOTIFY_EMAIL` = where "lead needs you" alerts go (use your own for the demo)
   - optional: `AI_DAILY_CAP` (default 200), `MODEL` (default `claude-sonnet-5`)
5. The demo business is preset to **Cabinet and Closet Express** (contact: Mark). Update the phone and service area in `DEMO_BRAND` if you know them. For a different business later, edit `DEMO_BRAND` at the top of `Code.gs` (you can also change it later in the app's Brand tab).
6. Select the function **setup** in the toolbar and click **Run**. Approve the permissions (Sheets, Gmail send, Drive, external requests, triggers). It creates the tabs, seeds demo data, a photo folder in Drive, and the hourly trigger.
7. **Deploy → New deployment →** type *Web app*. Execute as: **Me**. Who has access: **Anyone**. Deploy and copy the **Web app URL** (ends in `/exec`).

> Every time you change `Code.gs`: **Deploy → Manage deployments → pencil → Version: New version → Deploy.** Saving alone does not update the live URL. This is the #1 gotcha.

## 2. Cloudflare Worker

1. Cloudflare dashboard → **Workers & Pages → Create → Worker**, name it `foreman-demo`. Deploy the placeholder.
2. **Edit code**, paste `worker.js`, **Deploy**.
3. **Settings → Variables and Secrets:**
   - `APPS_SCRIPT_URL` (type Secret) = the `/exec` URL from step 1.7
   - `ALLOWED_ORIGINS` (type Text) = `https://foreman-demo.usebuildflow.io` (add `,http://localhost:8080` if you test locally)
4. Copy the Worker URL (e.g. `https://foreman-demo.yourname.workers.dev/`).
5. Test: open the Worker URL in a browser. You should see `{"ok":true,...}`.

## 3. Front end on GitHub Pages

1. In `index.html` and `lead-form.html`, set the Worker URL:
   - `index.html` → `CONFIG.apiUrl`
   - `lead-form.html` → `API_URL`
2. Create a GitHub repo `foreman-demo`. Upload all front-end files plus `buildflow-logo.png`.
3. Add a file named `CNAME` containing `foreman-demo.usebuildflow.io`.
4. Repo **Settings → Pages →** Deploy from branch `main`, folder `/root`. Tick **Enforce HTTPS** once available.
5. Cloudflare DNS for `usebuildflow.io`: if your wildcard doesn't already cover it, add CNAME `cabinet-demo` → `yourgithubname.github.io` (DNS only / grey cloud while GitHub issues the certificate).

## 4. Test before the meeting (10 minutes)

1. Open `https://foreman-demo.usebuildflow.io`, enter the PIN. Today screen shows demo data.
2. **Send a test lead** with your own email and a normal question. The reply lands in your inbox in about 10–20 seconds.
3. Send another asking "how much…". Check: no price in the reply, lead shows in **Needs you**, alert email arrives at `NOTIFY_EMAIL`.
4. **Create post from a job:** upload a photo, click **Write caption**, **Add to schedule**.
5. Click **Run next autopilot check.** A post publishes and a follow-up is sent (shown in the feed).
6. Open `/lead-form.html` on your phone and submit with your email. The lead appears in the app within a minute.
7. Before the meeting: **Autopilot → Reset demo data.**
8. If anything fails, check the Sheet's **Log** tab.

## 5. Demo script for the client (15 minutes)

1. **Today screen:** "This is what Foreman did while you were working."
2. Hand him your phone with `lead-form.html` open. He fills it out with **his** email. His inbox gets a reply in his business's voice within seconds.
3. Have him submit again asking about price. Show the handoff: no price given, and the alert email arrives.
4. **Posts:** snap a photo of something in his shop, generate a caption live.
5. Flip the brass **Autopilot** switch off and on. "You're always in control."
6. Open the proposal and walk through pricing.

## After the demo

- Change the PIN or remove `ANTHROPIC_KEY` if you don't want the demo used afterward.
- For the next prospect: edit `DEMO_BRAND`, deploy a new version, click **Reset demo data** in the app.
- Hourly autopilot keeps running (it sends real emails only to leads that entered an email). Pause it with the switch when you're not demoing.
