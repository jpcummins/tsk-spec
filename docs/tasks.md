# Tasks

Tasks are the atomic unit of tsk. Each task is a Markdown file with YAML front matter.

## Task File

```markdown
---
created_at: 2026-03-17T10:00:00Z
summary: Build user login
status: todo
---
Implement OAuth2 login flow.
```

The body of the file (after the front matter) contains detailed notes, specifications, or context.

## Front Matter

All fields are optional unless otherwise noted.

| Field          | Type                            | Description                                                    |
| -------------- | ------------------------------- | -------------------------------------------------------------- |
| `created_at`   | datetime                        | RFC3339 timestamp.                                             |
| `due`          | datetime                        | RFC3339 timestamp                                              |
| `assignee`     | string                          | Person or [team](teams.md) (e.g., `"alice"`, `"team:backend"`) |
| `dependencies` | list                            | Canonical paths to predecessor tasks                           |
| `summary`      | string                          | Short task description                                         |
| `estimate`     | [duration](#duration-rules)     | Estimated effort (e.g., `2h`, `1.5d`)                          |
| `status`       | enum                            | Custom status mapped to base category                          |
| `updated_at`   | datetime                        | RFC3339 timestamp                                              |
| `change_log`   | list                            | List of `{field, from, to, at}` changes                        |
| `labels`       | list                            | List of [label identifiers](#identifier-rules)                 |
| `type`         | [identifier](#identifier-rules) | Task type identifier                                           |
| `weight`       | number                          | Ordering weight (lower sorts first)                            |

## Canonical Paths

Canonical paths are the identity of a task. They are:

- Rooted at `tasks/` and omit file extensions
- Normalized and lowercased
- `README.md` resolves to the directory name

| File                              | Canonical Path           |
| --------------------------------- | ------------------------ |
| `tasks/launch/setup.md`           | `launch/setup`           |
| `tasks/launch/README.md`          | `launch`                 |
| `tasks/launch/web-ui/frontend.md` | `launch/web-ui/frontend` |

## Assignee

Tasks can be assigned to a person or a team:

```markdown
# Assign to a specific person
assignee: "alice"

# Assign to a team (awaiting triage)
assignee: "team:backend"
```

- Person values: email address or member identifier from [team.toml](teams.md)
- Team values: `team:<team-name>`

## Dependencies

Tasks can declare dependencies on other tasks:

```markdown
---
created_at: 2026-03-17T10:00:00Z
dependencies: [launch/setup, launch/build]
summary: Deploy the application
---
```

Dependencies use canonical paths. A task cannot be considered complete until its dependencies are complete.

### Redirect Stubs

Use redirect stubs when a task moves:

```markdown
---
redirect_to: platform/tasks/enable-sso
---
```

## Identifier Rules

Identifiers are used by `labels`, `type`, team names, SLA IDs, etc.

- Regex: `[a-z0-9][a-z0-9-]*`
- Lowercase alphanumeric with optional `-`

## Duration Rules

Duration format is `<number><unit>` with no spaces.

- Examples: `2h`, `1.5d`, `2w`, `3m`
- Units: `h` (hours), `d` (days), `w` (weeks), `m` (months)

## Examples

### Minimal Task

```markdown
---
created_at: 2026-03-17T10:00:00Z
summary: Build user login
status: todo
---
Implement OAuth2 login flow.
```

### Task with Dependencies

```markdown
---
created_at: 2026-03-17T10:00:00Z
dependencies:
  - backend/api-design
  - backend/auth-service
  - infra/database
summary: Create frontend login page
assignee: "team:frontend"
---
Wire up the login form to the API.
```
