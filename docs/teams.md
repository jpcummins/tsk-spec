# Teams

Teams are optional and live under `teams/`.

## Structure

```text
teams/
  backend/
    team.toml
```

- Team directory names are [identifiers](basics.md#identifier-rules).
- Team config lives in `teams/<team>/team.toml`.

## Team Members

Members are defined in a `[members]` table:

```toml
[members]
alice = "Alice Smith <alice@example.com>"
bob = "Bob Smith"
carol = "carol@example.com"
```

- Key (`alice`, `bob`, `carol`) is the member identifier.
- Member identifiers follow the standard [identifier](basics.md#identifier-rules) format.
- Values may be:
  - `"First Last <email@example.com>"`
  - `"First Last"`
  - `"email@example.com"`

## Assignee Resolution

- Tasks may assign to a team (`assignee: "team:backend"`) or to a member identifier (`assignee: "alice"`).
- Member identifiers are resolved across all teams.
- If the same identifier appears in multiple teams with different values, implementations should warn and pick the lexically first team directory.
- If an identifier cannot be resolved, implementations may emit a warning or info.

## Examples

### Team Definition

```toml
# teams/backend/team.toml
[members]
alice = "Alice Smith <alice@example.com>"
bob = "Bob Jones"
charlie = "charlie@example.com"
```

# 
