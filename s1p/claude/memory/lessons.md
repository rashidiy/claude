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

## Never Overwrite User Files From Git History

**Source:** Illustration fix (2026-03-13)
**What happened:** User spent hours manually fixing 14 illustration PNGs (extracting, removing backgrounds, cleaning edges). Agent restored "originals" from an old git commit, overwrote all uncommitted work. Hours of manual work permanently lost — binary files can't be recovered from git.
**Lesson:** Before touching ANY user file:
1. Check `git status` — uncommitted changes ARE the user's work
2. Never write directly to source files. Always process in `.tmp/`, show the user, copy only after confirmation
3. If you need "originals", backup the CURRENT on-disk files first — not git history versions
4. Binary files (images) can't be recovered if overwritten — extra caution required

## Landing Pages Need Premium Component Libraries, Not Hand-Written Tailwind

**Source:** Landing page rebuild (2026-03-15)
**What happened:** First 2 attempts (hand-written Tailwind components) scored 4-5/10. Looked like generic AI-generated templates. Switching to dark theme + Magic UI components (Particles, ShimmerButton, NumberTicker, BorderBeam, BlurFade, Marquee) + custom fonts (Unbounded + Manrope with Cyrillic) jumped quality to 7.5/10.
**Lesson:** For marketing/landing pages:
1. Never use Inter font — use distinctive display fonts with Cyrillic support (Unbounded, Geologica)
2. Use Magic UI / Aceternity UI copy-paste components instead of hand-writing
3. Dark themes with strategic light/color spills look dramatically more premium than white backgrounds
4. CIS CRM competitors (Bitrix24, Kommo) all use white/light — dark theme is an instant differentiator
5. Magic UI only needs `clsx` + `tailwind-merge` — no shadcn init required, works alongside Ant Design
6. Change framer-motion imports in Magic UI source from `motion/react` to `framer-motion` for compatibility

## Production Audit Checklist — Run Before Every Deploy

**Source:** Production hardening audit (2026-03-17)
**What happened:** Full audit found 10 critical, 14 high issues across both repos. Key gaps: Dockerfile with --reload, container running as root, multi-tenant isolation gap in stats queries, broken frontend build (Zustand store function used as value), dead code paths, CSP allowing unsafe-eval, wrong env var names in docker-compose.
**Lesson:** Before any production deploy, check:
1. `npm run build` / app import — does it even compile?
2. Dockerfile — no --reload, non-root user, proper workers count
3. Multi-tenant — every query filters by company_id (grep for queries without it)
4. Docker-compose — no source mounts, no exposed DB/Redis ports, correct env var names
5. CSP headers — no unsafe-inline/unsafe-eval in production
6. Zustand store functions — always call them `isAdmin()` not `isAdmin`
7. Dependencies — `npm audit` / check for unmaintained packages (python-jose)

## Phone-Based Subqueries Need company_id — Not Just FK Joins

**Source:** QA audit (2026-03-17)
**What happened:** contacts.py `calls_subquery` matched `CallEvent.phone_1/phone_2 == Contact.phone` without `company_id` filter. Since phone matching crosses entities (not a FK join), calls from ALL companies were counted for a contact. This is unlike `Lead.contact_id == contact.id` (FK-safe by UUID) — phone-based joins MUST always include `company_id`.
**Lesson:** Any query that joins by business data (phone number, email) rather than FK (UUID) MUST include explicit `company_id` filter. FK-based queries should still include `company_id` for defense-in-depth.

## Constants Outside Components: Store i18n Keys, Resolve at Render Time

**Source:** QA audit (2026-03-17)
**What happened:** `constants.ts` had 10 label maps (`LEAD_STATUS_LABELS`, `CALL_STATUS_LABELS`, etc.) with hardcoded English. They were used directly in JSX without translation.
**Lesson:** Pattern established: rename `*_LABELS` → `*_KEYS`, store i18n key paths (`statuses.new`, `directions.inbound`), resolve with `tFields(KEY)` at render time. Options arrays use `{ value, key }` instead of `{ value, label }`.

## Soft Delete Needs Full-Stack Thinking

**Source:** Pre-prod audit (2026-03-08)
**What happened:** Soft-deleted contacts left orphaned leads/deals with no indication in UI.
**Lesson:** When adding soft delete: list query filters, related entity display ("deleted" labels), cascade behavior.
