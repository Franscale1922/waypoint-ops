# Waypoint Franchise Advisors — Ops & Intelligence Repository

This repo is the **single source of truth** for all Waypoint operational intelligence, content production, and marketing assets. It syncs across machines (laptop + Mac Mini) via GitHub and is designed to be consumed by any AI agent.

---

## Directory Structure

```
waypoint-ops/
├── knowledge-base/
│   ├── research/          # Third-party source analyses (Connor Groce, etc.)
│   └── intelligence/      # Market insights, industry data, competitive intel
├── content/
│   ├── articles/
│   │   ├── tier-1/        # High-priority articles (drafted first)
│   │   ├── tier-2/        # Nurture sequence articles
│   │   └── tier-3/        # Vertical deep-dives
│   ├── templates/         # Article brief template, voice guide
│   └── queue/             # Content production tracker
└── assets/                # Images, PDFs, downloadable resources
```

---

## Cross-Machine Sync

```bash
# Pull latest from any machine
cd ~/Downloads/waypoint-ops && git pull origin main

# After making changes, push back
git add -A && git commit -m "update: [describe what changed]" && git push origin main
```

---

## Agent Instructions

Any AI agent working on Waypoint content should:
1. **Start by reading** `content/templates/voice_tone_guide.md` — all content must match Kelsey's voice
2. **Check** `content/queue/content_queue.md` before drafting — don't duplicate work in progress
3. **Use** `content/templates/article_brief_template.md` before writing any article
4. **Reference** `knowledge-base/research/` for source material — never plagiarize, always reframe
5. **Save all new articles** to the correct `content/articles/tier-X/` folder
6. **Commit and push** after completing any work session

---

## Content Sources Cataloged

| Source | File | Notes |
|---|---|---|
| Connor Groce (Franchise Gateway) | `knowledge-base/research/connor_groce_content_database.md` | 90+ articles cataloged with Waypoint repurposing angles |
