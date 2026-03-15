# TSK Spec

TSK is a fast, developer-first, non-proprietary, file-based project management
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

The intent is for task tracking data to live in its own repository, separate
from application code, to keep git history clean. This is not required; the
system is flexible enough to live alongside a codebase in the same repo.

## Principles
- Fast by default; no heavy infrastructure required.
- Developer-first UX; text is the source of truth.
- Non-proprietary formats and portable data.
- Minimal surface area with strong reporting.
- Scales from solo dev workflows to large multi-org teams.
- No API or servers required; just edit plain text files.
- A clear spec enables robust implementations for non-technical users.

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
