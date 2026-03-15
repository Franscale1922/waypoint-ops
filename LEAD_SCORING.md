# Waypoint Lead Scoring Engine

> **Purpose:** Define exactly how a raw LinkedIn lead gets a score 0–100 and whether they enter the cold email sequence. Score < 70 → SUPPRESSED. Score ≥ 70 → ENRICHED → personalization begins.

---

## Scoring Philosophy

We score on **fit** (does this person match the Waypoint candidate profile?) and **signal** (are they showing buying-adjacent behavior?). We are not scoring on demographics alone — a $200K/yr Director who has no career trigger is a worse lead than a $110K VP who just posted about being burned out.

**Target ICP:** Corporate professional, US-based, 10+ years experience, $100K+ deployable capital likely, currently employed or recently transitioned.

---

## Scoring Rubric (0–100)

### Base Score: 40 points (everyone starts here)

All leads that clear the hard gates (name + LinkedIn URL present) start at 40.

---

### Title / Seniority: up to +25 pts

| Condition | Points |
|---|---|
| Title contains: C-suite (CEO, COO, CFO, CTO, CMO, CHRO) | +25 |
| Title contains: VP, Vice President | +20 |
| Title contains: Director | +15 |
| Title contains: Senior Manager, Principal | +10 |
| Title contains: Manager | +5 |
| No title or junior title | +0 |

---

### Career Trigger Signal: up to +20 pts

A career trigger is any detectable signal of transition, frustration, or momentum change.

| Condition | Points |
|---|---|
| `careerTrigger` field populated **and** references: layoff, job loss, company shutdown, "what's next" | +20 |
| `careerTrigger` references: promotion, new role started (within 90 days) | +15 |
| `careerTrigger` references: relocation, life change, or franchise/entrepreneurship interest | +15 |
| `careerTrigger` present but generic (e.g., "career growth") | +5 |
| No career trigger detected | +0 |

---

### LinkedIn Activity / Post Content: up to +10 pts

| Condition | Points |
|---|---|
| `recentPostSummary` populated AND post references: burnout, autonomy, ownership, side business, W2 frustrations | +10 |
| `recentPostSummary` populated but post is generic professional content | +3 |
| No recent post data | +0 |

---

### Persona Fit Bonus: up to +5 pts

| Condition | Points |
|---|---|
| Company is a known Fortune 500 / large enterprise (likely high W2, golden handcuffs profile) | +5 |
| Company is a mid-market firm (50–500 employees) | +3 |
| Company type unknown | +0 |

---

### Hard Suppression Rules (override all scoring)

These conditions immediately set status = `SUPPRESSED` regardless of score:

| Rule | Reason |
|---|---|
| `country` is not US, CA, AU, or UK | Outside serviceable markets |
| LinkedIn URL is a company page, not a personal profile | Not a person |
| `name` matches known suppression list | Prior opt-out |
| `email` domain matches suppression list | Competitor / media / legal |
| Email bounce rate on domain currently > 2.0% | Deliverability health gate |

---

## Scoring Gate

```
Score < 70   → status = SUPPRESSED (excluded from all sends)
Score 70–79  → status = ENRICHED (enters sequence, lower priority)
Score 80–89  → status = ENRICHED (enters sequence, standard priority)
Score 90–100 → status = ENRICHED (enters sequence, high priority — send first)
```

---

## Max Possible Score Breakdown

| Component | Max |
|---|---|
| Base | 40 |
| Title/Seniority | 25 |
| Career Trigger | 20 |
| Post Content | 10 |
| Persona Fit | 5 |
| **Total** | **100** |

---

## Next Steps for the Code

The current `lead.hunter.start` Inngest function uses a **mock scoring function** (base 60, +20 for careerTrigger, +20 for "vp" in title). Replace the `enrich-and-score` step with the real rubric above.

When the real email finder (ZeroBounce or Hunter.io) is integrated:
1. Call the API to find and validate the email
2. If email bounces or is invalid → add to `SuppressionList` + set `SUPPRESSED`
3. If email is valid → run the scoring rubric and set accordingly

**Priority order for warm-up:** sort SEQUENCED leads by `score DESC` before each daily batch send.
