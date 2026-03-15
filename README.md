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

## Vision and Requirements
- Replace heavier task tracking systems (e.g., ClickUp, Linear) with a developer-first tool.
- Minimal, text-file based (Markdown) with front matter.
- Git as the version control system for project data.
- CLI-first interaction; future web UI for reporting and org-level visibility.

## Core Principles
- Plain text storage; diff-friendly and human-readable.
- Minimal schema, but strong reporting capability.
- Works for single-person projects up to multi-team, multi-organization scale.
- `tsk` data lives in a separate git repo from work code.
- Eventually integrate with external repos (possibly via submodules or references).

## Open Questions / TBD
- Integration approach with external repos (submodules vs references).
- Roll-up strategy configurations and UX.
- CLI command surface and behavior.
