# tsk

tsk is a fast, developer-first, non-proprietary, file-based project management
spec. It is intentionally minimal and a bit of a rebellion against the bloat of
Jira, ClickUp, and Linear. The core idea is simple: tasks and project data live
in plain text files, tracked in git, with tooling built around a stable, open
specification. It should scale from a single-developer workflow to a complex
multi-org setup at a large company.

This repository defines the specification for multiple implementations (CLI,
web UI, API, and others). The spec is versioned using semantic versioning so
breaking changes are clearly communicated.

This repo does not contain an implementation. It is for specs, documentation,
and examples for the task management system.

The intent is for a tsk repository to live in its own git repo, separate
from application code, to keep git history clean. This is not required; the
system is flexible enough to live alongside a codebase in the same repo.

## Implementations

- **[tsk-cli](https://github.com/jpcummins/tsk-cli)** - Terminal UI with DSL queries and fuzzy search
- **[tsk-lib](https://github.com/jpcummins/tsk-lib)** - Go library for parsing and querying tsk repositories


## Principles
- Instantaneous search.
- Text is the source of truth.
- Clear specification for humans and AI.
- Scales from solo dev workflows to large multi-org teams.

## Spec and Versioning
- The formal spec lives in `SPEC.md`.
- The spec follows semantic versioning.
  - Breaking changes require a major version bump.
  - Backwards-compatible changes require a minor version bump.
  - Fixes and clarifications require a patch version bump.
 - Canonical task paths are rooted at `tasks/`.

## Examples
- `example/minimal-todo/` shows a minimal solo TODO list.
- `example/complex-shopping-cart/` shows a multi-team, SLA-tracked project.
