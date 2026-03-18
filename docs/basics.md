# Basics

This guide explains the core tsk model.

## Core Idea

- [Tasks](tasks.md) are Markdown files with front matter. 
- [Projects](projects.md) are directories. A project can have a `README.md` that represents the project itself.
- Tasks and projects live under `tasks/`.

## Example

```text
/
  tasks/
    launch/
      README.md
      setup.md
      build-cli.md
      web-ui/
        README.md
        frontend.md
        backend.md
    ops/
      monitoring.md
      alerting.md
```

### Example Task File

This is the task at `tasks/launch/setup.md`:

```markdown
---
created_at: 2026-03-17T10:00:00Z
summary: Set up CI/CD pipeline
status: in_progress
assignee: "alice"
labels: ["priority"]
due: 2026-03-20T17:00:00Z
estimate: "4h"
---
Configure GitHub Actions for automated testing and deployment.
```

The canonical path is `launch/setup`.

## Example Queries

```text
# All open tasks in the launch project (matches launch/setup)
path ~ "launch/" AND status.category != done

# Tasks assigned to me (matches launch/setup if current user is alice)
assignee = me() AND status.category != done
```

See [Search](search.md) for the full query language.

 

## What's Next

- [Tasks](tasks.md) — task file structure and fields
- [Projects](projects.md) — directory structure and nested projects
- [Teams](teams.md) — assigning tasks to people or teams
- [Search](search.md) — querying your task repository
