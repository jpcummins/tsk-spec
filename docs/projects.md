# Projects

Projects are directories within `tasks/`. A project can optionally have a `README.md` that represents the project itself.

## Project Structure

```text
tasks/
  launch/
    README.md       # The launch project
    setup.md        # A task in the launch project
    web-ui/
      README.md     # A subproject: launch/web-ui
      frontend.md   # A task in web-ui
      backend.md
```

- `tasks/launch/` is the `launch` project
- `tasks/launch/web-ui/` is the `launch/web-ui` subproject
- Each `.md` file that is not `README.md` is a task
- `README.md` represents the project directory itself

## Canonical Paths

Projects and tasks use the same canonical path system. A project's canonical path is its directory path relative to `tasks/`.

| Directory              | Canonical Path  |
| ---------------------- | --------------- |
| `tasks/launch/`        | `launch`        |
| `tasks/launch/web-ui/` | `launch/web-ui` |

## Project README

A project's `README.md` is a task that represents the project itself:

```markdown
---
created_at: 2026-03-17T09:00:00Z
summary: Launch the new product
status: in_progress
labels: ["epic"]
type: initiative
---
High-level goals for this launch.
```

The project README can have all the same fields as any other task.

## Front Matter

All fields are optional unless otherwise noted.

| Field          | Type                                    | Description                                            |
| -------------- | --------------------------------------- | ------------------------------------------------------ |
| `created_at`   | datetime                                | RFC3339 timestamp.                                     |
| `due`          | datetime                                | RFC3339 timestamp                                      |
| `assignee`     | string                                  | Person or [team](teams.md)                             |
| `dependencies` | list                                    | Canonical paths to predecessor tasks                   |
| `summary`      | string                                  | Short task description                                 |
| `estimate`     | [duration](tasks.md#duration-rules)     | Estimated effort (e.g., `2h`, `1.5d`)                  |
| `status`       | enum                                    | Custom status mapped to base category                  |
| `updated_at`   | datetime                                | RFC3339 timestamp                                      |
| `change_log`   | list                                    | List of `{field, from, to, at}` changes                |
| `labels`       | list                                    | List of [label identifiers](tasks.md#identifier-rules) |
| `type`         | [identifier](tasks.md#identifier-rules) | Task type identifier                                   |
| `weight`       | number                                  | Ordering weight (lower sorts first)                    |

## Nested Projects

Projects can nest arbitrarily deep:

```text
tasks/
  platform/
    infrastructure/
      networking.md
      security.md
    api/
      v1/
        endpoints.md
      v2/
        endpoints.md
```

Each directory is a project. You can add a `README.md` to any directory to represent that project as a task.
