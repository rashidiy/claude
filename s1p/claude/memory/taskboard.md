# S1P — Taskboard

Persistent task state. Survives session crashes. Update on every status change.

---

## Active Tasks

| ID | Task | Status | Repo | Blocked By | Notes |
|---|---|---|---|---|---|
| T115 | Event 3 should set state=ANSWER (safety net for missing event 2) | todo | Backend | none | P1 — see sipuni-webhook-events study |
| T116 | External call answer_timestamp validation (broken for /external calls) | todo | Backend | none | P2 — fake answer_ts on NOANSWER, < start on ANSWER |
| T117 | Orphan call cleanup — mark NULL-state calls >5min as FAILED | todo | Backend | none | P3 — calls #92,93,98 stuck with no state |
| T118 | Outbound call notification shows "Unknown" — should show destination phone | todo | Backend | none | P2 — phone_1=SIP ext, need to show phone_2 |
| T119 | Operator not resolved from last_called SIP extension | todo | Backend | none | P3 — last_called=201 not matched to user |

## Completed (Last 15)

| ID | Task | Completed | Branch |
| T114 | Sipuni webhook event investigation — 9 scenarios tested, findings documented | 2026-03-24 | N/A (research) |
| T113 | Call duration fix — compute from timestamps when billing_sec missing | 2026-03-24 | dev (direct) |
| T112 | Recording playback fix — use residential proxy for Sipuni downloads | 2026-03-24 | dev (direct) |
| T111b | Mini App grouped call detail — iPhone-style with individual recordings | 2026-03-24 | dev (direct) |
| T87 | Currency cleanup — remove EUR, UZS default + formatting | 2026-03-24 | dev (direct) |
| T88 | Lead status change works in card view | 2026-03-24 | dev (direct) |
| T89 | Consistent actions between card and table views | 2026-03-24 | dev (direct) |
| T90 | Mini App contact create page with phone prefill | 2026-03-24 | dev (direct) |
| T91 | Mini App lead create form on pipeline page | 2026-03-24 | dev (direct) |
| T94 | Call recording as sendAudio in Telegram | 2026-03-24 | dev (direct) |
| T95 | Escalation message templates i18n (3 tiers, ru/en/uz) | 2026-03-24 | dev (direct) |
| T99 | Quiet hours timezone support (offset, default +5) | 2026-03-24 | dev (direct) |
| T100 | DM deep links use signed startapp format | 2026-03-24 | dev (direct) |
| T104 | Quiet hours time picker UI | 2026-03-24 | dev (direct) |
| T105 | Company picker roles translated | 2026-03-24 | dev (direct) |
| T106 | Dialer max phone length (15 digits) | 2026-03-24 | dev (direct) |
| T107 | React error boundary for Mini App | 2026-03-24 | dev (direct) |
| T108 | Recording processing note when URL not yet available | 2026-03-24 | dev (direct) |
| T109 | Daily digest topic routing fallback | 2026-03-24 | dev (direct) |

## Previously Completed

| ID | Task | Completed | Branch |
| T110 | Mark Handled in DM — skip group message edit | 2026-03-24 | dev (direct) |
| T111 | /search MarkdownV2 escaping | 2026-03-24 | dev (direct) |
| T86 | Invite code monospace formatting | 2026-03-24 | dev (direct) |
| T92 | Deal detail startapp deep link fix | 2026-03-24 | dev (direct) |
| T93 | Lead detail startapp deep link fix | 2026-03-24 | dev (direct) |
| T96 | Remove dead cb: callback, replace with Mini App startapp URL | 2026-03-24 | dev (direct) |
| T97 | DM notification button labels i18n | 2026-03-24 | dev (direct) |
| T98 | Missed call DM broadcast fix (operator_id=None) | 2026-03-24 | dev (direct) |
| T101 | Bot webhook handlers — 17 hardcoded English strings → i18n | 2026-03-24 | dev (direct) |
| T102 | Mini App auth error — hide debug info from users | 2026-03-24 | dev (direct) |
| T103 | User invite message i18n (was hardcoded Russian) | 2026-03-24 | dev (direct) |
| T84 | Mini App complete rewrite | 2026-03-23 | dev (direct) |
| T83 | Mini App complete redesign | 2026-03-22 | dev (direct) |
| T82 | Telegram improvements — notifications, Mini App, DM, bot polish | 2026-03-22 | feature/telegram-improvements → dev |
| T81 | Sipuni one-click setup | 2026-03-21 | dev (direct) |

---

## Status Legend

- `todo` — not yet started
- `doing` — actively being worked on
- `blocked` — waiting on another task
- `done` — merged into `dev` and smoke test passed
