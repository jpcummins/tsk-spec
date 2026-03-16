# tsk spec

Version: 1.2.0

## Versioning
- The spec follows semantic versioning.
- Breaking changes require a major version bump.
- Backwards-compatible changes require a minor version bump.
- Fixes and clarifications require a patch version bump.

This document defines the data model and file conventions for the `tsk` system.
It is implementation-agnostic and focused on structure, fields, and semantics.

## 1. Storage Model
- All records are Markdown files with front matter (header matter).
- Front matter format is YAML (Hugo-compatible).
- Directories represent hierarchical nodes. A directory may have an implied README.
- `README.md`, when present, represents the parent task for the directory.
- A task file may also have a subdirectory (node and leaf are both allowed).
- Canonical task paths are rooted at the `tasks/` directory.
- Stub files redirect moved tasks to their new paths.
- Top-level layout:
  - `.config.toml` is optional (root configuration; may include `version`).
  - `tasks/` is required and is the root for canonical task paths.
  - `teams/` is optional.
  - `sla.toml` is optional.

## 2. Identity and Addressing
- The canonical identity for a task is the extensionless path relative to
  `tasks/`.
- Task identity is derived from the canonical path; there are no explicit IDs.

### 2.1 Canonical Path Normalization
- Input is the path from `tasks/` to the file or directory.
- Normalize path separators to `/` and trim leading/trailing `/`.
- Lowercase the path, except for the special `README.md` filename which may be upper or lower case (both are valid).
- Remove the file extension for canonical paths.
- Replace spaces with `-` inside each path segment.
- Remove non-alphanumeric characters within each segment except `-`.
- Preserve `/` between segments.
- Resulting path is the canonical identity.
- Example: `Launch/Phase 1/Implement CLI.md` ->
  `launch/phase-1/implement-cli`.

### 2.2 Canonical Identity (Path-Based)
- Canonical paths omit file extensions.
- Example: `launch/phase-1/implement-cli` corresponds to
  `tasks/launch/phase-1/implement-cli.md`.
- Canonical paths must be lowercase to avoid filesystem casing differences.
- Directory container paths never include a trailing slash.
- If a directory and task file share the same canonical path, the directory
  container takes precedence; implementations should warn.
- Implementations should warn when on-disk paths include uppercase characters
  on case-sensitive filesystems, except for `README.md` which may be upper or
  lower case.
- If both lowercase and uppercase variants of the same path exist on a
  case-sensitive filesystem (except for `README.md` which allows both), the
  lowercase path takes precedence and the uppercase variant is ignored.

### 2.3 README Canonical Path
- The canonical path for a directory node maps to the directory itself.
- `README.md` does not add a path segment to the canonical identity.
- Example: `tasks/launch/README.md` resolves to `launch`.

### 2.3 Identifier Syntax
- A **label** is a lowercase alphanumeric string with `-` allowed. It must match `[a-z0-9][a-z0-9-]*`.
- A **type** follows the same rules as a label.
- This identifier format is defined here once and referenced throughout the spec.

### 2.4 Redirect Stub Files
- Redirect stubs are the only reference mechanism in the system.
- When a task is moved, the old path may optionally remain as a stub file. This is an implementation decision.
- If a task needs to appear in multiple locations, use a redirect stub instead of duplicating the task file.
- Stub files are Markdown files with front matter containing `redirect_to`.
- `redirect_to` must be a canonical path (no file extension) relative to
  `tasks/`.
- Stub files contain no task fields other than `redirect_to`.
- Stub file body content, if present, is ignored during resolution.
- Stub files are excluded from roll-ups, reporting, and search results.
- Resolution: if a path is a stub, resolve immediately to `redirect_to`.
- Redirect chaining is allowed with a maximum depth of 3; exceeding this is an error.
- `redirect_to` may point to a task or a directory node.

#### Redirect stub example
```markdown
---
redirect_to: launch/phase-1/implement-cli
---
```

#### Using a redirect stub to reference a task from multiple locations
```markdown
---
redirect_to: platform/tasks/enable-sso
---
```

## 3. Ordering
- Default ordering is lexicographic by filename.
- `weight` in front matter overrides ordering (lower first).

## 4. Status Model
- Base categories: `todo`, `in_progress`, `done`.
- Custom statuses must map to a base category in config.
- Status is always explicit (no inheritance unless configured defaults apply).
- Iterations use the same base categories but have a separate status mapping.

## 5. Task Entity
Tasks are the atomic unit. A task is a Markdown file with front matter.

### Required fields
- `date` (RFC3339 timestamp)

### Optional fields
- `due` (RFC3339 timestamp)
- `assignee` (string; a person or team — see **Assignee Semantics**)
- `dependencies` (canonical paths relative to `tasks/`, without file extensions)
- `summary` (use Hugo summary behavior)
- `estimate` (duration tokens like `2h`, `1.5d`)
- `status` (custom enum mapped to base categories)
- `updated_at` (RFC3339 timestamp)
- `change_log` (ordered list of `{field, from, to, at}` changes; tracks changes to any header field)
- `labels` (list of identifiers; see **Identifier Syntax**)
- `type` (identifier; see **Identifier Syntax**)
- `weight`

### Reference Resolution
- Dependency paths must resolve through redirect stubs.

### Assignee Semantics
- `assignee` is a single string representing the current owner of a task.
- A task is assigned to either a person or a team, never both.
- Person values are an email address or alphanumeric username token
  (e.g., `"alex@example.com"`, `"jp"`).
- Team values use the `team:` prefix followed by a team directory name
  (e.g., `"team:backend"`).
- The team name in a `team:` value must correspond to a directory under
  `teams/`.
- Reassignment (e.g., team triages and assigns to a person) is modeled by
  updating the `assignee` value.

### Labels
- `labels` is an optional list of freeform strings.
- Labels are case-insensitive for query matching.
- Labels inherit from parent tasks with union semantics: a child's labels
  are merged with its parent's labels, not replaced. This is an exception
  to the general array replacement rule in Section 10.
- Labels inherit automatically — no `[inherit]` opt-in is required.
- A label on a project `README.md` applies to all subtasks in that
  directory tree.
  - Queryable via `has(labels, "value")` for membership checks.

### Example
```markdown
---
date: 2026-03-14T09:30:00Z
updated_at: 2026-03-16T12:00:00Z
due: 2026-04-01T17:00:00Z
assignee: "alex@example.com"
dependencies: ["launch/plan", "bootstrap"]
summary: "Ship CLI MVP"
estimate: "12h"
labels: ["capitalizable", "mvp"]
status: "up_next"
change_log:
  - field: status
    from: todo
    to: up_next
    at: 2026-03-15T10:00:00Z
---
Notes here...
```

## 6. Subtasks and Hierarchy
- Subtasks are tasks located inside a directory.
- Directories can nest, enabling multi-level subtasks.
- If a task file has the same base name as its parent directory, the directory
  container takes precedence in hierarchy resolution.
- Implementations should warn when a task shares the same name as its parent
  directory.
- Cross-links between tasks are allowed via redirect stubs.

## 7. Projects
- A project is a directory node; its `README.md` (if present) is a task.
- Projects and tasks share the same schema; no separate project entity.
- Projects may contain subtasks and nested subprojects.

## 8. Teams
- Teams are global and live under `teams/`.
- Each team has a directory at `teams/<team>/`.
- `teams/<team>/README.md` is optional.
- Team-level configuration is stored in `teams/<team>/team.toml`.

### Team Members
- Team members are listed in `team.toml` under the `members` key.
- `members` is an array of strings in `"First Last <email@example.com>"` format.
- The `members` field is optional.

### team.toml example
```toml
members = [
  "Alice Smith <alice@example.com>",
  "Bob Jones <bob@example.com>",
]
```

## 9. Iterations
- Iterations live under `teams/<team>/iterations/`.
- An iteration is a Markdown file with ordered task references.
- Task references are canonical paths relative to `tasks/` (no file extensions).
- A task may belong to multiple iterations.
- Iteration status is explicit (not computed).
- Iteration `team` is derived from the directory path; an explicit `team` field is optional.

### Iteration Status
- Iterations use the same base categories (`todo`, `in_progress`, `done`).
- Iterations have their own configurable status mapping with `order`.
- Iteration status mapping is defined in `team.toml` under
  `[iteration.status.map]`.

### Required fields
- `start` (RFC3339)
- `end` (RFC3339)
- `status`
- `tasks` (ordered list of canonical paths)

### Optional fields
- `name`
- `capacity` (duration tokens like `80h`)

### Example
```markdown
---
name: "2026 Week 12"
team: platform
start: 2026-03-16T00:00:00Z
end: 2026-03-22T23:59:59Z
status: in_progress
capacity: "80h"
tasks:
  - launch/m2/implement-cli
  - platform/tasks/enable-sso
---
Goals and notes.
```

## 10. Configuration and Inheritance
- Configuration file format: TOML.
- Project configuration files are named `.config.toml`.
- Team configuration files are named `team.toml` and live under `teams/<team>/`.
- Project configuration files live in project directories and apply to that
  directory subtree.
- A global configuration may exist at the repository root (`.config.toml`).

### Repository Version
- The root `.config.toml` may include a `version` field specifying the
  spec version the repository conforms to (semver string, e.g., `"0.12.0"`).
- `version` is only valid at the repository root; it is ignored in
  subdirectory configs.
- `version` is optional. If absent, implementations should assume the
  latest version they support and warn.
- Implementations should error if the repository version is newer than
  what they support.
- Implementations may use `version` to apply migrations when upgrading
  a repository to a newer spec version.

### Inheritance
- When multiple configs apply, the most specific (nearest) config takes
  precedence.
- Configurations are deep-merged by section; nearest values override outer values.
- Arrays and lists are replaced (not merged) by nearer configs.
  Exception: `labels` use union semantics (see Section 5, Labels).
- Configuration can define default field values.
- Defaults apply to all tasks in the current project directory and recursively.
- Task front matter overrides inherited values.
- Inheritance is opt-in by field (only fields listed in config are inherited).
- Status enums are configured here and must map to base categories.
- Iteration status enums are configured separately and must map to base categories.

### Configuration example
```toml
version = "0.12.0"

[defaults]
assignee = "jp@example.com"
status = "up_next"
estimate = "4h"

[inherit]
assignee = true
status = true
estimate = true

[status.map]
todo = { category = "todo", order = 10 }
up_next = { category = "todo", order = 20 }
in_progress = { category = "in_progress", order = 30 }
review = { category = "in_progress", order = 40 }
done = { category = "done", order = 50 }

[iteration.status.map]
todo = { category = "todo", order = 10 }
queued = { category = "todo", order = 20 }
in_progress = { category = "in_progress", order = 30 }
stabilizing = { category = "in_progress", order = 40 }
done = { category = "done", order = 50 }
```

## 11. Roll-ups
- Implementations may compute derived metrics (e.g., percent complete,
  estimate totals) from subtask data.
- The spec does not prescribe roll-up behavior; this is an
  implementation concern.

## 12. SLAs
- SLA rules are defined globally in `sla.toml`.
- Tasks may explicitly select or override SLA rules in front matter.
- SLA applicability is defined by a search query in the rule.
- SLA start/stop timing is configurable per rule.
- If multiple SLA rules match a task, reporting behavior is configurable.

### 12.0 SLA Rule Storage
- SLA rules live in `sla.toml` at the repository root.
- SLA rules are global in scope; they do not support per-directory overrides.
- When multiple rules match a task, reporting configuration determines whether
  to apply all rules or prioritize.

#### SLA storage example
```
/
  sla.toml                      # global rules
  tasks/
    launch/
      phase-1/
        implement-cli.md
```

### 12.0.1 SLA Rule Schema (TOML)
```toml
[[rule]]
id = "security-30d"
name = "Security fixes in 30 days"
query = 'task.summary ~ "security"'
target = "30d"
start = "date"
stop = "status:done"
severity = "high"

[[rule]]
id = "ops-respond"
name = "Ops triage in 4h"
query = 'task.summary ~ "ops"'
target = "4h"
start = "status:up_next"
stop = "status:in_progress"
severity = "medium"
```

#### Rule fields
- `id` (string): unique within the file.
- `name` (string): human-readable label.
- `query` (string): SLA query language expression (Section 12.1).
  SLA rule queries may reference `task.*` and `iteration.*` fields
  but must not reference `sla.*` fields, as SLA results do not exist
  at rule evaluation time.
- `target` (duration): allowed time window.
- `start` (string): start event (`date`, `due`, `status:<value>`).
- `stop` (string): stop event (`status:<value>` or `due`).
- `severity` (string): reporting label (e.g., `low`, `medium`, `high`).

#### SLA Event Types
- `date`: use `task.date`.
- `due`: use `task.due`.
- `status:<value>`: use the most recent transition into the given status value
  from `change_log`.
- If `change_log` is absent, use `updated_at` as the transition time when
  `task.status` matches the requested value.
- If `status` is set and `updated_at` is missing, status-based SLA timing
  cannot be enforced and the task is skipped for status-based SLA evaluation.
- Default SLA rule application is apply all matching rules.
- Status values may be custom task statuses; they map to base categories via
  status mapping.

## 12.1 SLA Query Language (DSL)
The SLA query language is a domain-specific language for selecting tasks.
It is declarative, human-readable, and stable across implementations.

### 12.1.1 Goals
- Express task selection using task fields and derived attributes.
- Support boolean logic, grouping, and negation.
- Be safe and deterministic (no side effects, no execution).

### 12.1.2 Grammar (Conceptual)
- `expr := term ( ("AND" | "OR") term )*`
- `term := "NOT"? (predicate | "(" expr ")")`
- `predicate := field op value | function`

### 12.1.2.1 Grammar (BNF)
```
<expr>       ::= <term> ( ("AND" | "OR") <term> )*
<term>       ::= ["NOT"] ( <predicate> | "(" <expr> ")" )
<predicate>  ::= <field> <op> <value> | <function>
<op>         ::= "=" | "!=" | "<" | "<=" | ">" | ">=" | "~" | "IN"
<value>      ::= <string> | <number> | <date> | <duration> | <list>
<list>       ::= "[" <value> ("," <value>)* "]"
<function>   ::= <ident> "(" [<args>] ")"
<args>       ::= <value> ("," <value>)*
```

### 12.1.2.2 Precedence and Associativity
- Precedence: `NOT` > `AND` > `OR`.
- Operators of the same precedence evaluate left-to-right.

### 12.1.3 Operators
- Equality: `=`
- Inequality: `!=`
- Ordering: `<`, `<=`, `>`, `>=` (for dates, numbers, durations)
- Containment: `~` (substring or token contains, case-insensitive)
- Set membership: `IN` (comma-separated list)

### 12.1.3.1 Operator Type Rules
- Ordering operators apply only to comparable types: dates, numbers, and durations.
- Enumerations (e.g., `status`, `status.category`) support only `=`, `!=`, and `IN`.
- Non-comparable predicates (e.g., `status < "foo"`) are invalid and must error.
- Predicates against missing fields are no-ops and evaluate to false.

### 12.1.4 Fields
These fields are available for queries:
- Fields may be namespaced by entity: `task.<field>` or `iteration.<field>`.
- Unqualified fields apply to tasks by default.
- `sla.<field>` is available only in reporting contexts after SLA
  evaluation. SLA fields are task-scoped — they attach to the task that
  matched the SLA rule. They compose with `task.*` and `iteration.*`
  predicates using the same rules below.
- When a query references both `task.*` and `iteration.*` fields, it
  returns tasks that belong to at least one iteration satisfying the
  iteration predicates. The iteration's `tasks` list defines membership.
  Task predicates (including `sla.*`) are then applied to filter the
  matching tasks.
- Task fields:
  - `task.status` (custom status)
  - `task.status.category` (base category: `todo`, `in_progress`, `done`)
  - `task.assignee` (person or `team:` prefixed team name)
  - `task.due` (RFC3339)
  - `task.date` (RFC3339)
  - `task.updated_at` (RFC3339)
  - `task.estimate` (duration)
  - `task.path` (canonical path relative to `tasks/`, no file extension)
  - `task.summary`
  - `task.type` (identifier; see **Identifier Syntax**)
  - `task.dependency` (matches any dependency path)
  - `task.labels` (matches any label value)
- Iteration fields:
  - `iteration.team` (team directory name)
  - `iteration.status` (custom status)
  - `iteration.status.category` (base category: `todo`, `in_progress`, `done`)
  - `iteration.start` (RFC3339)
  - `iteration.end` (RFC3339)
  - `iteration.path` (canonical path relative to `tasks/`, no file extension)
- SLA fields (reporting only):
  - `sla.id` (SLA rule id)
  - `sla.status` (enum: `ok`, `at_risk`, `breached`)
  - `sla.target` (duration)
  - `sla.elapsed` (duration)
  - `sla.remaining` (duration)

### 12.1.4.1 Field Type Reference
- `task.status`: enum (custom status value)
- `task.status.category`: enum (`todo`, `in_progress`, `done`)
- `task.assignee`: string (person or `team:` prefixed team name)
- `task.due`: datetime (RFC3339)
- `task.date`: datetime (RFC3339)
- `task.updated_at`: datetime (RFC3339)
- `task.estimate`: duration
- `task.path`: string (canonical path relative to `tasks/`)
- `task.summary`: string
- `task.type`: identifier
- `task.dependency`: list of strings (canonical paths)
- `task.labels`: list of strings
- `iteration.team`: string (team directory name)
- `iteration.status`: enum (custom status value)
- `iteration.status.category`: enum (`todo`, `in_progress`, `done`)
- `iteration.start`: datetime (RFC3339)
- `iteration.end`: datetime (RFC3339)
- `iteration.path`: string (canonical path relative to `tasks/`)
- `sla.id`: string
- `sla.status`: enum (`ok`, `at_risk`, `breached`)
- `sla.target`: duration
- `sla.elapsed`: duration
- `sla.remaining`: duration

### 12.1.5 Functions
- `exists(field)`
- `missing(field)`
- `has(field, value)` (list membership; e.g., dependency)
- `date(value)` (convert to RFC3339; e.g., `date("yesterday")`)
- `team(name)` (expand to match `"team:<name>"` or any member listed in
  `teams/<name>/team.toml` `members`; for use with `assignee`)
- `me()` (resolve to the current user's identifier; for use with
  `assignee`. Identity resolution is implementation-defined — e.g.,
  git config, session, or environment.)
- `my_team()` (resolve to all teams the current user belongs to, based
  on `members` in each `team.toml`. Expands to match `"team:<name>"`
  and all members for every team the user is in. A user may belong to
  multiple teams.)

### 12.1.6 Values
- Strings can be quoted with double quotes, e.g. `"security"`.
- Dates use RFC3339, e.g. `2026-04-01T17:00:00Z`.
- Relative date strings are allowed only inside `date(...)`.
- Durations follow estimate tokens, e.g. `12h`, `1.5d`.

### 12.1.6.1 Duration Units
- Supported units: `m` (minutes), `h` (hours), `d` (days), `w` (weeks).
- Durations may be positive or negative (negative values indicate past offsets).

### 12.1.7 Examples
- All security tasks with a 30-day SLA:
  `summary ~ "security" AND status.category != done`
- Tasks assigned to specific people and due this week:
  `assignee IN ["alex@example.com", "jp"] AND due <= 2026-03-22T23:59:59Z`
- Tasks assigned to the backend team (awaiting triage):
  `assignee = "team:backend" AND status.category != done`
- All open tasks for the backend team (including individual assignments):
  `assignee = team("backend") AND status.category != done`
- Tasks depending on a specific file:
  `dependency = "launch/plan"`
- Unestimated in-progress tasks:
  `status.category = in_progress AND missing(estimate)`
- Explicit iteration status query:
  `iteration.status = in_progress AND iteration.start <= date("today")`
- SLA breach reporting query:
  `sla.id = "security-30d" AND sla.status = "breached"`
- Completed capitalizable tasks:
  `has(labels, "capitalizable") AND status.category = done`

### 12.1.8 Redirect Stub Handling
- Queries operate on resolved canonical tasks.
- Stub files are not matchable records and are excluded from search results.
- If a query input path references a stub, it resolves to the target task before matching.

## 13. Reporting Expectations
- Reporting spans tasks, projects, initiatives, teams, iterations, and SLAs.
- Virtual nodes (directories without README.md) appear in reports as normal.

## 14. Search as a Core Primitive
- Search is a first-class capability across CLI, web UI, and API.
- The SLA query language is the baseline DSL for search across the system.
- All major views should be expressible as saved or ad-hoc queries.
