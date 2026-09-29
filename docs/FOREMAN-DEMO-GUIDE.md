# Foreman AI — Demo Guide
**Client:** Cabinet and Closet Express (contact: Mark)
**Built by:** Weblina LLC · Foreman AI · Powered by BuildFlow
**Last updated:** September 28, 2026 (code version v5)

This is the single reference for the Foreman demo: what it does, how it's set up, how to update it, how to run the demo, what to do when something breaks, and what's planned next.

---

## 1. What Foreman does

Foreman is an AI marketing assistant for a trade business. His team uses it; Foreman does the busywork.

1. **Answers every new lead in seconds**, by email in the business owner's voice.
2. **Holds the email conversation.** When a customer replies, Foreman reads it and answers.
3. **Follows up automatically** (Day 1, 3, 7, 14) until the customer replies, books, or asks to stop.
4. **Hands serious buyers to Mark.** If a customer asks about price, wants a visit/measurement, proposes a day, or is upset, Foreman says Mark will reach out and alerts him.
5. **Writes and schedules social posts** from job photos.

### Real vs. simulated in this demo

| Real | Simulated (until the client's accounts are connected) |
|---|---|
| AI writing (Gemini) | Posting to Instagram / Facebook / Google Business |
| Email replies, sent from the demo Gmail | Text messages (see section 12 for the plan) |
| Two-way email conversations | |
| New leads from emails to the lead inbox | |
| "Lead needs you" alert emails | |
| Website contact form | |
| Photo uploads, hourly autopilot, 5-minute inbox check | |

---

## 2. Your demo environment

| Item | Value |
|---|---|
| App | https://foreman-demo.usebuildflow.io |
| Website form (demo) | https://foreman-demo.usebuildflow.io/lead-form.html |
| Cloudflare Worker | https://foremanai.weblinallc.workers.dev |
| Google account running Foreman | moonlighttrucking21@gmail.com |
| **Lead inbox** (emails here become leads) | **moonlighttrucking21+foreman@gmail.com** |
| Google Sheet | "Foreman Demo" (ID `1UprUxuk62l8Tvof2fXyXn96PXxvrzi8H9ADsevXlDvM`) |
| GitHub repo | `foreman-demo` (GitHub Pages, custom domain above) |
| Demo PIN | Set in Apps Script → Script Properties → `APP_PIN` |

Secrets (Gemini key, PIN) live only in Script Properties. Never put them in the code or on GitHub.

---

## 3. How it's built

```
Phone / browser (GitHub Pages PWA)
        │  POST
        ▼
Cloudflare Worker  (foremanai.weblinallc.workers.dev, hides the Google URL)
        │  POST, redirect: follow
        ▼
Google Apps Script web app  (Code.gs, runs as moonlighttrucking21@gmail.com)
        ├── Google Sheet: Posts, Leads, Activity, Config, Log
        ├── Gemini API: writes replies, follow-ups, captions, drafts
        ├── Gmail: sends as "Mark at Cabinet and Closet Express", reads replies
        └── Drive: stores uploaded job photos
```

### Files and where they go

| File | Where |
|---|---|
| `Code.gs`, `appsscript.json` | Google Apps Script |
| `worker.js` | Cloudflare Worker (Apps Script URL and allowed site are filled in) |
| `index.html`, `lead-form.html`, `manifest.json`, `sw.js`, `icon-192.png`, `icon-512.png` | GitHub repo root |
| `CNAME` (one line: `foreman-demo.usebuildflow.io`) | GitHub repo root |
| `FOREMAN-DEMO-GUIDE.md` | Your reference only; don't upload |

Never upload `Code.gs` or `worker.js` to the public GitHub repo.

### Sheet tabs

| Tab | Holds |
|---|---|
| Posts | Scheduled and published posts (JSON per row) |
| Leads | Every lead with its full conversation (JSON per row) |
| Activity | The "What Foreman did" feed |
| Config | Brand info and Autopilot settings |
| Log | Errors, for troubleshooting |

---

## 4. Script Properties (Apps Script → Project Settings ⚙️)

| Property | Value | Required |
|---|---|---|
| `APP_PIN` | Your demo PIN | Yes |
| `GEMINI_KEY` | Key from aistudio.google.com | Yes |
| `NOTIFY_EMAIL` | Where "Lead needs you" alerts go | Yes |
| `GEMINI_MODEL` | `gemini-3.6-flash` | Recommended |
| `GEMINI_FALLBACK_MODEL` | `gemini-3.5-flash, gemini-3.1-flash-lite` | Recommended |
| `SHEET_ID` | Only if you ever use a different sheet (the ID is built into the code) | No |
| `LEAD_INBOX` | Override the lead inbox address | No |
| `AI_DAILY_CAP` | Max AI calls per day (default 200) | No |
| `ANTHROPIC_KEY` | If present, Foreman uses Claude instead of Gemini | No |

Property changes take effect immediately. No redeploy needed.

---

## 5. Setup from scratch (for the next prospect)

### Google (about 15 minutes)
1. Create a Google Sheet. Copy its ID from the URL (between `/d/` and `/edit`) and set `SHEET_ID_DEFAULT` in `Code.gs`, or add a `SHEET_ID` property.
2. **Extensions → Apps Script.** Paste `Code.gs`. Show and replace `appsscript.json` (Project Settings → "Show appsscript.json").
3. Edit `DEMO_BRAND` at the top of `Code.gs` (business name, owner, phone, service area, products, offer, tone, never-say rules).
4. Add the Script Properties from section 4.
5. Run **setup** → allow permissions (Sheets, Gmail, Drive, external requests, triggers). This creates the tabs, demo data, photo folder, hourly autopilot trigger, and 5-minute inbox trigger.
6. **Deploy → New deployment → Web app.** Execute as: **Me**. Who has access: **Anyone**. Copy the `/exec` URL.
7. Open the `/exec` URL in an incognito window. You should see `{"ok":true,"app":"foreman-demo","version":"v5",...}`.

### Cloudflare (about 5 minutes)
1. Workers & Pages → Create Worker → paste `worker.js` → Deploy.
2. Either edit `DEFAULT_APPS_SCRIPT_URL` in the code, or add a Secret `APPS_SCRIPT_URL` (the secret overrides the code).
3. Make sure `DEFAULT_ALLOWED_ORIGINS` matches the app's address exactly.
4. Open the Worker URL. It should show `{"ok":true,"app":"foreman-demo"}` (not "Hello World!").

### GitHub (about 10 minutes)
1. Put the Worker URL into `index.html` (`CONFIG.apiUrl`) and `lead-form.html` (`API_URL`).
2. Upload the six front-end files plus `CNAME` to the repo root.
3. Settings → Pages → deploy from `main` / root. Custom domain: `foreman-demo.usebuildflow.io`.
4. Cloudflare DNS: CNAME `foreman-demo` → `yourgithubname.github.io`, proxy **off** (grey cloud), unless your wildcard already covers it.
5. After the domain check passes, turn on **Enforce HTTPS**.

---

## 6. Updating the code (read this every time)

**Saving `Code.gs` does not update the live app.** The app runs the *deployed version*.

After every change to `Code.gs`:
1. Save (Cmd + S).
2. **Deploy → Manage deployments.** If there's more than one deployment, update **every** one: pencil ✏️ → Version: **New version** → Deploy.
3. Open the `/exec` URL and confirm `"version":"v5"` (or whatever the latest is).
4. If you created a *new* deployment, its URL is different: update `APPS_SCRIPT_URL` in Cloudflare.

Time triggers (the 5-minute inbox check, hourly autopilot) always run the latest *saved* code. The app runs the *deployed* version. If they behave differently, a deployment is out of date.

After changing `index.html`: replace it on GitHub, wait about a minute, then **Cmd + Shift + R** in the browser.

---

## 7. How each feature works

### Where new leads come from
| Source | How |
|---|---|
| App | **Send a test lead** button (Today or Leads) |
| Website form | `lead-form.html` |
| Email | Anything sent to **moonlighttrucking21+foreman@gmail.com**. Foreman creates the lead from sender name, subject and body, replies in the same thread, marks it read, and adds a "Foreman" Gmail label. Your normal inbox is never touched. |

### Email conversations
- Foreman's first email starts a Gmail thread; later emails reply inside it.
- When the customer replies, Foreman decides:
  - **Normal question** → answers and invites them to the free consultation. Stage: **In conversation**.
  - **Price, visit, measuring, a proposed day, upset, or something it can't answer** → says Mark will reach out today, stage **Needs you**, alert email to `NOTIFY_EMAIL`.
  - **"Stop contacting me"** → confirms and never contacts them again.
- After 6 AI replies in one conversation, Foreman hands it to Mark.
- Foreman never quotes prices or promises dates.

### In the app's conversation view
| Button | What it does |
|---|---|
| **Send email** | Sends Mark's reply in the same Gmail thread. Switches the lead to "You're handling". |
| **✨ Draft with Foreman** | AI writes a suggested reply into the box. Edit, then send. |
| **Take over** | Foreman stops messaging this customer. |
| **Let Foreman handle it** | Hands back to Foreman. If the customer's last message is unanswered, Foreman answers it immediately. |
| **Check for new replies** | Checks Gmail right now. |
| **Answer their reply now** | Appears when a reply is queued because the AI was busy. |
| **Send Foreman's reply now** | Appears when the first reply to a new lead didn't go out. |
| **Mark booked** | Stops all follow-ups. |

### Timing
| Event | How fast |
|---|---|
| New lead from app or form | 5–20 seconds |
| Email to lead inbox / customer reply, app **open** | Within 30 seconds (the app checks Gmail every 30 s) |
| Email to lead inbox / customer reply, app **closed** | Within 5 minutes (background check) |
| Gmail indexing delay | New emails can take up to ~1 minute to become searchable |
| Automatic follow-ups | Checked hourly; paused during quiet hours |

The "🤖 Foreman is answering this customer" line is a status label (who's in charge), not a timer.

### Quiet hours
Quiet hours (default 8 pm – 8 am) pause **automatic follow-ups only**. Replies to customers who write in go out any time.

### When the AI is busy or out of quota
- Foreman tries the main model twice, then each backup model.
- If all fail, the reply is **queued**. Retries back off: 15 min, 30, 60, then every 120 min, so they don't burn the quota.
- The feed shows "⏳ AI was busy, so the reply is queued."
- Leads are always saved, even if the AI fails.

### Autopilot switch
- **On:** Foreman replies, follows up, and publishes posts.
- **Off:** nothing goes out automatically; customer replies go to **Needs you**.
- Turn it off after demos so follow-up emails don't keep going to test addresses.

---

## 8. Test checklist (before every demo)

1. Autopilot → **Reset demo data**.
2. Run **testAI** in Apps Script → log shows `Code version: v5` and `AI OK`.
3. Open the Cloudflare `/exec` URL → `"version":"v5"` and `"model":"gemini-3.6-flash"`.
4. Send a test lead with your personal email → reply arrives in the inbox.
5. Reply to it → Foreman answers within about 30 seconds with the lead open.
6. Send "How much would a closet cost? Can you come Thursday?" → no price in the reply, lead in **Needs you**, alert email arrives.
7. Email `moonlighttrucking21+foreman@gmail.com` from your personal email → becomes a lead and gets a reply.
8. Create a post from a photo → **Write caption** works.
9. Check the Log tab is clean.

**Always test with an email that is not moonlighttrucking21@gmail.com.** Foreman sends from that account, so it ignores that address as a customer.

---

## 9. Demo script for Mark (15 minutes)

1. **Today screen (2 min).** "This is what Foreman did while you were working." Point out the activity feed, **Needs you**, and the brass autopilot switch.
2. **Live lead (3 min).** Hand Mark your phone with `lead-form.html` open. He fills it out with **his own email**. Show his inbox: a reply signed by Mark in seconds.
3. **Real conversation (3 min).** Mark replies from his phone ("Do you do shoe storage?"). Keep the lead open in the app. Foreman's answer arrives in his inbox and appears in the app within about 30 seconds.
4. **Handoff (2 min).** He replies "How much? Can you come Thursday?" No price given, lead moves to **Needs you**, alert email arrives. "Serious buyers come to you."
5. **Mark takes over (2 min).** Click **✨ Draft with Foreman**, adjust, **Send email**. It lands in his inbox in the same thread. Then **Let Foreman handle it**.
6. **Email leads (1 min).** "Customers who email your address become leads automatically."
7. **Posts (1 min).** Photo of something in his shop → **Write caption** → schedule.
8. **Control (1 min).** Flip autopilot off and on. Show quiet hours and the approve-first setting.
9. **Close.** Walk through the proposal: build price, support plans, next steps.

Be upfront that social posting and texting are simulated until his accounts are connected. "In your version, these connect to your real Instagram, Facebook, Google and phone number."

---

## 10. Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `Cannot read properties of null (reading 'getSheetByName')` | Live app on old code, or script not attached to the sheet | Paste latest `Code.gs`, deploy a new version of **every** deployment; check the Cloudflare URL points at a v5 deployment |
| PIN accepted but screen doesn't change | Old `index.html` (PIN-screen bug) | Replace `index.html` on GitHub, Cmd + Shift + R |
| Worker URL shows "Hello World!" | Worker code never deployed | Edit code → paste `worker.js` → Deploy |
| "Backend returned HTML" | Apps Script access isn't **Anyone**, or the deployment is stale | Manage deployments → Anyone → New version |
| `AI error 404 … model … no longer available` | Model retired | Set `GEMINI_MODEL` to a current model (run **listModels**) |
| `AI error 503 … high demand` | Gemini overloaded | Use `gemini-3.6-flash` plus backups; turn on billing |
| `AI error 429 … quota … limit: 20` | Free-tier daily limit used up | Turn on billing for the Gemini key in AI Studio, or wait for reset |
| Test lead never appears | AI failed on old code and the lead was dropped | Current code always saves leads; redeploy |
| Customer replied but nothing happens | Replied from moonlighttrucking21 itself, the reply is still indexing, or autopilot is off | Use another address; wait 30–60 s; check the autopilot switch |
| Email to the plain Gmail address isn't a lead | By design | Send to `moonlighttrucking21+foreman@gmail.com` |
| Updates only show after refreshing the page | Old `index.html` | Replace it; current version checks every 30 s |
| Console 404 for `buildflow-logo.png` | No logo uploaded | Harmless; upload a logo with that name if you want one |

### Diagnostic functions (run from the Apps Script editor)
| Function | Tells you |
|---|---|
| `testAI` | Code version, fallback list, and whether the AI answers |
| `testEmail` | Whether Foreman can send email; remaining daily quota |
| `testGmail` | Sending account, lead inbox address, each lead's email, and whether replies are found |
| `listModels` | Every Gemini model your key can use |
| `setup` | Recreates tabs and triggers; asks for any new permissions |
| `resetDemo` | Also available in the app: Autopilot → Reset demo data |

**Where to look:** Apps Script → **Executions** (every run, with errors) and the sheet's **Log** tab.

---

## 11. AI provider notes

- Default: Gemini via `GEMINI_KEY`, main model `gemini-3.6-flash`, backups `gemini-3.5-flash`, `gemini-3.1-flash-lite`.
- Avoid the newest model as the main one; it gets the most traffic and the most "high demand" errors.
- The free tier is fine for building but **not for a live demo** (limits as low as 20 requests per model). Turn on billing before meeting Mark; demo volume costs pennies.
- Free-tier data may be used by Google to improve its products, so use test data only. Use a paid key for Mark's live system.
- Add `ANTHROPIC_KEY` at any time to switch Foreman to Claude with no other changes.

---

## 12. Planned: two-way text messages (not built yet)

Texting will work the same way as email.

### How it will work
- New lead with a phone number → Foreman texts right away from the business number.
- Customer texts back → Foreman answers or hands off to Mark, same rules as email.
- Someone texts the number first → they become a new lead.
- Texts appear in the app conversation; Mark can reply by text from there.
- Customer texts STOP → Twilio blocks further texts automatically.
- Foreman checks for new texts through the Twilio API every 30 seconds with the app open (no webhook needed), and every 5 minutes in the background.

### Demo version (Twilio trial)
- Free credit and a phone number (pick an 804 number).
- Can only text **verified** numbers (yours and Mark's); messages start with "Sent from your Twilio trial account".
- US texting rules have tightened; the trial may require some verification before sending works.
- Cost: about $1–2/month for the number plus about a penny per text (check Twilio's current pricing).

### Setup steps (when ready)
1. Sign up at twilio.com (trial).
2. Buy a phone number (804 area code).
3. Verify your cell (and Mark's) under **Verified Caller IDs**.
4. Add Script Properties: `TWILIO_SID` (starts with `AC…`), `TWILIO_TOKEN`, `TWILIO_NUMBER`.
5. Ask Claude to build the texting module; paste the new `Code.gs`, run `setup`, deploy a new version, update `index.html`.

### Production version (Mark's system)
- Mark's own Twilio account and number, under his business.
- **A2P 10DLC registration** (brand: his LLC and EIN; campaign: Customer Care or Mixed; sample messages; opt-in description; privacy policy stating numbers aren't shared). Takes days to weeks, so start it when he signs.
- Website form needs the SMS consent checkbox (already in `lead-form.html`).
- Quiet hours apply to texts.

---

## 13. From demo to Mark's real system

Everything moves into accounts **Mark owns**, with Weblina as admin:

| Piece | Demo | Production |
|---|---|---|
| Google | moonlighttrucking21 | Mark's business Google account |
| Email | +foreman address | leads@hisdomain.com forwarding to Foreman's inbox |
| AI | Free Gemini key | Paid Gemini (or Anthropic) key with spend cap |
| Social posting | Simulated | Meta Graph API (Facebook + Instagram) and Google Business Profile API; approvals take weeks |
| Texting | Simulated | Twilio + A2P 10DLC |
| App address | foreman-demo.usebuildflow.io | e.g. cce.usebuildflow.io |
| Branding | Demo data | Mark's real phone, service area, offer, photos |

Start the Meta, Google Business Profile and Twilio approvals the week he signs; build and test while they're pending.

**Pricing (see the proposal doc):** build $5,500 (50/50); support Essential $199 / Plus $349 / Priority $599 per month; $1,000 off the build with 12 months of Plus or Priority; he pays AI and texting usage directly (about $25–60/month).

---

## 14. Quick checklists

### Before the meeting
- [ ] Gemini billing on (or Anthropic key added)
- [ ] `GEMINI_MODEL` = `gemini-3.6-flash`; backups set
- [ ] `/exec` shows v5; app hard-refreshed
- [ ] `NOTIFY_EMAIL` set (use Mark's email if you want him to receive the alert live)
- [ ] Autopilot → Reset demo data
- [ ] Full test run from section 8
- [ ] `lead-form.html` open on your phone
- [ ] Proposal doc open

### After the meeting
- [ ] Flip autopilot **off** (stops follow-ups to test addresses)
- [ ] Change the PIN if you shared it
- [ ] Remove or rotate `GEMINI_KEY` if the demo shouldn't stay live
- [ ] For the next prospect: edit `DEMO_BRAND`, deploy a new version, Reset demo data
