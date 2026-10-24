Second Cup – Digital Daily Report
A mobile-friendly web page (hosted free on GitHub Pages) that replaces the paper daily report. When the duty manager signs and submits, everything goes into a Google Sheet. The Sheet also holds the weekly staff schedule, the staff wage list and an automatic labor-cost summary.
```
 Phone / tablet at the till                 Google (free)
┌──────────────────────────┐   HTTPS   ┌──────────────────────────────┐
│ GitHub Pages: index.html │ ────────▶ │ Apps Script web app (Code.gs) │
│  • Daily report + sign   │ ◀──────── │        │                      │
│  • Weekly schedule       │   JSON    │        ▼                      │
│  • Labor cost dashboard  │           │ Google Sheet                  │
│  • Staff & wages         │           │  Staff · Schedule · Daily     │
└──────────────────────────┘           │  Reports · Timesheet · Labor  │
                                       │  Summary  + Drive signatures  │
                                       └──────────────────────────────┘
```
What you need: a Google account (ideally the café's, not a personal one), a free GitHub account, and about 30–40 minutes.
---
Files
File	What it is	Where it goes
`index.html`	The whole web app (HTML + CSS + JS in one file)	Your GitHub repository
`Code.gs`	The backend that reads/writes the Sheet	Apps Script editor inside your Google Sheet
`README.md`	This guide	Your GitHub repository (optional)
---
Part 1 — Create the Google Sheet (5 min)
Go to https://sheets.google.com → Blank spreadsheet.
Name it, e.g. Second Cup – Operations.
Leave it empty. The script creates every tab for you.
> Tip: create it in a shared Drive folder that you and the owner can both see, so it doesn't vanish if one person leaves.
Part 2 — Add the backend script (10 min)
In the Sheet: Extensions → Apps Script. A new tab opens with a file called `Code.gs`.
Delete everything in that file and paste in the full contents of `Code.gs`.
At the top, change the PIN:
```js
   const PIN = 'CHANGE-ME-1234';
   ```
Pick something staff won't guess (e.g. `sc-7Qm4-2026`). Anyone who has both the web-app URL and the PIN can read and write reports, so treat the PIN like a till code.
Set the time zone so dates don't shift by a day:
Click the ⚙️ Project Settings (left sidebar).
Tick Show "appsscript.json" manifest file in editor.
Back in the editor, open `appsscript.json` and set `"timeZone": "Asia/Nicosia"`.
Click 💾 Save.
In the function dropdown at the top, choose `setup` → click ▶ Run.
Google asks for permission:
Review permissions → choose your account.
You'll see "Google hasn't verified this app". That's normal for your own scripts. Click Advanced → Go to (project name) (unsafe) → Allow.
It needs Sheets (to write reports) and Drive (to save signature images).
Go back to the Sheet. You should now have these tabs:
Tab	What's in it
Staff	ID · Name · Role · Hourly Wage · Active. Fill this in first.
Schedule	One row per planned shift (date, name, start, end, break, hours, wage, cost)
Daily Reports	One row per submitted report: sales, deductions, cash-up, labor, notes, signature link
Timesheet	Actual hours per person, taken from the "Staff on shift" section of each report
Labor Summary	Formula-driven weekly view: net sales vs scheduled and actual labor cost, labor %, OVER/OK flags
Open the Staff tab, delete the two example rows, and type in your real staff and hourly wages. (You can also do this later from the Staff tab of the web app.)
Part 3 — Deploy the script as a web app (5 min)
In Apps Script: Deploy → New deployment.
Click the ⚙️ next to "Select type" → Web app.
Settings:
Description: `v1`
Execute as: Me (the script writes to the Sheet as you)
Who has access: Anyone (needed so the GitHub page can call it. The PIN is what keeps strangers out.)
Deploy → copy the Web app URL. It ends in `/exec`.
Quick test: paste this in a browser (with your URL and PIN):
```
   https://script.google.com/macros/s/XXXX/exec?action=ping&pin=YOUR-PIN
   ```
You should see `{"ok":true,"data":{"pong":true}}`. A wrong PIN returns `"Wrong PIN"`.
> ⚠️ **Important for later changes:** every time you edit `Code.gs`, go to **Deploy → Manage deployments → ✏️ Edit → Version: New version → Deploy**. That keeps the **same URL**. If you use "New deployment" instead, you get a new URL and must update every device.
Part 4 — Put the page on GitHub Pages (10 min)
Sign in at https://github.com → + → New repository.
Name: `second-cup-report` (any name works)
Visibility: Public (free GitHub Pages needs public repos. The page holds no data, so that's fine. Your data lives in the Sheet behind the PIN.)
Tick Add a README file → Create repository.
Add file → Upload files → drag in `index.html` (and this `README.md` if you like) → Commit changes.
Settings → Pages (left sidebar):
Source: Deploy from a branch
Branch: main / (root) → Save.
Wait 1–2 minutes and refresh. Your link appears at the top:
`https://YOUR-USERNAME.github.io/second-cup-report/`
Optional: hard-wire the URL so staff don't have to type it
Open `index.html` on GitHub → ✏️ edit → find the `CONFIG` block near the top of the `<script>`:
```js
const CONFIG = {
  CAFE_NAME: 'Second Cup',
  API_URL: 'https://script.google.com/macros/s/XXXX/exec',   // ← paste here
  DEFAULT_FLOAT: 150,
  TARGET_LABOR_PCT: 28,
  EMPLOYER_ONCOST_PCT: 0,
  ...
```
Commit. Each device then only needs the PIN.
> Don't put the PIN in `index.html`. The repo is public, so anyone could read it.
Part 5 — Connect each device (2 min per device)
Open the GitHub Pages link on the café phone/tablet.
Tap ⚙️ Settings:
Web app URL: the `/exec` URL (skip it if you hard-wired it)
Access PIN: your PIN
Target labor %: e.g. 28
Employer on-cost %: the employer contributions you pay on top of wages. Get the right figure from your accountant. Leave it at 0 to use bare wages.
Save & connect. The pill in the header changes from Demo mode to Google Sheets.
Add to home screen (Safari: Share → Add to Home Screen; Chrome: ⋮ → Add to Home screen) so it opens like an app.
Part 6 — Daily workflow
Manager, weekly (e.g. Sunday):
Schedule tab → Copy last week → tap cells to adjust → Save schedule.
The bottom row shows hours and € cost per day before the week even starts.
Duty manager, end of every shift/day:
Daily report tab. Date defaults to today, and staff on shift pre-fill from the schedule.
Enter the Z-report figures (Cash, Card, Wolt, Foody, Other) plus discounts/refunds/voids.
Count the drawer. Either type the total or open Count by notes & coins.
Add any paid outs (milk top-up, etc.).
Correct staff in/out times to what actually happened.
Notes → choose Signed by → sign with a finger → Sign & submit report.
Re-submitting the same date + shift replaces the earlier one, so mistakes are easy to fix.
Owner/manager, any time: Labor cost tab or the Labor Summary sheet tab (change cell B1 to any Monday to view another week).
How the numbers are calculated
Gross sales = Cash + Card + Wolt + Foody + Other
Net sales = Gross − Discounts − Refunds − Voids (staff meals/comps are recorded but not deducted. Change this in `readReport()` if you prefer.)
Expected cash = Opening float + Cash sales − Paid outs
Over/short = Cash counted − Expected cash
Cash to bank = Cash counted − Opening float
Paid hours = (Out − In) − Break. Overnight shifts are handled.
Labor cost = Hours × Hourly wage × (1 + On-cost %)
Labor % = Labor cost ÷ Net sales. Uses actual hours from the report when one exists, otherwise scheduled hours.
---
Customising (the things you'll most likely tweak)
All in `index.html` unless noted:
Want to change…	Where
Café name, default float, target %, shift presets, coin/note list	`CONFIG` at the top of `<script>`
Sales channels (e.g. add Bolt Food, remove Foody)	HTML block `02 Sales` + `readReport()` `sales` object + `Code.gs` `REPORTS.headers` and `saveReport_()` row. Keep the column order identical in both.
Shift options (Morning/Evening/Full day)	`<select id="r_shift">`
Colours	CSS variables in `:root` at the top of `<style>`
Net-sales formula	`readReport()` → `const net = …`
After changing `Code.gs` headers on a Sheet that already has data, either add the new column manually in the same position or start a fresh Sheet and run `setup()` again.
---
Troubleshooting
Symptom	Fix
"Couldn't reach Google Sheets: Failed to fetch"	URL wrong, or deployment access isn't Anyone. Re-check Part 3. The URL must end in `/exec`, not `/dev`.
"Wrong PIN"	PIN in Settings ≠ PIN in `Code.gs`. Remember to deploy a new version after changing it.
Edited Code.gs but nothing changed	You didn't redeploy. Deploy → Manage deployments → Edit → New version.
Dates one day off	Set `"timeZone": "Asia/Nicosia"` in `appsscript.json` and in File → Settings of the Sheet, then redeploy.
Shift times show as strange dates in the Sheet	Re-run `setup()`. It formats the time columns as plain text.
GitHub page shows 404	Wait 2 minutes. Check Settings → Pages is on `main` / root and the file is named exactly `index.html`.
Settings disappear	They're stored per browser. Private/incognito mode wipes them. Use a normal tab or hard-wire `API_URL`.
Security notes (plain English)
The GitHub page is public, but it contains no data. All data stays in your Google Sheet.
The Apps Script URL + PIN together act as the key. Don't post either publicly. Change the PIN (and redeploy) if a staff member with access leaves.
Wages are visible in the app's Staff tab to anyone with the PIN. If you don't want shift staff to see wages, give them a device that only uses the Daily report, or ask for a two-PIN version (manager vs staff). That's an easy upgrade.
Signatures are saved as PNG files in a Drive folder called Daily Report Signatures. Each report row links to its signature.
