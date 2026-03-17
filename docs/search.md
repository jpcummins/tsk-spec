# Search Query Language

tsk search uses a DSL for filtering tasks, iterations, and SLA results.

## Core Concepts

- Unqualified fields are task fields by default.
- Use `task.*`, `iteration.*`, and `sla.*` when needed.
- Boolean operators: `AND`, `OR`, `NOT`.
- Operator precedence: `NOT` > `AND` > `OR`.

## Common Operators

| Operator             | Meaning              |
| -------------------- | -------------------- |
| `=`                  | Equality             |
| `!=`                 | Inequality           |
| `<`, `<=`, `>`, `>=` | Ordering             |
| `~`                  | Contains (substring) |
| `IN`                 | Membership           |

## Useful Functions

| Function            | Description                       |
| ------------------- | --------------------------------- |
| `exists(field)`     | True if field is present          |
| `missing(field)`    | True if field is absent           |
| `has(field, value)` | True if list field contains value |
| `date(value)`       | Convert to RFC3339 (see [basics.md](basics.md#duration-rules)) |
| `team(name)`        | Expand to team members            |
| `me()`              | Current user identifier           |
| `my_team()`         | All teams current user belongs to |

## Field Reference

### Task Fields

| Field             | Type       | Description                                     |
| ----------------- | ---------- | ----------------------------------------------- |
| `status`          | enum       | Custom status value                             |
| `status.category` | enum       | Base category: `todo`, `in_progress`, or `done` |
| `assignee`        | string     | Person or `team:` prefixed team name            |
| `due`             | datetime   | RFC3339 timestamp                               |
| `created_at`      | datetime   | RFC3339 timestamp                               |
| `updated_at`      | datetime   | RFC3339 timestamp                               |
| `estimate`        | [duration](basics.md#duration-rules) | Estimated effort (e.g., `2h`, `1.5d`) |
| `path`            | string     | Canonical path relative to `tasks/`             |
| `summary`         | string     | Task description                                |
| `type`            | [identifier](basics.md#identifier-rules) | Task type identifier |
| `labels`          | list       | List of label identifiers                       |
| `dependencies`    | list       | List of [canonical paths](basics.md#canonical-paths) |

### Iteration Fields

| Field             | Type       | Description                              |
| ----------------- | ---------- | ---------------------------------------- |
| `iteration.id`    | string     | Derived identifier (`<team>/<filename>`, see [iterations.md](iterations.md#iteration-id)) |
| `iteration.team`  | [identifier](basics.md#identifier-rules) | Team identifier |
| `iteration.start` | datetime   | RFC3339 timestamp                        |
| `iteration.end`   | datetime   | RFC3339 timestamp                        |

### SLA Fields

| Field           | Type     | Description                    |
| --------------- | -------- | ------------------------------ |
| `sla.id`        | string   | SLA rule id                    |
| `sla.status`    | enum     | `ok`, `at_risk`, or `breached` |
| `sla.target`    | [duration](basics.md#duration-rules) | Target duration |
| `sla.elapsed`   | [duration](basics.md#duration-rules) | Elapsed time |
| `sla.remaining` | [duration](basics.md#duration-rules) | Remaining time |

## Examples

### By Assignee

| Description                                                            | Query                                                                      |
| ---------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| My open tasks                                                          | `assignee = me() AND status.category != done`                              |
| Tasks assigned to specific people and due this week                    | `assignee IN ["alex@example.com", "jp"] AND due <= 2026-03-22T23:59:59Z`   |
| Tasks assigned to the backend team (awaiting triage)                   | `assignee = "team:backend" AND status.category != done`                    |
| All open tasks for the backend team (including individual assignments) | `assignee = team("backend") AND status.category != done`                   |
| Overdue tasks across my teams                                          | `assignee = my_team() AND due < date("today") AND status.category != done` |

### By Content

| Description                        | Query                                              |
| ---------------------------------- | -------------------------------------------------- |
| All security tasks                 | `summary ~ "security" AND status.category != done` |
| Tasks depending on a specific file | `has(dependencies, "launch/plan")`                 |
| Bug tasks without a due date       | `type = "bug" AND NOT exists(due)`                 |
| All tasks under a project subtree  | `path ~ "launch/phase-1"`                          |

### By Labels

| Description                                     | Query                                                                                    |
| ----------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Completed capitalizable tasks in the last month | `has(labels, "capitalizable") AND status.category = done AND updated_at >= date("-30d")` |

### By Estimate

| Description                                       | Query                                                 |
| ------------------------------------------------- | ----------------------------------------------------- |
| Unestimated in-progress tasks                     | `status.category = in_progress AND missing(estimate)` |
| Tasks created in the last 7 days with no estimate | `created_at >= date("-7d") AND missing(estimate)`     |

### By Iteration

| Description                       | Query                                                                                                                            |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Tasks in current iteration        | `iteration.team = "backend" AND iteration.start <= date("today") AND iteration.end >= date("today") AND status.category != done` |
| Active iteration (all teams)      | `iteration.start <= date("today") AND iteration.end >= date("today")`                                                            |
| Active iteration for backend team | `iteration.team = "backend" AND iteration.start <= date("today") AND iteration.end >= date("today")`                             |
| Last iteration for a team         | `iteration.id = "backend/2026-03-09" AND status.category != todo`                                                                |
| Current sprint blocked tasks      | `iteration.start <= date("today") AND iteration.end >= date("today") AND status = "blocked"`                                     |

### By SLA

| Description                    | Query                                                 |
| ------------------------------ | ----------------------------------------------------- |
| Breached SLAs                  | `sla.status = "breached"`                             |
| SLA breach reporting           | `sla.id = "security-30d" AND sla.status = "breached"` |
| At-risk SLAs for platform team | `sla.status = "at_risk" AND path ~ "platform/"`       |
