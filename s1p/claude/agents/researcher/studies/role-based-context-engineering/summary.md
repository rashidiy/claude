# Role-Based Context Engineering Architecture
**Date:** 2026-03-04
**Status:** complete
**Requested by:** Founder

## Key Findings

- **Claude Code has two native mechanisms** for project-scoped roles: **Skills** (`.claude/skills/*/SKILL.md` — run in main context, invoked via `/command`) and **Subagents** (`.claude/agents/*.md` — run in isolated context with tool restrictions). Both are project-scoped by default.
- **The community has converged on a 5-layer architecture**: CLAUDE.md (constitution) → Rules (auto-load by file path) → Skills (user-invoked roles) → Agents (isolated specialists) → Teams (parallel workers).
- **Roles are not just prompts** — the real power is prompt + tool restrictions + model selection + context isolation. A "researcher" with `disallowedTools: Write, Edit` running on Haiku behaves fundamentally differently from the same prompt with full access on Opus.
- **Project scoping is built in**: `.claude/skills/`, `.claude/agents/`, `.claude/rules/` are all project-local by default. User-global equivalents live in `~/.claude/`. Monorepos get nested discovery (each package can have its own `.claude/`).
- **Path-scoped rules** (`.claude/rules/*.md` with `paths:` frontmatter) enable automatic role switching — work on frontend files, frontend rules load; work on API files, backend rules load.
- **Leading repos** (wshobson/agents, zhsama/claude-sub-agent, contains-studio/agents) show 70-100+ agents organized by department (engineering, design, product, marketing).

## Recommendation

**Migrate your `claude/roles/` pattern into Claude Code's native `.claude/` structure.** Specifically:

1. **Skills for role invocation** — Move each role brain into `.claude/skills/<role>/SKILL.md` so you get `/cto`, `/researcher` as slash commands. Use `disable-model-invocation: true` so only you trigger them.
2. **Agents for delegation** — Create `.claude/agents/researcher.md` with `tools: Read, Grep, Glob, WebSearch, WebFetch` and `disallowedTools: Write, Edit` so the researcher can't accidentally modify code.
3. **Rules for auto-loading** — Use `.claude/rules/` with path-scoped frontmatter if you want automatic context loading based on what files you're touching.
4. **Keep studies/ where it is** — Research outputs are data, not config. `claude/roles/researcher/studies/` is fine as content storage.

## Decision Needed

- Do you want to restructure roles into `.claude/skills/` + `.claude/agents/` now, or keep the current `claude/roles/` pattern and migrate later?
- Should the researcher role be a **skill** (runs in your context, you see everything), a **subagent** (isolated context, returns summary), or **both**?
