# Lead Sourcing Setup Guide

> **Last updated:** March 25, 2026 — Corrected to reflect actual live process.
> **NOT in use:** Apify, Hunter.io, Apollo.io. Do not reference these tools for setup.
> **Source of truth:** `COLD_EMAIL_STACK.md` in `waypoint-core-system/docs/`

---

## Full Pipeline (Manual Steps + Automated Steps)

```
Sales Navigator (Kelsey — manual)
    ↓
Evaboot Chrome extension — export CSV with server-verified emails (Kelsey — manual)
    ↓
Admin dashboard — ImportLeadForm → POST /api/leads → PENDING_CLAY (Kelsey — manual)
    ↓
Clay table — import Evaboot CSV, enrichment runs automatically (Kelsey — manual import)
    ↓
Google Apps Script — fires automatically on Clay export to Google Sheet
    ↓
/api/webhooks/clay → lead scored → personalized → Instantly campaign
```

---

## Step 1: Build the Sales Navigator Search

Log into [linkedin.com/sales](https://linkedin.com/sales) and create a **Leads** search:

### Required Filters
| Filter | Value |
|---|---|
| **Geography** | United States |
| **Seniority Level** | Director, VP, C-Level, Owner, Partner |
| **Company Headcount** | 201–500, 501–1000, 1001–5000, 5001–10000 |
| **Function** | Operations, Finance, Sales, General Management, Business Development |

### Spotlight Filters (priority signals)
- ✅ **Posted on LinkedIn in past 30 days** → confirms active engagement (most important signal)
- ✅ **Open to Work** → maps to scoring bonus

### Do NOT use
- ❌ "Changed jobs in past 90 days" — catches people settling into new roles, opposite of our ICP

### WARN Act workflow (weekly, 15 min)
1. Check WARNTracker.com, Layoffs.fyi, TheLayoff.com for new company names
2. Search each company by name in Sales Navigator
3. Filter: VP/Director/CXO + "Posted on LinkedIn in past 30 days"
4. Export via Evaboot

---

## Step 2: Export with Evaboot

**Tool:** Evaboot Chrome extension (installed, active subscription)

1. Run your Sales Navigator search with filters set (Step 1)
2. Click **"Extract with Evaboot"** in the browser toolbar
3. Evaboot exports, cleans, deduplicates, and server-verifies emails
4. In Evaboot dashboard → download CSV
5. **Export "safe emails only"** — do NOT include "riskier" (catch-all) emails during warmup

**Email tiers:**
- `safe` — server confirmed the address exists (97% deliverability) ✅ Use only these during warmup
- `riskier` — catch-all domain, cannot verify without sending (~83%) ⚠️ Post-warmup only, max 30% of list
- blank — no email found → pass row to Clay for Findymail fallback enrichment

**Typical match rate:** ~60–70% at safe tier.

---

## Step 3: Load Leads into Admin Dashboard

1. Go to `https://waypointfranchise.com/admin/leads`
2. Use the **Import Lead Form** (or navigate to the import section)
3. Upload the Evaboot CSV
4. Leads are created with status `PENDING_CLAY` — they wait here until Clay enrichment arrives

**What happens:** `POST /api/leads` upserts each lead (dedup by LinkedIn URL). Status = `PENDING_CLAY`.

> **No Inngest pipeline fires yet.** Leads stay in PENDING_CLAY until the Clay webhook fires.

---

## Step 4: Import CSV into Clay and Run Enrichment

1. Open [clay.com](https://clay.com) → your **"Cold Email Test 1"** table
2. Import the same Evaboot CSV into the Clay table
3. Clay automatically enriches each row:
   - `recentPostSummary` — LinkedIn post content (via Clay LinkedIn integration)
   - `companyNewsEvent` — company news signal (PredictLeads / Google News)
   - `yearsInCurrentRole` — tenure in current title (Clay native)
   - Company attributes: headcount range, industry, function, seniority, OpenToWork
4. For leads where Evaboot didn't find an email: Clay's **Findymail** waterfall runs as fallback
   - ZeroBounce verification runs after Findymail to validate before export

**Clay table column requirements (must match Apps Script column map):**
| Col | Header | Clay source |
|-----|--------|-------------|
| A | First Name | Clay `First Name` |
| B | Last Name | Clay `Last Name` |
| C | LinkedIn URL | Clay `LinkedIn URL (Slash)` |
| D | Email | Clay `work email` (Evaboot primary OR Findymail fallback, ZeroBounce verified) |
| E | Title | Clay `Job Title` |
| F | Company | Clay `Company Name` |
| G | Country | Clay `Country` |
| H | Recent Post Summary | Clay `Recent Post Summary` |
| I | Company News Event | Clay `Company News` |
| J | Years in Position | Clay native `Years in Position` |
| K | Company Size Range | Clay `Company Headcount` (bucketed via formula) |
| L | Industry Vertical | Clay `Industry` |
| M | Function Area | Clay `Department` |
| N | Seniority Level | Clay `Seniority` |
| O | Is Open To Work | Clay `Open To Work` (boolean) |
| P | Was Recently Promoted | Clay formula: promoted < 6 months |

---

## Step 5: Google Apps Script Fires Automatically

When Clay exports enriched rows to the Google Sheet, the **Apps Script** (`Code.gs`) fires automatically and POSTs each row to the pipeline webhook.

**No action needed from Kelsey** — this is fully automated once Clay finishes enrichment.

**What Apps Script does:**
- Reads columns A–P from the Google Sheet
- Constructs a JSON payload per row
- POSTs to `POST https://waypointfranchise.com/api/webhooks/clay`
- Auth: `x-clay-secret` header (value stored as Script Property in Apps Script settings)

**Google Sheet:** [Cold Email Test 1](https://docs.google.com/spreadsheets/d/1YF73at-vjjXPvfYuUEmiP7NeGI7Ku4G3gskJcUSr-ko/edit)

> If you need to manually re-fire for a batch: open the Sheet → Extensions → Apps Script → Run `postAllRows()`

---

## Step 6: Automated Pipeline (No Manual Action)

Once the Clay webhook fires, the rest is automatic:

1. **`/api/webhooks/clay`** — updates Lead record with enrichment signals → status: `RAW` → fires `workflow/lead.hunter.start`
2. **`leadHunterProcess`** — scores the lead (0–100). Below 70 → `SUPPRESSED`. Above 70 → `ENRICHED` → fires `workflow/lead.personalize.start`
3. **`personalizerProcess`** — GPT-4o writes the outbound email → status: `SEQUENCED`
4. **`warmupScheduler`** (8 AM MT, Mon–Fri) — picks top 25 SEQUENCED leads by score, fires `senderProcess` for each with 90-second stagger
5. **`senderProcess`** — calls Instantly v2 API → lead added to campaign → email sent → status: `SENT`

---

## Env Vars Required

| Var | Where set | Purpose |
|---|---|---|
| `CLAY_WEBHOOK_SECRET` | Vercel + Apps Script Script Properties | Authenticates Clay → webhook |
| `INSTANTLY_API_KEY` | Vercel | senderProcess calls Instantly v2 API |
| `INSTANTLY_CAMPAIGN_ID` | Vercel | Target campaign for lead injection |
| `OPENAI_API_KEY` | Vercel | personalizerProcess + replyGuardianProcess |
| `INBOUND_WEBHOOK_SECRET` | Vercel | Instantly inbound reply webhook auth |

---

## Tools NOT in the Active Stack

| Tool | Status |
|---|---|
| **Apify** | ❌ Not used, not subscribed. Dead code at `/api/webhooks/apify`. Do not start. |
| **Hunter.io** | ❌ Not subscribed. Key exists in Vercel as legacy artifact. No active code path. |
| **Apollo.io** | ❌ Not subscribed. Evaluated March 2026, not purchased. |

---

## pendingClayFallback Safety Net

If Clay enrichment doesn't arrive within 24 hours (e.g. Clay table misconfigured, Apps Script error), the **`pendingClayFallback`** Inngest cron (7 AM MT, Mon–Fri) automatically advances stuck `PENDING_CLAY` leads to `RAW` and fires scoring. This ensures no lead is permanently lost due to a Clay/Apps Script failure.
