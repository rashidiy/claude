# S1P — Cross-Cutting Lessons

Lessons that affect ALL agents. Read before every session.

For agent-specific lessons, see `claude/agents/[role]/lessons.md`.

---

## Cross-Repo Changes Need a Contract

**Source:** Auth flow refactor (2026-03-08)
**What happened:** httpOnly cookie auth broke refresh flow — backend required body param while frontend stopped sending it.
**Lesson:** When a change spans backend + frontend, write the exact request/response contract and give it to BOTH agents.

## Tell QA About Known Exceptions

**Source:** QA cycles (2026-03-08)
**What happened:** Intentionally deferred Alembic migrations but didn't tell QA. QA flagged as FAIL.
**Lesson:** When sending to QA, list what was intentionally deferred.

## Constants Outside Components Need Key-Based i18n

**Source:** Frontend overhaul QA (2026-03-12)
**What happened:** Constants like `OUTCOME_OPTIONS` and `NOTIFICATION_TOGGLES` defined outside React components had hardcoded English labels. `useTranslations` can only be called inside components.
**Lesson:** When i18n-ing pages, store translation **keys** (not labels) in constants. Resolve at render time: `{ value: 'interested', key: 'interested' }` then `t(option.key)`.

## Worktree Agents Must Actually Use Worktrees — Don't Block the User

**Source:** Frontend i18n wiring (2026-03-12)
**What happened:** Launched 3 agents with `isolation: "worktree"` but they wrote to the main working directory instead. User was blocked for 30+ minutes waiting — couldn't do any other work because agents were writing to the same directory.
**Lesson:** When parallelizing with agents:
1. Verify agents actually commit to their worktree branches (check `git log` in worktree dir).
2. If agents write to main dir instead, user is blocked — defeats the purpose of parallel work.
3. Prefer separate tmux sessions or true worktree isolation so the user can keep working on other things in the main repo while agents run.
4. Never make the user wait idle — if agents take >5 min, something is wrong with the approach.

## Pre-Add All i18n Keys Before Parallelizing Page Wiring

**Source:** Frontend overhaul i18n pass (2026-03-12)
**What happened:** Spawned 3 agents to wire 22 pages with useTranslations. By adding all 629 keys to JSON files FIRST (in one commit), agents only touched .tsx files — zero merge conflicts on shared JSON files.
**Lesson:** For parallel i18n work, add all translation keys to the shared JSON files in advance. Agents should never modify message files.

## Soft Delete Needs Full-Stack Thinking

**Source:** Pre-prod audit (2026-03-08)
**What happened:** Soft-deleted contacts left orphaned leads/deals with no indication in UI.
**Lesson:** When adding soft delete: list query filters, related entity display ("deleted" labels), cascade behavior.
