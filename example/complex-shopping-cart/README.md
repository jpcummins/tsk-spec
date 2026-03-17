# Complex Shopping Cart Example

This example models a multi-team shopping cart build with cross-team
dependencies, iterations, and SLA-tracked support/incident work.

## Layout
- `tasks/` contains all task records.
- `teams/` contains team iterations.
- `sla.toml` defines global SLA rules.

## Teams
- billing
- checkout
- backend
- frontend

## Notes
- Tasks are referenced by canonical paths relative to `tasks/`.
- Backend tasks use custom task statuses (see `tasks/shopping-cart/backend/config.toml`).
- Frontend iterations use custom iteration statuses (see `teams/frontend/team.toml`).
- Support and incident tickets live outside the core project and are tracked with SLAs.

## Useful Search Queries

These examples use the query language from the spec (Section 13). All queries
reference data in this example as of 2026-03-19. See `sla.toml` for
the SLA rules applied to support and incident tasks.

### Incident tickets with a breached SLA

The `incident-4h` rule already selects tasks matching
`summary ~ "incident"` with a 4-hour mitigation window. Querying SLA
status after evaluation only needs the rule ID and breach status:

```
sla.id = "incident-4h" AND sla.status = "breached"
```

| Task | Status | Assignee | SLA target | Breached? |
|---|---|---|---|---|
| `incidents/incident-2026-03-18-payment-failures` | in_progress | team:billing | 4h | yes |
| `incidents/incident-2026-03-18-cart-sync` | in_progress | alice@example.com | 4h | yes |
| `incidents/incident-2026-03-18-api-latency` | in_progress | team:backend | 4h | yes |

### Open tickets assigned to me

Using `me()` to resolve the current user (in this example,
`alice@example.com`):

```
assignee = me() AND status.category != done
```

| Task | Status | Estimate |
|---|---|---|
| `incidents/incident-2026-03-18-cart-sync` | in_progress | 4h |
| `incidents/incident-2026-03-18-cart-sync/fix-session-merge` | in_progress | 2h |
| `incidents/incident-2026-03-18-cart-sync/verify-sync-restored` | todo | 1h |

### Open tickets for my team's current iteration

Using `my_team()` to resolve the current user's teams. In this example,
`alice@example.com` is on the `backend` team (via `teams/backend/team.toml`),
so this finds tasks in Backend Iteration 1 (2026-03-17 to 2026-03-30):

```
iteration.team = my_team()
  AND iteration.start <= date("today")
  AND iteration.end >= date("today")
  AND status.category != done
```

Note: if the user belongs to multiple teams, `my_team()` expands to
match iterations for all of them.

| Task | Status | Assignee | Estimate |
|---|---|---|---|
| `shopping-cart/backend/cart-service-endpoints` | dev | — | 8h |
| `shopping-cart/backend/inventory-check` | queued | — | 6h |
| `incidents/incident-2026-03-18-cart-sync` | in_progress | alice@example.com | 4h |
| `incidents/incident-2026-03-18-api-latency` | in_progress | team:backend | 5h |

### Tickets that need to be triaged

Tasks assigned to a team (not yet assigned to a specific person) and
still open:

```
assignee ~ "team:" AND status.category != done
```

| Task | Status | Assignee |
|---|---|---|
| `incidents/incident-2026-03-18-payment-failures` | in_progress | team:billing |
| `incidents/incident-2026-03-18-payment-failures/apply-mitigation` | in_progress | team:billing |
| `incidents/incident-2026-03-18-payment-failures/postmortem` | todo | team:billing |
| `incidents/incident-2026-03-18-api-latency` | in_progress | team:backend |
| `incidents/incident-2026-03-18-api-latency/monitor-and-confirm` | todo | team:backend |

### Unassigned tasks

Tasks with no assignee at all:

```
missing(assignee) AND status.category != done
```

| Task | Status | Estimate |
|---|---|---|
| `support/support-bug-payment-timeout` | in_progress | 3h |
| `support/support-bug-cart-duplicates` | in_progress | 4h |
| `support/support-bug-email-receipts` | todo | 2h |
| `shopping-cart/frontend/cart-ui-shell` | in_progress | 6h |
| `shopping-cart/frontend/cart-item-quantity` | in_progress | 4h |
| `shopping-cart/frontend/promo-code-ui` | todo | 3h |
| `shopping-cart/frontend/cart-summary` | todo | 5h |
| `shopping-cart/backend/cart-service-endpoints` | dev | 8h |
| `shopping-cart/backend/pricing-engine` | queued | 10h |
| `shopping-cart/backend/inventory-check` | queued | 6h |
| `shopping-cart/billing/tax-calculation` | in_progress | 7h |
| `shopping-cart/billing/invoice-generation` | todo | 6h |
| `shopping-cart/billing/refund-flow` | todo | 8h |
| `shopping-cart/checkout/checkout-flow` | todo | 9h |
| `shopping-cart/checkout/payment-methods` | todo | 6h |
| `shopping-cart/checkout/order-confirmation` | todo | 4h |

### All open support bugs

```
path ~ "support/" AND status.category != done
```

| Task | Status | Estimate | SLA rule |
|---|---|---|---|
| `support/support-bug-payment-timeout` | in_progress | 3h | support-7d |
| `support/support-bug-cart-duplicates` | in_progress | 4h | support-7d |
| `support/support-bug-email-receipts` | todo | 2h | support-7d |

### All open work for the backend team (including individual assignments)

Using `team("backend")` for a specific team, or `my_team()` for the
current user's teams:

```
assignee = my_team() AND status.category != done
```

| Task | Status | Assignee |
|---|---|---|
| `incidents/incident-2026-03-18-cart-sync` | in_progress | alice@example.com |
| `incidents/incident-2026-03-18-cart-sync/fix-session-merge` | in_progress | alice@example.com |
| `incidents/incident-2026-03-18-cart-sync/verify-sync-restored` | todo | alice@example.com |
| `incidents/incident-2026-03-18-api-latency` | in_progress | team:backend |
| `incidents/incident-2026-03-18-api-latency/scale-checkout-service` | in_progress | bob@example.com |
| `incidents/incident-2026-03-18-api-latency/monitor-and-confirm` | todo | team:backend |

### Capitalizable tasks

The `shopping-cart/README.md` has `labels: ["capitalizable"]`. All
subtasks inherit this label automatically (labels use union semantics).
Incident tasks are *not* capitalizable — they have the `incident` label
instead. This query finds all capitalizable tasks and their estimates:

```
has(labels, "capitalizable")
```

| Task | Status | Estimate | Labels (effective) |
|---|---|---|---|
| `shopping-cart` | in_progress | — | capitalizable |
| `shopping-cart/backend/cart-service-endpoints` | dev | 8h | capitalizable |
| `shopping-cart/backend/pricing-engine` | queued | 10h | capitalizable |
| `shopping-cart/backend/inventory-check` | queued | 6h | capitalizable |
| `shopping-cart/frontend/cart-ui-shell` | in_progress | 6h | capitalizable |
| `shopping-cart/frontend/cart-item-quantity` | in_progress | 4h | capitalizable |
| `shopping-cart/frontend/promo-code-ui` | todo | 3h | capitalizable |
| `shopping-cart/frontend/cart-summary` | todo | 5h | capitalizable |
| `shopping-cart/billing/tax-calculation` | in_progress | 7h | capitalizable |
| `shopping-cart/billing/invoice-generation` | todo | 6h | capitalizable |
| `shopping-cart/billing/refund-flow` | todo | 8h | capitalizable, not-capitalizable |
| `shopping-cart/checkout/checkout-flow` | todo | 9h | capitalizable |
| `shopping-cart/checkout/payment-methods` | todo | 6h | capitalizable |
| `shopping-cart/checkout/order-confirmation` | todo | 4h | capitalizable |

Note: most subtasks inherit `capitalizable` from the parent
`shopping-cart/README.md` without declaring any labels themselves.
The `refund-flow` task explicitly adds `not-capitalizable` — since
labels use union semantics, it carries both labels.

### Capitalizable tasks (excluding opt-outs)

The `refund-flow` task is regulatory compliance work and has been
labeled `not-capitalizable`. Using `NOT` to exclude it:

```
has(labels, "capitalizable") AND NOT has(labels, "not-capitalizable")
```

| Task | Status | Estimate | Labels (effective) |
|---|---|---|---|
| `shopping-cart` | in_progress | — | capitalizable |
| `shopping-cart/backend/cart-service-endpoints` | dev | 8h | capitalizable |
| `shopping-cart/backend/pricing-engine` | queued | 10h | capitalizable |
| `shopping-cart/backend/inventory-check` | queued | 6h | capitalizable |
| `shopping-cart/frontend/cart-ui-shell` | in_progress | 6h | capitalizable |
| `shopping-cart/frontend/cart-item-quantity` | in_progress | 4h | capitalizable |
| `shopping-cart/frontend/promo-code-ui` | todo | 3h | capitalizable |
| `shopping-cart/frontend/cart-summary` | todo | 5h | capitalizable |
| `shopping-cart/billing/tax-calculation` | in_progress | 7h | capitalizable |
| `shopping-cart/billing/invoice-generation` | todo | 6h | capitalizable |
| `shopping-cart/checkout/checkout-flow` | todo | 9h | capitalizable |
| `shopping-cart/checkout/payment-methods` | todo | 6h | capitalizable |
| `shopping-cart/checkout/order-confirmation` | todo | 4h | capitalizable |

The `refund-flow` task is excluded even though it inherits
`capitalizable` from the parent — the `NOT` filter handles this at
query time without needing to remove the inherited label.

## Summary
- Shopping cart feature work spans four teams with cross-team dependencies.
- Each team runs four two-week iterations under `teams/`.
- Support bugs and incident tickets are modeled as standalone tasks.
- SLAs are defined in `sla.toml` for support and incident work.
