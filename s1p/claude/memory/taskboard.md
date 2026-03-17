# S1P — Taskboard

Persistent task state. Survives session crashes. Update on every status change.

---

## Active Tasks

| ID | Task | Status | Repo | Blocked By | Notes |
|---|---|---|---|---|---|
| T81 | Sipuni one-click setup — contract + backend (client, model, migration, router) + frontend (types, api, page, i18n) | doing | both | none | All code written, needs commit + QA |

## Completed (Last 15)

| ID | Task | Completed | Branch |
| T79 | QA audit fix — backend: test isolation, PII masking, SSRF hardening, race condition fix, operator enumeration, UUID validation, webhook secrets, bulk contact dedup, stale tests cleanup | 2026-03-18 | dev (direct) |
| T80 | QA audit fix — frontend: remove sample analytics data, 59 console.error calls, toast consistency, i18n fixes, type safety, duplicate constants | 2026-03-18 | dev (direct) |
|---|---|---|---|
| T77 | QA audit fix — backend: cross-tenant leaks (contacts calls_subquery, contact activity, dashboard, auto-link), unbounded queries (outcomes summary, pipeline summary), defense-in-depth filters | 2026-03-17 | fix/qa-audit-findings → dev |
| T78 | QA audit fix — frontend: i18n constants (10 label maps → key maps, 6 option arrays), outcome options, ProtectedRoute on 4 admin settings pages, "answered" i18n key | 2026-03-17 | fix/qa-audit-findings → dev |
| T75 | Production hardening — backend (Dockerfile, multi-tenant fix, JWT validation, lifespan, Redis warnings, docker-compose.prod) | 2026-03-17 | dev (direct, 8 commits) |
| T76 | Production hardening — frontend (build fix, dead code removal, docker env fix, CSP fix, nginx fix, deps audit) | 2026-03-17 | dev (direct, 7 commits) |
| T60-T74 | Telegram Bot V2 | 2026-03-15 | feature/telegram-bot-v2 |
| T53 | Landing page — dark theme, Magic UI, Unbounded font | 2026-03-15 | feat/landing-page |
| T60 | Extend TelegramBotConfig model + migration | 2026-03-15 | feature/telegram-bot-v2 |
| T61 | Pyrogram dependency + config | 2026-03-15 | feature/telegram-bot-v2 |
| T62 | Pyrogram userbot service | 2026-03-15 | feature/telegram-bot-v2 |
| T63 | Group setup endpoints + topic creation | 2026-03-15 | feature/telegram-bot-v2 |
| T64 | Frontend setup wizard UI | 2026-03-15 | feature/telegram-bot-v2 |
| T65 | i18n notification templates | 2026-03-15 | feature/telegram-bot-v2 |
| T66 | TelegramService refactor (topics + i18n) | 2026-03-15 | feature/telegram-bot-v2 |
| T67 | Recording signed URLs + dual-auth endpoint | 2026-03-15 | feature/telegram-bot-v2 |
| T68 | Interactive callback handlers | 2026-03-15 | feature/telegram-bot-v2 |
| T69 | Bot commands (/today, /search, /myleads) | 2026-03-15 | feature/telegram-bot-v2 |
| T70 | Daily digest scheduler | 2026-03-15 | feature/telegram-bot-v2 |
| T71 | Smart grouping (Redis dedup) | 2026-03-15 | feature/telegram-bot-v2 |
| T72 | DM notification prefs + sending | 2026-03-15 | feature/telegram-bot-v2 |
| T73 | DM preferences frontend (in T64) | 2026-03-15 | feature/telegram-bot-v2 |
| T74 | Mini App auth endpoint | 2026-03-15 | feature/telegram-bot-v2 |
| T52 | Webhooks settings page | 2026-03-15 | feat/api-keys-page |
| T51 | API Keys settings page | 2026-03-15 | feat/api-keys-page |
| T50 | Custom Fields settings page | 2026-03-15 | feat/api-keys-page |
| T49 | Frontend shared file prep (types, API, i18n, sidebar, settings hub) | 2026-03-15 | dev (direct) |
| T48 | Public API cursor pagination | 2026-03-15 | feat/public-api-cursor-pagination |
| T47 | Contacts operator scoping + Admin invite email | 2026-03-15 | feat/admin-invite-email |
| T46 | Telegram cleanup: consolidate services, remove dead webhook, fix schemas | 2026-03-14 | fix/telegram-cleanup |
| T45 | Fix Telegram reg_ webhook handler + User avatar system | 2026-03-13 | main (direct) |
| T43 | Telegram Auth Overhaul (backend + frontend) | 2026-03-12 | feature/telegram-auth-overhaul + feature/telegram-auth-frontend |
| T42 | Frontend Production-Ready Overhaul | 2026-03-12 | dev (direct) |
| T34 | Fix deals.py data.stage AttributeError | 2026-03-08 | verified done |
| T35 | Fix custom fields enum case mismatch | 2026-03-08 | verified done |
| T36 | Add created_by migration to leads | 2026-03-08 | verified done |
| T37 | Fix notes.entity_id type mismatch in tests | 2026-03-08 | verified done |
| T38 | Fix sipuni test fixtures int32 overflow | 2026-03-08 | verified done |
| T39 | Remove duplicate /settings route | 2026-03-08 | verified done |
| T40 | Fix ESLint rule not found | 2026-03-08 | verified done |
| T41 | Fix statistics.py CallEvent import | 2026-03-08 | direct fix |
| T33 | Enable webhook IP whitelist for Sipuni | 2026-03-08 | fix/pre-prod-audit-4 |
| T32 | Handle soft-deleted contacts — show "(deleted)" label | 2026-03-08 | fix/pre-prod-audit-4 |
| T31 | Add /health endpoint (DB + Redis check) | 2026-03-08 | fix/pre-prod-audit-4 |
| T30 | Fix refresh token flow + DealResponse schema | 2026-03-08 | fix/pre-prod-audit-3 |
| T29 | Analytics sample data — Phase 2 TODOs + banner | 2026-03-08 | fix/pre-prod-audit-3 |
| T28 | httpOnly cookie auth (backend + frontend) | 2026-03-08 | fix/pre-prod-audit-3 |
| T27 | Add created_by column to Lead and Deal | 2026-03-08 | fix/pre-prod-audit-3 |

---

## Status Legend

- `todo` — not yet started
- `doing` — actively being worked on
- `blocked` — waiting on another task (see Blocked By column)
- `done` — merged into `dev` and smoke test passed
