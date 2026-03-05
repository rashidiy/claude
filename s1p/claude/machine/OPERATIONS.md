# S1P Machine — Operations Guide

**Version:** 2
**Last updated:** 2026-03-06

How to run the dev team. Read this on every session start.

---

## Starting a Session

1. Open terminal, `cd ~/projects/sip`, run `claude`
2. CTO loads automatically (CLAUDE.md)
3. Tell CTO what to work on: "Spin up the team, work on [feature]"
4. CTO creates team and spawns agents

---

## Team Structure

```
Founder (you)
  └── CTO (main Claude Code session)
        ├── Team Lead (teammate, plan mode)
        ├── Backend Engineer (teammate, plan mode)
        ├── Frontend Engineer (teammate, plan mode)
        ├── QA Engineer (teammate, read-only — never writes production code)
        └── Researcher (teammate — investigates, analyzes, delivers findings)
```

All agents are teammates at the same level. CTO coordinates.
Team Lead can spin up the Researcher when a task needs investigation before implementation.

---

## The Cycle

### 1. Scope → Team Lead
CTO gives Team Lead the feature scope.

### 2. Team Lead Consults Agents
Before drafting tasks, Team Lead messages backend/frontend agents for technical input:
- "What's the best approach for X?"
- "Any blockers you see?"
- "How much effort for Y?"

Agents respond with technical recommendations from their expertise + codebase knowledge.

### 3. Team Lead Drafts Plan
Team Lead synthesizes input into a task breakdown and enters plan mode.

### 4. CTO Reviews
- **Concrete:** CTO approves or adjusts the plan.
- **Unsure:** CTO asks the Founder with context and options.
- CTO notes the decision in `claude/agents/cto/decisions-cache.md`.

### 5. Tasks Created & Assigned
Team Lead creates tasks. CTO assigns to backend/frontend agents.

### 6. Agents Propose (Plan Mode)
Each agent reads the task, explores the codebase, and proposes an implementation plan.
- CTO approves if solid.
- CTO adjusts if issues spotted.
- CTO asks Founder if product/business decision needed.

### 7. Agents Implement
After plan approval, agents write code. They follow their brain file patterns exactly.

### 8. QA Review
Agent finishes → messages Team Lead → Team Lead sends to QA with task spec + files changed.

QA reviews against 10-point checklist (backend or frontend):
- **PASS:** QA approves. Team Lead marks task done. Next task.
- **FAIL:** QA lists specific issues. Team Lead relays to agent. Agent fixes. Back to QA.
- **FAIL x2:** Team Lead escalates to CTO.

No code merges without QA approval. No exceptions.

**Commit rules:**
- Never add `Co-Authored-By` lines to commit messages. All code is authored by the Founder.
- No AI/Claude attribution in commits, PRs, or code comments.

---

## Learning System

Every agent maintains a lessons file at `claude/agents/[role]/lessons.md`.

### What Gets Noted

**Mistakes:**
```markdown
## Mistake: [date]
**What happened:** Forgot company_id filter on new query
**Impact:** Would have leaked tenant data
**Lesson:** Always grep for company_id in every new query before committing
**Prevention:** Added to personal pre-commit checklist
```

**Decisions:**
```markdown
## Decision: [date]
**Context:** Needed to choose between aiogram and python-telegram-bot
**Choice:** python-telegram-bot — more stable, better docs, webhook support built-in
**Reasoning:** [why]
**Outcome:** [fill in after implementation]
```

**Patterns discovered:**
```markdown
## Pattern: [date]
**What:** The codebase uses X pattern for Y
**Where:** file_path:line_number
**When to use:** [conditions]
```

### When to Read Lessons
- **Before every task:** Read your own lessons.md to avoid repeating mistakes
- **Before every plan:** Check if a similar decision was already made

### When to Write Lessons
- **After every mistake** (test failure, QA rejection, wrong approach)
- **After every non-obvious decision** (library choice, pattern choice, workaround)
- **After discovering something about the codebase** that isn't in the brain file

---

## CTO Decisions Cache

CTO maintains `claude/agents/cto/decisions-cache.md`.

### Purpose
When a question comes to CTO that has been answered before, CTO references the cache instead of asking the Founder again.

### Rules
1. Every decision CTO makes gets logged (question + answer + date)
2. If the same or similar question appears again, CTO uses the cached answer
3. If a question appears 3+ times, CTO promotes it to a permanent rule in the relevant brain file
4. CTO tells the Founder when promoting a rule: "This keeps coming up, I'm making it permanent"

### Format
```markdown
## Q: [question]
**Date:** 2026-03-05
**Asked by:** [agent role]
**Answer:** [CTO's decision]
**Reasoning:** [why]
**Promoted to rule:** No | Yes → [which brain file]
```

---

## Escalation Chain

```
Agent question
  → Team Lead (scope, priority, task clarity)
    → CTO (architecture, technical decisions)
      → Founder (product direction, business decisions)
```

### What Each Level Handles

| Level | Handles | Example |
|---|---|---|
| Team Lead | Task scope, priority conflicts, coordination | "Should I do task A or B first?" |
| CTO | Technical decisions, architecture, patterns | "Should the Telegram bot use webhooks or polling?" |
| Founder | Product direction, business logic, market | "Should we support Binotel bot notifications in Phase 1?" |

### Rules
- Never skip levels. Agent → Team Lead → CTO → Founder.
- CTO only escalates when it's genuinely a business/product question.
- CTO always provides context + options when asking Founder, never open-ended questions.

---

## Communication Protocol

### How Agents Talk: SendMessage (Claude Code Teams)

All live communication uses **SendMessage** — the native Claude Code Teams tool. Agents are teammates in a shared team created by CTO at session start.

**Message flows:**
```
CTO → Team Lead:       Feature scope, plan approval/rejection
Team Lead → Backend:    Task spec, clarification, feedback relay from QA
Team Lead → Frontend:   Task spec, API contract from backend, feedback relay
Team Lead → QA:         Review request (task spec + files changed)
Backend → Team Lead:    "Done", "blocked on X", technical question
Frontend → Team Lead:   "Done", "blocked on X", technical question
QA → Team Lead:         PASS/FAIL verdict
Team Lead → CTO:        Status update, escalation, decision needed
Any agent → CTO:        CRITICAL security issue (bypasses Team Lead)
```

**No direct agent-to-agent.** Backend never messages Frontend directly. Everything routes through Team Lead. This prevents chaos and keeps Team Lead aware of all state.

### Message Format

Every message must include:
- **What** they need (decision, approval, clarification, status update)
- **Context** (what they're working on, what they tried)
- **Options** (if applicable, with pros/cons)
- **Recommendation** (what they think is best)

### CTO → Founder Messages
Must include:
- **The question** (clear, specific)
- **Who asked** (which agent, which task)
- **Options with tradeoffs** (never open-ended)
- **CTO's recommendation** (always have one)

### Bad Example
> "How should we handle the Telegram bot?"

### Good Example
> "Backend agent asks: should we use python-telegram-bot (stable, good docs, 15K GitHub stars) or aiogram (async-native, faster, 4K stars)? For our use case (webhook-based, inside FastAPI monolith), I recommend python-telegram-bot — more battle-tested and we don't need aiogram's async advantage since we're already async via FastAPI. Your call."

---

## Session Resumption

When starting a new session after a break:

1. CTO reads: `claude/agents/cto/instructions.md` + `claude/agents/cto/decisions-cache.md`
2. CTO reads: `claude/agents/cto/taskboard.md` — what was in progress?
3. CTO cross-references with git branches: `git branch` in `repos/s1p-backend/` and `repos/s1p-frontend/`
   - Taskboard says "in-progress" but branch is merged → mark done
   - Taskboard says "in-progress" and branch exists with changes → resume
   - Orphan branches not in taskboard → investigate before deleting
4. Agents read: their brain + their lessons.md
5. Continue from where we left off

---

## Conflict Resolution — Simultaneous File Edits

### Rule 1: Git Worktrees (Isolation by Default)
Dev agents work in **isolated git worktrees** (`isolation: "worktree"` on the Agent tool). They never share a working directory. Each agent gets its own copy of the repo, branches from `dev`, and merges via PR.

### Rule 2: Team Lead Avoids Overlap (Soft Rule)
When assigning parallel tasks, Team Lead checks `FILES LIKELY TOUCHED` in each task spec. If two tasks touch the same files, they run sequentially — not in parallel.

### Rule 3: Conflicts at PR Time (Git Handles It)
If two agents modify the same file despite precautions, git catches the conflict at merge time. The second PR to merge must rebase and resolve. This is normal git workflow — not a failure.

### What This Means in Practice
- Backend and Frontend agents can always work in parallel (different repos)
- Two Backend tasks that touch different files can run in parallel
- Two Backend tasks that touch the same router file → sequential
- Team Lead makes this call at assignment time, not mid-task

---

## Error Recovery — Agent Crashes

### The Taskboard
`claude/agents/cto/taskboard.md` is the persistent state tracker. Team Lead updates it:
- When a task is **created** → add row with `backlog` status
- When a task is **assigned** → update to `in-progress`, note the branch name
- When a task is **submitted for review** → update to `in-review`
- When a task is **done** → move to Completed table
- When a task is **blocked** → update status, note the blocker

### Recovery Protocol (Session Dies Mid-Task)
1. New session starts → CTO reads `taskboard.md`
2. For each `in-progress` task:
   - Check if the branch exists: `git branch` in the relevant repo
   - Check the branch state: `git log dev..branch-name --oneline`
   - If branch has commits but no PR → agent was mid-implementation, resume
   - If PR exists → check if QA reviewed, continue from there
   - If no branch exists → agent hadn't started, reassign
3. For each `in-review` task:
   - Check PR status on GitHub: `gh pr status`
   - If QA passed → merge and mark done
   - If QA failed → relay feedback, agent fixes
4. Update taskboard to reflect current reality

### What Survives a Crash
- **Always survives:** taskboard file, git branches, PRs, committed code
- **Lost:** uncommitted changes in the crashed agent's working directory
- **Mitigable:** agents should commit early and often (WIP commits are fine on feature branches)

---

## Quality Gates

### Branching Strategy
- `dev` — primary development branch. All PRs target `dev`. Default branch on GitHub.
- `main` — production checkpoints only. CI/CD deploys from `main`. Merge `dev` → `main` when CTO/Founder approves a release.
- Feature branches — short-lived, one per task, branched from `dev`.

### Before Any PR Merges (into `dev`)
1. Agent verifies all acceptance criteria from task spec
2. Agent ran tests successfully
3. Team Lead checks scope compliance (no gold-plating)
4. CTO spot-checks high-risk changes (auth, multi-tenancy, data model)

### After Every Sprint
1. Team Lead reports: tasks done, tasks remaining, blockers
2. CTO reviews: any recurring mistakes? Update brain files if patterns emerge
3. Founder reviews: is this what I wanted? Any course corrections?
