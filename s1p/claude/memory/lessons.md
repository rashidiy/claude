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

## CSP Blocks Telegram Mini App SDK Injection

**Source:** Mini App launch failure (2026-03-22)
**What happened:** Mini App showed "Not authenticated" because `window.Telegram.WebApp` was undefined. Root cause: `X-Frame-Options: DENY` and `script-src` without `https://telegram.org` blocked Telegram's SDK injection. Also `frame-ancestors 'none'` prevented the WebView.
**Lesson:** Telegram Mini App routes need relaxed CSP:
1. No `X-Frame-Options` header
2. `script-src` must include `https://telegram.org`
3. `frame-ancestors` must allow `https://web.telegram.org https://*.telegram.org`
4. Use separate Next.js `headers()` rules for `/miniapp` vs other routes
5. Next.js `/:path*` catch-all overrides specific rules — use negative lookahead `/((?!miniapp).*)`

## Next.js Middleware Blocks Routes on Bare Domain

**Source:** Mini App showing landing page (2026-03-22)
**What happened:** Middleware redirected all non-landing routes on bare domain (`s1p.uz`) to `/`. Mini App at `/miniapp` was redirected to the landing page.
**Lesson:** When adding new public routes that should work on the bare domain, add them to the middleware whitelist alongside `/login`, `/register`, etc.

## Mini App ≠ Mini CRM — Design for 10-Second Sessions

**Source:** Mini App rewrite (2026-03-23)
**What happened:** First mini app version was a read-only data browser (contacts list, leads list, deals list, calls list — separate tabs). Users had no reason to come back because they couldn't DO anything.
**Lesson:** Telegram Mini Apps need to be action-oriented:
1. Show what needs attention (missed calls, new leads) — not dashboards with numbers
2. Let users act (change lead status, add notes, call back, create contacts) — not just view
3. DM notifications must deep-link to mini app pages with action buttons — not dead-end text
4. Use Telegram SDK features (MainButton, showPopup, haptics) — not custom web UI
5. Role-based views (operator vs manager) — not one-size-fits-all
6. Combined views (Pipeline = leads + deals toggle) — not separate tabs for related data
7. 4 tabs max — not 6. Profile goes behind avatar tap.

## Soft Delete Needs Full-Stack Thinking

**Source:** Pre-prod audit (2026-03-08)
**What happened:** Soft-deleted contacts left orphaned leads/deals with no indication in UI.
**Lesson:** When adding soft delete: list query filters, related entity display ("deleted" labels), cascade behavior.

## Webhook Event 1 Races With Call Endpoints

**Source:** Sipuni events 1-4 support (2026-03-19)
**What happened:** Added handling for Sipuni event 1 (call start). When user clicks "Call", Sipuni fires event 1 webhook faster than the API response returns. The webhook handler created the CallEvent row first, then the call endpoint tried to create the same row → 409 Conflict "Resource already exists".
**Lesson:** When adding new webhook event handlers:
1. Check ALL code paths that create CallEvent rows — not just the webhook handler
2. Call endpoints (`/calls/sipuni/external`, `/number`, `/tree`) must use upsert logic (check existing before insert)
3. Always test the real call flow after changes, not just unit tests — race conditions only manifest in async production flows
4. Before running `alembic upgrade head` on a running DB, verify new code paths don't break existing functionality

## Worktree Isolation Does NOT Isolate Sub-Repos

**Source:** T86/T88/T89 parallel agents (2026-03-24)
**What happened:** Launched 3 agents with `isolation: "worktree"` expecting each to work in isolated copies. But worktrees are created from the **claude repo** (`/home/rashidiy/claude/`), and `repos/` is git-ignored. The worktree doesn't contain copies of `s1p-frontend` or `s1p-backend`. When agents `cd repos/s1p-frontend`, they hit the **original shared repo**. T89 checked out its feature branch in the user's main frontend directory, overwriting their work. Bug fixes applied to the main repo got overwritten by agent operations on the same files.
**Lesson:** For this project structure (parent repo + git-ignored sub-repos):
1. `isolation: "worktree"` only isolates the parent claude repo files — NOT the application repos
2. For true isolation, agents must create worktrees **inside** each sub-repo: `cd repos/s1p-frontend && git worktree add ...`
3. Never let agents `git checkout` branches in the main working directory — they must work in their own worktree path
4. If agents can't be truly isolated, run them **sequentially** instead of in parallel, or at minimum ensure they never touch files the user is actively editing
5. Always verify the user's working directory branch hasn't changed after agent work completes
