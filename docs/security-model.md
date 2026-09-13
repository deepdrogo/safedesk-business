# Security model

[← Documentation index](README.md) · [SafeDesk overview](../README.md) · [Live guide](https://safedesk.business/docs/security-and-troubleshooting)

SafeDesk uses layered controls: TLS at the edge, secure server-side sessions, CSRF protection, tenant checks, role-based permissions, optional organization IP allowlists, signed webhooks, immutable audit records and transactional persistence.

**No individual control guarantees security.** That sentence is in the product's own documentation, and it is the honest frame for everything below.

<img src="../assets/business-security.png" width="100%" alt="The SafeDesk security posture"/>

## What is claimed

| Control | Implementation |
|---|---|
| **Session hardening** | `HttpOnly`, `Secure` session cookies with CSRF protection on every write. Sessions last 12 hours and are revoked server-side when a password changes. |
| **Password storage** | Argon2 hashing, 12-character minimum. Owner, admin, billing and viewer passwords are one-way hashed and unrecoverable. |
| **Operator MFA** | TOTP required for every SafeDesk operations account; optional for organization users. |
| **Rate limits** | Per-route limits that return `Retry-After` guidance instead of dropping silently. Sign-in is limited at the edge. |
| **Edge only** | Cloudflare-fronted edge; the origin application listener is never exposed directly. |
| **Hidden staff surface** | `/admin/` deliberately returns `404`. Staff administration lives on an environment-configured random path behind MFA, RBAC, rate limits, IP policy and auditing. |
| **Secrets at rest** | Webhook, source and agent secrets are encrypted at rest with the installation's field-encryption key, and shown exactly once where applicable. |
| **Tenant isolation** | The server is the authority. Every organization request carries `X-Organization-ID`; membership is checked server-side. |
| **Signed inbound integrations** | Device and payment webhooks require exact-raw-body HMAC-SHA256, a fresh timestamp, header/body consistency and constant-time comparison. |
| **Immutable audit** | Actor, organization, target, source IP, status and time on every privileged action. |
| **Append-only money** | Ledger entries can never be updated or deleted. Balance writes take row locks; a negative-going debit is refused. |
| **Transactional fulfilment** | Delivery deduplication and business mutation share one transaction — no half-applied payment. |
| **Backups** | Encrypted, tested backups. |
| **Data handling** | Designed around GDPR principles. Data location is as configured by the operator. |

## What is deliberately not claimed

- **No certifications are asserted.** SafeDesk does not claim SOC 2, ISO 27001 or any other attestation it does not hold. What it publishes instead is the control list above and the audit trail to verify it.
- **Audit records are control-plane records.** They are not a substitute for endpoint forensic logging.
- **`online` is not a network probe.** It is the latest accepted source value, and it expires after three minutes without an event.
- **A `2xx` on a webhook means accepted, not fulfilled.** Money and access change only on the specific verified events documented in [billing and payments](billing-and-payments.md).
- **Liveness and readiness checks prove little.** Liveness confirms the application process responds; readiness also checks the database. Neither proves that queues, workers, external payment processing, every source or all customer workflows are healthy.
- **A checksum is not a signature.** It detects file changes; it does not prove publisher identity on its own.
- **IP policy is a supplement.** It does not replace strong passwords, MFA, session hygiene, device authorisation or audit review.

## The credential separation that matters most

| Credential | Where it lives | Where it does **not** live |
|---|---|---|
| Source-server API token | Only on the customer's own source server | **Never in SafeDesk.** Not requested, not accepted, not stored. Configuration payloads containing one are refused. |
| Per-source webhook secret | Customer's secret manager, plus an encrypted copy in SafeDesk | Never in a URL, a log line, a ticket, shell history or client-side code |
| DrogoPay API and webhook secrets | SafeDesk backend configuration only | Never in the browser application |
| Agent password | Encrypted at rest, readable by the issuing workspace's owner/admin | Never readable by another agent |
| Session cookie | `HttpOnly` browser cookie | Never in JavaScript-accessible storage, a URL or a support ticket |

The point of the first row: a control plane holding an API token for every customer's server is one breach away from controlling every customer's fleet. SafeDesk removes that class of incident by never possessing the token. The worst case for one customer is a receive-side HMAC secret, which allows writing device records into that single organization — and nothing else.

## Security baseline for customers

- Use individual accounts and least privilege.
- Require MFA wherever the workflow supports it, especially for staff.
- Keep browsers, clients, agents and operating systems current.
- Verify downloads by origin, SHA-256 checksum and publisher signature.
- Restrict organization access to known egress CIDRs where operationally safe.
- Rotate source and payment webhook secrets after suspected exposure.
- Keep secrets in server-side secret managers, never in frontend code.
- Synchronise clocks on API, worker, proxy and source hosts.
- Retain the minimum necessary device and audit data.
- Test backups and incident procedures.

## Error contract

Application API errors use a single shape, so a client can branch on a machine-readable code rather than parsing prose:

```json
{ "error": { "code": "<machine-readable-code>",
             "message": "<safe-summary>",
             "details": {} } }
```

| Status | Typical cause |
|---|---|
| `401` | Missing or expired session, or an invalid webhook signature |
| `402 subscription_required` | No active plan on the workspace |
| `403` | CSRF, role, membership or IP policy denial |
| `404` | Outside scope, absent, inactive or intentionally hidden |
| `405` | Wrong HTTP method |
| `409` | State conflict |
| `413 payload_too_large` | Source webhook body above 2 MB |
| `429` | Throttled — honour `Retry-After` |
| `5xx` | Temporary; retry only safe or idempotent operations with bounded exponential backoff and jitter |

For `400`, inspect `error.details` for field validation. Raw exception data is never exposed to end users.

## Incident response

If compromise is suspected:

1. End active sessions and isolate affected endpoints if it is safe to do so.
2. Revoke user sessions and access.
3. Disable affected device sources.
4. Rotate exposed source, payment, application or signing credentials.
5. Preserve audit events, webhook delivery IDs, request IDs, timestamps and relevant endpoint logs.
6. Determine the organization and device scope.
7. Restore from a known-good state and notify affected parties as required.
8. Record the root cause and corrective actions.

**Never put passwords, session cookies, API credentials, HMAC secrets or signatures, private keys, or full customer payloads into a ticket or a chat message.** Masking a displayed value is not the same as rotating it.

## Escalation information

When reporting a problem, provide: the UTC timestamp and timezone, the affected organization and resource ID, the route and HTTP status, the safe error code and a redacted request or delivery ID, expected versus observed behaviour, any recent configuration change, and whether the problem reproduces. Omit secrets and minimise personal and device data.

## Abuse and lawful use

SafeDesk is a remote-support product for **authorised** access only. Installing an agent does not by itself establish authorisation — endpoint-owner approval, user notice and any consent required by law or policy come first.

SafeDesk reviews reported or detected misuse and may suspend or terminate a business account, disable registrations and access, preserve relevant account, billing, payment and audit records, and cooperate with competent law-enforcement, regulatory and government authorities worldwide in response to valid lawful requests. Each business customer is solely responsible for unlawful or malicious use by its users, employees, contractors and agents.

See the [Acceptable Use Policy](https://safedesk.business/legal/acceptable-use) and [Compliance Policy](https://safedesk.business/legal/compliance).

## Related

- [Architecture](architecture.md) — where each boundary sits
- [The signed device webhook](device-webhook.md) — the inbound contract
- [IP restrictions](ip-restrictions.md) — the network layer
- [Roles and permissions](roles-and-permissions.md) — the identity layer
