# Master Strategy & To-Do List: Waypoint Franchise Autopilot

> **Purpose:** Lightweight project index — just enough context to know what each project is and where it fits.
>
> **Last Updated:** March 24, 2026 — Scorecard nurture system fully hardened. Session: Inngest migration, dedup guard, Slack high-score alert, booking/reply suppression on both nurture sequences, admin Scorecard page.
>
> **Status key:** `[ ]` Not started · `[/]` In progress · `[x]` Complete · `[~]` Deferred (Horizon 2) · `[!]` Needs attention

---

## 1. 🔍 System Audits & Foundation

- [x] **GitHub Code Repository**: All Next.js website code in `waypoint-core-system` on GitHub.
- [x] **GitHub Migration**: Cold email system, n8n workflows, and ops assets in `waypoint-operations` and `waypoint-ops` on GitHub.
- [x] **Data Schema Definition**: Fully documented in `waypoint-ops/DATA_SCHEMA.md`. All webhooks, Inngest flows, and Prisma model mapped.
- [x] **Custom CRM API Mapping**: All endpoints documented via `DATA_SCHEMA.md` + active Inngest/webhook code.
- [/] **Google Drive Brain Architecture**: Google Drive folder exists. Syncing across machines but not yet being used actively as a live context source. URL: https://drive.google.com/drive/folders/1RpB1rYawDEsDQ5OEaIILBszEzkn7w2g6
- [ ] **Workflow Evaluation**: Audit n8n workflows and Claude skills — determine what to migrate, deprecate, or duplicate into AntiGravity.
- [!] **Cold Email Audit**: Must be done **before** scaling Instantly sends. Audit existing sequences against `docs/VOICE_GUIDE.md` and B.L.A.S.T architecture. *See §4 note.*
- [ ] **Persona Expansion Architecture**: Define the remaining 50% of the client base (non-Corporate Refugees). Map motivational drivers.
- [/] **CRM API Integration**: Core pipeline (Apify → Inngest → Neon DB → Resend) live. Clay enrichment connected. TidyCal webhook live. Remaining: direct API write-back from external tools.

---

## 2. 🧲 High-Value Lead Magnets

- [x] **Franchise Readiness Scorecard** *(live at `/scorecard`)*: Interactive quiz with email capture, personalized results, and score-aware CTAs.
- [x] **High-Depth SEO/AEO Articles**: 34+ articles live at `/resources`. Tier 1/2/3 library complete. Article queue active.
- [x] **Franchise Checklists** *(live at `/checklists`)*: 6 downloadable checklists with email capture + 5-email nurture sequence via Inngest.
- [ ] **The Corporate Escape Kit (PDF/Web)**: NotebookLM-generated guide — "financial safety nets of franchising vs. W2." Pre-decision stage lead magnet. *Not started.*

---

## 3. 🏗️ The Waypoint Website

- [x] **Full site build**: Next.js 15, Tailwind, Playfair Display. Live at `waypointfranchise.com` via Vercel + Cloudflare.
- [x] **TidyCal Integration**: Live at `/book`. 30-min intro call.
- [x] **Sprint 3 — Technical SEO**: Next.js Image, FAQ JSON-LD, OG images, sitemap, robots.txt, internal linking — all confirmed live.
- [x] **Testimonials**: 8 live testimonials. Mobile carousel + dot indicators. Pull-quote on About page.
- [x] **Email Nurture (Post-Checklist)**: 5-email drip on checklist download. Unsubscribe handling. Branded HTML emails. Upgraded March 2026: booking + reply suppression added (`shouldSuppress` checks `Lead.bookedAt` + `Lead.status === REPLIED` before each send, in addition to unsubscribe flag).
- [x] **CRO Sprints A–I**: All confirmed live (CTA hierarchy, Start Here, keyword search, investment cards, mobile nav SMS, About depth, Process micro-outcomes).
- [x] **Kelsey Welcome Video**: Vimeo embed on About page with lazy-load facade.
- [x] **LinkedIn DM Queue**: Admin at `/admin/linkedin`. Inngest function Mon–Fri 9 AM. Mark sent/skip with DB persistence.
- [x] **Admin Dashboard**: Leads Manager, LinkedIn Queue, Inbox & Replies, Settings — all live at `/admin`.
- [x] **Email Nurture (Post-Scorecard)**: 3-email sequence (Day 0 / Day 3 / Day 7) via Inngest + Resend. `ScorecardSubmission` model tracks state. HMAC-signed `List-Unsubscribe` header + `/api/scorecard-unsubscribe`. Mid-sequence suppression: stops on unsubscribe link, TidyCal booking (`Lead.bookedAt`), or reply (`Lead.status === REPLIED`). Dedup guard prevents double-sequence on re-submission. Fully tested and confirmed live March 2026.
- [x] **High-Score Slack Alert**: Real-time Slack notification fires for any scorecard submission ≥ 70 — name, email, score, driver, fear, admin link. Routed to #hot-replies.
- [x] **Admin Scorecard Page**: `/admin/scorecard` — table of all submissions with score, nurture step, driver, fear, unsubscribe flag, and 4-stat overview strip. Added to admin sidebar.

---

## 4. ⚙️ The AntiGravity Lead Machine (Cold Email Outreach)

> **⚠️ Instantly Status as of March 23, 2026:**
> - 6 sending accounts, all at **100% warmup health**, 80 warmup emails/day each.
> - Campaign: **"Waypoint Outbound — Corporate-to-Franchise"** — Active.
> - 32 leads loaded, 20 emails sent, **6 bounces (30% bounce rate).**
> - 0 replies received. Open/click tracking disabled (correct for deliverability).
> - **The 30% bounce rate is above the 2% safety threshold. Cold email audit + list hygiene must happen before scaling.**

- [x] **Lead Qualification Scoring Engine**: Live in `lead.hunter.start` Inngest. Score 0–100 rubric (title, career trigger, post content, persona fit). Cutoff: 70. Documented in `LEAD_SCORING.md`.
- [x] **Data Flow (Apify → CRM → Resend)**: Full pipeline live. Apify → Prisma → Inngest enrich/score/personalize/send.
- [x] **Reply Guardian**: Live. GPT-4o classifies replies. Auto-suppresses unsubscribes. Slack alerts on interest.
- [/] **Warm-Up Sequences**: 6 accounts warming. All 100% health. Single-step outbound campaign active but bounce rate needs investigation before scaling.
- [!] **Cold Email Audit**: Rewrite the `personalizerProcess` prompt + Instantly sequences against `docs/VOICE_GUIDE.md`. Fix bounce issue. *Priority before scaling.*
- [ ] **Navigation Engine (Architect Phase)**: Outreach path selection based on career trigger signals (title change, layoff, etc.). Not built.
- [ ] **Email Triage Agent**: Separate from Reply Guardian — an Instantly/Gmail layer for autonomous inbox routing. Not built.
- [ ] **Referral Generation Engine**: Sequence for prior clients and non-converted leads to extract referrals.

---

## 4.5. 🦾 Gravity Claw — Autonomous Web Agent Layer

*Build trigger: After §4 sequences are running cleanly and generating replies.*

- [ ] **Core Agent Build**: Adapt OpenClaw into Gravity Claw, scoped to Waypoint use cases.
- [ ] **Lead Sourcing Module**: Autonomous LinkedIn + Reddit prospecting → scored leads → CRM.
- [ ] **Content Intelligence Module**: Daily franchise Reddit scrape → feeds n8n Video Content Generator.
- [ ] **Inbox Monitoring Module**: Reply signals → triggers Email Triage Agent (§4).
- [ ] **Franchisor Portal Watcher**: Monitors partner portals for recycled leads → routes to §8.

---

## 5. 🎥 Content Ecosystem: Omni-Channel Creation Agent

- [x] **n8n Foundation**: Reddit Intelligence Pipeline, Video Content Generator, LinkedIn/Twitter Automation operational in `waypoint-operations`.
- [ ] **ClaudeMax Teammate Integration**: Claude Projects for persistent context across sessions.
- [ ] **Repurposing Engine API Map**: NotebookLM → Higgsfield → Descript / Opus Clips → VidIQ.
- [ ] **Cross-Platform Scheduling**: Auto-resize + schedule for LinkedIn, YouTube, Facebook, Instagram, Twitter, TikTok.
- [ ] **Beehiiv + LinkedIn Newsletters**: Deep, SEO-optimized content distribution.
- [ ] **"Raw Truth" Podcast**: Transparent entrepreneurship. Honest mistakes, real numbers.

---

## 6. 🎓 Horizon 2: Autonomous Education (Course Engine)

*Deferred until §4 is generating active pipeline.*

- [~] **"AI for Beginners" Course**: NotebookLM + AntiGravity produces curriculum, flashcards. Auto-updating.
- [~] **"Intermediate AI" Course**: Higher-level tactical operations.

---

## 7. 🤝 Horizon 2: Community & Monetization (Skool)

*Deferred. Build audience via Newsletter + Podcast (§5) first.*

- [~] **Skool Mentorship Program**: Paid community for new franchisees.
- [~] **Unconverted Lead Funnel**: Routes Skool non-buyers back into Waypoint consultation pipeline.

---

## 8. ♻️ Franchisor Lead Recycling (B2B Channel)

*Partner franchisors send "wrong fit" leads to Waypoint for re-matching.*

- [ ] **Recycling Engine**: Intake logic for leads who want a franchise — just not the franchisor's concept.
- [ ] **Partner Portal / CRM Ingestion**: Automated lead ingestion from franchisor partners.
- [ ] **Follow-Up Sequence**: Email + AI avatar sequence to re-engage recycled leads into other portfolios.

---

## 9. 🤖 Horizon 2: Advanced Autonomous Agents

*Deferred until primary lead flow is stabilized.*

- [~] **Reddit Lead Gen Agent** *(Gravity Claw)*: Agent threads/replies in business/franchising subreddits.

---

## Active Phase (March 2026)

| What | Status |
|---|---|
| Website + all pages live | 🟢 Done |
| Article library (34+ articles) | 🟢 Done |
| Scorecard + checklists + lead magnets | 🟢 Done |
| Checklist 5-email nurture (+ booking/reply suppression) | 🟢 Done |
| CRM pipeline (Apify → Score → Send → Reply) | 🟢 Done |
| CRO Sprints A–I + Technical SEO | 🟢 Done |
| LinkedIn DM Queue admin panel | 🟢 Done |
| Post-scorecard 3-email nurture (fully hardened) | 🟢 Done |
| High-score Slack alert (#hot-replies) | 🟢 Done |
| Admin Scorecard page | 🟢 Done |
| Instantly warm-up (6 accounts, 100% health) | 🟡 Running — bounce rate issue to fix |
| **Cold email audit (voice + list hygiene)** | 🔴 Urgent before scaling |
| Corporate Escape Kit lead magnet | 🔵 Next to build |
| n8n workflow evaluation | 🔵 Up next |
| Content repurposing engine (YouTube/Podcast) | 🟡 Queued |
| Gravity Claw | 🟡 Queued — after email scales |
| Franchisor Lead Recycling (B2B) | 🟡 Queued |
