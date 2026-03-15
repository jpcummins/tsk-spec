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
- Backend tasks use custom task statuses (see `tasks/shopping-cart/backend/.config.toml`).
- Frontend iterations use custom iteration statuses (see `teams/frontend/.config.toml`).
- Support and incident tickets live outside the core project and are tracked with SLAs.

## Summary
- Shopping cart feature work spans four teams with cross-team dependencies.
- Each team runs four two-week iterations under `teams/`.
- Support bugs and incident tickets are modeled as standalone tasks.
- SLAs are defined in `sla.toml` for support and incident work.
