# SLA Rules

SLA rules are defined globally in `sla.toml`.

## Structure

Rules are an array of TOML tables (`[[rule]]`):

```toml
[[rule]]
id = "incident-4h"
name = "Incidents mitigated in 4 hours"
query = 'task.summary ~ "incident"'
target = "4h"
warn_at = "2h"
start = "status:in_progress"
stop = "status:done"
severity = "high"
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | [identifier](basics.md#identifier-rules) | yes | Unique rule identifier |
| `name` | string | yes | Human-readable rule name |
| `query` | string | yes | DSL expression to select matching tasks |
| `target` | [duration](basics.md#duration-rules) | yes | Breach threshold duration |
| `warn_at` | [duration](basics.md#duration-rules) | no | At-risk threshold duration |
| `start` | string | yes | Start event (`due` or `status:<value>`) |
| `stop` | string | yes | Stop event (`due` or `status:<value>`) |
| `severity` | [identifier](basics.md#identifier-rules) | yes | Reporting label identifier |

## Status Outcomes

When SLA timing is active:

- `breached` if elapsed time is greater than `target`
- `at_risk` if `warn_at` is set and elapsed time is greater than `warn_at`
- `ok` otherwise

## Validation Notes

- `warn_at` must be between `0` and `target` (inclusive).
- SLA IDs and severity values follow [identifier](basics.md#identifier-rules) rules.

## Examples

### Incident SLA Rule

```toml
[[rule]]
id = "incident-4h"
name = "Incidents mitigated in 4 hours"
query = 'task.summary ~ "incident"'
target = "4h"
warn_at = "2h"
start = "status:in_progress"
stop = "status:done"
severity = "critical"
```

### Support Bug SLA Rule

```toml
[[rule]]
id = "support-7d"
name = "Support bugs resolved in 7 days"
query = 'has(labels, "support")'
target = "7d"
start = "status:in_progress"
stop = "status:done"
severity = "medium"
```
