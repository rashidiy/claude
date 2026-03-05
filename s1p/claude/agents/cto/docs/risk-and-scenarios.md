# Risk Register & Scenario Playbook

**Source:** CTO Brain v100
**Last updated:** 2026-03-03

---

## Risk Register

| # | Risk | Probability | Impact | Mitigation |
|---|---|---|---|---|
| 1 | Sipuni distributor relationship breaks | Low | Critical | Diversify to Binotel and direct sales by Q3. Never be 100% dependent on one channel. |
| 2 | Sipuni builds their own CRM | Medium | High | Move fast. By the time they notice us, we should have 500+ clients and Phase 2 AI moat. |
| 3 | Price too high for Uzbek market | Medium | High | Start at $29/month base. Can always raise prices later. Never lower them — it signals weakness. |
| 4 | Agent-generated code quality is poor | Medium | Medium | QA agent reviews every PR. Integration tests must pass before merge. Founder spot-checks weekly. |
| 5 | Can't find good Russian/Uzbek STT for Phase 2 | Medium | Medium | Research starts NOW during Phase 1. Evaluate Whisper, Yandex SpeechKit, SaluteSpeech, Deepgram by June. |
| 6 | 300 clients overwhelm support with no support team | High | Medium | Self-service onboarding. Telegram group for support (community-driven). FAQ/docs in Russian/Uzbek. |
| 7 | Multi-tenant data leak between companies | Low | Critical | Every query MUST filter by company_id. Add automated tests that verify tenant isolation. |
| 8 | Call recordings storage costs explode | Medium | Medium | Don't store recordings ourselves in Phase 1. Stream from provider. Phase 3 changes this. |
| 9 | Founder gets distracted by Phase 2-5 before Phase 1 ships | High | Critical | I (CTO) block all non-Phase 1 work until first 50 paying clients. Research is OK. Building is not. |
| 10 | Competitors copy Telegram bot idea | Low | Low | Execution speed is the moat. By the time they copy, we have 6 months of user data and Phase 2 AI. |

---

## Scenario Playbook — When Things Go Wrong

### Scenario: Sipuni distributor delivers 30 clients instead of 300
**Reality check:** 10% conversion of a "warm pipeline" is common.
**Response:**
1. Don't panic. 30 paying clients is validation.
2. Ask WHY the other 270 didn't convert. Likely: wrong timing, "we'll try later", competitor already embedded.
3. Offer 14-day free trial to lower friction.
4. Get testimonials from the 30. Social proof sells in Uzbekistan.
5. Activate Plan B: approach Binotel Uzbekistan directly. Approach Zadarma UZ users.
6. Start Telegram group marketing: post demo videos in Uzbek business groups.

### Scenario: A client's data leaks to another tenant
**Severity:** CRITICAL. Company-ending if not handled perfectly.
**Response:**
1. Take the system offline immediately. Data integrity > uptime.
2. Identify the exact query that leaked data. Fix the company_id filter.
3. Audit EVERY query in the codebase for missing company_id filters.
4. Notify affected clients within 24 hours (legal requirement in most jurisdictions).
5. Add automated tenant isolation tests that run on every deploy.
6. Post-mortem in the Lessons section.

### Scenario: Sipuni changes their API or deprecates endpoints
**Response:**
1. Provider abstraction layer (already built) limits blast radius.
2. Fix the SipuniProvider implementation. Other providers unaffected.
3. This is why Phase 3 (own telephony) exists — eliminate provider dependency.
4. Accelerate Phase 3 timeline if this becomes a pattern.

### Scenario: A competitor launches a Telegram CRM bot for Uzbekistan
**Response:**
1. Don't panic. Execution > ideas. They're at least 3-6 months behind us in telephony integration.
2. Accelerate Phase 2. AI analysis is not copyable in weeks.
3. Lock in clients with annual discounts. Data gravity protects existing users.
4. Differentiate: our bot is connected to a real telephony provider with real calls. Theirs is probably just a messenger CRM.

### Scenario: AI agent writes code that passes tests but has a security vulnerability
**Response:**
1. This WILL happen. Plan for it.
2. Every PR gets a security-focused review pass: SQL injection, missing auth checks, exposed secrets.
3. Maintain a security checklist that QA agents must verify:
   - [ ] All queries filter by company_id
   - [ ] All endpoints have @require_permissions
   - [ ] No raw SQL with string interpolation
   - [ ] No secrets in code or logs
   - [ ] No PII logged
   - [ ] API keys are hashed in DB
4. Monthly security audit: run OWASP ZAP or similar against the API.

---

## SaaS Killer Patterns — Things I've Learned to Spot

| Pattern | How it shows up | What I do |
|---|---|---|
| **Building in the dark** | 3 months of coding, zero client conversations | Block. Ship incomplete MVP to 5 clients this week. |
| **Feature factory** | "Just one more feature then we launch" | Launch now. Add features based on client demand, not imagination. |
| **Premature scaling** | Kubernetes setup before 10 clients | Delete it. Single VPS until it hurts. |
| **Wrong audience** | Building for enterprise while selling to SMB | All architecture decisions must match the $29-79/month buyer. |
| **Founder does support** | Founder answering "how do I login?" questions | Self-service must be good enough. Founder's time is strategy + sales. |
| **Ignoring metrics** | No tracking of sign-ups, activation, retention | Instrument on day 1. If we can't measure it, we can't improve it. |
| **CTO does coding** | I start writing code "just this once" | STOP. I think. Agents code. The moment I code, I stop thinking strategically. |

---

## Strategic Blind Spots to Watch

- **We're betting heavily on Sipuni distributor.** If they underperform, we need Plan B by April.
- **Uzbek market alone won't hit $1M.** CIS expansion prep must start by June, not September.
- **Phase 2 AI is the real product.** Phase 1 CRM is the distribution mechanism. Don't fall in love with the CRM — it's the wrapper, not the candy.
- **AI agent competition (Phase 5) is moving FAST.** Bland AI, Vapi, Retell, Synthflow — all funded, all building. Our advantage: CIS market knowledge + Uzbek language + existing client base. Speed matters.
- **The Founder is a backend engineer.** Frontend and mobile are potential quality risks. Agent output for frontend must be scrutinized harder. Consider hiring a part-time design reviewer if AI-generated UI looks generic.
