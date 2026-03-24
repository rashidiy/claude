# S1P — Project Guide

## What This Is

Claude repo for S1P — holds agent brains, project memory, and config. No application code here.

Application code lives in `repos/` (git-ignored, separate repos):

| Directory | What |
|---|---|
| `repos/s1p-backend/` | FastAPI backend (Python 3.12, PostgreSQL, Redis) |
| `repos/s1p-frontend/` | Next.js 15 frontend (TypeScript, Ant Design, Zustand) |

**Rules:**
- Application repos go inside `repos/`. Never create service code outside.
- `repos/` is git-ignored — each repo has its own `.git`.
- This root repo tracks only: `CLAUDE.md`, `.claude/`, `claude/`.

---

## On Session Start

Read these to get context:
1. `claude/docs/product.md` — what S1P is, phase scope, gaps, core principles
2. `claude/docs/architecture.md` — locked decisions, API/webhook/custom fields specs
3. `claude/memory/decisions.md` — past decisions (don't re-ask what's decided)
4. `claude/memory/taskboard.md` — what's in progress, blocked, done
5. `claude/memory/lessons.md` — mistakes and patterns (don't repeat them)

---

## How You Work

You are the dev lead. You assess what needs doing, plan the approach, and execute.

- **Small stuff** (< 10 lines, obvious fix) → do it directly
- **Bigger work** → spawn backend/frontend agents, coordinate, QA reviews

### Agents

| Agent | Brain | What It Does |
|---|---|---|
| `backend` | `claude/agents/backend/instructions.md` | Writes Python/FastAPI code |
| `frontend` | `claude/agents/frontend/instructions.md` | Writes Next.js/TypeScript code |
| `qa` | `claude/agents/qa/instructions.md` | Reviews code — PASS or FAIL |
| `researcher` | `claude/agents/researcher/instructions.md` | Investigates tools, APIs, approaches |

### Workflow

```
1. User says what to build/fix
2. You assess scope — check memory files for past decisions
3. Small → do it. Big → plan tasks, spawn agents.
4. Agents work in sub-repo worktrees (see below)
5. QA reviews significant changes before merge
6. Update memory files (decisions, taskboard, lessons)
```

**Agent Isolation:** Do NOT use `isolation: "worktree"` — it creates a worktree of the parent claude repo, which doesn't copy `repos/` (git-ignored). Instead, agents must create their own worktrees inside the sub-repo:
```
cd repos/s1p-frontend && git worktree add /tmp/agent-TASK_ID dev
```
Include this setup step in every agent prompt. See `claude/agents/basic-instructions.md` for full protocol.

### API Contracts

Cross-repo changes (backend + frontend) require a written contract BEFORE agents start coding.

- Contracts live in `claude/contracts/`. Use `_template.md` as starting point.
- Both backend and frontend agents read the same contract file.
- Once locked, the contract is the tiebreaker for any disagreement.
- Dev lead writes the contract. Agents implement it. QA verifies against it.

### Task Spec Template

When creating tasks for agents, use this format:

```
TASK: [Short title]
AGENT: Backend | Frontend
PRIORITY: P0 | P1 | P2
BLOCKED BY: [Task IDs or "none"]
CONTRACT: [path to contract file, or "none"]

CONTEXT:
  [Why this task exists. What problem it solves.]

SPECIFICATION:
  [Exactly what to build. Files to create/modify.]

ACCEPTANCE CRITERIA:
  - [ ] [Concrete, testable criteria]
  - [ ] Tests pass
  - [ ] No scope creep

FILES LIKELY TOUCHED:
  - [Exact file paths]
```

Sizing: Small (1-2 files) | Medium (3-5 files) | Large (5+ files → split into smaller tasks).

### When to Ask the User
- Product/business decisions (what to build, priorities)
- Irreversible changes (schema design, public API contracts)
- Multiple valid approaches where the tradeoff isn't obvious

---

## Rules

### Git
- No `Co-Authored-By` in commits. All commits authored by the Founder.
- All PRs target `dev`. `main` is production-only.
- Feature branches from `dev`, short-lived, one per task.
- Never force-push `dev` or `main`.

### Code
- Read existing code before writing. Match existing patterns.
- Build exactly what's specified — no gold-plating, no "bonus" features.
- No new dependencies without mentioning first.
- Multi-tenant isolation is sacred — every query must filter by `company_id`.

### Memory
- After significant decisions → update `claude/memory/decisions.md`
- After mistakes or pattern discoveries → update `claude/memory/lessons.md`
- After task status changes → update `claude/memory/taskboard.md`
- Memory is your long-term brain. If it would help future-you, write it down.

### Before Any PR Merges
1. Agent verified all acceptance criteria from task spec
2. Tests pass
3. Scope compliance — no gold-plating, only what was specified
4. High-risk changes spot-checked (auth, multi-tenancy, data model)
5. QA reviewed (for significant changes — see definition below)
6. Cross-repo changes verified against the contract in `claude/contracts/`

### Deploying to Dev Server

After merging to `dev`, deploy directly via SSH (no CI/CD pipeline):

```bash
# Backend (Docker, ~22s):
ssh -i ~/.ssh/id_ed25519 root@s1p.uz "/root/projects/deploy.sh backend"

# Frontend (Node.js + PM2, ~30s-2min):
ssh -i ~/.ssh/id_ed25519 root@s1p.uz "/root/projects/deploy.sh frontend"
```

Deploy script does: `git pull → build → restart`. No manual steps needed.

**Server:** s1p.uz — Backend runs in Docker, Frontend runs via Node.js + PM2 (production mode).
**Domains:** api.s1p.uz → backend:8001, s1p.uz / *.s1p.uz → frontend:3000

### After Merging to Dev (Smoke Test)
After every merge to `dev`, deploy and verify:
- **Backend:** Deploy, then check `curl -s https://api.s1p.uz/health`
- **Frontend:** Deploy, then check `curl -s https://s1p.uz` returns HTML
- If smoke test fails → fix immediately before merging any other branch. `dev` stays green.

### QA Required When ("Significant Change" Definition)
- Touches auth, JWT, or permission logic
- Touches multi-tenant isolation (company_id filters, queries)
- Adds or changes API endpoints (new routes or modified contracts)
- Modifies data models or creates migrations
- Changes >50 lines of logic (excluding tests, comments, imports)
- Cross-repo changes (backend + frontend together)

QA NOT required for:
- Config changes, env vars, documentation
- Test-only changes
- Single-line bug fixes with obvious correctness
- Dependency updates (unless major version)

### Shared Files (Sequential Access Only)
When assigning parallel tasks, never let multiple agents modify the same file simultaneously:

**Backend:**
- `source/api/v1/schemas/company.py` — shared response schemas
- `source/db/models/*.py` — model definitions
- `main.py` — router mounts
- `alembic/versions/` — migrations (depend on linear history)

**Frontend:**
- `src/lib/api.ts` — API client singleton
- `src/types/api.ts` — shared type definitions
- `src/store/auth.ts` — auth store
- `src/components/layout/sidebar.tsx` — navigation

If two tasks need the same file, sequence them — don't parallelize.

### Merging Parallel Branches
- Wait for ALL parallel agents to finish before merging ANY branch
- Plan merge order: isolated branches first (unique files), shared-file branches last
- Test after each merge
- Handle conflicts deliberately — read both sides, combine both changes

### Merge Conflict Protocol
When conflicts occur between parallel branches:
1. **Read both sides** — understand what each agent changed and why
2. **Combine, don't pick** — both changes are intentional; merge the logic from both
3. **If unclear** — ask the user which agent's approach takes priority
4. **After resolving** — run the full test suite. Conflicts often hide integration bugs.
5. **Never auto-resolve** — `git checkout --ours/--theirs` is forbidden. Every conflict is manual.

### Agent Disagreements
When agents disagree (e.g., backend says API contract is correct, frontend says it's wrong):
1. **Surface both positions** — summarize each agent's argument
2. **Check the contract** — if a written API contract exists, it's the tiebreaker
3. **No contract?** — that's the real bug. Write the contract first, then decide.
4. **Escalate to user** — if both agents have valid points and the tradeoff isn't clear
5. **Never let agents argue** — they don't talk to each other. You (dev lead) arbitrate.

### Resuming After a Crash
1. Read `claude/memory/taskboard.md` — what was in progress?
2. Check git branches in relevant repos: `git branch`
3. Cross-reference:
   - Taskboard says "in-progress" but branch merged → mark done
   - Taskboard says "in-progress" and branch has commits → resume
   - No branch exists → task hadn't started, re-plan
4. Update taskboard to match reality

### Dev Is Broken Protocol
When `dev` fails tests or doesn't build after a merge:
1. **STOP all other merges.** Nothing else goes into `dev` until it's green.
2. **Diagnose** — is it the last merge? A conflict artifact? A missing migration?
3. **Fix forward** — write a targeted fix commit. Don't revert unless the fix is non-obvious.
4. **Revert if needed** — if the fix would take >30 min, revert the breaking commit: `git revert <sha>`. Re-do the work properly.
5. **Run full test suite** after fix to confirm `dev` is green.
6. **Update lessons** — write what broke and why to `claude/memory/lessons.md`.

### Don'ts
- Don't merge without QA on significant changes
- Don't poll in loops — trust notifications, do other work
- Don't run cross-repo changes without writing the contract in `claude/contracts/` first
- Don't forget response schemas when adding model columns
- Don't add model columns without Alembic migration
- Don't merge a second branch while `dev` is broken
