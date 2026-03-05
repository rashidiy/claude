# Taskboard

Persistent task state. Survives session crashes. Read on every session start.

Team Lead owns this file. Update when tasks are created, started, finished, or blocked.

---

## Active Tasks

| ID | Task | Agent | Status | Branch | Repo | Notes |
|---|---|---|---|---|---|---|
| — | — | — | — | — | — | — |

## Completed (Last 10)

| ID | Task | Agent | Completed | Branch |
|---|---|---|---|---|
| — | — | — | — | — |

---

## Status Legend

- `backlog` — created, not yet assigned
- `in-progress` — agent is working on it
- `in-review` — PR open, waiting for QA
- `blocked` — waiting on dependency or decision
- `done` — merged into `dev`
- `failed` — abandoned or rejected after 2 QA failures
