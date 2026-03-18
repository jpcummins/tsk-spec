# tsk

tsk is a minimal file-based project management
specification. It is a bit of a rebellion against the bloat of
Jira, ClickUp, and Linear. This repository defines the specification of a tsk repository and contains documentation and examples. This repo does not contain an implementation.

The core idea is simple:

- Tasks are markdown files
- Tasks belong to projects, which are directories with an optional readme.md

A simple tsk repository looks like this:

```
/
  config.toml
  tasks/
    project-a/
      README.md
      task-1.md
      task-2.md
    project-b/
      README.md
      task-1.md
      task-2.md
      task-3.md
      project-b-big-subtask/
        README.md
        task-1.md
        task-2.md
        task-3.md
```

Optionally, you can add additional functionality (see [shopping cart example](example/complex-shopping-cart/)):

- [teams](docs/teams.md)
- [iterations](docs/iterations.md)
- [SLAs](docs/sla.md)

The primary way to interact with tsk is via the [query interface](docs/search.md), though, implementations are free to abstract this away to make a clean UI for non-technical users. For example, to find all open bugs in the `support` project:

```
path ~ "support/" AND status.category != done
```

tsk repositories are intended to live in their own git repo, separate
from application code, to keep the application's git history clean. This is not required; the system is flexible enough to live alongside a codebase in the same repo.

## Implementations

- **[tsk-cli](https://github.com/jpcummins/tsk-cli)** - Terminal UI with queries and fuzzy search
- **[tsk-lib](https://github.com/jpcummins/tsk-lib)** - Go library for parsing and querying tsk repositories

## Examples

- [example/minimal-todo](example/minimal-todo/) shows a minimal solo TODO list.
- [example/complex-shopping-cart](example/complex-shopping-cart/) shows a multi-team, SLA-tracked project.

## Docs

- [docs/README.md](docs/README.md) - Guide to the human-friendly documentation.
- [SPEC.md](SPEC.md) - Formal normative specification.

## Future Plans

- A CLI tool to create, update, and delete tasks
- A web frontend with advanced reporting
- Comments
