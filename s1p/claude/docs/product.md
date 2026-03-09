# S1P — Product Reference

Lean product context for coding agents. Business/strategy docs live at `~/claude/cto/`.

---

## What S1P Is

Multi-tenant CRM with integrated telephony (Sipuni, Binotel) for the CIS market. Telegram bot for real-time call management. Goal: replace Bitrix24/AmoCRM for small-medium call centers in Uzbekistan, Russia, Kazakhstan.

---

## 5 Phases

1. **Phase 1 (current):** Integrated CRM — multi-tenant CRM + telephony + Telegram bot + public API + i18n
2. **Phase 2:** Call Intelligence — AI-powered STT transcription, operator scoring, sentiment analysis
3. **Phase 3:** Own Telephony — FreePBX + SIP trunks, native Bitrix24/AmoCRM connectors
4. **Phase 4:** Custom SIP Middleware — purpose-built SIP stack, real-time AI audio injection
5. **Phase 5:** AI Agents Replace Operators — virtual call centers for small businesses

Every Phase 1 decision should make Phase 5 easier. Store the data. Build the abstractions. Design for the endgame.

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

### Telegram Bot
- Notifications: call completed, missed call, new lead, deal stage change
- Rich messages: caller info, matched contact, recording, duration, operator
- Interactive: assign lead, mark handled, trigger callback, deep link to CRM
- Configurable: per team, per group, filter by call type
- Shared bot instance, company mapped by chat_id, webhook-based

### Public REST API
- Endpoints: calls, contacts, leads, deals (read)
- Auth: API key per company (hashed with bcrypt, shown once at creation)
- Rate limited, cursor-based pagination
- Swagger/Redoc documentation

### Outbound Webhooks
- Events: call.completed, call.missed, lead.created, deal.stage_changed, contact.created
- JSON payload, HMAC-SHA256 signed
- Retry: 3 attempts (10s, 60s, 300s), delivery log

### Internationalization
- Three languages: Russian (primary), Uzbek, English
- Frontend: next-intl, JSON locale files
- Backend: error codes (frontend translates), Jinja2 email templates per language

### NOT in Phase 1
- AI transcription (Phase 2)
- Native CRM connectors (Phase 3)
- Own telephony (Phase 3)
- Mobile app, billing system, real-time WebSocket/SSE

---

## Current State

### Backend (~80% of Phase 1)
- FastAPI multi-tenant backend
- Sipuni + Binotel with provider abstraction
- CRM entities: contacts, leads, deals, tasks, notes
- Auth: JWT with RBAC (Owner, Admin, Manager, Operator), httpOnly cookies
- Call recording streaming, analytics service with caching
- Docker setup (dev + prod), Alembic migrations

### Frontend (~70% of Phase 1)
- Next.js 15 with Ant Design
- Owner panel + Company panel
- Subdomain routing, auth flows
- Basic CRM pages

---

## Gap to Phase 1 Completion

### MUST SHIP (launch blockers)
- [ ] Telegram bot — core notifications (completed, missed calls) with contact info
- [ ] Telegram bot — interactive buttons (assign, mark handled, callback link)
- [ ] Telegram bot — company configuration (connect bot, select group, set filters)
- [ ] Admin bootstrapping — owner creates first company admin, invitation email
- [ ] Operator scoping — operators only see their own leads/deals/tasks
- [ ] Fix tasks permission (trivial — add decorator)

### SHOULD SHIP (high value, not blockers)
- [ ] Custom fields — backend model + API endpoints
- [ ] Custom fields — frontend UI (settings page + forms)
- [ ] Public REST API — read endpoints + API key generation
- [ ] Outbound webhooks — event system + delivery with retry
- [ ] i18n — Russian + English frontend (Uzbek can follow in week 2)

### CAN WAIT (week 2)
- [ ] Tags router, audit trail
- [ ] Email service with Jinja2 templates
- [ ] Uzbek language translations
- [ ] Webhook test button in UI
- [ ] Custom field search/filter

---

## Core Principles

1. **Ship beats perfect.** Phase 1 must reach 300 real users. Polish comes later.
2. **Platform thinking.** Every feature should expose an API, not just a UI.
3. **Multi-tenant always.** Every query, every cache key, every webhook — scoped to company_id.
4. **Don't build what you can buy.** Use existing services until they limit you.
5. **Telegram is the frontend.** For Uzbek managers, the Telegram bot IS the product. The web CRM is the back-office.
6. **Revenue before features.** Never build a feature that doesn't lead to a paying client within 30 days.
7. **One provider deep, then wide.** Nail Sipuni perfectly before expanding to more providers.
8. **Localization is not translation.** Uzbek/Russian UI must feel native — date formats, phone numbers (998xx), currency (UZS/USD), business terminology.
9. **Data is the moat.** Every call, conversation, interaction we store becomes training data for Phase 2 AI. Design schema with this in mind.
10. **Every decision enables Phase 5.** We're playing a 5-phase game. Store the data. Build the abstractions. Design for the endgame.
