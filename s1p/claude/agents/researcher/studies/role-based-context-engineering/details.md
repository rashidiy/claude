# Role-Based Context Engineering Architecture — Full Research

## Question

How are others setting up role-based context engineering for AI coding agents? Specifically: how to make roles invocable like skills, scoped to a project, and switchable mid-session?

## Sources

| Source | Type | URL |
|---|---|---|
| Claude Code Skills Docs | Official docs | https://code.claude.com/docs/en/skills |
| Claude Code Subagents Docs | Official docs | https://code.claude.com/docs/en/sub-agents |
| Claude Code MCP Docs | Official docs | https://code.claude.com/docs/en/mcp |
| Anthropic's official skills repo | GitHub | https://github.com/anthropics/skills |
| Inside Claude Code Skills — Mikhail Shilkov | Blog | https://mikhail.io/2025/10/claude-code-skills/ |
| When to Use Skills vs Workflows vs Agents — Daniel Miessler | Blog | https://danielmiessler.com/blog/when-to-use-skills-vs-commands-vs-agents |
| Context Engineering for Coding Agents — Martin Fowler | Article | https://martinfowler.com/articles/exploring-gen-ai/context-engineering-coding-agents.html |
| Claude Code 6 Pillars Framework — ClaudeFast | Guide | https://claudefa.st/blog/guide/mechanics/context-engineering |
| How I Use Every Claude Code Feature — Shrivu Shankar | Blog | https://blog.sshh.io/p/how-i-use-every-claude-code-feature |
| Claude Code Customization Guide — Alex Op | Blog | https://alexop.dev/posts/claude-code-customization-guide-claudemd-skills-subagents/ |
| Context Engineering Deep Knowledge — Thomas Landgraf | Substack | https://thomaslandgraf.substack.com/p/context-engineering-for-claude-code |
| zhsama/claude-sub-agent | GitHub | https://github.com/zhsama/claude-sub-agent |
| contains-studio/agents | GitHub | https://github.com/contains-studio/agents |
| wshobson/agents | GitHub | https://github.com/wshobson/agents |
| VoltAgent/awesome-claude-code-subagents | GitHub | https://github.com/VoltAgent/awesome-claude-code-subagents |
| iannuttall/claude-agents | GitHub | https://github.com/iannuttall/claude-agents |
| valllabh/claude-agents | GitHub | https://github.com/valllabh/claude-agents |
| subagents.app | Directory | https://subagents.app/ |
| Empirical study of 253 CLAUDE.md files | Paper | https://arxiv.org/html/2509.14744v1 |

## Analysis

### The Two Native Mechanisms in Claude Code

Claude Code provides **two distinct systems** for making roles project-scoped and invocable:

#### 1. Skills (`.claude/skills/*/SKILL.md`)

Skills are the successor to custom slash commands. A file at `.claude/skills/researcher/SKILL.md` creates `/researcher` as an invocable command.

```yaml
---
name: researcher
description: Full-spectrum research intelligence for market and technical analysis
disable-model-invocation: true
---

You are the Researcher of S1P. Read `claude/roles/researcher/core.md` and operate from it.
...
```

Key properties:
- **Runs in main conversation context** — shares your context window, you see all its work
- **Project-scoped by default** — `.claude/skills/` only loads for this project
- **User-invocable** — triggered via `/researcher` slash command
- `disable-model-invocation: true` prevents Claude from auto-triggering it
- `user-invocable: false` makes it background knowledge Claude loads automatically (invisible to user)
- Supports `$ARGUMENTS` placeholder for parameterized invocation

Skills also support `context: fork` to delegate to a subagent:
```yaml
---
name: deep-research
context: fork
agent: Explore
---
Research $ARGUMENTS thoroughly...
```

#### 2. Subagents (`.claude/agents/*.md`)

Subagents run in isolated context windows with configurable tool access:

```yaml
---
name: researcher
description: Research agent for market and technical intelligence
tools: Read, Grep, Glob, WebSearch, WebFetch, Bash
disallowedTools: Write, Edit
model: sonnet
memory: project
---

You are the Researcher of S1P...
```

Key properties:
- **Isolated context window** — doesn't pollute main conversation
- **Tool restrictions** — can limit what the agent can do (e.g., read-only)
- **Model selection** — can run cheaper models for research tasks
- **Persistent memory** — `memory: project` creates a `MEMORY.md` that persists across sessions
- **Preloaded skills** — can inject domain knowledge via `skills:` frontmatter
- **Spawning control** — `tools: Agent(worker, tester)` restricts which sub-subagents it can create

### The 5-Layer Architecture (Community Consensus)

```
Layer 1: CLAUDE.md
  └── Always-on project identity and conventions
  └── "Constitution" — defines what Claude IS for this project
  └── Cannot be turned off within a session

Layer 2: .claude/rules/ (path-scoped)
  └── Auto-loads based on file paths you're working on
  └── Frontmatter: paths: ["src/api/**/*.ts"]
  └── Enables implicit role switching (edit frontend → frontend rules load)

Layer 3: .claude/skills/ (user-invoked)
  └── Explicit role switching via /command
  └── Runs in main context (shared conversation)
  └── Good for: roles you want to "become" temporarily

Layer 4: .claude/agents/ (delegated specialists)
  └── Isolated context, tool restrictions, model selection
  └── Good for: roles you want to "send off" to do work
  └── Returns structured results to main conversation

Layer 5: Agent Teams (experimental)
  └── Multiple full Claude Code instances
  └── Shared task list coordination
  └── Good for: parallel autonomous work on complex projects
```

### How Others Organize Roles

#### By Department (contains-studio pattern)
```
agents/
  design/           (ui-designer, brand-guardian, whimsy-injector)
  engineering/      (backend-architect, rapid-prototyper, test-writer)
  marketing/        (tiktok-strategist, growth-hacker)
  product/          (feedback-synthesizer, sprint-prioritizer, trend-researcher)
  project-management/
  testing/
```

#### By Workflow Phase (zhsama pattern)
```
agents/
  spec-agents/
    spec-orchestrator.md    # Phase coordinator
    spec-analyst.md         # Requirements → specs
    spec-architect.md       # Specs → system design
    spec-planner.md         # Design → task breakdown
    spec-developer.md       # Tasks → code
    spec-tester.md          # Code → tests
    spec-reviewer.md        # Tests → review
    spec-validator.md       # Review → production
```

#### By Capability (Daniel Miessler pattern)
```
skills/          (77 domain containers — Blogging, Research, Security)
  └── workflows/ (task procedures nested inside skills)
agents/          (20+ parallel workers that execute skills)
```

### Project Scoping Mechanisms

| Mechanism | Scope | How It Works |
|---|---|---|
| `.claude/skills/` | Project-only | Loads only when working in this repo |
| `~/.claude/skills/` | All projects | Global skills available everywhere |
| `.claude/agents/` | Project-only | Agents scoped to this repo |
| `~/.claude/agents/` | All projects | Global agents |
| `.claude/rules/` + `paths:` | File-scoped | Loads only when touching matching files |
| Monorepo nested `.claude/` | Package-scoped | Each package discovers its own `.claude/` |
| `--agents` CLI flag | Session-only | Override agents for one session |

### Role-as-Skill: The Concrete Pattern for S1P

To make a role invocable as `/researcher`:

**Option A: Skill (thin wrapper pointing to brain file)**
```
.claude/skills/researcher/SKILL.md
```
```yaml
---
name: researcher
description: Full-spectrum market and technical research intelligence
disable-model-invocation: true
---

Switch to Researcher mode. Read and follow `claude/roles/researcher/core.md`.
All research output goes into `claude/roles/researcher/studies/`.
```

**Option B: Agent (isolated, tool-restricted)**
```
.claude/agents/researcher.md
```
```yaml
---
name: researcher
description: Research agent — market intelligence, technical analysis, competitive research
tools: Read, Grep, Glob, WebSearch, WebFetch, Bash
disallowedTools: Write, Edit, NotebookEdit
model: sonnet
---

You are the Researcher of S1P. Read `claude/roles/researcher/core.md` for your full identity and methodology.

Your outputs go into `claude/roles/researcher/studies/<study-name>/` with `summary.md` and `details.md`.
```

**Option C: Both (recommended)**
- Skill `/researcher` for when the Founder wants to think-as-researcher in conversation
- Agent `researcher` for when the CTO wants to delegate research to a background worker

### Cursor Comparison

Cursor uses `.cursor/rules/*.mdc` files with frontmatter for file-path-scoped rules (similar to Claude Code's `.claude/rules/`). No equivalent to skills or subagents — Cursor's "agent mode" is single-agent only. Claude Code's architecture is significantly more flexible for role-based workflows.

### Aider Comparison

Aider has built-in modes (`/architect`, `/code`, `/ask`) but no custom role system. The Architect/Editor pattern (reasoning model proposes, editing model implements) achieved 85% on SWE-bench — interesting for a CTO/Developer split but not extensible to custom roles.

## Data & Comparisons

### Community Scale

| Repository | Agents | Skills | Stars |
|---|---|---|---|
| wshobson/agents | 112 agents | 72 plugins | Active |
| VoltAgent/awesome-claude-code-subagents | 100+ agents | — | Curated list |
| contains-studio/agents | 30+ agents | — | Department-organized |
| zhsama/claude-sub-agent | 8 core + domain agents | — | Spec-driven workflow |
| brolag's installer | 33 agents | — | One-click install |
| anthropics/skills (official) | — | Creative, technical, enterprise | Reference impl |

### Skill vs Agent Decision Matrix

| Factor | Skill | Agent |
|---|---|---|
| Context | Shared (main conversation) | Isolated (own window) |
| Tool access | Full (same as main) | Configurable (can restrict) |
| Model | Same as main session | Can override (e.g., Haiku for speed) |
| Memory | Shares main memory | Can have own persistent MEMORY.md |
| Invocation | `/command` by user | Auto-delegated or manual |
| Cost | No overhead | New context window = more tokens |
| Visibility | You see everything | Returns summary only |
| Best for | "I want to think as X" | "Go do X and report back" |

### Empirical Data on CLAUDE.md Patterns

An academic study of 253 real CLAUDE.md files found:
- 64.8% include architecture-level content (role definitions)
- 44.3% include testing patterns
- 35.2% define coding style rules
- CLAUDE.md files average ~500 lines in serious projects

## Risks & Caveats

1. **Skills and agents are Claude Code-specific** — If you ever move to Cursor or another tool, only the `claude/roles/` brain files are portable. The `.claude/` config is vendor-locked.
2. **Agent isolation has token cost** — Each subagent creates a new context window. Research tasks on Sonnet are cheaper, but still add up with frequent delegation.
3. **`disable-model-invocation` is critical** — Without it, Claude may auto-invoke roles at inappropriate times. Always set this for roles that should only be user-triggered.
4. **Memory persistence is experimental** — Agent memory (`memory: project`) works but is a newer feature. Brain files in `claude/roles/` are more stable as plain markdown.
5. **The 5-layer model adds complexity** — For a team of 1-2, using all 5 layers is overkill. Start with Layers 1+3 (CLAUDE.md + Skills) and add agents only when you need delegation.
6. **Community repos are fast-moving** — Many of the referenced repos are weeks old. Patterns may shift as Claude Code's agent features mature.

## Appendix

### Recommended Reading Order

1. [Claude Code Skills Docs](https://code.claude.com/docs/en/skills) — Understand the native skill system
2. [Claude Code Subagents Docs](https://code.claude.com/docs/en/sub-agents) — Understand isolation and delegation
3. [Daniel Miessler — Skills vs Workflows vs Agents](https://danielmiessler.com/blog/when-to-use-skills-vs-commands-vs-agents) — Organizational philosophy
4. [ClaudeFast 6 Pillars](https://claudefa.st/blog/guide/mechanics/context-engineering) — The layered architecture
5. [zhsama/claude-sub-agent](https://github.com/zhsama/claude-sub-agent) — Full spec-driven implementation

### S1P Migration Path

Current state:
```
claude/roles/
  cto/core.md
  researcher/core.md
  researcher/studies/
```

Proposed state (additive, keeps brain files):
```
claude/roles/                    # Brain files stay (portable, vendor-neutral)
  cto/core.md
  researcher/core.md
  researcher/studies/

.claude/
  skills/
    cto/SKILL.md               # /cto → loads claude/roles/cto/core.md
    researcher/SKILL.md        # /researcher → loads claude/roles/researcher/core.md
  agents/
    researcher.md              # Delegatable researcher with tool restrictions
  rules/
    frontend.md                # Auto-loads for src/components/**, src/pages/**
    backend.md                 # Auto-loads for src/api/**, src/services/**
```

This keeps brain files portable while adding Claude Code's native invocation and isolation features on top.
