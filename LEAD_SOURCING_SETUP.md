# Lead Sourcing Setup Guide

> Reference this when setting up or modifying lead sourcing. The full pipeline is:
> **Sales Navigator → Apify → /api/webhooks/apify → Score → GPT-4o → Instantly**

---

## Step 1: Build the Sales Navigator Saved Search

Log into [linkedin.com/sales](https://linkedin.com/sales) and create a **Leads** search with these filters:

### Required Filters
| Filter | Value |
|---|---|
| **Geography** | United States |
| **Seniority Level** | Director, VP, C-Level, Owner, Partner |
| **Company Headcount** | 201–500, 501–1000, 1001–5000, 5001–10000 |
| **Function** | Operations, Finance, Sales, General Management, Business Development |

### Spotlight Filter (highest-value signal)
Under **Spotlights**, enable:
- ✅ **Changed jobs in past 90 days** → maps to `careerTrigger +20` in scoring rubric

### Save the Search
After applying filters, click **Save Search** → name it `Waypoint ICP — Job Changers`.

Copy the URL from the address bar — it will look like:
```
https://www.linkedin.com/sales/search/people?query=...&savedSearchId=...
```

This URL is your Apify actor input.

---

## Step 2: Configure the Apify Actor

**Actor:** `curious_coder/linkedin-sales-navigator-search-scraper`
**Cost:** ~$39/mo rental + ~$2–4 per 1,000 leads in compute

### Input Configuration
```json
{
  "searchUrl": "<paste your Sales Navigator search URL here>",
  "cookie": "<paste LinkedIn session cookies as JSON array>",
  "userAgent": "<paste your browser user agent>",
  "limit": 100,
  "deepSearch": false
}
```

### Getting LinkedIn Cookies
1. Install [Cookie-Editor](https://cookie-editor.com/) Chrome extension
2. Go to linkedin.com while logged in as YOUR account (the Sales Nav account)
3. Click Cookie-Editor → Export → Copy ALL cookies as JSON
4. Paste into the `cookie` field above

### Getting Your User Agent
Open Chrome DevTools (F12) → Console → paste:
```javascript
navigator.userAgent
```
Copy the output string → paste into `userAgent` field.

### Webhook Output
In Apify actor settings → **Integrations** → **Webhook**:
- **URL:** `https://waypointfranchise.com/api/webhooks/apify`
- **Method:** POST
- **Authorization Header:** `Authorization: Bearer <APIFY_WEBHOOK_SECRET from .env>`
- **Events:** Run Succeeded

---

## Step 3: Schedule the Actor Run

In Apify → the actor task → **Schedule** tab:
- **Frequency:** Monday–Friday
- **Time:** 6:00 AM MT (12:00 UTC)
- **Runs before** the 8 AM MT warm-up scheduler fires

This ensures fresh leads are scored and personalized before the daily send batch.

---

## Step 4: Email Discovery (Hunter.io)

The `enrich-and-score` Inngest step calls Hunter.io to find validated emails.

1. Sign up at [hunter.io](https://hunter.io) → grab API key
2. Add to Vercel env vars: `HUNTER_API_KEY`
3. Recommended plan: **Starter ($49/mo)** = 500 email finds/month

> **If Hunter finds nothing or confidence < 60%:** the lead's score is capped at 60 and fails the 70-point gate → goes to SUPPRESSED. Only leads with verified emails enter the sequence.

---

## Apify Output → Webhook Field Mapping

The Apify actor outputs these fields which map to the Lead schema:

| Apify Field | Lead Field | Notes |
|---|---|---|
| `fullName` | `name` | Required |
| `linkedInProfileUrl` | `linkedinUrl` | Required (dedup key) |
| `jobTitle` | `title` | Drives seniority score |
| `companyName` | `company` | Used for Hunter.io domain |
| `location` | `country` | For suppression rules |
| `summary` | `recentPostSummary` | Post content score |

> If the actor outputs different field names, update the mapping in `/api/webhooks/apify/route.ts`.
