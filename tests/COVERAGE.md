# Conformance Coverage

This document tracks test coverage against `SPEC.md` section references.

## Current Coverage Summary

- Total test files: 21
- Total test cases: 90
- DSL test cases (Section 7.\*): 44

## Covered Sections

| Spec Ref | Cases | Files                                                                                                                             |
| -------- | ----: | --------------------------------------------------------------------------------------------------------------------------------- |
| `1.1`    |     2 | `tests/cases/1-basics/storage-and-task-entity.toml`                                                                               |
| `1.2.1`  |     2 | `tests/cases/1-basics/paths.toml`                                                                                                 |
| `1.2.2`  |     2 | `tests/cases/1-basics/paths.toml`                                                                                                 |
| `1.2.3`  |     2 | `tests/cases/1-basics/paths.toml`                                                                                                 |
| `1.2.4`  |     4 | `tests/cases/1-basics/identifiers.toml`                                                                                           |
| `1.2.5`  |     3 | `tests/cases/1-basics/stubs.toml`                                                                                                 |
| `1.3`    |     2 | `tests/cases/1-basics/model.toml`                                                                                                 |
| `1.4`    |     1 | `tests/cases/1-basics/model.toml`                                                                                                 |
| `1.5`    |     2 | `tests/cases/1-basics/storage-and-task-entity.toml`                                                                               |
| `1.6`    |     1 | `tests/cases/1-basics/model.toml`                                                                                                 |
| `1.7`    |     1 | `tests/cases/1-basics/model.toml`                                                                                                 |
| `2`      |     5 | `tests/cases/2-teams/assignee.toml`                                                                                               |
| `3`      |     5 | `tests/cases/3-iterations/status.toml`                                                                                            |
| `4`      |     4 | `tests/cases/4-config/parse.toml`                                                                                                 |
| `6.0`    |     2 | `tests/cases/6-sla/storage.toml`                                                                                                  |
| `6.0.1`  |     4 | `tests/cases/6-sla/evaluate.toml`                                                                                                 |
| `7.2`    |     2 | `tests/cases/7-query/syntax-errors.toml`                                                                                          |
| `7.3`    |     4 | `tests/cases/7-query/precedence.toml`                                                                                             |
| `7.4`    |     6 | `tests/cases/7-query/operators.toml`                                                                                              |
| `7.4.1`  |     3 | `tests/cases/7-query/operator-errors.toml`                                                                                        |
| `7.4.2`  |     6 | `tests/cases/7-query/cross-entity.toml`                                                                                           |
| `7.5`    |    12 | `tests/cases/7-query/iteration-queries.toml`, `tests/cases/7-query/sla-fields.toml`, `tests/cases/7-query/fields-and-values.toml` |
| `7.6`    |     1 | `tests/cases/7-query/syntax-errors.toml`                                                                                          |
| `7.6.1`  |     4 | `tests/cases/7-query/functions.toml`                                                                                              |
| `7.6.2`  |     3 | `tests/cases/7-query/functions.toml`                                                                                              |
| `7.7`    |     3 | `tests/cases/7-query/fields-and-values.toml`, `tests/cases/7-query/syntax-errors.toml`                                            |
| `8`      |     2 | `tests/cases/8-reporting/virtual-nodes.toml`                                                                                      |
| `9`      |     2 | `tests/cases/9-search/core.toml`                                                                                                  |

## Error/Warning Code Coverage

Every code in `SPEC.md` Section 10 has at least one test assertion.

- `QUERY_INVALID_SYNTAX` - covered in `tests/cases/7-query/syntax-errors.toml`
- `QUERY_INVALID_OPERATOR` - covered in `tests/cases/7-query/operator-errors.toml`
- `QUERY_OR_CROSS_NAMESPACE` - covered in `tests/cases/7-query/cross-entity.toml`
- `QUERY_UNKNOWN_FIELD` - covered in `tests/cases/7-query/fields-and-values.toml`
- `QUERY_UNKNOWN_FUNCTION` - covered in `tests/cases/7-query/syntax-errors.toml`
- `QUERY_INVALID_VALUE` - covered in `tests/cases/7-query/syntax-errors.toml`, `tests/cases/7-query/fields-and-values.toml`
- `QUERY_UNQUALIFIED_SLA` - covered in `tests/cases/7-query/sla-fields.toml`
- `PATH_UPPERCASE` - covered in `tests/cases/1-basics/paths.toml`
- `PATH_CASE_CONFLICT` - covered in `tests/cases/1-basics/paths.toml`
- `REDIRECT_CHAIN_TOO_DEEP` - covered in `tests/cases/1-basics/stubs.toml`
- `REDIRECT_INVALID_TARGET` - covered in `tests/cases/1-basics/stubs.toml`
- `ASSIGNEE_UNKNOWN_MEMBER` - covered in `tests/cases/2-teams/assignee.toml`
- `ASSIGNEE_MEMBER_CONFLICT` - covered in `tests/cases/2-teams/assignee.toml`
- `SLA_STATUS_MISSING_UPDATED_AT` - covered in `tests/cases/6-sla/evaluate.toml`

## Notes on Indirect Coverage

- Section `5` (Roll-ups) is intentionally implementation-defined; no strict
  pass/fail assertions are required by the spec.
- Section `10` is covered through code assertions across the suite rather than
  dedicated standalone tests.
