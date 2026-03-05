# Roadmap — 5 Phases

**Source:** CTO Brain v100
**Last updated:** 2026-03-03

---

## The Vision (5 Phases)

### Phase 1 — Integrated CRM (MVP) ← CURRENT
Multi-tenant CRM connected to Sipuni & Binotel. Telegram bot for real-time call management. Public REST API for external integrations. Three languages: uz, ru, en.

### Phase 2 — Call Intelligence
AI-powered STT transcription + conversation analysis. Operator scoring, sentiment analysis, compliance checks, lead quality signals. Research starts during Phase 1.

### Phase 3 — Own Telephony (FreePBX) + Native CRM Connectors
Cut provider dependency. Offer own telephony via FreePBX + SIP trunks. Add native Bitrix24, AmoCRM integrations.

### Phase 4 — Custom SIP Middleware
Replace FreePBX with purpose-built SIP stack. Real-time audio streaming to AI pipeline. Full control over call routing, recording, conferencing.

### Phase 5 — AI Agents Replace Operators
AI voice agents handle inbound/outbound calls. Small businesses get virtual call centers without human operators. The endgame.

---

## Phase 1 — Defined Scope

### CRM Core
- Entities: Contact, Lead, Deal, Task, Note
- Fixed base fields + custom fields per entity (JSONB storage)
- Configurable pipeline stages per company
- Multi-tenant: Owner → Company → User hierarchy

### Telephony Integration
- Sipuni (primary) — done
- Binotel (secondary) — done
- Provider abstraction layer — done

### Telegram Bot (Full)
- Notifications: call completed, missed call, new lead, deal stage change
- Rich messages: caller info, matched contact, recording, duration, operator
- Interactive: assign lead, mark handled, trigger callback, deep link to CRM
- Configurable: per team, per group, filter by call type
- Shared bot instance, company mapped by chat_id
- Webhook-based (not polling)

### Public REST API
- Endpoints: calls, contacts, leads, deals (read)
- Authentication: API key per company
- Rate limited
- Swagger/Redoc documentation
- Separate from internal JWT-authenticated API

### Outbound Webhooks
- Company configures target URL
- Events: call.completed, call.missed, lead.created, deal.stage_changed, contact.created
- JSON payload, HMAC signed
- Retry: 3 attempts with exponential backoff

### Internationalization
- Three languages from day one: Uzbek, Russian, English
- Both frontend and backend responses

### What's NOT in Phase 1
- AI transcription / Call Intelligence (Phase 2)
- Native CRM connectors like Bitrix24/AmoCRM (Phase 3)
- Own telephony infrastructure (Phase 3)
- Mobile app (TBD)
- Billing system / payment processing (manual invoicing initially)

---

## Current State Assessment

### What Exists (S1P Backend — ~80% of Phase 1)
- FastAPI multi-tenant backend
- Sipuni + Binotel integration with provider abstraction
- CRM entities: contacts, leads, deals, tasks, notes
- Auth: JWT with RBAC (Owner, Admin, Manager, Operator)
- Call recording streaming
- Analytics service with caching
- Contract enforcement
- Docker setup (dev + prod)
- Alembic migrations

### What Exists (Frontend — ~70% of Phase 1)
- Next.js 15 with Ant Design
- Owner panel + Company panel
- Subdomain routing
- Auth flows
- Basic CRM pages

### Gap to Phase 1 Completion

**MUST SHIP (launch blockers):**
- [ ] Telegram bot — core notifications for calls (completed, missed) with contact info
- [ ] Telegram bot — interactive buttons (assign, mark handled, callback link)
- [ ] Telegram bot — company configuration (connect bot, select group, set filters)
- [ ] Admin bootstrapping — owner creates first company admin, invitation email
- [ ] Operator scoping — operators only see their own leads/deals/tasks
- [ ] Fix tasks permission (trivial — add decorator)

**SHOULD SHIP (high value, not blockers):**
- [ ] Custom fields — backend model + API endpoints
- [ ] Custom fields — frontend UI (settings page to define, forms to fill)
- [ ] Public REST API — read endpoints for calls, contacts, leads, deals
- [ ] Public REST API — API key generation in company settings
- [ ] Outbound webhooks — event system + delivery with retry
- [ ] i18n — Russian + English frontend (Uzbek can follow in week 2)

**CAN WAIT (ship in week 2):**
- [ ] Tags router
- [ ] Audit trail
- [ ] Email service with Jinja2 templates (use simple text emails for now)
- [ ] Uzbek language translations
- [ ] Webhook test button in UI
- [ ] Custom field search/filter

**Parallelization plan:**
```
Day 1-2: Agent A: Telegram bot backend    | Agent B: Operator scoping + permissions
Day 2-3: Agent A: Telegram bot frontend   | Agent B: Admin bootstrapping + email
Day 3-4: Agent C: Public API + API keys   | Agent D: Custom fields backend
Day 4-5: Agent C: Outbound webhooks       | Agent D: Custom fields frontend
Day 5:   All agents: Integration testing, bug fixes, i18n pass
```

---

## Phase 2 → Phase 5 Bridge — How Each Phase Enables the Next

```
Phase 1: CRM + Telephony
  → Generates: Call recordings, contact data, operator behavior patterns
  → Enables Phase 2 because: We have the audio data to train on

Phase 2: Call Intelligence (STT + AI Analysis)
  → Generates: Transcripts, conversation patterns, script templates, quality scores
  → Enables Phase 3 because: We understand call flows deeply enough to build our own PBX
  → Revenue unlock: AI add-on at $29-99/month per company

Phase 3: Own Telephony (FreePBX)
  → Generates: Real-time audio streams (not just recordings), lower latency, full control
  → Enables Phase 4 because: We control the audio pipeline end-to-end
  → Revenue unlock: Telephony margin + no provider dependency

Phase 4: Custom SIP Middleware
  → Generates: Programmable call routing, real-time AI injection into live calls
  → Enables Phase 5 because: AI can now speak in real-time on a call, not just analyze after
  → Revenue unlock: Platform fees for AI-powered call features

Phase 5: AI Agents
  → Generates: Fully automated call handling, 24/7 virtual operators
  → This IS the product. Everything before was building toward this.
  → Revenue unlock: Per-minute AI agent pricing. Small business pays $99/month for a
    virtual call center that would cost $2,000/month in human operators.
  → TAM expands 100x: Not just call centers — every small business with a phone number.
```

---

## Phase 2 Research Checklist (Start During Phase 1)

### STT (Speech-to-Text) Evaluation
| Solution | Russian quality | Uzbek quality | Cost/hour | Self-hosted? | Notes |
|---|---|---|---|---|---|
| OpenAI Whisper (large-v3) | Excellent | Poor-Medium | Free (self-hosted) or $0.36/hr (API) | Yes | Best open-source option. Uzbek needs fine-tuning. |
| Yandex SpeechKit | Excellent | None | ~$0.60/hr | No (cloud only) | Best Russian quality. No Uzbek. Russian company — geopolitical risk. |
| Sber SaluteSpeech | Very good | None | ~$0.50/hr | No | Good Russian. No Uzbek. |
| Deepgram | Good | None | $0.25/hr | No | Fast, cheap. No Uzbek. |
| Google Speech-to-Text | Good | Uzbek beta | $0.24-1.44/hr | No | Uzbek support exists but quality unverified. |
| Azure Speech | Good | None | $1.00/hr | No | Enterprise-grade. No Uzbek. |

**Recommendation:** Whisper large-v3 self-hosted for Russian. Fine-tune Whisper on Uzbek call center audio for Uzbek support. This gives us: zero per-call cost (self-hosted), both languages, full data control.

**CRITICAL INSIGHT — Uzbek STT is unsolved commercially:**
- No major provider (Google, Azure, Yandex, Deepgram, Sber) supports Uzbek.
- Whisper lists Uzbek but quality is poor (WER 20-40%+).
- Mozilla CommonVoice has only ~50-100 hours of validated Uzbek audio — insufficient.
- **Code-switching problem:** Uzbek business conversations mix Russian and Uzbek constantly. The model needs to handle both languages in a single utterance.
- **This is simultaneously our biggest challenge AND our biggest moat.** If we build production-quality Uzbek STT, no competitor can easily replicate it.

**Action items for Phase 2 research (start NOW):**
1. Start collecting Uzbek call recordings from Phase 1 clients (with consent). Every call is training data.
2. Target: 200-500 hours of labeled Uzbek call center audio for fine-tuning.
3. Build a small labeling pipeline — hire 2-3 Uzbek speakers to transcribe samples.
4. Test Whisper fine-tuning on 50 hours first to validate the approach.
5. Evaluate Google Speech-to-Text Uzbek beta — quality is unverified but worth testing.

### LLM for Conversation Analysis
- Claude API (Anthropic) — best reasoning, best for nuanced analysis
- GPT-4o (OpenAI) — good alternative, slightly cheaper
- Self-hosted (Llama 3, Mistral) — cheapest at scale, but lower quality
