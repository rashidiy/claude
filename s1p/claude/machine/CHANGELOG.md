# S1P Dev Machine — Changelog

All notable changes to the machine architecture, agent brains, and configuration.

Format: [Semantic Versioning](https://semver.org/)
- **MAJOR** — Architecture redesign, agent hierarchy change, tool swap
- **MINOR** — New agent added, new brain file, new workflow rule
- **PATCH** — Brain file tuning, rule adjustment, config tweak

---

## [0.1.0-draft] — 2026-03-05

### Status: Designing — not yet operational

### Added
- Initial blueprint (`BLUEPRINT.md`) with full architecture design
- CTO brain file (`claude/agents/cto/instructions.md` v100)
- Researcher brain file (`claude/agents/researcher/instructions.md` v1)
- Research study: Role-Based Context Engineering
- Research study: Autonomous AI Dev Team Feasibility
- Study templates (`claude/agents/researcher/studies/_template/`)
- Machine directory (`claude/machine/`)
- This changelog

### Decided
- GitHub as single monitoring/communication hub (replaces Notion + Linear + Telegram)
- Separate repos (s1p-frontend, s1p-backend)
- Sonnet 4.6 for Lead + Dev + QA agents
- Haiku for Researcher + Monitor agents
- Modular monolith backend (5 modules + shared core)
- Ant Design rules for frontend (no human designer)
- Auto-merge on QA pass (Founder doesn't review PRs)
- Lead Agent = CTO + PM combined (1-level hierarchy workaround)
- Machine versioning via BLUEPRINT.md + CHANGELOG.md

### Not Yet Done
- Brain files: QA, Monitor
- `.claude/agents/*.md` agent definitions
- `.claude/skills/*.md` skill definitions
- `MACHINE.md` operational config
- GitHub Discussions/Projects/Actions setup
- First test run on real feature

## [0.2.0-draft] — 2026-03-05

### Added
- Team Lead brain file (`claude/agents/team-lead/instructions.md` v1) — task breakdown, dependency mapping, cross-agent coordination, quality checklists
- Backend Engineer brain file (`claude/agents/backend/instructions.md` v1) — 10+ years FastAPI expertise, full codebase patterns, multi-tenant rules, all Phase 1 specs
- Frontend Engineer brain file (`claude/agents/frontend/instructions.md` v1) — 10+ years Next.js/Ant Design expertise, full codebase patterns, i18n plan, component rules
- Full codebase audit: Backend (16 models, 60+ endpoints, 14 test files) and Frontend (44 pages, 80+ API methods, design system documented)
- Phase 1 gap analysis: 23 done, 4 partial, 14 not started
- `.claude/agents/team-lead.md` — Team Lead agent definition (plan mode, delegates to backend/frontend)
- `.claude/agents/backend.md` — Backend Engineer agent definition (plan mode, FastAPI/Python)
- `.claude/agents/frontend.md` — Frontend Engineer agent definition (plan mode, Next.js/Ant Design)

### Decided
- Workflow: agents use plan mode, CTO/Founder confirms before execution
- Priority order: i18n Russian (P0) → Telegram bot (P0) → operator scoping (P1) → custom fields (P2) → public API (P2)
- Team Lead orchestrates Backend + Frontend agents
- Machine design deferred — build with lightweight team first, scale when proven
- Team Lead MUST consult backend/frontend agents before drafting task plans
- CTO maintains decisions cache — never asks Founder the same question twice
- All agents maintain lessons.md — mistakes and decisions noted for future reference
- Escalation chain: Agent → Team Lead → CTO → Founder (never skip levels)

### Added (Operations)
- `claude/machine/OPERATIONS.md` — full operational guide (session start, team cycle, learning system, escalation, communication protocol)
- `claude/agents/cto/decisions-cache.md` — CTO's cached decisions
- `claude/agents/team-lead/lessons.md` — Team Lead learning log
- `claude/agents/backend/lessons.md` — Backend learning log
- `claude/agents/frontend/lessons.md` — Frontend learning log
- Updated all agent definitions with learning system + escalation rules
- Updated CLAUDE.md with operations reference
- QA Engineer brain file (`claude/agents/qa/instructions.md` v1) — 10-point backend checklist, 10-point frontend checklist, severity levels, verdict format
- `.claude/agents/qa.md` — QA agent definition (read-only, never writes code)
- `claude/agents/qa/lessons.md` — QA learning log
- Updated Team Lead brain with QA handoff workflow (review request template, failure handling, escalation after 2 failures)
- Updated CLAUDE.md agents table with QA
