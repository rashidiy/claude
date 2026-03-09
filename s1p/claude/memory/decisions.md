# S1P — Decisions

Past decisions. Read before every session. If it's been decided here, don't re-ask.

---

| Date | Question | Answer | Context |
|---|---|---|---|
| 2026-03-06 | Custom field types: text-only or typed? | **Typed** — text, number, dropdown, date, boolean. All 5 from day one. | |
| 2026-03-06 | i18n: which languages at launch? | **All 3** — Russian, English, Uzbek simultaneously. | |
| 2026-03-06 | Telegram bot library? | **aiogram** (async-native, fits FastAPI). Not python-telegram-bot. | |
| 2026-03-06 | Public API + webhooks in initial push? | **Defer.** Focus on core CRM + Telegram + security first. | |
| 2026-03-06 | UI component system? | **Ant Design only.** Remove all shadcn/radix. Use Ant directly everywhere. | |
| 2026-03-06 | Telegram bot deployment? | **Inside FastAPI monolith.** Webhook-based, not polling. Revisit at 1000+ companies. | |
| 2026-03-06 | Pipeline customization depth? | **Stages only** for Phase 1. No automations yet. | |
| 2026-03-08 | Soft delete cascade when contact deleted? | **No cascade.** Show "(deleted)" label on orphaned leads/deals. | |
| 2026-03-08 | Webhook signature validation (HMAC)? | **No HMAC.** Sipuni doesn't sign. IP whitelist is the only protection. | |
| 2026-03-08 | Frontend tests before launch? | **Skip.** Backend has 31 test files. Frontend tests deferred. | |
| 2026-03-08 | Structured logging? | **Skip.** Basic Python logging fine for Phase 1. | |
| 2026-03-08 | Real-time features (WebSocket/SSE)? | **Later.** Telegram bot handles operator notifications for now. | |
| 2026-03-08 | Payment/billing integration? | **Later.** Manual contracts fine for Phase 1. | |
