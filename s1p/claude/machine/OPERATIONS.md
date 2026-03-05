# S1P Machine — Operations Guide

**Version:** 1
**Last updated:** 2026-03-05

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

### Agent → CTO Messages
Must include:
- **What** they need (decision, approval, clarification)
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
2. CTO checks: `claude/machine/BLUEPRINT.md` for current state
3. CTO reads: task list (if team was running) for progress
4. Agents read: their brain + their lessons.md
5. Continue from where we left off

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
