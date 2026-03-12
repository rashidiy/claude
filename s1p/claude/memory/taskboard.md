# S1P — Taskboard

Persistent task state. Survives session crashes. Update on every status change.

---

## Active Tasks

| ID | Task | Status | Repo | Blocked By | Notes |
|---|---|---|---|---|---|

_No active tasks._

## Completed (Last 15)

| ID | Task | Completed | Branch |
|---|---|---|---|
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
