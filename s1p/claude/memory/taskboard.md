# S1P — Taskboard

Persistent task state. Survives session crashes. Update on every status change.

---

## Active Tasks

| ID | Task | Status | Repo | Blocked By | Notes |
|---|---|---|---|---|---|
| T83 | Mini App polish — dark theme sync, styling, i18n, detail screens | todo | frontend | none | Foundation working, needs UI refinement |

## Completed (Last 15)

| ID | Task | Completed | Branch |
| T82 | Telegram improvements — notifications, Mini App, DM, bot polish | 2026-03-22 | feature/telegram-improvements → dev |
| T81 | Sipuni one-click setup — contract + backend (client, model, migration, router) + frontend (types, api, page, i18n) | 2026-03-21 | dev (direct) |
| T79 | QA audit fix — backend: test isolation, PII masking, SSRF hardening, race condition fix, operator enumeration, UUID validation, webhook secrets, bulk contact dedup, stale tests cleanup | 2026-03-18 | dev (direct) |
| T80 | QA audit fix — frontend: remove sample analytics data, 59 console.error calls, toast consistency, i18n fixes, type safety, duplicate constants | 2026-03-18 | dev (direct) |
|---|---|---|---|
| T77 | QA audit fix — backend: cross-tenant leaks, unbounded queries, defense-in-depth filters | 2026-03-17 | fix/qa-audit-findings → dev |
| T78 | QA audit fix — frontend: i18n constants, ProtectedRoute, outcome options | 2026-03-17 | fix/qa-audit-findings → dev |
| T75 | Production hardening — backend | 2026-03-17 | dev (direct, 8 commits) |
| T76 | Production hardening — frontend | 2026-03-17 | dev (direct, 7 commits) |
| T60-T74 | Telegram Bot V2 | 2026-03-15 | feature/telegram-bot-v2 |
| T53 | Landing page — dark theme, Magic UI, Unbounded font | 2026-03-15 | feat/landing-page |
| T52 | Webhooks settings page | 2026-03-15 | feat/api-keys-page |
| T51 | API Keys settings page | 2026-03-15 | feat/api-keys-page |
| T50 | Custom Fields settings page | 2026-03-15 | feat/api-keys-page |
| T49 | Frontend shared file prep | 2026-03-15 | dev (direct) |
| T48 | Public API cursor pagination | 2026-03-15 | feat/public-api-cursor-pagination |

---

## T82 Details (completed 2026-03-22)

Massive Telegram session — 15+ commits across both repos:

**Notifications:**
- Compact scannable format (not data dumps)
- Emoji on all buttons
- 3 topics instead of 5 (merged Missed into Calls)
- "Create Contact" button for unknown callers
- Phone numbers as tappable tel: links
- Bot reactions (✅ handled, 👤 lead assigned)
- Improved daily digest

**Bot:**
- /help command i18n (ru/en/uz) with Mini App button (DM only)
- /start i18n based on group config language
- /today fixed parse_mode (was showing raw MarkdownV2)
- Bot description + short description in 3 languages
- Localized command descriptions (ru/en/uz)
- Mini App menu button (📋 CRM)
- OTP message i18n with tap-to-copy monospace

**Mini App:**
- 6 screens: dashboard, contacts, leads, deals, calls, profile
- Bottom tab bar (icons only)
- Company picker (for users in multiple companies)
- Auth via Telegram initData HMAC → JWT
- CSP fix to allow Telegram SDK
- Middleware fix to allow /miniapp on bare domain

**DM Notifications:**
- Wired _send_dm_notifications (was dead code)
- Scoped to assignee (not broadcast)
- Triggered on: call completed, call missed, new lead, deal stage change

**QA:**
- UUID validation on all callback actions
- Phone pattern validation on create contact
- tel: URL sanitization
- Proper API types (no any)
- Error states on all pages
- is_suspended filter on /companies endpoint

**Infrastructure:**
- FRONTEND_URL added to docker-compose.yml
- Settings renamed "Telegram Bot" → "Telegram Group"
- Sipuni contract webhook URL fixed

---

## Status Legend

- `todo` — not yet started
- `doing` — actively being worked on
- `blocked` — waiting on another task
- `done` — merged into `dev` and smoke test passed
