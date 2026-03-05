# Technical Decisions

**Source:** CTO Brain v100
**Last updated:** 2026-03-03

---

## Locked Decisions

| Decision | Choice | Reason |
|---|---|---|
| Custom fields storage | JSONB column per entity | Already using JSONB in codebase, PostgreSQL 16 GIN indexing, fast to implement |
| External CRM integration | Push (webhooks) + Pull (public API) | Platform approach, not integration marketplace |
| Telegram bot model | Shared bot, webhook-based | Simpler onboarding for 300 clients |
| Public API auth | API key per company | Simple, standard for B2B SaaS, OAuth2 later if needed |
| CRM entity model | Fixed core + custom fields | Balance of structure and flexibility |

---

## Open Decisions (Awaiting Discussion)

| # | Decision | Options | Status |
|---|---|---|---|
| 1 | i18n architecture | Frontend-only vs full-stack | Open |
| 2 | Telegram bot deployment | Inside FastAPI vs separate service | Open |
| 3 | Public API versioning | URL prefix vs header | Open |
| 4 | Custom field types | Text-only vs typed (text, number, dropdown, date, boolean) | Open |
| 5 | Pipeline customization depth | Stages only vs stages + automations | Open |

---

## Architecture Wisdom — Patterns That Save Months

### Public API Design — Get It Right First Time
The public API is a **hard door decision**. Once clients integrate, you can't change it without breaking them.

```
Rules:
1. Version in URL: /api/v1/public/... (not headers — headers confuse junior devs)
2. Use cursor-based pagination, not offset: ?cursor=xxx&limit=20
   (offset breaks when new records are added between pages)
3. Filter by date range on EVERY list endpoint: ?from=2026-01-01&to=2026-03-01
   (clients will need this immediately, don't make them pull everything)
4. Return consistent envelope: { "data": [...], "cursor": "next_cursor", "has_more": true }
5. Error format: { "error": { "code": "RATE_LIMITED", "message": "...", "retry_after": 30 } }
6. Rate limit headers on every response: X-RateLimit-Remaining, X-RateLimit-Reset
7. Idempotency keys on POST/PUT: clients can safely retry without duplicating data
8. Never return internal IDs that expose sequence (use UUIDs in public API even if internal is int)
```

### Webhook Design — Lessons From Every SaaS That Got It Wrong
```
Rules:
1. Include a "type" field: { "type": "call.completed", "data": {...} }
2. Include a timestamp: { "timestamp": "2026-03-03T12:00:00Z" }
3. Include the full object, not just an ID (clients shouldn't need to call back to get data)
4. Sign with HMAC-SHA256 in X-Webhook-Signature header
5. Retry: 3 attempts at 10s, 60s, 300s — then mark as failed
6. Webhook log: store every delivery attempt with status code and response body
   (clients WILL ask "did you send it?" — you need proof)
7. Webhook test button in UI: "Send test event" so clients can verify integration
8. Allow multiple webhook URLs per company (they might send to CRM + analytics + Zapier)
```

### Custom Fields — Implementation Guardrails
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
  - Custom fields are returned in API responses as a nested object
  - Public API exposes custom field definitions so external systems can map them
  - Don't index individual custom field values in Phase 1 — JSONB containment queries are fast enough for < 100K records per company
```

### Telegram Bot Architecture — Decision
```
Option A: Inside FastAPI (monolith)
  + Simple deployment (one container)
  + Shares DB connection pool
  + Shares auth/permission logic
  - Telegram webhook processing blocks web request workers
  - If bot goes down, API goes down (and vice versa)

Option B: Separate service
  + Independent scaling
  + Bot can crash without affecting CRM API
  + Cleaner codebase separation
  - Needs its own DB connection
  - Inter-service communication complexity
  - More Docker containers to manage

DECISION: Option A for Phase 1 (monolith). Reason:
  - Speed. We ship in 5 days. Separate service adds 1-2 days of infra work.
  - 300 clients x ~150 calls/day = ~45,000 Telegram messages/day = ~0.5 msg/sec.
    This is trivially handled by the existing FastAPI workers.
  - Revisit at 1,000+ companies or if bot processing latency affects API response times.

Implementation:
  - Telegram webhook endpoint: POST /api/v1/webhooks/telegram
  - Background task for message sending (don't block the webhook response)
  - Company-to-chat mapping in DB: TelegramConfig model (company_id, chat_id, settings)
```

### i18n Architecture — Decision
```
Frontend (Next.js):
  - next-intl or react-i18next
  - JSON locale files: /messages/en.json, /messages/ru.json, /messages/uz.json
  - Locale detection: browser preference → user setting → company default
  - All UI strings through translation function t('key')

Backend (FastAPI):
  - API error messages: return error codes, frontend translates
  - Email templates: Jinja2 templates per language (/templates/email/{lang}/...)
  - Do NOT translate database content (entity names, custom field labels)
  - User model gets a 'language' field (en|ru|uz), default 'ru'

Phase 1 priority: Russian first (90% of initial users), Uzbek second, English third.
Don't block launch on Uzbek translations — ship Russian, add Uzbek within 2 weeks.
```
