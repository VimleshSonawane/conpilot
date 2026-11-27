# ConPilot: Scrum Backlog Reconstructed from Git History

> **Source of evidence:** `git log` (4 commits, 2026-10-17 → 2026-10-19), `conpilot/README.md`, `conpilot/package.json`, `conpilot/vercel.json`, `conpilot/api/ask.js`, `conpilot/public/*`, and the root `index.html` (including its earlier version, `index (9).html` at `b7e0166`).
> **Rule followed:** everything listed below exists in the code or the history. Anything uncertain is marked **[unclear]**.

---

## 0. Evidence summary

| Hash | Date | Message | What it contains |
|---|---|---|---|
| `8b8dbb1` | 2026-10-17 | Add files via upload | The modular app in `conpilot/`: `README.md`, `api/ask.js` (157 lines, AI endpoint), `public/app.js` (734), `public/index.html` (171), `public/styles.css` (208), `public/vendor/README.txt`, `package.json`, `vercel.json`. 1,374 lines added. |
| `b7e0166` | 2026-10-17 | Add files via upload | `index (9).html` (1,176 lines): a single-file build of the same app with CSS and JS inline, plus a `CONPILOT_CONFIG` settings block. |
| `e43e5fa` | 2026-10-17 | Rename index (9).html to index.html | Rename only (0 lines changed). |
| `eae5e20` | 2026-10-19 | Update index.html | Major expansion of the root `index.html`: +2,131 / −530 lines (now 2,777 lines). |

**Caveats that affect how this backlog should be read:**
- Every commit was made through the GitHub web UI ("Add files via upload", "Update index.html"). The code was written somewhere else and uploaded in large batches. Commit dates are **upload dates, not development dates**, and one commit can hold many stories. **[unclear]** how long each feature really took to build.
- The repo has **two parallel versions of the front end**: `conpilot/public/` (modular, set up for Vercel) and the root `index.html` (single file). Sprint 2 work went only into the root file. **[unclear]** which one is deployed at the live URL.
- There are no branches, PRs, tags, issues, or tests in the repo. Story points are estimated from the size and complexity of the code, as you asked.

---

## 1. Epics

| ID | Epic | One-line goal |
|---|---|---|
| **E1** | Schedule Upload & Parsing | Let a PM drop in the schedule export they already have (Excel, CSV, later PDF) and have it read correctly in the browser. |
| **E2** | Data Quality Checks | Catch schedule and cost data errors automatically before they mislead a report. |
| **E3** | Cost Control (Earned Value) | Show where the money is going and where it will land, using EV metrics, charts and what-if scenarios. |
| **E4** | Schedule Control | Show whether the job is on time, what drives the finish date, and what's coming up next. |
| **E5** | AI Assistant | Answer project questions and write briefings with Claude, using only the project's own data, through a secure server. |
| **E6** | Dashboard & UI / UX | Make the analysis easy to navigate: tabs, filters, drill-down, theming, landing pages and an in-app guide. |
| **E7** | Advanced Project Controls | Add quantity takeoff checks, week-over-week comparison, a risk register, trade performance and a PDF report. |
| **E8** | Security & Deployment | Ship safely on Vercel with locked-down headers, secrets kept on the server, and a custom domain. |

---

## 2. User Stories

Personas used: **PM** = construction project manager, **SE** = site engineer, **User** = any visitor.
"Delivered by" lists the commit(s) that contain the code.

### E1: Schedule Upload & Parsing

**US-1.1: Upload a schedule file** · 3 pts · Sprint 1
*As a PM, I want to drag and drop my Excel or CSV schedule export so that I can analyse it without retyping data.*
- Accepts `.xlsx`, `.xls`, `.xlsm`, `.csv` by drag-and-drop or the file picker (`#drop`, `#file`, `handleFile()`).
- The file is parsed in the browser with SheetJS 0.20.3 and never uploaded. Limits are 15 MB and 5,000 rows (`LIMITS`).
- Libraries load from `/vendor/` first, then fall back to CDNs (`LIBS`, `loadScript()`).
- Delivered by: `8b8dbb1`, `b7e0166`

**US-1.2: Flexible column mapping** · 3 pts · Sprint 1
*As a PM, I want ConPilot to recognise my columns even when the names differ by tool so that exports from Smartsheet, MS Project, Primavera or AutoCAD all work.*
- `mapHeaders()` and `fromGrid()` detect WBS, Task, Start, Finish, % Complete, Budget, Actual Cost, Predecessors and Owner.
- `parseDate()`, `parseNum()` and `parsePred()` handle mixed date, number and predecessor formats.
- The detected mapping is shown to the user in an expandable line (`#mapline`).
- Delivered by: `8b8dbb1`, `b7e0166`

**US-1.3: Sample project and template** · 2 pts · Sprint 1
*As a User, I want to try a sample project and download a blank template so that I can see the value before I use my own data.*
- "Try the sample project" loads a generated schedule with dates relative to today (`sampleGrid()`).
- "Download template" saves `conpilot-schedule-template.csv` with the expected headers and example rows.
- Delivered by: `8b8dbb1`, `b7e0166`

**US-1.4: Import a schedule from PDF** · 5 pts · Sprint 2
*As a SE, I want to upload a PDF schedule printout so that I can analyse schedules I only have as PDFs.*
- `.pdf` is accepted on the upload zones. pdf.js 3.11.174 extracts text with eval disabled (`readPdf()`, `isEvalSupported:false`).
- `gridFromPdfPages()` rebuilds rows and columns from text positions, then feeds the normal parser.
- If pdf.js fails to load, the user is told to export to Excel or CSV instead.
- Delivered by: `eae5e20`

**US-1.5: Clear rejection of unusable files** · 1 pt · Sprint 2
*As a User, I want a clear message when my file can't be used so that I know what to change.*
- `rejectFile()` shows "This file can't be used." with a reason in an alert box.
- The drop zone gets a temporary `bad-file` visual state and scrolls into view.
- Delivered by: `eae5e20`

### E2: Data Quality Checks

**US-2.1: Automatic data-error detection** · 5 pts · Sprint 1
*As a PM, I want every row checked for errors on upload so that I catch problems before my client does.*
- `analyse()` flags: duplicate WBS codes, missing or unreadable dates, finish before start, % complete outside 0–100, missing budget, predecessors that don't exist, a task starting before its predecessor finishes, and summary budgets that don't match their child tasks.
- It also flags cost problems: complete with no cost booked, progress with no cost, spending with no progress, over budget by more than 5% or 15%, and CPI below 0.85.
- Each issue has a severity (high, med, low), a category (data, cost, schedule) and a plain-English suggested fix. Issues are sorted by severity.
- Delivered by: `8b8dbb1`, `b7e0166`

**US-2.2: Problems tab and issue-log export** · 3 pts · Sprint 1
*As a PM, I want to filter problems by severity and export them so that I can send a fix list to the scheduler.*
- The Problems tab shows a count badge and severity filters (`#sevFilters`, `renderIssues()`).
- "Export issue log (CSV)" saves `issue-log-<status date>.csv`.
- CSV cells starting with `=`, `+`, `-`, `@`, tab or CR are escaped so they can't run as formulas (`csvCell()`).
- Delivered by: `8b8dbb1`, `b7e0166`

### E3: Cost Control (Earned Value)

**US-3.1: Earned-value headline readings** · 5 pts · Sprint 1
*As a PM, I want BAC, EV, PV, AC, CPI, SPI, EAC and VAC calculated as of a status date so that I know the health of the job at a glance.*
- PV uses straight-line planned % between start and finish. EV = budget × % complete. CPI = EV/AC. SPI = EV/PV. EAC = BAC/CPI (`analyse()`).
- Summary rows are excluded from the totals so costs aren't counted twice (`isSummary` detection by WBS prefix).
- The status date can be changed from the header (`#statusDate`).
- Delivered by: `8b8dbb1`, `b7e0166`

**US-3.2: Cost charts** · 5 pts · Sprint 1
*As a PM, I want charts of cost performance so that I can explain the overrun in a meeting.*
- Overview charts include an S-curve of planned vs earned vs spent with the forecast, budget/earned/spent by package, CPI by trade, and planned spend by month (Chart.js 4.4.1).
- Chart colours follow light or dark mode (`themeChart()`).
- Delivered by: `8b8dbb1`, `b7e0166`

**US-3.3: What-if scenario sliders** · 3 pts · Sprint 1
*As a PM, I want to test how the remaining work could perform so that I can show best and worst cases.*
- Sliders for remaining-work CPI (0.6–1.3) and SPI (0.5–1.4) (`#wiCpi`, `#wiSpi`).
- Forecast cost and time update live (`renderWhatIf()`).
- Delivered by: `8b8dbb1`, `b7e0166`

**US-3.4: Task table with CSV export** · 2 pts · Sprint 1
*As a PM, I want a full task table I can export so that I can use the analysis in other tools.*
- The Cost tab lists all tasks (`renderTable()`).
- "Export analysis (CSV)" includes status, planned %, earned, CPI and forecast at completion.
- Delivered by: `8b8dbb1`, `b7e0166`

**US-3.5: Dedicated cash-flow view** · 2 pts · Sprint 2
*As a PM, I want a cash-flow tab so that I can tell the owner how much funding I need in the coming months.*
- A new `cash` tab with a summary and a monthly table (`renderCash()`, `cashData()`).
- Months are split around the status date. "No dated tasks with budgets" is shown when there's nothing to plot.
- Delivered by: `eae5e20`

### E4: Schedule Control

**US-4.1: Interactive timeline (Gantt)** · 5 pts · Sprint 1
*As a SE, I want a colour-coded timeline of tasks so that I can see what's late at a glance.*
- `renderGantt()` draws planned bars plus status: on track, behind, overdue, done.
- Clicking a task opens a details drawer with its problems and predecessors (`openDrawer()`). Escape closes it.
- Delivered by: `8b8dbb1`, `b7e0166`

**US-4.2: Forecast finish date (earned schedule)** · 3 pts · Sprint 1
*As a PM, I want a forecast finish based on real progress so that I can warn the client early.*
- Earned schedule (ES) is found by searching for the date when planned value equals today's EV. Schedule variance is reported in days.
- The forecast finish is never earlier than the planned finish while work remains.
- Delivered by: `8b8dbb1`, `b7e0166`

**US-4.3: Watch list** · 2 pts · Sprint 1
*As a PM, I want the riskiest tasks listed first so that I know where to focus this week.*
- `renderWatch()` ranks tasks by cost and schedule risk on the Overview tab.
- Delivered by: `8b8dbb1`, `b7e0166`

**US-4.4: Critical path analysis** · 8 pts · Sprint 2
*As a PM, I want the critical path calculated from my predecessors so that I know which tasks drive the finish date.*
- `cpm()` runs a forward and backward pass supporting FS, SS, FF and SF links with lag, and calculates float for each task.
- The Critical path tab lists tasks by float with forecast start and finish against the planned finish (`renderCritical()`), plus a "What drives the finish date" card.
- Completion slip on the critical path (`cpSlip`) feeds risks and the assistant's summaries.
- Delivered by: `eae5e20`

**US-4.5: Three-week look-ahead** · 3 pts · Sprint 2
*As a SE, I want a 3-week look-ahead with readiness status so that I can plan crews and clear blockers.*
- `lookaheadRows()` shows unfinished tasks starting in the next 21 days, grouped by week.
- Readiness labels include "In progress", "Waiting on predecessors" (based on unfinished FS, FF and SS links) and "Should have started".
- Delivered by: `eae5e20`

### E5: AI Assistant

**US-5.1: Secure AI endpoint** · 8 pts · Sprint 1
*As a PM, I want AI answers without exposing an API key so that I can share the tool publicly and safely.*
- `api/ask.js` is a Vercel function. The key comes only from `ANTHROPIC_API_KEY`, and a GET health check tells the page whether AI is on.
- Checks on every request: allowed origin, JSON content type, task type, context size (150k chars), message count and shape (12 max, 4k chars each), and a per-IP rate limit (default 30 per 10 minutes).
- The system prompt is fixed on the server and tells the model to use only the project data, treat it as untrusted, cite WBS codes, and decline unrelated topics. Responses are streamed to the browser.
- Delivered by: `8b8dbb1`

**US-5.2: One-click weekly status briefing** · 3 pts · Sprint 1
*As a PM, I want a weekly meeting briefing written for me so that I save time on reporting.*
- "Write briefing" sends `task:"brief"`. The server adds a fixed template: Headline, Top risks, Actions this week, Fix in the data (under 230 words).
- The output is rendered from a safe markdown subset (`md()` escapes HTML first).
- Delivered by: `8b8dbb1`, `b7e0166`

**US-5.3: Chat about the project** · 5 pts · Sprint 1
*As a PM, I want to ask questions such as "what's driving the overrun?" or "draft an email to the sub" so that I get answers grounded in my numbers.*
- The chat keeps the last 10 turns and streams replies. A Stop button cancels the request (`AbortController`).
- Suggested-question chips are built from the data, for example the worst-CPI owner or the most-late task (`renderChips()`).
- Only a compact JSON summary is sent: up to 400 tasks and 150 issues (`context()`).
- Delivered by: `8b8dbb1`, `b7e0166`

**US-5.4: Access-code gate for AI** · 2 pts · Sprint 1
*As the site owner, I want an optional access code so that strangers can't run up my API bill.*
- If `ACCESS_CODE` is set, the page shows a password field and sends it as `X-Access-Code`.
- The server compares it in constant time (`safeEqual()`) and returns 401 if it's wrong. The client then shows "Enter the access code".
- Delivered by: `8b8dbb1`

**US-5.5: Run inside Claude as well as on Vercel** · 2 pts · Sprint 1
*As a User, I want the AI to work when the page is opened inside Claude so that it can be demoed without a deployed server.*
- If `window.claude.use` exists, the page uses Claude's built-in model and downloads, and applies the same rules as the server (`RULES`, `inClaude`).
- Otherwise it falls back to `/api/ask`, or shows a message explaining that AI is off.
- Delivered by: `8b8dbb1`, `b7e0166`. **[unclear]** whether this mode is still used.

### E6: Dashboard & UI / UX

**US-6.1: Tabbed dashboard with cross-filtering** · 5 pts · Sprint 1
*As a PM, I want to click a chart slice or package and have every view filter so that I can drill into one area quickly.*
- Tabs in Sprint 1: Overview, Schedule, Cost, Problems, Ask AI (`showTab()`).
- Filters by package, status, owner and free-text search, shown as removable chips (`setFilter()`, `renderFilters()`).
- Delivered by: `8b8dbb1`, `b7e0166`

**US-6.2: Light and dark mode** · 2 pts · Sprint 1
*As a User, I want the app to follow my system theme so that it's comfortable to use on site or in the office.*
- `styles.css` supports both themes. Charts re-render when `prefers-color-scheme` changes.
- Delivered by: `8b8dbb1`, `b7e0166`

**US-6.3: Configurable branding** · 2 pts · Sprint 1
*As the site owner, I want to set my name, role and links in one place so that the tool credits me.*
- `8b8dbb1`: a `BRAND` block in `app.js` feeds the footer, and links must be https.
- `b7e0166`: a larger `window.CONPILOT_CONFIG` block (name, tagline, headline, intro, features, website, GitHub, LinkedIn). `applyConfig()` truncates and escapes every value.
- Delivered by: `8b8dbb1`, `b7e0166`

**US-6.4: Multi-page landing site** · 5 pts · Sprint 2
*As a User (for example a recruiter or prospective client), I want a landing site explaining the problem, how it works, a demo and the author so that I understand ConPilot before uploading.*
- Hash-routed pages: home, why, how, demo, about (`route()`, `showLanding()`, `go()`).
- Includes a rotating headline, count-up stats, an animated mock dashboard with cycling toasts, and "revision cloud" SVG accents. Animations respect `prefers-reduced-motion` (`landing()`, `motion()`, `clouds()`).
- Content sections: "Where current tools fall short, and what ConPilot does about it", "What ConPilot finds in the sample project", and an About page with title-block details.
- Delivered by: `eae5e20`

**US-6.5: Choose a starting view** · 1 pt · Sprint 2
*As a User, I want to say what I want to check before uploading so that I land on the right view.*
- Goal chips (overview, schedule, critical, look-ahead, cost, cash, ...) with notes about required columns (`setGoal()`, `GOAL_NOTES`).
- Delivered by: `eae5e20`

**US-6.6: Manual theme toggle** · 1 pt · Sprint 2
*As a User, I want a light/dark switch that remembers my choice so that I don't depend on system settings.*
- `#themeBtn` toggles the theme and saves it to `localStorage` (`conpilot-theme`). It's applied before first paint to avoid a flash.
- Delivered by: `eae5e20`

**US-6.7: Floating navigation assistant** · 5 pts · Sprint 2
*As a User, I want to type "show the critical path" or "clear filters" so that I can find my way without learning the UI.*
- A floating button opens a chat panel (`openBot()`). Rules-based intents open views, load the sample, clear filters, and summarise look-ahead, cash, trades, risks, critical path or quantities (`INTENTS`, `botHandle()`).
- Free-text filters are tried first (`tryFilter()`). Real questions are routed to the AI when data is loaded.
- Delivered by: `eae5e20`

### E7: Advanced Project Controls (all Sprint 2)

**US-7.1: Quantity takeoff check** · 8 pts · Sprint 2
*As a PM, I want to upload a quantity takeoff (for example from AutoCAD) and check it against the schedule budget so that I catch double counts and maths errors.*
- A separate quantity file upload (`#qtyFile`, `qtyFromGrid()`, `loadQty()`) needs a description or code column plus a quantity column.
- `analyseQty()` flags unit mismatches (the same item in different units), duplicate item codes, missing or zero quantity, missing unit rate, amount ≠ quantity × rate, takeoff value vs schedule budget, and WBS codes not in the schedule.
- There's a quantities sample (`sampleQtyGrid()`) and a "Priced takeoff vs schedule budget" chart.
- Delivered by: `eae5e20`

**US-7.2: Compare with last week** · 5 pts · Sprint 2
*As a PM, I want to load last week's schedule and see what changed so that slippage doesn't creep in unnoticed.*
- `loadPrev()` takes a previous file and status date, and checks that the previous date is earlier.
- `compare()` matches tasks by WBS and name (falling back to name only) and reports added, removed and changed tasks plus "Earned vs spent this period" (`renderChanges()`).
- Delivered by: `eae5e20`

**US-7.3: Risk register** · 5 pts · Sprint 2
*As a PM, I want risks generated from the data with probability and impact scores so that I can bring a register to the meeting.*
- `buildRisks()` creates schedule and cost risks from overdue or behind tasks (weighted higher on the critical path) and from cost issues.
- `renderRisks()` draws a 3×3 probability × impact matrix and the top 3 risks with their response.
- Delivered by: `eae5e20`

**US-7.4: Trade / subcontractor performance** · 3 pts · Sprint 2
*As a PM, I want performance grouped by trade so that I know which subcontractor to call.*
- `tradeStats()` and `renderTrades()` give, per owner: task count, budget, actual, earned, overdue, behind, critical count, and forecast overrun.
- Delivered by: `eae5e20`

**US-7.5: PDF status report export** · 5 pts · Sprint 2
*As a PM, I want a one-click PDF report so that I can email a polished status pack.*
- `makeReport()` builds an A4 PDF with jsPDF 2.5.1. Charts are drawn off-screen in the light theme and embedded as images (`chartImage()`).
- Sections include: progress and cost curve, budget by package, planned spend by month, critical path, watch list, problems with "Why it matters / What to do", quantity takeoff check, what changed, and the AI status briefing.
- Delivered by: `eae5e20`. **[unclear]** whether the "reports" tab (`data-tab="reports"`) contains more than this export.

---

## 3. Bugs & Rework

No commit message in the history says "fix", "revert" or "refactor". The items below are rework that the diffs show.

| ID | Type | What broke / why | How it was resolved | Hash | Pts |
|---|---|---|---|---|---|
| **BR-1** | Bug (file naming) | The single-file build was uploaded as `index (9).html`, a browser download suffix. As the repo root page it wouldn't be served at `/`. | Renamed to `index.html` the same day (0 line changes). | `b7e0166` → `e43e5fa` | 1 |
| **BR-2** | Rework / restructure | The modular app (`conpilot/public` + `api`) was followed 18 minutes later by a second, single-file copy of the whole front end at the repo root. That's two codebases to maintain. | Not resolved. The single file became the main line of development (Sprint 2 changed only it). `conpilot/public/app.js` has stayed at its Sprint 1 version. | `b7e0166` | 2 |
| **BR-3** | Rework (UI and design overhaul) | The Sprint 1 UI was replaced: 530 lines removed or rewritten. The font changed from Barlow/Barlow Condensed to Inter, the favicon and hero copy were rewritten, and chart titles were renamed ("Progress and cost curve" → "Planned vs earned vs spent", "Cost performance by trade" → "Cost index by trade", "Data problems by type" → "Data problems"). The 5-tab layout grew to 13 tabs. | Done in one big commit together with all the Sprint 2 features. **[unclear]** whether this was driven by user feedback. | `eae5e20` | 3 |

### Open issues seen in the code (not fixed in the history)

These are not counted in any sprint. They are candidates for the next sprint's backlog.

| ID | Observation | Evidence |
|---|---|---|
| OI-1 | The README lists `.env.example`, but the file isn't in the repo. | `conpilot/README.md` project structure; file missing. |
| OI-2 | The root `index.html` (where Sprint 2 lives) is outside Vercel's `outputDirectory: "public"`, but it still calls `/api/ask`, which exists only under `conpilot/api/`. **[unclear]** how the Sprint 2 build is deployed. | `conpilot/vercel.json`, root `index.html`. |
| OI-3 | The rate limit is in memory and per server instance. The code and README both say to move to Vercel KV or Upstash. | `api/ask.js` comment; README "Optional hardening". |
| OI-4 | Vendor libraries aren't self-hosted yet. `public/vendor/` holds only a README, so the CSP still allows three CDN hosts. | `public/vendor/README.txt`, `vercel.json` CSP. |
| OI-5 | The README says to fill in `BRAND` in `app.js`. The root build uses `CONPILOT_CONFIG` instead, so the docs no longer match the main build. | README "Make it yours"; root `index.html`. |
| OI-6 | There are no automated tests anywhere in the repo. | Folder structure; `package.json` has no scripts. |

---

## 4. Technical Tasks (not user-facing)

Tasks are tracked without story points, which is common Scrum practice. The status column shows only what the repo proves.

| ID | Task | Evidence | Hash | Status |
|---|---|---|---|---|
| T-1 | Create the GitHub repo and upload the initial project | First commit | `8b8dbb1` | Done |
| T-2 | `package.json`: ES module, Node ≥ 18, private, no dependencies | `conpilot/package.json` | `8b8dbb1` | Done |
| T-3 | Vercel config: `outputDirectory: public`, `cleanUrls` | `conpilot/vercel.json` | `8b8dbb1` | Done |
| T-4 | Security headers: CSP, HSTS (2 years, preload), X-Frame-Options DENY, nosniff, Referrer-Policy, Permissions-Policy, COOP | `conpilot/vercel.json` | `8b8dbb1` | Done |
| T-5 | Environment variables: `ANTHROPIC_API_KEY` (required), `ACCESS_CODE`, `ALLOWED_ORIGINS`, `RATE_LIMIT`, `ANTHROPIC_MODEL` | `api/ask.js`, README | `8b8dbb1` | Code done. **[unclear]** whether they are set in Vercel. |
| T-6 | Anthropic API key and monthly spend limit | README step 3 | `8b8dbb1` | Documented. **[unclear]** whether done. |
| T-7 | Custom subdomain `conpilot.vimleshsonawane.com` (CNAME → `cname.vercel-dns.com`) | README "Use your subdomain" | `8b8dbb1` | Documented. **[unclear]** whether DNS is live. |
| T-8 | Library loading with fallbacks (vendor → CDN → CDN); SheetJS pinned to 0.20.3 for the parsing CVE fix | `LIBS` in `app.js` | `8b8dbb1` | Done |
| T-9 | Single-file build with inline CSS/JS and a settings block | `index (9).html` | `b7e0166` | Done (see BR-2) |
| T-10 | Add pdf.js 3.11.174 and jsPDF 2.5.1 to the library loader (with worker) | `LIBS` in root `index.html` | `eae5e20` | Done |
| T-11 | README: deploy, local dev (`vercel dev`), security notes, EV formulas | `conpilot/README.md` | `8b8dbb1` | Done |

---

## 5. Story Point Summary

| Sprint | Stories | Bugs/Rework | Total |
|---|---|---|---|
| Sprint 1 | US-1.1 (3), 1.2 (3), 1.3 (2), 2.1 (5), 2.2 (3), 3.1 (5), 3.2 (5), 3.3 (3), 3.4 (2), 4.1 (5), 4.2 (3), 4.3 (2), 5.1 (8), 5.2 (3), 5.3 (5), 5.4 (2), 5.5 (2), 6.1 (5), 6.2 (2), 6.3 (2) = **70** | BR-1 (1), BR-2 (2) = **3** | **73** |
| Sprint 2 | US-1.4 (5), 1.5 (1), 3.5 (2), 4.4 (8), 4.5 (3), 6.4 (5), 6.5 (1), 6.6 (1), 6.7 (5), 7.1 (8), 7.2 (5), 7.3 (5), 7.4 (3), 7.5 (5) = **57** | BR-3 (3) = **3** | **60** |
| **Total** | **127** | **6** | **133** |

---

## 6. Sprints

> **How the sprints were split:** the history has commits on only two days, 2026-10-17 and 2026-10-19, with no commits on 2026-10-18. Following your rule that a gap with no commits is a sprint boundary, there are **2 sprints**. Both are much shorter than the 1–2 week length you asked for, because all the work was uploaded in a 3-day window. The real development time behind each upload is **[unclear]** (see caveats in section 0).

### Sprint 1: Foundation & MVP
- **Dates:** 2026-10-17 → 2026-10-17 (all three commits uploaded between 22:42 and 23:00 EDT)
- **Sprint goal:** Ship a working, securely deployable MVP: upload a schedule, check it for errors, show earned value and timeline, and answer questions with AI.
- **Completed:**
  - Stories: US-1.1, US-1.2, US-1.3, US-2.1, US-2.2, US-3.1, US-3.2, US-3.3, US-3.4, US-4.1, US-4.2, US-4.3, US-5.1, US-5.2, US-5.3, US-5.4, US-5.5, US-6.1, US-6.2, US-6.3
  - Bugs/Rework: BR-1, BR-2
  - Tasks: T-1 to T-9, T-11
- **Commits:** `8b8dbb1`, `b7e0166`, `e43e5fa`
- **Total points:** **73**

### Sprint 2: Advanced Controls & Product Polish
- **Dates:** 2026-10-19 → 2026-10-19 (single commit at 01:24 EDT)
- **Sprint goal:** Turn the MVP into a full project-controls product with critical path, quantities, change tracking, risks and PDF reporting, wrapped in a portfolio-ready landing site.
- **Completed:**
  - Stories: US-1.4, US-1.5, US-3.5, US-4.4, US-4.5, US-6.4, US-6.5, US-6.6, US-6.7, US-7.1, US-7.2, US-7.3, US-7.4, US-7.5
  - Bugs/Rework: BR-3
  - Tasks: T-10
- **Commits:** `eae5e20`
- **Total points:** **60**

**Velocity:** 73 → 60. The average is about 66 points per sprint, but because the sprints are upload batches, velocity should not be used to forecast. **[unclear]**

---

## 7. Retrospective Notes

### Sprint 1 retro
- **Went well:** The MVP shipped complete and with security built in from the first commit: the API key stays on the server, the system prompt is fixed, there's origin, size and rate checking, a constant-time access-code check, CSV formula-injection escaping and strict security headers. None of that needed rework later.
- **To improve:** Within 18 minutes a second, single-file copy of the front end was added (BR-2), and it needed a same-day rename fix (BR-1). Pick one source of truth for the front end, and name files before uploading them.

### Sprint 2 retro
- **Went well:** A large amount of value was delivered: critical path, 3-week look-ahead, quantity takeoff checks, week-over-week comparison, risk register, trade view, PDF import and export, plus a landing site. All of it was built on the Sprint 1 analysis engine without changing the AI endpoint.
- **To improve:** Everything went into one 2,131-line commit with the generic message "Update index.html", mixing 14 stories with a UI overhaul (BR-3). That makes review, rollback and traceability hard. The new features also exist only in the root file, which has drifted from the deployable `conpilot/` version (OI-2, OI-5). Next time, use small commits or PRs per story with descriptive messages, and bring the two front ends back together.
