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
| 2026-03-12 | Company user auth method? | **Telegram OTP only.** Email+password removed for company users. Owner keeps email+password. | Deep link registration, prefill from TG profile |
| 2026-03-12 | telegram_user_id uniqueness? | **Per-company** (not global). One TG account can be in multiple companies. | UniqueConstraint('company_id', 'telegram_user_id') |
| 2026-03-12 | Registration flow? | **Deep link** via `/start inv_{TOKEN_ID_HEX}`. No more `/register` bot command. | Bot captures TG profile + avatar, creates challenge |
| 2026-03-12 | Owner admin invite? | **InviteToken flow** (same as company invite). No email+temp password. | Returns deep_link + copyable message |
| 2026-03-15 | Telegram V2 userbot library? | **Pyrogram** (async, MTProto). Inside monolith, single dedicated account. | FloodWait handling, asyncio.Lock serialization |
| 2026-03-15 | Telegram V2 group structure? | **5 forum topics**: Calls, Missed, Leads, Deals, General. Automated via Pyrogram, manual fallback. | Topic names localized per company language |
| 2026-03-15 | Telegram V2 recording delivery? | **Signed URL** (HMAC-SHA256, 24h expiry) + JWT cookie fallback. No file upload/storage. | Dual-auth: signed token for Telegram, JWT for CRM |
| 2026-03-15 | Telegram V2 callback data format? | **Shortened prefixes** (mh/al/cb/cc) to fit 64-byte limit. | Legacy prefixes preserved for backward compat |
| 2026-03-15 | Telegram V2 smart grouping? | **Redis-based**, 30min window, edit existing message with (x{count}). | Redis down → graceful fallback (always send new) |
| 2026-03-15 | Telegram V2 Mini App auth? | **initData HMAC validation → JWT**. Maps telegram_user_id to CRM user. | Foundation only, Mini App frontend is separate work |
| 2026-03-18 | Sipuni setup architecture? | **SipuniAsyncClient in-process** (httpx, no Playwright/sidecar). Password used once, never stored. Background task for setup. | Same UX pattern as Telegram one-click |
| 2026-03-18 | Sipuni setup model? | **SipuniSetupConfig** — separate from Company.provider_config. Tracks setup_status + method + services_enabled. | provider_config stores credentials, this tracks setup state |
| 2026-03-19 | Sipuni webhook events scope? | **All 4 events** (1=start, 2=hangup, 3=answer, 4=transfer) via HTTP webhook. No WebSocket in Phase 1. | Single row per call, updated by each event. Terminal state protection prevents out-of-order overwrites. |
| 2026-03-19 | Sipuni event notifications? | Event 2 → Telegram + outbound webhook. Events 1,4 → outbound webhook only. Event 3 → no notifications. | Telegram on every event would be too noisy. |
| 2026-03-19 | SIP extension uniqueness? | **No unique constraint** — multiple users can share one extension. | Phase 1 decision. |
| 2026-03-19 | Operator call popup behavior? | **Auto-fill for operators** (disabled dropdown), **dropdown for admins/managers** from Sipuni operators API. | Operators always use their assigned extension. |
| 2026-03-19 | Webhook phone overwrite? | **Skip phone_1/phone_2 in webhook update** if existing record already has values. | Prevents SIP extension (201) overwriting real phone numbers set by call endpoint. |
