# Master Strategy & To-Do List: Waypoint Franchise Autopilot

> **Purpose:** Lightweight project index — just enough context to know what each project is and where it fits. Deep documentation, research, and implementation plans are written when each project becomes active.
>
> **Status key:** `[ ]` Not started · `[/]` In progress · `[x]` Complete · `[~]` Deferred (Horizon 2)

---

## 1. 🔍 System Audits & Foundation

Establish the cross-machine brain architecture, audit existing tools, and define the data schema before building anything new.

- [x] **GitHub Code Repository**: All Next.js website code isolated in `waypoint-core-system` on GitHub.
- [x] **GitHub Migration**: Cold email system, n8n workflows, and ops assets saved to `waypoint-operations` and `waypoint-ops` on GitHub.
- [ ] **Google Drive Brain Architecture**: Relocate the core AntiGravity brain (all markdown strategies, guidelines, context profiles) into Google Drive and symlink to both machines. Enables zero-click sync to Claude Teams Project Knowledge Base and non-developer pipelines (Descript, NotebookLM).
- [ ] **Workflow Evaluation**: Audit all n8n workflows and Claude skills — determine what to migrate, deprecate, or duplicate into the AntiGravity architecture.
- [ ] **Cold Email Audit**: Audit existing system against `voice-tone.md` and B.L.A.S.T architecture. Rewrite scripts to match deterministic standards.
- [ ] **Data Schema Definition**: Map and document the full data flow: LinkedIn (Apify) → Email (Resend) → Website → CRM → TidyCal.
- [ ] **Persona Expansion Architecture**: Define and document the remaining 50% of the client base (non-Corporate Refugees). Map motivational drivers. Required before finalizing the lead scoring engine.
- [ ] **Custom CRM API Mapping**: Document all custom CRM endpoints (Auth, GET, POST) before piping any external data into it.
- [ ] **CRM API Integration**: Connect primary AntiGravity workflows (Lead Recycling, etc.) directly to the CRM via API.

---

## 2. 🧲 High-Value Lead Magnets

Capture leads with WIIFM-focused assets before they book a call.

- [x] **Franchise Readiness Scorecard** *(live at `/scorecard`)*: Interactive quiz with email capture and personalized results.
- [ ] **The Corporate Escape Kit (PDF/Web)**: NotebookLM-generated "Study Guide" detailing the financial safety nets of franchising vs. W2 employment. Designed for the pre-decision stage.
- [x] **High-Depth SEO/AEO Articles**: Tier 1/2/3 article library drafted (Connor Groce standard). All 28 articles complete in `waypoint-ops/content/`.

---

## 3. 🏗️ The Waypoint Website (B.L.A.S.T Protocol)

The primary conversion asset. Funnel: Traffic → Lead Magnet → Content → TidyCal booking.

- [x] **Blueprint Phase**: User journey finalized.
- [x] **TidyCal Integration**: Live at `/book`. 15-min discovery call configured.
- [x] **Stylize Phase**: Full site built (Next.js 14, Tailwind, Playfair Display). Glassmorphism accents on key CTAs.
- [x] **Trigger Phase**: `waypointfranchise.com` deployed via Vercel + Cloudflare DNS.
- [ ] **Sprint 3 — Technical SEO**: Migrate `<img>` → Next.js `<Image>`, FAQ schema markup, page-specific OG images, sitemap, robots.txt, internal linking.
- [ ] **Testimonials Section**: 2–3 quote cards with attribution. Requires real quotes from Kelsey.
- [ ] **Email Nurture Sequence (Post-Scorecard)**: 3-email drip triggered by scorecard completion. Day 0/3/7.

---

## 4. ⚙️ The AntiGravity Lead Machine (Cold Email Outreach)

Automated outbound system to generate inbound pipeline from LinkedIn-sourced leads.

- [ ] **Lead Qualification Scoring Engine**: Programmatic lead scoring before granting TidyCal access. Protects Kelsey's calendar.
- [ ] **Navigation Engine (Architect Phase)**: Logic that selects outreach paths based on career trigger signals (title changes, layoffs, new ventures).
- [ ] **Warm-Up Sequences (Trigger Phase)**: Launch low-volume sends (10–20/day). Monitor deliverability via GlockApps.
- [ ] **Email Triage Agent**: Autonomous inbox management — categorizes replies (booking request, question, spam) and triggers appropriate responses.
- [ ] **Referral Generation Engine**: Automated sequence for previous clients and non-converted leads designed to extract personal network referrals.

---

## 4.5. 🦾 Gravity Claw — Autonomous Web Agent Layer

*Waypoint's fork of the OpenClaw architecture. A browser-native autonomous agent that acts on the open web on behalf of AntiGravity systems — no human in the loop.*

**Build trigger:** Activate after §4 cold email sequences are running and generating replies.

- [ ] **Core Agent Build**: Adapt OpenClaw into Gravity Claw, scoped to Waypoint use cases. Modular design — each module below is independent.
- [ ] **Lead Sourcing Module**: Autonomous LinkedIn + Reddit prospecting. Outputs scored leads directly to CRM.
- [ ] **Content Intelligence Module**: Daily scrape of franchise Reddit threads → feeds n8n Video Content Generator pipeline.
- [ ] **Inbox Monitoring Module**: Watches for reply signals → triggers Email Triage Agent (§4).
- [ ] **Franchisor Portal Watcher**: Monitors partner portals for new recycled leads → routes to §8 Recycling Engine.

---

## 5. 🎥 Content Ecosystem: Omni-Channel Creation Agent

Repurpose one piece of content into every format, on every platform, autonomously.

- [x] **n8n Foundation**: Reddit Intelligence Pipeline, Video Content Generator, LinkedIn/Twitter Automation all operational in `waypoint-operations`.
- [ ] **ClaudeMax Teammate Integration**: Establish Claude as a virtual team member via the *Projects* feature. Maintains master project context across sessions without enterprise plan.
- [ ] **Repurposing Engine API Map**: NotebookLM (scripting) → Higgsfield (AI Avatar) → Descript / Opus Clips / Scriptr.ai (video editing) → VidIQ (YouTube optimization).
- [ ] **Cross-Platform Scheduling Engine**: Agent maps and resizes content for LinkedIn, YouTube, Facebook (Personal + Business), Instagram, Twitter, and TikTok.
- [ ] **Format Diversity Generator**: Agent produces the correct mix per platform — long-form video, short clips, text posts, carousels, infographics.
- [ ] **Multi-Model Enrichment**: Route tasks by model strength — Perplexity (live research), Grok (real-time Twitter trends), others as appropriate. Outputs feed back to the Claude/AntiGravity hub.
- [ ] **Beehiiv + LinkedIn Newsletters**: Private Beehiiv newsletter and LinkedIn newsletter for deep, SEO-optimized content distribution.
- [ ] **"Raw Truth" Podcast**: Transparent entrepreneurship podcast. Focus: honest mistakes, real numbers, no fluff.

---

## 6. 🎓 Horizon 2: Autonomous Education (Course Engine)

*Deferred. Do not build until the Lead Machine (§4) is generating active pipeline.*

- [ ] **"AI for Beginners" Course**: NotebookLM + AntiGravity produces curriculum, study guides, flashcards. Cron jobs keep content current.
- [ ] **"Intermediate AI" Course**: Higher-level tactical operations. Also updates autonomously.

---

## 7. 🤝 Horizon 2: Community & Monetization (Skool)

*Deferred. Build audience via Newsletter + Podcast (§5) before launching paid community.*

- [ ] **Skool Mentorship Program**: Paid subscription community for newly formed franchisees.
- [ ] **Unconverted Lead Funnel**: Routes unconverted Skool leads back into the Waypoint consultation pipeline.

---

## 8. ♻️ Franchisor Lead Recycling Program

B2B system — partner franchisors send their "wrong fit" leads to Waypoint for re-matching.

- [ ] **Recycling Engine**: Build intake logic for "I want to buy something, but not your something" leads from franchisor partners.
- [ ] **Partner Portal / CRM Ingestion**: Automated lead ingestion from franchisor partners into the Waypoint CRM.
- [ ] **Follow-Up Sequence**: Dedicated email + AI avatar sequence to re-engage recycled leads into different franchise portfolios.

---

## 9. 🤖 Horizon 2: Advanced Autonomous Agents

*Deferred. Do not deploy until primary inbound/outbound lead flow is stabilized.*

- [ ] **Reddit Lead Gen Agent** *(powered by Gravity Claw)*: Autonomous agent starts conversations and replies to comments in business/franchising subreddits. Drives inbound volume without paid ads.

---

## Active Phase (March 2026)

| What | Status |
|---|---|
| Website live on `waypointfranchise.com` | 🟢 Done |
| Article library (28 articles) | 🟢 Done |
| Scorecard live | 🟢 Done |
| Technical SEO Sprint 3 | 🔵 Next |
| Cold email warm-up sequences | 🔵 Next |
| Corporate Escape Kit lead magnet | 🔵 Next |
| Gravity Claw (after email sequences warm) | 🟡 Queued |
| Podcast + Newsletter | 🟡 Queued |
