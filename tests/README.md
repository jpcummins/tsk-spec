# tsk Conformance Tests

This directory contains language-agnostic conformance tests for the tsk specification.

## Test Structure

Tests are defined in TOML files under `tests/cases/`. Each file may contain multiple test cases using the `[[test]]` array syntax.

## Running Tests

A test runner should:

1. **Load test cases** from all `.toml` files in `tests/cases/` (recursive)
2. **Parse fixtures** from `test.files` into an in-memory filesystem
3. **Execute operations** as specified in each test case
4. **Compare results** against `test.expect`, `test.warnings`, and `test.errors`
5. **Report results** with pass/fail status and semantic error/warning matching

## Operations

| Operation                 | Description                                   | Expected Inputs                                     |
| ------------------------- | --------------------------------------------- | --------------------------------------------------- |
| `normalize_path`          | Normalize a path to canonical form            | `path` (string)                                     |
| `resolve_stub`            | Resolve a redirect stub to its target         | `path` (string)                                     |
| `resolve_task`            | Resolve a task path to its canonical identity | `path` (string)                                     |
| `resolve_hierarchy`       | Resolve directory/file hierarchy precedence   | `path` (string)                                     |
| `validate_identifier`     | Validate identifier syntax                    | `value` (string), optional `kind`                   |
| `run_query`               | Execute a DSL query against the task set      | `query` (string), optional `namespace`              |
| `order_tasks`             | Order child tasks by weight then filename     | `paths` (array), optional `weights` (table)         |
| `evaluate_sla`            | Evaluate SLA rules against tasks              | `task_path` (string), optional `time`               |
| `derive_iteration_status` | Compute iteration status from start/end       | `start` (RFC3339), `end` (RFC3339), `now` (RFC3339) |
| `parse_config`            | Parse and validate a config file              | `path` (string), `content` (string)                 |
| `resolve_assignee`        | Resolve an assignee to person/team            | `assignee` (string)                                 |
| `generate_report`         | Generate reporting view from tasks/projects   | `scope` (string)                                    |

## Semantic Matching

Warnings and errors are matched **semantically**, not by exact string:

- **code**: Must match a code from SPEC.md Section 10
- **level**: Must be `warn` or `error`
- **message_contains**: Optional substring match (case-insensitive)

Example:

```toml
[[test.errors]]
code = "QUERY_OR_CROSS_NAMESPACE"
level = "error"
message_contains = "namespace"
```

## Expected Results

### normalize_path

- `canonical_path` (string): The normalized path
- `warnings` (optional): Array of expected warnings

### run_query

- `matches` (array): Array of matching canonical paths
- `warnings` (optional): Array of expected warnings
- `errors` (optional): Array of expected errors

### resolve_stub

- `target` (string): The resolved redirect target
- `chain_depth` (number): Current chain depth during resolution

### resolve_hierarchy

- `resolved_kind` (string): `directory` or `file`
- `resolved_path` (string): Canonical resolved path

### validate_identifier

- `valid` (boolean): Whether identifier matches spec syntax

### order_tasks

- `ordered_paths` (array): Paths in final sorted order

### evaluate_sla

- `status` (string): `ok`, `at_risk`, or `breached`
- `elapsed` (duration): Elapsed time
- `remaining` (duration): Remaining time (may be negative)
- `warnings` (optional): Array of expected warnings

### derive_iteration_status

- `status` (string): `todo`, `in_progress`, or `done`

### parse_config

- `version` (optional): Parsed version string
- `status_map` (optional): Parsed status mappings
- `warnings` (optional): Array of expected warnings

### resolve_assignee

- `type` (string): `person` or `team`
- `value` (string): Resolved value (email, name, or team name)
- `warnings` (optional): Array of expected warnings

### generate_report

- `nodes` (optional): Reported nodes
- `virtual_nodes` (optional): Virtual directory nodes
- `teams` (optional): Team identifiers in report scope

## Test File Naming

Test files should be named to match their spec section:

- `tests/cases/1-basics/` - Sections 1.1 through 1.7
- `tests/cases/2-teams/` - Section 2
- `tests/cases/3-iterations/` - Section 3
- `tests/cases/4-config/` - Section 4
- `tests/cases/6-sla/` - Section 6
- `tests/cases/7-query/` - Section 7 (largest)
- `tests/cases/8-reporting/` - Section 8
- `tests/cases/9-search/` - Section 9

## Error Codes

See SPEC.md Section 10 for the canonical list of error and warning codes.
