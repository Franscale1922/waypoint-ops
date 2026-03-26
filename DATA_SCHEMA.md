# Waypoint Data Schema — Lead Pipeline

> **Purpose:** Single source of truth for how data flows through the Waypoint lead machine. Reference this before modifying any webhook, Inngest function, or database schema.
>
> **Last updated:** March 25, 2026 — Corrected to reflect actual live stack. Apify, Hunter.io, and Apollo are not used. See COLD_EMAIL_STACK.md for full tool details.

---

## Lead Lifecycle States

```
PENDING_CLAY → RAW → ENRICHED → SEQUENCED → SENT → REPLIED → BOOKED
                                                          ↓
                                                    SUPPRESSED (at any point)
```

| Status | Meaning | Set By |
|---|---|---|
| `PENDING_CLAY` | Imported to admin dashboard. Waiting for Clay enrichment to arrive via Apps Script webhook. | `POST /api/leads` (admin import) |
| `RAW` | Clay enrichment received. Ready to score. | Clay webhook → `/api/webhooks/clay` |
| `ENRICHED` | Score ≥ 70. Ready for personalization. | `leadHunterProcess` Inngest fn |
| `SEQUENCED` | GPT-4o email drafted and saved. Ready for warmup scheduler to send. | `personalizerProcess` Inngest fn |
| `SENT` | Lead added to Instantly campaign. Email delivered. | `senderProcess` Inngest fn |
| `REPLIED` | Prospect replied to the cold email. | `replyGuardianProcess` (Instantly inbound webhook) |
| `BOOKED` | Prospect booked a TidyCal consultation. | `tidycalBookingSync` cron |
| `SUPPRESSED` | Score < 70, unsubscribe, or "Not a fit" reply. Excluded from all sends. | `leadHunterProcess` / `replyGuardianProcess` |

---

## Actual Pipeline Flow (Live as of March 2026)

```
1. Sales Navigator — Kelsey filters by seniority, company size, function, behavioral spotlights
    ↓
2. Evaboot Chrome extension — "Extract with Evaboot" → CSV with server-verified emails
   Export "safe emails only" during warmup
    ↓
3. Admin dashboard — ImportLeadForm → POST /api/leads → DB (PENDING_CLAY status)
    ↓
4. Clay (Launch plan, $185/mo) — Evaboot CSV manually imported into Clay table
   Enriches: recentPostSummary, companyNewsEvent, yearsInCurrentRole, company attributes
    ↓
5. Google Apps Script (Code.gs on Google Sheet) — fires when Clay exports row to Sheet
   POSTs each row to POST /api/webhooks/clay (auth: x-clay-secret header)
    ↓
6. /api/webhooks/clay — updates Lead record → status: RAW
   Fires Inngest event: workflow/lead.hunter.start
    ↓
7. Inngest: leadHunterProcess — scores (0–100) using enriched signals
   No external email API — Evaboot-verified email already present
   Score < 70 → status = SUPPRESSED
    ↓ (gate: score ≥ 70)
8. Inngest: personalizerProcess — GPT-4o writes email → status: SEQUENCED
    ↓
9. Inngest: warmupScheduler (8 AM MT, Mon–Fri) — fires top 25 SEQUENCED leads by score
   senderProcess → Instantly v2 API → lead added to campaign → status: SENT
    ↓
10. Instantly sends from sending domain (getwaypointfranchise.com / meetwaypointfranchise.com)
    ↓
11. Inngest: replyGuardianProcess — classifies reply, sends Resend + Slack HITL alert
    ↓
12. HITL: Kelsey replies and shares TidyCal link
    ↓
13. tidycalBookingSync cron (10 AM MT, Mon–Fri) → matches booking email → status: BOOKED
```

---

## Tools NOT in the Active Stack

| Tool | Status | Notes |
|---|---|---|
| Apify | ❌ Not used, not subscribed | Old design artifact. `/api/webhooks/apify` route exists as dead code. |
| Hunter.io | ❌ Not used, not subscribed | Key exists in Vercel as legacy artifact. No active code path calls Hunter. |
| Apollo.io | ❌ Not used, not subscribed | Evaluated March 2026, not purchased. Evaboot sufficient at Stage 1 volume. |
| Resend (as sender) | ❌ Not used for outbound | Resend is used only for HITL alert emails to Kelsey. Instantly is the outbound sender. |

---

## Data Model (Prisma — current)

```prisma
model Lead {
  id                      String     @id @default(uuid())
  name                    String
  title                   String?           // Job title (e.g. "VP of Operations")
  company                 String?           // Employer
  linkedinUrl             String?    @unique
  country                 String?           // Location / market
  email                   String?           // Evaboot-verified email (primary source)
  score                   Int        @default(0)   // 0–100 lead quality score
  status                  LeadStatus @default(PENDING_CLAY)
  careerTrigger           String?           // e.g. "Layoff", "Recent promotion", "Job change"
  recentPostSummary       String?           // Clay: summary of prospect's most recent LinkedIn post
  companyNewsEvent        String?           // Clay: company-level news signal (WARN Act, reorg, etc.)
  yearsInCurrentRole      Int?              // Clay: years in current title
  companySizeRange        String?           // Clay: headcount bucket (e.g. "201-500")
  industryVertical        String?           // Clay: industry
  functionArea            String?           // Clay: department/function
  seniorityLevel          String?           // Clay: seniority tier
  isOpenToWork            Boolean?          // Clay/Evaboot: OpenToWork badge
  wasRecentlyPromoted     Boolean?          // Clay formula: promoted < 6 months
  yearsAtCompany          Int?              // Clay: total tenure at company
  geoMarket               String?           // Clay: US region bucket
  draftEmail              String?           // GPT-4o generated outbound email
  replies                 Reply[]
  createdAt               DateTime   @default(now())
  updatedAt               DateTime   @updatedAt
}
```

---

## Webhook Secrets (Env Vars Required)

| Var | Webhook | Status |
|---|---|---|
| `CLAY_WEBHOOK_SECRET` | `/api/webhooks/clay` + Apps Script Script Property | ✅ Required |
| `INBOUND_WEBHOOK_SECRET` | `/api/webhooks/resend` (Instantly inbound reply webhook) | ✅ Required |
| `TIDYCAL_WEBHOOK_SECRET` | `/api/webhooks/tidycal` (future — TidyCal doesn't support webhooks yet) | ⏳ Future |
| `INSTANTLY_API_KEY` | `senderProcess` Inngest fn | ✅ Required |
| `INSTANTLY_CAMPAIGN_ID` | `senderProcess` Inngest fn | ✅ Required |
| `OPENAI_API_KEY` | `personalizerProcess` + `replyGuardianProcess` | ✅ Required |
| `RESEND_API_KEY` | HITL alert emails to Kelsey | ✅ Required |
| `POSTGRES_PRISMA_URL` | Prisma connection (pooled) | ✅ Required |
| `POSTGRES_URL_NON_POOLING` | Prisma connection (migrations) | ✅ Required |
| `APIFY_WEBHOOK_SECRET` | `/api/webhooks/apify` — **dead code, Apify not used** | ⚠️ Legacy |

---

## Daily Send Monitor

Inngest `warmupScheduler` fires Mon–Fri at 8 AM MT. Reads `SystemSettings.maxSendsPerDay` from DB (currently **25**). Sends top-scored SEQUENCED leads with 90-second stagger between dispatches.

Domain health monitored via Instantly inbox health dashboard — all 6 sending inboxes at 100% health.
