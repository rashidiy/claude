# S1P — Architecture Reference

Locked technical decisions and implementation specs. For agents writing code.

---

## Locked Decisions

| Decision | Choice | Reason |
|---|---|---|
| Custom fields storage | JSONB column per entity | Already using JSONB, PostgreSQL 16 GIN indexing |
| External CRM integration | Push (webhooks) + Pull (public API) | Platform approach |
| Telegram bot model | Shared bot, webhook-based, inside FastAPI monolith | Simpler onboarding, fast to ship. Revisit at 1000+ companies. |
| Public API auth | API key per company (hashed with bcrypt) | Simple, standard for B2B SaaS |
| CRM entity model | Fixed core + custom fields | Balance of structure and flexibility |
| i18n frontend | next-intl, JSON locale files | Standard Next.js approach |
| i18n backend | Error codes (frontend translates), Jinja2 email templates per language | Don't translate DB content |
| Telegram bot library | aiogram (async-native) | Fits FastAPI. Not python-telegram-bot. |
| Public API versioning | URL prefix: /api/v1/public/... | Not headers — headers confuse junior devs |
| Custom field types | Typed: text, number, dropdown, date, boolean | All 5 from day one |
| Pipeline customization | Stages only (Phase 1) | No automations yet |

---

## Public API Design Rules

1. Version in URL: `/api/v1/public/...`
2. Cursor-based pagination: `?cursor=xxx&limit=20` (not offset — offset breaks when records are added between pages)
3. Date range filter on EVERY list endpoint: `?from=2026-01-01&to=2026-03-01`
4. Consistent envelope: `{ "data": [...], "cursor": "next_cursor", "has_more": true }`
5. Error format: `{ "error": { "code": "RATE_LIMITED", "message": "...", "retry_after": 30 } }`
6. Rate limit headers on every response: `X-RateLimit-Remaining`, `X-RateLimit-Reset`
7. Idempotency keys on POST/PUT: clients can safely retry without duplicating data
8. Never return internal IDs that expose sequence — use UUIDs in public API even if internal is int

---

## Webhook Design Rules

1. Include type field: `{ "type": "call.completed", "data": {...} }`
2. Include timestamp: `{ "timestamp": "2026-03-03T12:00:00Z" }`
3. Include the full object, not just an ID (clients shouldn't need to call back)
4. Sign with HMAC-SHA256 in `X-Webhook-Signature` header
5. Retry: 3 attempts at 10s, 60s, 300s — then mark as failed
6. Webhook log: store every delivery attempt with status code and response body
7. Webhook test button in UI: "Send test event" for client verification
8. Allow multiple webhook URLs per company

---

## Custom Fields Implementation

```
Model:
  CustomFieldDefinition:
    id, company_id, entity_type (contact|lead|deal|task),
    field_name, field_type (text|number|dropdown|date|boolean),
    options (JSONB — for dropdown choices),
    required (bool), sort_order (int)

  Entity (Contact, Lead, etc.):
    ... existing fields ...
    custom_fields: JSONB  →  { "field_id_1": "value", "field_id_2": 42 }

Rules:
  - Max 20 custom fields per entity type per company (prevent abuse)
  - field_name must be unique per entity_type per company
  - Validate types on write (don't let text go into number field)
  - Custom fields returned in API responses as nested object
  - Public API exposes custom field definitions for external mapping
  - Don't index individual custom field values in Phase 1 — JSONB containment
    queries are fast enough for < 100K records per company
```

---

## Telegram Bot Architecture

**Decision:** Inside FastAPI monolith (Phase 1).

- Webhook endpoint: `POST /api/v1/webhooks/telegram`
- Background task for message sending (don't block webhook response)
- Company-to-chat mapping in DB: `TelegramConfig` model (company_id, chat_id, settings)
- ~45,000 messages/day at 300 clients = ~0.5 msg/sec — trivially handled
- Revisit at 1,000+ companies or if bot latency affects API response times

---

## i18n Architecture

**Frontend (Next.js):**
- next-intl with JSON locale files: `/messages/en.json`, `/messages/ru.json`, `/messages/uz.json`
- Locale detection: browser preference → user setting → company default
- All UI strings through `t('key')`

**Backend (FastAPI):**
- API error messages: return error codes, frontend translates
- Email templates: Jinja2 per language (`/templates/email/{lang}/...`)
- Do NOT translate database content (entity names, custom field labels)
- User model has `language` field (en|ru|uz), default `ru`

Priority: Russian first (90% of initial users), Uzbek second, English third.

---

## Security Non-Negotiables

| Requirement | Implementation |
|---|---|
| Tenant isolation in every query | `company_id` filter on every DB query, automated tests verify |
| Call recordings are sensitive | Stream from provider, never store locally in Phase 1. HTTPS only. |
| API keys must be hashed | bcrypt, show full key only once at creation |
| Rate limiting on public API | Token bucket per API key, 100 req/min default |
| Webhook signatures | HMAC-SHA256 with per-company secret |
| No PII in logs | Sanitize phone numbers, emails from application logs |

---

## Tech Debt Budget

### Allowed in Phase 1
- Frontend polish (CSS, animations, responsive edge cases)
- Test coverage below 80%
- No automated deployment pipeline (manual deploy fine for < 100 clients)
- Plain text email templates (not styled HTML)
- Error messages in English only
- No rate limiting on internal API

### NOT Allowed in Phase 1 (zero tolerance)
- Multi-tenant isolation gaps
- Auth/JWT security shortcuts
- Bad data model design (custom fields JSONB must be right from day one)
- Breaking public API contract
- Unreliable Telegram bot notifications
