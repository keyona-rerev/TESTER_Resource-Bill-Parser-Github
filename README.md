# Bill Parser — Automated Utility Bill Extraction System

Automated pipeline for extracting structured data from Canadian utility bill PDFs. Bills dropped into a Google Drive folder are processed by Claude (primary extraction) and Grok (independent validation). Agreed results write directly to Google Sheets; conflicts route to a Review Queue with email alerts.

## Architecture

```
Drive Folder (PDF drop) → GAS Trigger (every 15 min)
  → Claude API (extract) + Grok API (validate independently)
  → Validator (compare + sanity check)
  → [Agreed] Google Sheets (Electricity / Natural Gas / Water tabs)
  → [Conflict] Review Queue tab + email alert to ALERT_EMAIL
  → GitHub API → data/bills.json → this dashboard
```

**Supported bill types:** Toronto Hydro, Alectra (electricity + water), Enbridge Gas, Toronto Water, Region of Peel Water

---

## Setup

### 1. Prerequisites
- Google Sheet (create one, copy the Sheet ID from the URL)
- Google Drive folder for bill PDFs (copy the Folder ID from the URL)
- Anthropic API key (Claude)
- xAI API key (Grok)
- GitHub fine-grained PAT with `contents:write` on this repo only

### 2. Script Properties
In Apps Script editor → Project Settings → Script Properties, add all 7:

| Key | Value |
|-----|-------|
| `CLAUDE_API_KEY` | Your Anthropic API key |
| `GROK_API_KEY` | Your xAI API key |
| `GITHUB_TOKEN` | Fine-grained PAT, contents:write on this repo |
| `GITHUB_REPO` | `keyona-rerev/TESTER_Resource-Bill-Parser-Github` |
| `SHEET_ID` | ID from Google Sheet URL |
| `DRIVE_FOLDER_ID` | ID of the Drive folder for bill PDFs |
| `ALERT_EMAIL` | Email for Review Queue and error alerts |

### 3. Initialize
Run these in order from the Apps Script editor or Bill Parser menu:

1. **Run `diagnosConfig()`** from the editor — all 7 properties must show `✓ SET`
2. **Bill Parser → Setup Sheets** — creates 5 tabs (Electricity, Natural Gas, Water, Review Queue, Audit Log)
3. **Bill Parser → Setup Trigger** — sets 15-minute time-based trigger
4. **Bill Parser → Test Email Alert** — confirms email delivery to ALERT_EMAIL

### 4. Test
Drop one PDF into the Drive folder, then run **Bill Parser → Run Now**. Check the Sheet and confirm `data/bills.json` updated in this repo.

---

## Google Sheet Tabs

| Tab | Purpose |
|-----|---------|
| Electricity | Auto-written rows for electricity bills (kWh, demand, cost) |
| Natural Gas | Auto-written rows for Enbridge gas bills (m³, cost) |
| Water | Auto-written rows for water bills (m³, cost) |
| Review Queue | Bills flagged for human review (conflicts or sanity failures) |
| Audit Log | Full run history — every file, every outcome |

---

## Review Queue Workflow

When Claude and Grok disagree (or sanity checks fail), the bill lands in Review Queue with status `PENDING` and an alert email fires. To resolve:

1. Open the Review Queue tab
2. Open the original bill via the Drive link
3. Manually enter correct values into the appropriate utility tab
4. Change the Review Queue row status to `RESOLVED`

---

## Dashboard

Live dashboard at [keyona-rerev.github.io/TESTER_Resource-Bill-Parser-Github](https://keyona-rerev.github.io/TESTER_Resource-Bill-Parser-Github)

Reads from `data/bills.json`, which is updated by GAS after every successful run.

---

## GAS Script ID
`1EAAaPHD24fNYbIq5Vhga8eQUWt4C39P9u6dp7qfes0zs1g4G0Q69CQXJ`
