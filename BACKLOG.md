v001 | 2026-08-27 | 41 lines

# BACKLOG — motiq-app

Tracks work not being done right now: bugs, UI improvements, and larger features. The
process is defined in CLAUDE.md Part C. This file is the source of truth; the running
block in chat is temporary and holds items only until they are flushed here.

## Status

| Status | Meaning |
|---|---|
| `open` | Raised and accepted. Not started. |
| `active` | Being worked on now. |
| `review` | Work is finished and awaiting Stella's judgment. Claude moves items here; Claude never closes. |
| `close` | Closed by Stella. Requires evidence in Closed-by. |
| `discard` | Will not be done. Requires the reason in Closed-by. |

## Category

| Category | Meaning |
|---|---|
| `bug` | Behaves incorrectly. |
| `ui` | Layout, wording, or interaction quality. |
| `task` | A task or condition change affecting what the battery does or records. |
| `data` | Capture, storage, or export of trial data. |
| `infra` | Repo, deploy, versioning, tooling. |
| `docs` | README and repo documentation. |

## Rules

- IDs are `BL-###`, assigned in cumulative sequence at flush, never reused.
- Claude never closes an item. Finished work goes to `review`.
- `close` and `discard` both require Closed-by: a PR number, or Stella's stated reason.
  Closed-by stays empty on every other status.
- Escape any `|` inside an Item cell, or it silently breaks the table.

## Items

| ID | Status | Category | Item | Raised | Closed-by |
|---|---|---|---|---|---|
