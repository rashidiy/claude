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

## Migrations Must Not Flag Old Data for Background Tasks

**Source:** Escalation spam on Logistic group creation (2026-03-24)
**What happened:** Migration set `needs_callback = true` on all existing inbound missed calls. When the Telegram group was created, the escalation scheduler found 30+ old calls and tried to send messages for all of them at once → Telegram flood control, spam in group.
**Lesson:**
1. Never retroactively flag old data for time-sensitive background tasks (escalation, notifications)
2. Only new records going forward should trigger background tasks
3. If you must backfill, add a `created_at` check in the task to skip old records

## Startapp Params Must Be Handled End-to-End

**Source:** Mini App validation error from Telegram callback buttons (2026-03-24)
**What happened:** Backend generated signed base64 startapp params for Mini App buttons. But Mini App frontend still expected raw company_id in `start_param`. Clicking any button showed validation error. Also, dialer read `number` param but startapp sent `dial`.
**Lesson:**
1. When changing a parameter format (raw → encoded), update BOTH producer (backend) and consumer (frontend)
2. Test the full flow: button in Telegram → Mini App opens → correct page with prefilled data
3. Keep backward compatibility — decode new format but still accept old raw format

## Never Manually Assign Auto-Increment PKs — Use the Sequence

**Source:** Logistic company 500 errors (2026-03-24)
**What happened:** `next_call_number()` computed `MAX(id) WHERE company_id = X` to assign "company-scoped" IDs, but `id` is a **global** primary key. Company A had ids 1-9; company B's first call tried id=1 → PK collision. Additionally, FOR UPDATE on an empty result set (new company) locks nothing, so concurrent webhooks both got id=1.
**Lesson:**
1. Never manually assign values to auto-increment PK columns — let PostgreSQL's sequence handle it
2. If you need company-scoped numbering, use a separate column, not the PK
3. `FOR UPDATE` on an empty result set is a no-op — it cannot prevent concurrent inserts
4. When catching IntegrityError in async SQLAlchemy, always `session.rollback()` first, and cache all ORM scalar values beforehand (objects expire after rollback → MissingGreenlet)
5. After fixing manual-id code, reset the sequence: `SELECT setval('seq', (SELECT MAX(id) FROM table))`

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

## Mirror Back Before Implementing — Don't Assume Understanding

**Source:** Mini App dialer redesign (2026-03-24)
**What happened:** User described what they wanted for the dialer. I implemented immediately without confirming my understanding. Got it wrong 5+ times in a row — wrong layout, wrong interaction model, wrong tap targets. User got increasingly frustrated. Each failed iteration wasted a build+deploy cycle (2 min each).
**Lesson:**
1. ALWAYS write back what you understood in plain language and get explicit "yes" before writing code
2. Draw ASCII layouts or describe the exact interaction model
3. If the user describes multiple changes, confirm each one separately
4. One failed implementation = acceptable. Three in a row = you're not listening

## Verify Deployments Actually Work — Don't Trust the Script

**Source:** Mini App dialer spam fix, CSS broken deploy (2026-03-24)
**What happened:** Deploy script said "deployed" but changes weren't live. Root causes: (1) `next build` silently OOM-killed — PM2 restarted with old `.next/` dir. (2) Next.js standalone mode doesn't serve `_next/static/` — CSS returned 404. Multiple "fix" iterations were wasted because the code was never actually deployed.
**Lesson:**
1. After EVERY deploy, verify the change is live: `ssh server "grep 'unique_string' .next/..."` or curl a known changed endpoint
2. Check for OOM: `next build` on a 1GB VPS can get killed — watch for `Killed` in output
3. Next.js `output: 'standalone'` does NOT serve static files — either use nginx for `_next/static/` or switch PM2 to `next start`
4. If deploy script says success but changes aren't visible, the build is stale — don't keep pushing more commits on top
5. Two concurrent build processes on a small VPS will OOM — never deploy frontend from two terminals simultaneously

## Flex Layout Needs Complete Parent Chain — flex:1 Inside Scroll Container Does Nothing

**Source:** Mini App dialer numpad positioning (2026-03-24)
**What happened:** Spent 5+ iterations trying to push the numpad to the bottom using `flex: 1` + `justify-content: flex-end`. It never worked because `.miniapp-content` (the parent) has `overflow-y: auto` and is NOT a flex container. `flex: 1` on a child of a non-flex parent is ignored.
**Lesson:**
1. `flex: 1` only works if the PARENT has `display: flex`
2. Scroll containers (`overflow: auto`) are not flex containers unless explicitly set
3. For fixed-bottom layouts inside scroll containers, use `min-height: calc(100dvh - known_offsets)` instead of relying on flex
4. Always trace the DOM parent chain to verify flex layout propagates correctly
5. Test on device, not just in theory — what looks correct in CSS logic may not render as expected

## Telegram WebView Haptic Feedback Blocks Main Thread

**Source:** Mini App dialer spam/lag fix (2026-03-24)
**What happened:** Users couldn't press numpad keys rapidly (5+ per second) — input froze. Tried: rAF batching, React.memo, event delegation. None worked. Root cause: `webApp.HapticFeedback.selectionChanged()` is a synchronous Telegram SDK call that blocks the JS main thread for ~50-100ms each time.
**Lesson:**
1. Telegram SDK haptic calls are synchronous and expensive — never call on every keypress
2. Either throttle to max 1 per 100ms, or disable entirely for rapid-fire interactions
3. Use `onPointerDown` instead of `onClick` in Telegram WebView — `onClick` has ~300ms delay for double-tap detection
4. CSS transitions on rapidly-pressed elements compound — remove `transition` for instant feedback
5. Event delegation (single handler on parent) is better than 12 individual button handlers for rapid interaction

## Operator Dropdown Cycles Through Items — Use Proper Selection UI

**Source:** Mini App dialer operator selector (2026-03-24)
**What happened:** First attempt at operator selection: tiny chevron button that cycled through operators one by one on each tap. User hated it — couldn't see options, couldn't pick directly. Then tried a bottom sheet (slides up from bottom) — user wanted dropdown that goes DOWN from the label.
**Lesson:**
1. Never use "cycle through" for selection — always show all options
2. Selection UI should match the context: dropdown for inline selection, bottom sheet for full-screen actions
3. Make the entire label row tappable (not just a tiny icon) — but don't add invisible padding that causes accidental taps
4. "Dropdown goes down, sheet goes up" — match user's spatial expectation

## Phone Display Logic: Inbound Shows Caller, Outbound Shows Callee

**Source:** Mini App call history showing wrong numbers (2026-03-24)
**What happened:** History page showed `phone_2 || phone_1` for all calls. For inbound missed calls, this showed the operator's SIP extension instead of the external caller's number. Dashboard missed calls count was 0 because query filtered by `operator_id == user.id` — missed calls often have no assigned operator.
**Lesson:**
1. Inbound calls: display `phone_1` (the caller/external party)
2. Outbound calls: display `phone_2` (the callee/external party)
3. Never use a single fallback (`phone_2 || phone_1`) — always check direction first
4. Missed/unanswered calls may not have `operator_id` — don't filter by it for missed call queries
5. Contact matching for inbound calls should join on `phone_1`, not `phone_2`

## i18n Keys Must Exist Before Referencing Them

**Source:** Mini App dialer SIP/External toggle (2026-03-24)
**What happened:** Added `t('calls.sipMode')` and `t('calls.externalMode')` in JSX but forgot to add the actual keys to the message JSON files. The deployed app showed raw key paths like `miniapp.calls.sipMode` instead of translated text.
**Lesson:**
1. When adding `t('new.key')` calls, ALWAYS add the key to ALL locale files (en/ru/uz) in the same commit
2. `next build` does NOT fail on missing i18n keys — they silently render as raw key paths
3. After adding i18n keys, verify by searching the JSON files: `grep 'newKey' messages/*.json`

## Telegram SDK startapp Param Is Immutable — Handle Deep Links Once

**Source:** Deep link redirect loop (2026-03-24)
**What happened:** `webApp.initDataUnsafe.start_param` is set once by Telegram when the Mini App opens and NEVER changes during the session. The layout's `authenticate()` callback re-read `startappData` on every navigation, causing an infinite redirect loop back to the deep link page. User was "locked" on the page they deep-linked to.
**Lesson:**
1. `start_param` is immutable — treat it as a one-time initialization value, not a reactive state
2. Use a `useRef` flag to track whether the deep link has been handled
3. Only redirect on the FIRST authentication, never on subsequent re-renders
4. Any value derived from Telegram SDK's `initDataUnsafe` should be consumed once, not re-read
