# Engineering decisions and tradeoffs

| Decision | Why | Cost or limit |
| --- | --- | --- |
| **ERPNext/Frappe as the base** | Reuse mature stock and accounting records while building commerce workflows around them. | Frappe and ERPNext conventions shape module boundaries, permissions, and deployment. |
| **One app, several domain modules** | Keep a business change and its event in one MariaDB transaction. | Code ownership and import boundaries need discipline; this is not independent-service deployment. |
| **Cursor polling is the intake backbone** | A webhook can be delayed, dropped, or disabled by a vendor; a cursor and replay window can recover. | Intake may lag up to the configured poll interval, and rate limits require planning. |
| **Webhook as an accelerator** | Reduce latency when a vendor sends a trustworthy notice. | Signature checks and ingress budgets remain necessary; the notice is not enough by itself. |
| **Database-backed inbox and outbox** | Preserve intent and business facts even if Redis or a worker disappears. | More rows and retention work than fire-and-forget jobs. |
| **No external write inside a user request** | A slow carrier cannot hold a packer or a database transaction hostage. | The interface must show pending, in-doubt, and failed states clearly. |
| **At-least-once delivery plus idempotent handlers** | Practical recovery from worker loss without pretending a network has exactly-once semantics. | Every handler and external action needs a stable key or a reconciliation path. |
| **Explicit in-doubt state** | A timed-out refund or label may already exist at the vendor. | Some cases require a person because a vendor cannot verify an outcome. |
| **One Parcel per physical box** | Labels, tracking, contents, and returns attach to the physical unit shipped. | Order-level UI must summarize multiple parcels. |
| **Rules and holds visible in the workflow** | A person can decide what to do when an order cannot proceed automatically. | More product design than a simple green/red automation status. |
| **Separate public sandbox** | Visitors can inspect real screens and roles without touching a merchant's records. | Sandbox behavior and test vendors must be described as such, not presented as production outcomes. |

## Deliberate limits

Kontor is an operational product, not a claim that all commerce cases are automated. A workflow can require an owner, packer, operations person, or tax adviser. Vendor capabilities differ; adapter declarations state what each connection can do. The public demo only proves the connected test flows and role screens visible there.

The architecture aims to make failure *legible*: every order should have a next action, every external call a state, and every uncertain outcome a recovery path. That is a stronger and more honest claim than “all integrations are reliable.”
