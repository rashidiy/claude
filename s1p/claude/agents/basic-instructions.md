# Basic Instructions — All Agents

Every agent MUST read this file before starting any work. These rules apply to every role without exception.

---

## Plan Mode

- Enter plan mode for every task. Propose your approach before implementing.
- Wait for approval. No cowboy coding.
- If something goes sideways mid-implementation, STOP and re-plan.

## Scope

- Spec is spec. Build exactly what's specified — nothing more, nothing less.
- No gold-plating. No "bonus" features, no refactoring outside the task, no "nice-to-haves."
- No new dependencies (npm, pip) without mentioning in your plan first.

## Existing Patterns

- Read existing code before writing new code. Find 2-3 similar files and match their structure.
- Follow codebase conventions. Don't invent new patterns unless the existing ones genuinely can't handle the task — and if so, escalate to CTO first.

## Git

- Feature branches from `dev`. PRs target `dev`. Never push directly to `main`.
- `main` is production-only — `dev` -> `main` merges happen when CTO/Founder approves a release.
- No `Co-Authored-By` lines. No AI/Claude attribution in commits.
- Dev agents work in **git worktrees** (isolated copy of the repo). Never edit the main working directory.
- Commit early and often on feature branches. WIP commits are fine — they survive crashes. Squash on merge.

## Taskboard

- Team Lead maintains `claude/agents/cto/taskboard.md` — the persistent task tracker.
- Dev agents: when you start a task, confirm the branch name with Team Lead so the taskboard is accurate.
- If your session crashes mid-task, the taskboard + your branch = recovery point for the next session.

## Communication

- Use **SendMessage** to communicate with teammates during a session.
- All messages route through Team Lead. Don't message other dev agents directly.
- Exception: CRITICAL security issues go directly to CTO.

## Escalation

- Never skip levels: Agent -> Team Lead -> CTO -> Founder.
- Task scope questions -> Team Lead.
- Architecture/technical decisions -> CTO (via Team Lead).
- Product/business decisions -> Founder (via CTO).
- When asking, always provide: what you need, context, options, and your recommendation.

## Ambiguity

- If a task spec is unclear, ask. Don't guess.
- Propose options with tradeoffs. Never send open-ended questions.

## Lessons — Non-Negotiable

Every agent maintains `claude/agents/[your-role]/lessons.md`. This is your memory.

**Before every task:** Read your lessons.md. Never repeat a past mistake.

**After every task:** If you discovered something valuable — a codebase pattern, a gotcha, a decision rationale, a mistake — write it to your lessons.md immediately. Don't wait. Don't skip. If it would help future-you avoid a problem or make a better decision, it belongs in lessons.md.

Categories:
- **Mistake:** What happened, impact, lesson, prevention
- **Decision:** Context, choice, reasoning, outcome
- **Pattern:** What, where (file:line), when to use
