# ConPilot — AI co-pilot for construction projects

Upload a construction schedule (Excel or CSV from Smartsheet, MS Project, Primavera or an AutoCAD data-extraction table) and ConPilot will:

- **Catch data errors** such as duplicate WBS codes, impossible dates, broken predecessor links, missing budgets, unbooked costs and summary roll-ups that don't add up.
- **Control cost** with earned value (CPI, EAC), cost by package and by trade, and a monthly cash-flow chart.
- **Control schedule** with an interactive timeline, overdue and behind-plan tasks, and an earned-schedule forecast finish date.
- **Test scenarios** with what-if sliders for the remaining work's cost and schedule performance.
- **Answer questions with AI** by writing a weekly status briefing, explaining what's driving an overrun, or drafting the email to a subcontractor.

## Project structure

```
public/index.html   page layout
public/styles.css   design (light and dark mode)
public/app.js       parsing, checks, earned value, charts, AI client
api/ask.js          secure server function that talks to the Claude API
vercel.json         security headers (CSP, HSTS, no framing)
.env.example        settings you add in Vercel
```

## Make it yours

Open `public/app.js` and fill in the `BRAND` block at the top with your name, role and LinkedIn link. It shows in the footer.

## Deploy to Vercel from GitHub

1. Create a new repository on GitHub and upload everything in this folder (or `git init`, commit and push).
2. In Vercel, click **Add New → Project**, import the repository, leave the framework as **Other**, and click **Deploy**.
3. Get an API key at https://console.anthropic.com. Set a **monthly spend limit** there too.
4. In Vercel, open **Project → Settings → Environment Variables** and add:
   - `ANTHROPIC_API_KEY`: your key (required)
   - `ACCESS_CODE`: a code people must type before using the AI (recommended if you share the link publicly)
5. Go to **Deployments**, open the latest one, and click **Redeploy** so the variables take effect.

## Use your subdomain

1. In Vercel, open **Project → Settings → Domains** and add `conpilot.vimleshsonawane.com`.
2. At your domain provider, add a DNS record:
   - Type: `CNAME`
   - Name: `conpilot`
   - Value: `cname.vercel-dns.com`
3. Wait a few minutes. Vercel issues the HTTPS certificate automatically.

## Security built in

- The uploaded file is read in the browser and never stored. Only a compact JSON summary is sent to the AI when you ask a question.
- The API key lives only in a Vercel environment variable. It's never sent to the browser or committed to GitHub.
- The system prompt is fixed on the server, so the endpoint can't be used as a free general chatbot.
- Requests are checked for origin, size, message shape and an optional access code (compared in constant time), and limited per IP.
- Spreadsheet text is treated as untrusted. It's HTML-escaped everywhere, and the AI is told never to follow instructions inside it.
- Exported CSVs neutralize formula injection (`=`, `+`, `-`, `@` at the start of a cell).
- Strict headers: Content Security Policy, HSTS, `X-Frame-Options: DENY`, `nosniff`, a locked-down Permissions-Policy.
- File size (15 MB) and row (5,000) limits. The spreadsheet library is loaded at SheetJS 0.20.3, which fixes the known parsing vulnerability in older versions.

### Optional hardening

- **Self-host the libraries.** Download `xlsx.full.min.js` (https://cdn.sheetjs.com/xlsx-0.20.3/package/dist/xlsx.full.min.js) and `chart.umd.js` (https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.js) into `public/vendor/`. The app loads them from there first. Then remove the three CDN hosts from `script-src` in `vercel.json`.
- **Shared rate limiting.** The built-in limiter is per server instance. For a busy public site, swap it for Upstash Redis or Vercel KV.

## Run locally

```
npm i -g vercel
vercel dev
```

Then open http://localhost:3000. Put your key in a local `.env` file (it's git-ignored).

## How the numbers work

- **PV (planned value):** budget × planned % complete at the status date (linear between start and finish).
- **EV (earned value):** budget × actual % complete.
- **AC (actual cost):** cost booked to date.
- **CPI = EV ÷ AC** and **SPI = EV ÷ PV.** Below 1.00 is unfavourable.
- **EAC (forecast cost) = BAC ÷ CPI.**
- **Forecast finish** uses *earned schedule*: the date on the plan when today's earned value should have been reached, projected to the end.
