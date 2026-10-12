# Security, data boundaries, and operations

## People, permissions, and records

Kontor has workflows for the owner, operations, packer, content, and tax adviser. Those are distinct jobs, not just different navigation menus. API methods check roles and, where relevant, Frappe document permissions and company access before returning records or accepting an action. Order details can mask address information by role. Cost and margin data belong to owner and finance roles; a packer receives the facts needed to pack.

The backend makes the permission decision. Hiding a button in the browser does not grant or deny access. A queued job may run as a system user to perform ERPNext bookkeeping after an authorized human action, but the human-facing endpoint still checks who may initiate that action.

## Vendor and customer data

An Integration Account binds a company, adapter, environment, and merchant identity. The queued event carries that identity. A later account edit cannot silently reinterpret old work as belonging to another merchant. Vendor DTOs are validated before domain services use them.

Integration Event payloads are encrypted with the Frappe site key and given an adapter-specific purge date. Secrets are fetched only when an adapter needs them, and tokens are cached with their identity. Logs and error events are designed around IDs, event names, and bounded error text rather than raw request bodies or buyer records. Correlation IDs let an operator find the related records without making every log line contain personal data.

The public sandbox is a separate demonstration with fictional company and buyer records. Its Shopify and DHL connections use test environments. Public role accounts do not include the Administrator account. Interviewers should use the [demo instructions on the product page](https://kontor.hammadbhutta.dev/#explore), not a credential in this repository.

## Ingress and egress

Webhooks carry an opaque account ingress key and are verified by the vendor adapter. A valid notice is stored once by external ID/version and normally advances a poll; it is not treated as the sole source of truth. The ingress layer applies request and unknown-key budgets, while the reverse proxy limits body size before Frappe parses a request. A request that reaches a webhook function by the wrong route is refused.

All vendor writes leave from workers. A marketplace or carrier outage therefore does not block the packer's HTTP request or hold an ERPNext transaction open across the network. Outbound calls inherit a correlation ID and are governed by topic-specific retry, rate-limit, and uncertainty rules.

## Observability and recovery

The system exposes per-account health based on poll lag, pause state, dead events, set-aside records, in-doubt calls, and expiring credentials. Structured logs carry release, account, event, order, and correlation identifiers where relevant. Prometheus metrics and Grafana dashboards cover integration and infrastructure behavior; an operations trace joins the calls, domain events, alerts, and log lines related to an order.

Recovery is intentionally visible:

| Problem | System response | Human action when needed |
| --- | --- | --- |
| Temporary vendor or network failure | Retry on a schedule, respecting rate limits. | Inspect if attempts run out. |
| Authentication rejected | Pause the account and alert. | Restore access, then resume. |
| One unreadable item in a page | Set that item aside; keep the rest and the cursor. | Retry the item after a fix or dismiss with a reason. |
| Worker dies before completing a claimed event | Tick finds the stuck claim and re-enqueues safe work. | Inspect only if repeated failures persist. |
| Non-idempotent external call may have succeeded | Mark in doubt; verify before any permitted resend. | Reconcile manually if the vendor cannot answer. |
| Domain-event handler fails | Its delivery records a retry/dead state independently. | Replay after the cause is corrected. |

The production recipe builds a pinned image and runs MariaDB, Redis, HTTP/WebSocket services, scheduler, and separate queue workers. Site data and secrets are mounted at runtime. Healthchecks and service supervision are part of the deployment; backup and restore are operational responsibilities, not a claim made by this case study.

## Verification strategy

The private codebase has pure tests for keys, retry schedules, DTOs, rules, and adapters; integration tests for claims, queue behavior, uncertain calls, poll failures, and permission boundaries; and role journey scenarios for owner, operations, packer, content, returns, and tax-adviser work. Fake adapters let workflows run without vendor credentials. Development and production-like stacks are separate, with a smoke/restore path for deployment changes.

This repository does not contain those tests or assert a coverage percentage. It documents the behaviors they target. In a technical interview, I can walk through the relevant code paths and explain the failure cases behind them.
