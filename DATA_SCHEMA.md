# Waypoint Data Schema — Lead Pipeline

> **Purpose:** Single source of truth for how data flows through the Waypoint lead machine. Reference this before modifying any webhook, Inngest function, or database schema.

---

## Lead Lifecycle States

```
RAW → ENRICHED → SEQUENCED → SENT → REPLIED → (SUPPRESSED)
```

| Status | Meaning | Set By |
|---|---|---|
| `RAW` | Just ingested. Not yet scored or emailed. | Apify webhook on create |
| `ENRICHED` | Email found, score ≥ 70. Ready for personalization. | `lead.hunter.start` Inngest fn |
| `SEQUENCED` | GPT-4o draft written. Ready for Resend. | `lead.personalize.start` Inngest fn |
| `SENT` | Email delivered via Resend. | `lead.send.start` Inngest fn |
| `REPLIED` | Reply received from lead OR TidyCal booking made. | Resend webhook / TidyCal webhook |
| `SUPPRESSED` | Score < 70, unsubscribe, or "Not a fit" reply. Excluded from all sends. | `lead.hunter.start` / Reply Guardian |

---

## Data Model (Prisma)

```prisma
model Lead {
  id                      String     @id @default(uuid())
  name                    String
  title                   String?           // Job title (e.g. "VP of Operations")
  company                 String?           // Employer
  linkedinUrl             String?    @unique
  country                 String?           // Location / market
  email                   String?           // Found during enrichment
  score                   Int        @default(0)   // 0–100 lead quality score
  status                  LeadStatus @default(RAW)
  careerTrigger           String?           // e.g. "Layoff", "Recent promotion", "Job change"
  recentPostSummary       String?           // Summary of prospect's most recent LinkedIn post
  pulledQuoteFromPost     String?           // Verbatim quote from post (for personalization)
  specificProjectOrMetric String?           // A metric or project mentioned publicly
  placeOrPersonalDetail   String?           // Location detail, sport, hobby (personalization)
  franchiseAngle          String?           // How franchise ownership maps to their situation
  draftEmail              String?           // GPT-4o generated outbound email
  replies                 Reply[]
  createdAt               DateTime   @default(now())
  updatedAt               DateTime   @updatedAt
}

model Reply {
  id             String   @id @default(uuid())
  leadId         String
  lead           Lead     @relation(fields: [leadId], references: [id])
  content        String            // Full reply text
  classification String?           // "Interested" | "Curious" | "Not now" | "Not a fit" | "Unsubscribe" | "Out of office" | "Ambiguous"
  createdAt      DateTime @default(now())
}
```

---

## Data Flow Map

### Channel 1: Outbound (LinkedIn → Cold Email)

```
LinkedIn Scrape (Apify)
  ↓
POST /api/webhooks/apify
  Auth: Bearer APIFY_WEBHOOK_SECRET
  Validates: name + linkedinUrl required
  ↓
Prisma Lead.upsert (dedup by linkedinUrl)
  status: RAW
  ↓
Inngest: workflow/lead.hunter.start
  → Enrich (find email via ZeroBounce/Hunter)
  → Score (see LEAD_SCORING.md)
  → If score < 70: status = SUPPRESSED
  → If score ≥ 70: status = ENRICHED
  ↓
Inngest: workflow/lead.personalize.start
  → GPT-4o generates personalized 50–90 word email
  → draftEmail saved to DB
  → status = SEQUENCED
  ↓
Inngest: workflow/lead.send.start (triggered by admin or scheduler)
  → Random 3–8 min send delay (warm-up protection)
  → Resend API sends from: kelsey@waypointfranchise.com
  → status = SENT
  ↓
Reply received (Resend webhook)
POST /api/webhooks/resend
  → Reply saved to DB
  ↓
Inngest: workflow/lead.reply.received
  → GPT-4o classifies reply
  → If "Interested" / "Curious": console alert (→ Slack in production)
  → If "Unsubscribe" / "Not a fit": status = SUPPRESSED
  → Otherwise: status = REPLIED
```

### Channel 2: Inbound (Website → Scorecard → CRM)

```
Visitor completes Readiness Scorecard
  ↓
POST /api/scorecard-complete (or /api/webhooks/inbound)
  Validates: name, email, score (0–100), primaryDriver?, biggestFear?
  ↓
Prisma Lead.upsert (dedup by email)
  If new: status = ENRICHED (pre-qualified inbound)
  If existing: merge score + careerTrigger fields
  ↓
[Future] Trigger nurture email sequence via Inngest
```

### Channel 3: Booking (TidyCal → CRM)

```
Lead books a call via TidyCal
  ↓
POST /api/webhooks/tidycal
  Auth: Bearer TIDYCAL_WEBHOOK_SECRET
  Matches booking email to existing Lead record
  ↓
Lead.status = REPLIED (highest intent signal)
```

---

## Webhook Secrets (Env Vars Required)

| Var | Webhook |
|---|---|
| `APIFY_WEBHOOK_SECRET` | `/api/webhooks/apify` |
| `RESEND_WEBHOOK_SECRET` | `/api/webhooks/resend` |
| `TIDYCAL_WEBHOOK_SECRET` | `/api/webhooks/tidycal` |
| `INBOUND_WEBHOOK_SECRET` | `/api/webhooks/inbound` |
| `OPENAI_API_KEY` | Inngest personalizer + reply guardian |
| `RESEND_API_KEY` | Inngest sender |
| `POSTGRES_PRISMA_URL` | Prisma connection (pooled) |
| `POSTGRES_URL_NON_POOLING` | Prisma connection (migrations) |

---

## Daily Send Monitor (Cron)

Inngest `monitor-process` runs daily at 9 AM. Checks domain reputation:
- Bounce rate > 2.0% → alert + pause sends
- Complaint rate ≥ 0.3% → alert + pause sends

Target during warm-up: **10–20 sends/day**. Scale to 50/day after 4 weeks of clean metrics.
