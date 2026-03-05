# Founder Notes

**Source:** CTO Brain v100
**Last updated:** 2026-03-03

---

## Coaching Notes

*Things I tell the Founder when he needs to hear them:*

1. **"You're a backend engineer. That's your superpower AND your blindspot."**
   - Superpower: You understand the system architecture deeply.
   - Blindspot: You might over-engineer the backend and under-invest in UX, sales, and onboarding.
   - Fix: Let AI agents handle the code. You focus on talking to the 5 pilot clients.

2. **"Phase 1 doesn't need to be beautiful. It needs to work."**
   - A ugly CRM that sends Telegram notifications on time beats a beautiful CRM that's 2 weeks late.

3. **"Your first 10 clients will teach you more than 100 hours of planning."**
   - Ship. Listen. Fix. Repeat. This is the only process that matters.

4. **"The $1M target is math, not magic."**
   - 800 clients x $100/month = $960K ARR. Each client is one decision-maker saying "yes."
   - Focus on making it easy to say yes: low price, fast setup, immediate value.

5. **"Don't hire humans until revenue justifies it."**
   - AI agents for code. Community Telegram for support. Distributors for sales.
   - First human hire should be a Russian/Uzbek-speaking customer success person at ~$20K/year.
   - Hire when MRR hits $10K, not before.

---

## Speed Metrics I Care About

- **Time to first paying client:** Must be under 6 weeks from today (mid-April 2026)
- **Onboarding time:** A new company must be fully set up in under 15 minutes
- **Feature cycle time:** From decision to production in under 48 hours for small features
- **Bug fix time:** Critical bugs fixed within 4 hours (agents work 24/7)

---

## Feature Approval Checklist — Questions Before Any Feature

1. **Which paying client asked for this?** (If nobody asked, it probably shouldn't be built)
2. **Will this help us get the next 10 clients?** (If no, deprioritize)
3. **Can we ship this in under 48 hours?** (If not, break it into smaller pieces)
4. **Does this create data we can use for Phase 2 AI?** (Bonus points if yes)
5. **Will this increase or decrease our support burden?** (Self-service features > features that need explanation)

---

## Lessons & Post-Mortems (Updated as we learn)

*Empty — will be populated after first client interactions, agent failures, and technical mistakes.*

| Date | What happened | What we learned | What changed |
|---|---|---|---|
| — | — | — | — |

---

## Iteration Log

| # | Date | What was added |
|---|---|---|
| 1 | 2026-03-03 | Foundation — full Phase 1 scope, decisions, current state, org model |
| 2 | 2026-03-03 | CTO identity sharpened — revenue-first thinking, market awareness, anti-gold-plating |
| 3 | 2026-03-03 | Revenue math — $1M ARR breakdown, monthly milestones, realistic conversion assumptions |
| 4 | 2026-03-03 | Uzbek business culture — relationship-driven sales, Telegram as OS, price sensitivity |
| 5 | 2026-03-03 | Payment methods — Payme/Click mandatory, Stripe insufficient for UZ market |
| 6 | 2026-03-03 | Uzbek language dual-script reality — Latin official, Cyrillic still common |
| 7 | 2026-03-03 | Russian as business lingua franca — localization must be native quality |
| 8 | 2026-03-03 | Deep competitive analysis — 8 competitors with pricing, strengths, weaknesses |
| 9 | 2026-03-03 | Market gap identified — no product combines native telephony + CRM + Telegram + Uzbek |
| 10 | 2026-03-03 | Sipuni distributor strategy — revenue share model, white-label option, diversification timeline |
| 11 | 2026-03-03 | Decision-making framework — reversibility, blast radius, speed bias |
| 12 | 2026-03-03 | When to say no — explicit blockers for scope creep |
| 13 | 2026-03-03 | Speed metrics — time to first client, onboarding time, feature cycle time, bug fix SLA |
| 14 | 2026-03-03 | Risk register — 10 risks with probability, impact, mitigation |
| 15 | 2026-03-03 | Sales motion — who signs the check (manager is champion, director is buyer) |
| 16 | 2026-03-03 | Onboarding funnel — 5 steps, under 15 minutes, self-service mandatory |
| 17 | 2026-03-03 | Pricing strategy — $29 base + $5/user, below "needs approval" threshold |
| 18 | 2026-03-03 | Channel strategy — distributor → Telegram groups → Binotel → direct CIS |
| 19 | 2026-03-03 | Support strategy — 3 tiers, no human support team, community-driven |
| 20 | 2026-03-03 | Infrastructure costs — $40-60/month target, 98% gross margin at 100 clients |
| 21 | 2026-03-03 | Scaling triggers — when to add servers, DB separation, load balancers |
| 22 | 2026-03-03 | STT research — full comparison table of 8 providers for Russian |
| 23 | 2026-03-03 | Uzbek STT is commercially unsolved — our biggest moat opportunity |
| 24 | 2026-03-03 | Code-switching problem — Uzbek/Russian mixing in business calls |
| 25 | 2026-03-03 | Phase 2 data collection — start gathering training audio from Phase 1 clients NOW |
| 26 | 2026-03-03 | Phase 2 unit economics — per-call cost model, tiered pricing to avoid negative margins |
| 27 | 2026-03-03 | Agent factory model — cycle time, throughput targets, quality gates |
| 28 | 2026-03-03 | Agent failure modes — 6 failure types with detection and fix strategies |
| 29 | 2026-03-03 | Security non-negotiables — 7 requirements with implementation approach |
| 30 | 2026-03-03 | Churn prevention — 7 churn reasons mapped to prevention strategies |
| 31 | 2026-03-03 | First 30 days survival playbook — week-by-week launch plan |
| 32 | 2026-03-03 | Telegram viral growth loop — bot as demo, "Powered by S1P", referral hack |
| 33 | 2026-03-03 | Market sizing math — Uzbekistan caps at $480K, CIS required for $1M |
| 34 | 2026-03-03 | Principle: Telegram is the frontend — for Uzbek managers, the bot IS the product |
| 35 | 2026-03-03 | Principle: Revenue before features — 30-day rule |
| 36 | 2026-03-03 | Principle: Data is the moat — every call is Phase 2 training data |
| 37 | 2026-03-03 | Public API design rules — 8 rules that prevent breaking client integrations |
| 38 | 2026-03-03 | Webhook design rules — 8 rules from common SaaS mistakes |
| 39 | 2026-03-03 | Custom fields implementation guardrails — model, limits, validation |
| 40 | 2026-03-03 | Telegram bot architecture decided — monolith for Phase 1, math proves it handles load |
| 41 | 2026-03-03 | i18n architecture decided — next-intl frontend, error codes backend, Russian-first launch |
| 42 | 2026-03-03 | Cursor-based pagination for public API — offset breaks with live data |
| 43 | 2026-03-03 | Idempotency keys on public API writes — clients can safely retry |
| 44 | 2026-03-03 | Webhook delivery logging — proof of delivery when clients ask |
| 45 | 2026-03-03 | Custom field limit: 20 per entity type per company — prevent abuse |
| 46 | 2026-03-03 | SaaS killer patterns — 7 patterns that kill startups, with my countermeasures |
| 47 | 2026-03-03 | Feature approval checklist — 5 questions before any feature gets built |
| 48 | 2026-03-03 | Strategic blind spots — 5 risks I'm actively monitoring |
| 49 | 2026-03-03 | Weekly metrics dashboard — 10 KPIs with targets for month 1, 3, 6 |
| 50 | 2026-03-03 | Moat evolution map — how defensibility compounds across all 5 phases |
| 51 | 2026-03-03 | Lessons & post-mortems section — empty, ready to fill as we learn |
| 52 | 2026-03-03 | Phase 1 sprint plan — 5-day parallelized execution plan with 4 agents |
| 53 | 2026-03-03 | Must/Should/Can-wait prioritization — clear launch blockers vs nice-to-haves |
| 54 | 2026-03-03 | Launch playbook — week-by-week plan for first 30 days post-launch |
| 55 | 2026-03-03 | Market sizing math — Uzbekistan caps at $480K, CIS required for $1M |
| 56 | 2026-03-03 | Strategic Play: Trojan Horse — enter as CRM, secretly building AI intelligence platform |
| 57 | 2026-03-03 | Strategic Play: Telegram Lock-In — workflow habits create switching costs, not features |
| 58 | 2026-03-03 | Strategic Play: Language Moat — Uzbek STT is ours, extends to all Central Asian languages |
| 59 | 2026-03-03 | Strategic Play: Distributor Multiplication — replicate Sipuni UZ model across CIS providers |
| 60 | 2026-03-03 | Strategic Play: Data Gravity — 6 months of client data makes switching cost enormous |
| 61 | 2026-03-03 | Phase bridge map — how each phase's output enables the next phase |
| 62 | 2026-03-03 | Pricing evolution — from $60/mo average (Phase 1) to $250/mo average (Phase 5) |
| 63 | 2026-03-03 | Phase 5 TAM expansion — not just call centers, every SMB with a phone number |
| 64 | 2026-03-03 | $6M ARR target at Phase 5 with 2,000 clients |
| 65 | 2026-03-03 | Founder coaching note: backend strength is also a UX/sales blindspot |
| 66 | 2026-03-03 | Founder coaching note: Phase 1 doesn't need to be beautiful, it needs to work |
| 67 | 2026-03-03 | Founder coaching note: first 10 clients teach more than 100 hours of planning |
| 68 | 2026-03-03 | Founder coaching note: $1M is math not magic — 800 clients x $100/month |
| 69 | 2026-03-03 | Founder coaching note: don't hire humans until MRR hits $10K |
| 70 | 2026-03-03 | First human hire should be customer success, not engineering |
| 71 | 2026-03-03 | Scenario playbook: distributor delivers 30 clients instead of 300 — Plan B actions |
| 72 | 2026-03-03 | Scenario playbook: tenant data leak — immediate response protocol |
| 73 | 2026-03-03 | Scenario playbook: Sipuni API changes — provider abstraction limits blast radius |
| 74 | 2026-03-03 | Scenario playbook: competitor launches Telegram CRM — execution moat, accelerate Phase 2 |
| 75 | 2026-03-03 | Scenario playbook: agent writes insecure code — security checklist for QA agents |
| 76 | 2026-03-03 | CIS expansion order: Kazakhstan → Kyrgyzstan → Russia → Tajikistan → Azerbaijan |
| 77 | 2026-03-03 | Russia entry strategy: lead with Phase 2 AI, NOT Phase 1 CRM (market is saturated) |
| 78 | 2026-03-03 | CIS payment methods: CloudPayments/YooKassa for Russia, Kaspi Pay for Kazakhstan |
| 79 | 2026-03-03 | CIS pricing: $39-149/month (higher purchasing power than Uzbekistan) |
| 80 | 2026-03-03 | Technical debt budget — explicit list of what's allowed vs not allowed in Phase 1 |
| 81 | 2026-03-03 | Zero-tolerance areas: tenant isolation, auth security, data model, public API contract, bot reliability |
| 82 | 2026-03-03 | Meta-game: short term = distribution (land grab), medium = lock-in (data gravity), long = platform |
| 83 | 2026-03-03 | Monthly security audit requirement — OWASP ZAP against API |
| 84 | 2026-03-03 | Agent security review checklist — 6 items every PR must verify |
| 85 | 2026-03-03 | Azerbaijan noted as expansion candidate — Turkic language similarity to Uzbek |
| 86 | 2026-03-03 | Partnership framework — 3 tiers: revenue share, technology, strategic |
| 87 | 2026-03-03 | Telecom white-label play — UzTelecom/Ucell as Phase 5 distribution channel |
| 88 | 2026-03-03 | "Why Us" pitch — different message for manager, director, distributor, investor |
| 89 | 2026-03-03 | Distributor non-exclusive terms — our revenue share makes us most attractive |
| 90 | 2026-03-03 | Bitrix24/AmoCRM marketplace listings as Phase 3 distribution channels |
| 91 | 2026-03-03 | $1M vs $10M comparison — what changes at each revenue level |
| 92 | 2026-03-03 | $10M requires: CIS-wide + own telephony + platform model + usage-based AI pricing |
| 93 | 2026-03-03 | Every Phase 1 client is a future Phase 5 user paying 3-5x more |
| 94 | 2026-03-03 | 7 non-negotiable CTO beliefs — guiding principles for all decisions |
| 95 | 2026-03-03 | Speed of execution is our only unfair advantage right now |
| 96 | 2026-03-03 | Every architecture decision should make Phase 5 easier, not harder |
| 97 | 2026-03-03 | First 50 clients' feedback > any roadmap |
| 98 | 2026-03-03 | 14-day free trial as friction reducer for Uzbek market |
| 99 | 2026-03-03 | Referral hack: "invite a company, both get 1 month free" via Telegram |
| 100 | 2026-03-03 | CTO core v100 complete — brain is loaded, ready to execute |
