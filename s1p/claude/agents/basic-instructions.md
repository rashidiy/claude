# Basic Instructions — All Agents

Read this before starting any work.

---

## Plan Mode

- Propose your approach before implementing. Wait for approval.
- If something goes sideways mid-implementation, STOP and re-plan.

## Scope

- Build exactly what's specified — nothing more, nothing less.
- No gold-plating. No "bonus" features, no refactoring outside the task.
- No new dependencies (npm, pip) without mentioning in your plan first.

## Existing Patterns

- Read existing code before writing new code. Find 2-3 similar files and match their structure.
- Follow codebase conventions. Don't invent new patterns unless existing ones genuinely can't handle the task.

## Git

- Feature branches from `dev`. PRs target `dev`. Never push directly to `main`.
- No `Co-Authored-By` lines. No AI/Claude attribution in commits.
- Commit early and often on feature branches. WIP commits are fine — they survive crashes.

### Worktree Isolation (CRITICAL)

The application repos (`repos/s1p-backend`, `repos/s1p-frontend`) are **separate git repos inside a git-ignored directory**. The parent claude repo's worktree does NOT copy them. You MUST create worktrees inside the sub-repo itself:

```bash
# 1. Create worktree from the sub-repo
cd /home/rashidiy/claude/s1p/repos/s1p-frontend
git worktree add /tmp/agent-YOUR_TASK_ID dev

# 2. Work entirely in the worktree — NEVER cd back to the original repo
cd /tmp/agent-YOUR_TASK_ID
git checkout -b fix/your-branch-name

# 3. Do all your work here, commit here

# 4. When done, the dev lead merges your branch in the main directory
# 5. Clean up
git worktree remove /tmp/agent-YOUR_TASK_ID
```

**NEVER** run `git checkout` in the main repo directory (`repos/s1p-frontend` or `repos/s1p-backend`). The user works there. Checking out your branch there will overwrite their work.

## Pre-Flight (Before Writing Any Code)

1. Read `claude/agents/[your-role]/lessons.md` — your past mistakes. Don't repeat them.
2. Read `claude/memory/lessons.md` — cross-cutting project lessons.
3. Read `claude/memory/decisions.md` — locked decisions. Don't re-ask or contradict.
4. If your task involves a cross-repo change, read the relevant contract in `claude/contracts/`. If no contract exists, STOP and ask the dev lead to write one first.

## Memory

- After every mistake, decision, or pattern discovery — write it to your lessons file. If it affects multiple agents, write to `claude/memory/lessons.md`.

## Communication

- Use **SendMessage** to communicate with the main session during work.
- If blocked or facing a decision, escalate with: what you need, context, options, and your recommendation.
- Never send open-ended questions. Always propose options with tradeoffs.

## Ambiguity

- If a task spec is unclear, ask. Don't guess.
