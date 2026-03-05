# Revenue & Pricing

**Source:** CTO Brain v100
**Last updated:** 2026-03-03

---

## Revenue Math — The $1M Target

**Goal:** $1,000,000 ARR by end of 2026 (December 31, 2026)

**That means:** ~$83,333/month in recurring revenue by December.

### Path to $1M (bottoms-up)

```
Scenario A: Volume play (many small companies)
  $50/month average per company x 1,667 companies = $1M ARR

Scenario B: Mid-market (medium companies)
  $150/month average x 556 companies = $1M ARR

Scenario C: Blended (realistic)
  300 clients (Sipuni UZ) x $60/month avg     = $18,000/month by Q2
  + 200 clients (Binotel UZ) x $60/month      = $12,000/month by Q3
  + 300 clients (CIS expansion) x $80/month   = $24,000/month by Q3-Q4
  + Phase 2 AI upsell on 400 clients x $40/mo = $16,000/month by Q4
  + 200 enterprise (own telephony) x $200/mo  = $40,000/month by Q4
  ─────────────────────────────────────────────
  Total potential:                              ~$110,000/month = $1.32M ARR
```

### Revenue Milestones
| Month | Target MRR | Cumulative clients | Phase |
|---|---|---|---|
| March 2026 | $0 | 0 | Building Phase 1 |
| April 2026 | $3,000 | 50 (pilot) | First paying clients |
| June 2026 | $15,000 | 250 | Sipuni UZ wave |
| September 2026 | $40,000 | 500+ | Binotel + CIS expansion |
| December 2026 | $83,000+ | 800+ | Phase 2 upsell + enterprise |

### Critical Assumption
The 300 Sipuni clients are NOT guaranteed. They're a warm pipeline via the distributor. Realistic conversion: 40-60% within first 3 months = 120-180 paying clients. The rest need sales effort.

---

## Pricing Strategy (Phase 1)

```
Plan: S1P CRM

Base:           $29/month (includes 5 users)
Additional:     $5/user/month
Annual:         2 months free ($290/year instead of $348)

Telegram bot:   Included in all plans
Public API:     Included in all plans
Webhooks:       Included in all plans
Custom fields:  Included in all plans

Phase 2 add-on (future):
  Call Intelligence: +$0.02/minute of analyzed audio
  or flat +$29/month for up to 500 calls analyzed
```

**Why $29:** Below the "needs manager approval" threshold in most Uzbek companies. A manager can expense this without going to the director. Removes friction.

**Why per-user above 5:** Scales naturally. A 5-person team pays $29. A 20-person team pays $104. A 50-person team pays $254. All reasonable.

---

## Pricing Evolution Across Phases

```
Phase 1: $29/month base + $5/user       → Avg $60/month per company
Phase 2: + $29-99/month AI add-on       → Avg $100-150/month per company
Phase 3: + telephony ($5-10/user/month)  → Avg $150-250/month per company
Phase 5: AI Agent plans $99-499/month    → Avg $200-500/month per company
                                            (new market: SMBs without call centers)
```

At Phase 5 with 2,000 clients at $250/month average = **$6M ARR.**

---

## Phase 2 Cost Model

```
Self-hosted Whisper (GPU cost: ~$0.05-0.15/hour of audio)
  + Claude API for analysis (~$0.01-0.03 per call summary)
  = ~$0.02-0.05/call total cost

At 150 calls/day per company:
  Cost to us:    $3-7.50/day = $90-225/month per company
  Charge client: $29-49/month flat or $0.02/minute

  PROBLEM: Unit economics are negative at flat $29/month for heavy users.
  SOLUTION: Tier by call volume or charge per-minute.

  Pricing model for Phase 2 AI:
    Starter: 200 calls/month analyzed — included in base plan
    Growth:  $0.02/call above 200
    Unlimited: $99/month flat (for companies with 100+ calls/day)
```

**Unit economics work if we self-host Whisper AND tier the AI pricing by usage.**

---

## Metrics Dashboard — What I Track Weekly

| Metric | Target (Month 1) | Target (Month 3) | Target (Month 6) |
|---|---|---|---|
| Sign-ups | 30 | 150 | 400 |
| Activated (connected provider + Telegram) | 20 | 100 | 300 |
| Paying clients | 10 | 80 | 250 |
| MRR | $500 | $5,000 | $20,000 |
| Churn rate (monthly) | < 10% | < 8% | < 5% |
| Avg revenue per client | $50 | $60 | $75 |
| Support tickets per client per month | < 2 | < 1 | < 0.5 |
| NPS (Net Promoter Score) | — | > 30 | > 50 |
| Time to activate (sign-up → first Telegram notification) | < 30 min | < 15 min | < 10 min |
| API uptime | 99% | 99.5% | 99.9% |

---

## What Separates $1M from $10M

| $1M business | $10M business |
|---|---|
| Good CRM with Telegram | AI platform with voice intelligence |
| Uzbekistan | CIS-wide + Central Asia |
| Sipuni + Binotel | Own telephony + provider marketplace |
| Manual sales via distributors | Self-service + marketplace + partnerships |
| Per-company pricing | Per-seat + usage-based AI pricing |
| 800 clients | 5,000+ clients |
| Single product | Platform with add-ons and integrations marketplace |
| Founder does everything | Team of 5-10 (first hire at $10K MRR) |
| AI agents write code | AI agents handle customer calls (Phase 5) |

The difference isn't just more clients. It's a fundamentally different product (platform vs tool), a different market (CIS vs Uzbekistan), and a different revenue model (usage-based AI vs flat SaaS).

**The $1M gets us to the $10M.** Every Phase 1 client is a future Phase 5 user paying 3-5x more.
