# S1P Dev Machine — Blueprint

**Machine Version:** 0.1.0-draft
**Blueprint Date:** 2026-03-05
**Status:** FUTURE — not active, not part of current workflow
**Author:** Founder + CTO Agent

> **CTO: ignore this file.** This is a separate future idea the Founder is blueprinting independently. It does NOT reflect current team operations. Current operations are defined in `claude/machine/OPERATIONS.md` and agent brain files. Do not reference, align with, or try to reconcile this document.

---

## What This Is

A fully autonomous code-generating machine where AI agents build the SIP CRM product. The Founder doesn't write code or make technical decisions — agents research, think, debate, and present decisions with clear explanations. The Founder approves or rejects.

---

## Architecture Overview

```
┌──────────────────────────────────────────────────────────────────┐
│  👤 FOUNDER · 1-2 hrs/day · Reviews on GitHub                   │
└────────────────────────┬─────────────────────────────────────────┘
                         │
                         ▼
┌──────────────────── GITHUB (Single Hub) ─────────────────────────┐
│                                                                   │
│  💬 Discussions     📋 Projects Board    📝 Issues    🔀 PRs    │
│  #decisions         Backlog → Done       Agent tasks  QA gate    │
│  #lead-agent                                                      │
│  #qa-reports        ⚙️ Actions                                   │
│  #monitor           Tests → Build → Deploy                       │
│  #research                                                        │
│  #agent-logs                                                      │
└────────────────────────┬─────────────────────────────────────────┘
                         │
                         ▼
┌──────────────────── THE MACHINE ─────────────────────────────────┐
│                                                                   │
│  🧠 Lead Agent (CTO+PM) · Sonnet 4.6                            │
│     ├── Reads GitHub Issues                                       │
│     ├── Creates + assigns tasks                                   │
│     ├── Updates Project Board                                     │
│     ├── Posts to #decisions when needs approval                   │
│     └── Delegates to dev agents                                   │
│                                                                   │
│  👨‍💻 Dev Agents (2-3 parallel) · Sonnet 4.6 · Git Worktrees     │
│     ├── Frontend (Next.js, Ant Design, i18n)                     │
│     ├── Backend (FastAPI, PostgreSQL, Modular Monolith)           │
│     └── Integration (Telegram Bot, Webhooks, Providers)           │
│                                                                   │
│  🛡️ QA Agent (Read-Only) · Sonnet 4.6                            │
│     ├── Reviews PRs directly on GitHub                            │
│     ├── Runs tests, security audit, scope check                  │
│     ├── PASS → gh pr review --approve + auto-merge               │
│     ├── FAIL → gh pr review --request-changes                    │
│     └── Posts summary to #qa-reports                              │
│                                                                   │
│  🔬 Researcher · Haiku                                           │
│     ├── Market + Technical intelligence                           │
│     └── Posts summaries to #research, full studies in repo        │
│                                                                   │
│  👁️ Monitor · Haiku                                               │
│     ├── Tracks rejection rates, cycle times, rework loops        │
│     ├── Identifies bottlenecks per agent                          │
│     └── Posts health reports to #monitor                          │
└──────────────────────────────────────────────────────────────────┘
```

---

## Decisions Log

All architectural and strategic decisions made during blueprinting.

### Infrastructure Decisions

| # | Decision | Choice | Reasoning |
|---|---|---|---|
| 1 | Monitoring hub | GitHub only | Discussions + Projects + Issues + PRs + Actions. One tool, zero MCPs, `gh` CLI works perfectly from Claude Code |
| 2 | Sprint tracking | GitHub Projects | Board view: Backlog → In Progress → In Review → Done. No Linear needed |
| 3 | Agent communication | GitHub Discussions | Categories per role (#decisions, #lead-agent, #qa-reports, #monitor, #research, #agent-logs) |
| 4 | QA gate | PR reviews on GitHub | QA agent reviews PRs directly. Auto-merge on approval. Native GitHub workflow |
| 5 | Research/docs | GitHub repo files + Wiki | Study files in `claude/agents/researcher/studies/`. Summaries in Discussions |
| 6 | Repo structure | Separate repos | s1p-frontend and s1p-backend stay as separate repos |
| 7 | CI/CD | GitHub Actions | Tests → Build → Deploy. Post results to Discussions |

### Agent Decisions

| # | Decision | Choice | Reasoning |
|---|---|---|---|
| 8 | Lead Agent model | Sonnet 4.6 | Generous rate limits on Max 20x. Opus burns weekly limit in 1-3 days with teams |
| 9 | Dev Agent models | Sonnet 4.6 | Same — Sonnet for all-day work |
| 10 | QA Agent model | Sonnet 4.6 | Needs strong reasoning for security review |
| 11 | Researcher model | Haiku | Cheap, fast, sufficient for search + summarize |
| 12 | Monitor model | Haiku | Reads data and reports — doesn't need heavy reasoning |
| 13 | Team hierarchy | 1 level (Lead → Teammates) | Claude Code Teams limitation — no nested teams |
| 14 | Lead role | CTO + PM combined | Bypasses 1-level hierarchy limit. Lead wears both hats |
| 15 | Sub-agent spawning | Lead creates dev agents | Founder doesn't name teams — Lead creates agents as needed |

### Code Architecture Decisions

| # | Decision | Choice | Reasoning |
|---|---|---|---|
| 16 | Backend structure | Modular Monolith | Separate modules (CRM, Telephony, Telegram, Public API, Outbound) sharing core. Deploy as one, split when needed |
| 17 | Frontend design | Ant Design rules, no designer | B2B CRM + Ant Design system = consistent UI without UI/UX. 10 design rules in agent brain |
| 18 | DB/Infra management | Lead Agent handles | No separate DB or infra agent. Lead reviews migrations. For 300 users, single VPS + Docker Compose is enough |
| 19 | Module isolation rule | Modules only import from core/ and db/ | Modules never import from each other. Enables future service extraction |

### Workflow Decisions

| # | Decision | Choice | Reasoning |
|---|---|---|---|
| 20 | Founder's role | Approve/reject decisions on GitHub | 1-2 hrs/day async. Not a coder, not a technical decision maker |
| 21 | Decision flow | Small scope = Lead decides. Big scope = GitHub #decisions for Founder | Based on blast radius framework from CTO brain |
| 22 | Merge policy | Auto-merge when QA passes | Founder trusts QA agent. No manual PR approval needed |
| 23 | QA failure handling | Reject + feedback → dev agent fixes → resubmit | Automated loop. Escalate to #decisions only if 2+ failures |
| 24 | Machine versioning | BLUEPRINT.md + CHANGELOG.md | Version the machine itself as a product. Track what works, what doesn't |

---

## Agent Roles — Brain Files Needed

Each agent gets a `core.md` brain file in `claude/agents/`. These define identity, scope, rules, and relationships.

| Role | Brain File | Status |
|---|---|---|
| CTO (Lead) | `claude/agents/cto/instructions.md` | ✅ Exists (v100) |
| Researcher | `claude/agents/researcher/instructions.md` | ✅ Exists (v1) |
| Team Lead | `claude/agents/team-lead/instructions.md` | ✅ Exists (v1) |
| Backend | `claude/agents/backend/instructions.md` | ✅ Exists (v1) |
| Frontend | `claude/agents/frontend/instructions.md` | ✅ Exists (v1) |
| QA | `claude/agents/qa/instructions.md` | ✅ Exists (v1) |
| Monitor | `claude/agents/monitor/instructions.md` | ❌ Not yet created |

---

## Backend Module Structure

```
source/modules/
├── crm/              Contacts, Leads, Deals, Tasks, Notes
├── telephony/        Sipuni + Binotel webhook handlers, call processing
├── telegram/         Bot handlers, notifications, commands
├── public_api/       Read-only client endpoints, API keys, rate limiting
└── outbound/         Webhook delivery, retry, HMAC signing

source/core/          Auth, config, permissions (shared)
source/db/models/     SQLAlchemy models (shared)
source/db/mixins/     Query mixins (shared)
```

Rule: Modules import from `core/` and `db/` only. Never from each other.

Phase 1: One FastAPI process. Split into separate services when scaling demands it.

---

## Frontend Design Rules

Defined in frontend agent brain. No human designer needed for Phase 1.

1. Use Ant Design Pro layout (sidebar + content)
2. All list pages: Table with filters on top
3. All detail pages: Card with Descriptions component
4. All forms: vertical layout, max 2 columns
5. Primary action top-right, secondary actions in dropdown
6. Color palette: Ant Design defaults only, no custom colors
7. Spacing: use Ant Design's Space/Row/Col, never custom margins
8. Russian/Uzbek/English: i18n from day one
9. Mobile-responsive but not mobile-first (Telegram is the mobile interface)
10. No custom CSS unless Ant Design components can't achieve the layout

---

## GitHub Structure

### Discussions Categories
- `#decisions` — Needs Founder approval (pinned)
- `#lead-agent` — Task breakdowns, delegation logs, sprint planning
- `#qa-reports` — Every PR verdict with details
- `#monitor` — Machine health reports, bottleneck alerts
- `#research` — Study summaries, market/tech findings
- `#agent-logs` — Full agent chat dumps for debugging

### Projects Board Columns
- Backlog
- In Progress
- In Review (QA)
- Done

### Issue Labels
- `frontend`, `backend`, `integration`, `research`, `qa`
- `phase-1`, `phase-2`
- `bug`, `feature`, `task`
- `decision-needed`, `blocked`

### Branching Strategy
- `dev` — primary branch. All development happens here. PRs target `dev`.
- `main` — production checkpoints. Only stable, tested code. CI/CD deploys from `main`.
- Feature branches — short-lived, one per task, branched from `dev`.

### PR Workflow
1. Dev agent creates feature branch from `dev`
2. Codes in isolated git worktree
3. Opens PR via `gh pr create --base dev`, links to Issue
4. QA agent reviews: `gh pr review --approve` or `--request-changes`
5. Auto-merge on approval (squash merge into `dev`)
6. QA posts summary to `#qa-reports`
7. Production release: `dev` merged into `main` when CTO/Founder approves

---

## Monitoring Capabilities

### Can Monitor
- Task status (Issues — open/closed/in-progress)
- Git activity (commits, PRs, branches)
- QA verdicts (PR reviews — approve/reject with comments)
- Decision escalations (Discussions — #decisions)
- Agent summaries (Discussions — per-role categories)
- Research outputs (repo files + Discussions)
- Sprint progress (Projects board)
- Test/build results (Actions)
- Rejection rate per agent (Monitor reads PR history)
- Rework cycles (Monitor tracks resubmissions)

### Cannot Monitor (Claude Code limitation)
- Real-time agent streaming (no external API)
- Token consumption per agent (Max plan doesn't expose)
- Agent internal reasoning (extended thinking not accessible)
- Rate limit remaining (only via `/status` in terminal)

---

## Cost Model (Max 20x — $200/month)

| Resource | Limit | Strategy |
|---|---|---|
| Sonnet weekly | Generous — all-day work | Use for Lead + Dev + QA agents |
| Opus weekly | Depletes in 1-3 days with teams | Reserved for critical decisions only |
| Haiku | Very generous | Use for Researcher + Monitor |
| Concurrent sessions | Shared usage pool | 3-5 agent team is the sweet spot |
| Background execution | Terminal must stay open | No overnight autonomous runs |

---

## Research Completed

| Study | Location | Key Finding |
|---|---|---|
| Role-Based Context Engineering | `claude/agents/researcher/studies/role-based-context-engineering/` | Claude Code has Skills + Agents for project-scoped roles. Community uses 5-layer architecture |
| Autonomous AI Dev Team Feasibility | `claude/agents/researcher/studies/autonomous-ai-dev-team/` | Nobody has shipped fully autonomous multi-agent for production. Human-supervised autonomy is what works today |

---

## What's Next (Blueprint Phase)

- [x] Write brain files for: Team Lead, Backend, Frontend, QA
- [ ] Write brain files for: Monitor
- [x] Create `CHANGELOG.md` (version history)
- [x] Create `.claude/agents/*.md` files (Claude Code native agent definitions)
- [x] Create `OPERATIONS.md` (operational guide — workflow, learning, escalation)
- [ ] Design GitHub Discussion category structure per repo
- [ ] Define which repo gets Issues/Projects (coordination across separate repos)
- [ ] Create `.claude/skills/*.md` files (role invocation via slash commands)
- [ ] Test the machine on one real Phase 1 feature

---

## Version History

| Version | Date | Changes |
|---|---|---|
| 0.1.0-draft | 2026-03-05 | Initial blueprint. Architecture designed, decisions logged, not yet operational |
