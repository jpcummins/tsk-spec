# Iterations

Iterations are time-boxed periods during which a team works on a set of tasks.

## Structure

```text
teams/
  backend/
    team.toml
    iterations/
      2026-03-17.md
```

- Iterations live under `teams/<team>/iterations/`.
- An iteration is a Markdown file with ordered task references.
- The [team](teams.md) is derived from the directory path.
- Task references are [canonical paths](basics.md#canonical-paths) relative to `tasks/` (no file extensions).
- A task may belong to multiple iterations.

## Iteration ID

The iteration ID is derived from the file path: `<team>/<filename>` (lowercase, no extension).

Example: `teams/backend/iterations/2026-03-17.md` → `backend/2026-03-17`

Use the body of the file for descriptive text.

## Fields

| Field   | Type       | Description                           |
| ------- | ---------- | ------------------------------------- |
| `start` | RFC3339    | Start of the iteration time window.   |
| `end`   | RFC3339    | End of the iteration time window.     |
| `tasks` | list       | Ordered list of canonical task paths. |

## Example

```markdown
---
start: 2026-03-16T00:00:00Z
end: 2026-03-22T23:59:59Z
tasks:
  - launch/m2/implement-cli
  - platform/tasks/enable-sso
---
Let's ship SSO!
```
