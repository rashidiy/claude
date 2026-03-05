# Go-to-Market Strategy

**Source:** CTO Brain v100
**Last updated:** 2026-03-03

---

## Who Signs the Check?

In an Uzbek call center:
- **Owner/Director** — makes the buying decision. Doesn't use the CRM daily. Cares about: cost, ROI, "is my team working?"
- **Call center manager** — the champion. Uses Telegram. Wants visibility into calls without asking operators. THIS is who you demo to.
- **IT person (if they have one)** — does the integration. Needs: good docs, simple API, Docker setup that works.

**Sales motion:** Demo the Telegram bot to the manager. Show them a missed call notification with client info and a "callback" button. That's the moment they say "I need this."

---

## The Onboarding Funnel

```
Step 1: Manager sees demo (via Sipuni distributor or Telegram group)
Step 2: Signs up on web → creates company (< 2 minutes)
Step 3: Connects Sipuni API key (settings page, < 3 minutes)
Step 4: Adds Telegram bot to their group (< 1 minute)
Step 5: First call comes in → notification appears in Telegram
Step 6: "Holy shit, this works" moment → calls operators to set up CRM

Total time to value: < 15 minutes
```

**Critical insight:** Steps 1-5 must work WITHOUT talking to a human from our side. Self-service or die. We have no support team. The product must sell itself.

---

## Channel Strategy

| Channel | Timeline | How |
|---|---|---|
| Sipuni distributor | Month 1-3 | They demo, we provide materials. Revenue share: 15% forever. |
| Telegram groups | Month 2+ | Join Uzbek business/call center Telegram groups. Share bot demo videos. |
| Binotel partnership | Month 3+ | Replicate the Sipuni distributor model with Binotel's Uzbek partners. |
| Word of mouth | Month 3+ | Happy clients tell other call centers. The Telegram bot is inherently viral — managers show it off. |
| Direct sales (CIS) | Month 6+ | Russian-language landing page, Google Ads targeting "call center CRM" in Russian. |

---

## Client Support Strategy (No Human Support Team)

### Tier 1: Self-Service
- In-app tooltips and onboarding wizard (frontend)
- Video tutorials: 3-5 minute screencasts in Russian and Uzbek
- FAQ page covering top 20 questions
- API documentation (Swagger + examples)

### Tier 2: Community
- Telegram group: "S1P Users" — moderated by Founder
- Clients help each other. Power users become advocates.
- We monitor the group for bug reports and feature requests.

### Tier 3: Direct (Founder only, temporary)
- Critical issues only: data loss, can't log in, billing disputes
- Via Telegram DM, not email
- Goal: eliminate Tier 3 by making Tier 1 and 2 good enough

---

## Churn Prevention — What Kills SaaS in Emerging Markets

| Churn reason | How we prevent it |
|---|---|
| "I don't understand how to use it" | 15-minute onboarding, Russian/Uzbek video tutorials, Telegram support group |
| "It's too expensive for what I get" | Price anchored below Bitrix24 paid tier. Show ROI: "you missed X calls this week worth $Y" |
| "My team won't adopt it" | Telegram bot = zero training needed. Managers already live there. |
| "It doesn't work with my other tools" | Public API + webhooks = connect to anything |
| "Support is unresponsive" | Community Telegram group, self-service docs, 4-hour critical bug fix SLA (agents are 24/7) |
| "Feature X is missing" | Collect requests in Telegram group, build weekly based on demand, announce publicly |
| "I found something better" | Move fast on Phase 2 AI — that's the lock-in feature nobody else has for this market |

---

## The First 30 Days After Launch — Survival Playbook

```
Week 1: Pilot with 5-10 companies (hand-picked from Sipuni distributor's best clients)
  - Founder personally onboards each one
  - Collect every complaint, confusion, friction point
  - Fix critical issues same-day (agents work 24/7)

Week 2: Expand to 20-30 companies
  - Self-service onboarding must work by now
  - Telegram support group is active
  - Founder stops doing manual onboarding

Week 3: Open to all 300 Sipuni clients via distributor
  - Distributor sends announcement
  - Landing page in Russian + Uzbek with demo video
  - Conversion target: 40-60 sign-ups in first week

Week 4: Measure and iterate
  - Metrics: sign-ups, activation (connected Sipuni + Telegram), 7-day retention
  - If activation < 50%: onboarding is broken, fix immediately
  - If 7-day retention < 70%: core value proposition isn't landing, investigate
```

---

## The "Why Us" Pitch — Different Audience, Different Message

### To a call center manager:
"You won't miss another call. Every missed call, every completed call — instant notification in your Telegram with the client's name and a callback button. Try it for 14 days free."

### To a company director:
"Your operators are losing leads and you don't know it. S1P shows you exactly who called, what happened, and what fell through the cracks. $29/month for your whole team."

### To a Sipuni distributor:
"Your clients use Sipuni for calls but need a CRM. Instead of losing them to Bitrix24 (which has its own telephony), give them S1P — a CRM built FOR Sipuni. You earn 15% of every subscription forever."

### To an investor (Phase 2+):
"We're building the AI infrastructure for voice-based business communication in Central Asia. 500 clients, $50K MRR, growing 30% month-over-month. We own the only production Uzbek speech-to-text system in the world. Our Phase 5 AI agents will let any small business have a professional call center for $99/month."
