# 🔄 Enterprise Data Pipeline — n8n Automation v2.0

> **A production-grade automated data workflow** built with n8n — featuring schema validation, quality gates, parallel processing, error handling, multi-channel alerting, Google Sheets audit logging, and styled stakeholder email reports — triggered every Monday with zero manual intervention.

---

## 🏗️ Full Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ENTERPRISE DATA PIPELINE v2.0                    │
└─────────────────────────────────────────────────────────────────────┘

⏰ Cron Trigger (Mon 9AM)
        │
        ▼
🔧 Initialise Context      → Creates Run ID, timestamps, config
        │
        ▼
🌐 Fetch API Data          → HTTP GET with auto-retry (3×, 2s delay)
        │
        ▼
✅ Schema Validation        → Field checks, type checks, null checks
        │
        ▼
🔍 Quality Gate (≥80%?)    ──── FAIL ────► 🔥 Error Handler
        │ PASS                                      │
        ▼                                           ▼
🧹 Clean & Transform       ← normalise, enrich  🚨 Slack Error Alert
        │                     feature engineering
        ▼
📤 Split Batches (n=10)    → Parallel processing
        │
        ▼
🔗 Merge & Aggregate       → Combines batches, computes final KPIs
        │
   ┌────┴────────────────┐
   ▼                     ▼
📊 Google Sheets     📄 Build HTML Report
(Audit Trail)             │
                     ┌────┴────────────────┐
                     ▼                     ▼
               📧 Email Report      💬 Slack Success
               (Stakeholders)       (#data-pipeline-alerts)
```

---

## 🔑 Key Features

| Feature | Description |
|---------|-------------|
| **Schema Validation** | Checks all required fields, types, and nulls before processing |
| **Quality Gate** | Halts pipeline if data quality drops below 80% threshold |
| **Auto-Retry** | HTTP requests retry 3× with 2-second delay on failure |
| **Parallel Batching** | Splits records into batches of 10 for parallel processing |
| **Dynamic Position Sizing** | Each record enriched with user tier and content features |
| **Error Routing** | All failures route to centralised error handler + Slack alert |
| **Audit Trail** | Every run appends a row to Google Sheets for full history |
| **Styled HTML Email** | KPI cards, data table, and tier breakdown in a polished report |
| **Slack Notifications** | Success AND failure messages with run ID and key metrics |
| **Run Context** | Unique Run ID + timestamps for every execution (traceable) |

---

## 📋 Node-by-Node Breakdown

### 1. ⏰ Cron Trigger
Fires every Monday at 09:00 AM via cron expression `0 9 * * 1`. Can also be triggered manually from the n8n dashboard.

### 2. 🔧 Initialise Run Context
Creates a unique `Run ID` (e.g. `RUN_20240115090000`), stores pipeline config (API URL, retry settings, thresholds), and sets week range for the report.

### 3. 🌐 Fetch Data from API
HTTP GET request with:
- Configurable timeout (15 seconds)
- Automatic retry: 3 attempts with 2-second backoff
- Returns raw JSON array

### 4. ✅ Schema Validation
For every record checks:
- `id` — exists and is a number
- `title` — exists and not empty
- `body` — exists and not empty
- `userId` — exists
- Counts valid vs invalid; throws error if <5 valid records returned

### 5. 🔍 Quality Gate
If valid records ÷ total records < 80% → routes to error handler. This prevents bad data from reaching downstream systems.

### 6. 🧹 Clean & Transform
- Strips and normalises all text (whitespace, special chars)
- Computes `word_count`, `title_length`, `content_length` category
- Assigns `user_tier` (premium / standard / basic) based on user ID
- Adds `processed_at` timestamp

### 7. 📤 Batch Splitting
Splits records into batches of 10 — enables parallel downstream processing and prevents timeout on large datasets.

### 8. 🔗 Merge & Aggregate
Combines all batches, computes final KPIs:
- total records, unique users, avg word count
- breakdown by user tier and content length category

### 9. 📊 Google Sheets
Appends one audit row per run with Run ID, timestamp, record counts, and pass rate — creates a historical run log.

### 10. 📄 HTML Report Builder
Generates a styled HTML email with:
- KPI summary cards (4 metrics)
- User tier breakdown list
- Top 15 records table with tier colour-coding

### 11. 📧 Gmail + 💬 Slack
Sends report email to stakeholders and posts success/failure notifications to Slack channel.

---

## 🚀 How to Set Up

### Step 1 — Install n8n
```bash
npm install -g n8n
n8n start
# Open http://localhost:5678
```

### Step 2 — Import Workflow
1. n8n Dashboard → **+ New Workflow** → **⋮ menu** → **Import from File**
2. Select `workflow.json`
3. Click **Save**

### Step 3 — Connect Credentials
| Node | Credential Needed |
|------|------------------|
| Google Sheets | Google OAuth2 — [guide](https://docs.n8n.io/integrations/builtin/credentials/google/) |
| Gmail | Gmail OAuth2 |
| Slack | Slack OAuth2 / Bot Token |

Then update:
- Google Sheets: Replace `YOUR_GOOGLE_SHEET_ID` with your actual Sheet ID
- Gmail: Update `fromEmail` and `toEmail`
- Slack: Update `#data-pipeline-alerts` channel name

### Step 4 — Test
Click **Execute Workflow** → inspect each node's output panel → verify data flows correctly

### Step 5 — Activate
Toggle **Inactive → Active** to enable the Monday schedule

---

## 📁 File Structure
```
02-n8n-automation/
├── workflow.json    ← Import directly into n8n
└── README.md
```

---

## 🛠️ Tech Stack
- **n8n** — Open-source workflow automation engine
- **JavaScript (ES2022)** — All Code nodes
- **Google Sheets API** — Audit trail storage
- **Gmail API** — Report delivery
- **Slack API** — Real-time alerting

---

## 💡 Production Extensions
- Replace JSON placeholder API with your real data source (PostgreSQL, Airtable, Salesforce, etc.)
- Add a **Webhook Trigger** node for real-time event-driven processing
- Connect to **Telegram Bot** instead of Slack for mobile alerts
- Add a **Merge** node to compare this week's data with last week's (trend detection)
- Deploy n8n on a VPS (DigitalOcean, Railway) for 24/7 uptime

---

*Built by Meghraj Nikalje | B.Sc. Data Science, Mumbai University*
