# Developer API

[← Documentation index](README.md) · [SafeDesk overview](../README.md) · [Live guide](https://safedesk.business/docs/developer-api)

SafeDesk exposes a REST API under `https://safedesk.business/api/v1/`. **The SafeDesk web application is built on this API**, so everything visible in the dashboard is available programmatically — there is no private back channel that the browser uses and integrators cannot.

Live references for the production installation:

| | |
|---|---|
| OpenAPI 3 schema | `GET https://safedesk.business/api/schema/` |
| Interactive Swagger UI | [safedesk.business/api/docs/](https://safedesk.business/api/docs/) |
| Health | `GET https://safedesk.business/api/v1/health/` |

<img src="../assets/business-webhook-guide.png" width="100%" alt="SafeDesk developer documentation"/>

## Authentication model

Requests use a **same-origin `HttpOnly` session cookie plus CSRF protection**. The server — not the browser — is the authority for authentication, organization isolation, permissions, billing and audit records.

```http
GET  /api/v1/auth/csrf/     → { "data": { "csrf_token": "…" } }  (also sets the csrftoken cookie)
POST /api/v1/auth/login/    → { "email" | "username", "password", "otp"? }
                              200 user · 202 { "mfa_required": true }
                                      · 202 { "mfa_setup_required": true } for staff
GET  /api/v1/auth/me/       → current user + memberships (organization_id, organization_name, role)
POST /api/v1/auth/logout/
POST /api/v1/auth/mfa/setup/  ·  /mfa/confirm/
POST /api/v1/auth/password-reset/  ·  /password-reset/confirm/  ·  /verify-email/
POST /api/v1/register/      → creates user + organization (owner membership)
```

Sign-in is rate-limited at the edge to **10 requests per minute per IP**. Sessions last **12 hours** and are **revoked server-side when a password changes**.

## Organization scope

Every authenticated tenant request must carry `X-Organization-ID`. The server verifies membership and scopes the objects it returns; the header is a *request* for a scope, not a grant of one.

```js
const response = await fetch("/api/v1/devices/?page_size=50", {
  credentials: "same-origin",
  headers: {
    Accept: "application/json",
    "X-Organization-ID": "<organization-uuid>",
  },
});
```

Writes (`POST`, `PUT`, `PATCH`, `DELETE`) additionally require:

```http
Content-Type: application/json
X-CSRFToken: <csrf-token>
```

Platform staff may act inside a customer workspace by sending `X-Organization-ID`; **without it**, workspace-scoped endpoints such as `/dashboard/` and `/agents/` answer `400 organization_required`. Cross-tenant access requires staff permission *and* deliberate organization filtering — it is never implicit.

> Never place cookies or CSRF tokens in URLs, logs, `localStorage`, source code or documentation.

## Resource surface

| Collection | Notes |
|---|---|
| `organizations/` | Owners/admins may change `name`, `contact_email` and `ip_restriction_enabled`; everything else is staff-only |
| `memberships/` | Filter by `role` |
| `ip-allowlists/` | CIDR entries; see [IP restrictions](ip-restrictions.md) |
| `plans/` | **Publicly readable**, with a full per-term pricing array |
| `subscriptions/` `balances/` `ledger/` `provisioning/` `notifications/` `audit-events/` | **Read-only** in the tenant API |
| `orders/` `payments/` | Purchase and checkout |
| `device-sources/` | Webhook URL + secret, shown once |
| `devices/` | Read-only **except** the notes action |
| `downloads/` | Releases; administration needs owner/admin or staff |
| `agents/` | Issue, rotate, revoke agent credentials |
| `support/messages/` · `support/guest/messages/` · `support/threads/` | Customer · visitor · staff |
| `users/` · `platform-settings/` | Staff only |

### Notable custom actions

```http
POST /api/v1/orders/                        { "plan": <id>, "term_months": 1|2|3|6|12 }
POST /api/v1/orders/{id}/checkout/          { "asset": "USDT", "network": "tron",
                                              "success_url", "cancel_url" }
                                            → { "data": { "checkout_url": "https://drogopay.com/pay/…" } }
POST /api/v1/orders/{id}/pay-from-balance/
POST /api/v1/balances/top-up/
POST /api/v1/device-sources/                → webhook_url + webhook_secret, returned once
POST /api/v1/device-sources/{id}/rotate-secret/
POST /api/v1/agents/                        → username, email, generated_password, returned once
POST /api/v1/agents/{id}/rotate-password/
PATCH /api/v1/devices/{id}/notes/           { "notes": "…" }
PATCH /api/v1/organizations/{id}/           { "ip_restriction_enabled": true }
GET  /api/v1/dashboard/                     → devices: { total, online, managed,
                                                          over_limit, limit,
                                                          usage_percent, limit_warning }
GET  /api/v1/entitlement/                   → plan state, trial state, device limit
```

Hosted checkout returns a URL — **fulfilment happens only when DrogoPay's signed webhook confirms the payment.** See [billing and payments](billing-and-payments.md).

### Releases and resumable uploads

A release carries a `kind` of `file` or `link`. A **link-only** release sends users to the customer's own download page instead of a hosted installer, and its public payload sets `download_url` to `null` so the card offers *Open download page*. Every release needs at least one of a file or a `website_url`.

Installers above a few megabytes use a resumable upload session:

```http
POST  /api/v1/downloads/uploads/            { filename, size, platform, version,
                                              organization?, website_url?, replace? }
                                            → { received, chunk_size, … }
PATCH /api/v1/downloads/uploads/{id}/chunk/  raw application/octet-stream
                                             Upload-Offset: <received>
POST  /api/v1/downloads/uploads/{id}/complete/   hashes, ClamAV-scans, publishes
```

A `409 offset_mismatch` includes the `offset` to resume from, which makes the flow safe across network drops and page reloads. Abandoned sessions are purged after 24 hours.

Platform identifiers are exactly `windows`, `macos`, `android` and `agent`. Do not infer other operating systems, architectures or package formats.

### The company download code

Every company has a short **download code** of 2–3 characters (for example `A8V`). End users enter it on [safedesk.org/downloads](https://safedesk.org/downloads) — or open `https://safedesk.org/downloads?code=A8V` — and receive exactly the installers SafeDesk operators published for that company rather than the standard public builds. Users without a code see only public builds; company installers are never listed or downloadable without the code, and an unknown code returns *not found*.

**A download code is a routing label, not authentication.** Requests are rate-limited to slow guessing, but the code should never be treated as a secret.

## Pagination

List endpoints use **cursor pagination**, default page size 50, maximum requested `page_size` 200:

```json
{
  "data": [],
  "page": {
    "next": "https://safedesk.business/api/v1/devices/?cursor=<opaque>",
    "previous": null
  }
}
```

Follow the returned URL. Cursors are opaque and ordering-dependent — they are not permanent bookmarks. Some custom actions return `data` without `page` because they are not paginated.

## Errors

```json
{
  "error": {
    "code": "invalid_request",
    "message": "Request validation failed.",
    "details": { "field": ["<validation-message>"] }
  }
}
```

Handle the machine-readable code and the HTTP status together. Known integration codes include `invalid_request`, `invalid_signature`, `method_not_allowed`, `not_found`, `payload_too_large`, `ip_not_allowed`, `subscription_required` and `internal_error`; framework-generated authentication, permission, throttling and validation codes may also appear.

Retry only safe operations. For `429` or retryable `5xx`, use bounded exponential backoff with jitter and honour `Retry-After`. **Never blindly retry a state-changing request without an idempotency guarantee.**

## Filtering and ordering

Standard DRF conventions apply — `?search=`, `?ordering=-created_at`, `?page_size=100` — with per-resource allowlists:

| Resource | Filters |
|---|---|
| `devices` | `online`, `enabled`, `platform`, `source` |
| `memberships` | `role` |
| `subscriptions` | `status` |
| `ledger` | `kind` |
| `orders` | `status`, `plan` |
| `audit-events` | `action`, `target_type` |

Consult the schema rather than sending arbitrary fields.

## Real-time notifications

```text
wss://safedesk.business/ws/v1/notifications/?organization=<organization-uuid>
```

The same session cookie authenticates the socket, and the **organization IP allowlist is enforced on it**. Support messages also arrive here as `{ "type": "support.message", … }`.

## Webhooks are a separate trust boundary

Webhook routes live **outside** `/api/v1/` and use neither browser sessions nor CSRF:

```text
POST https://safedesk.business/api/webhooks/drogopay/
POST https://safedesk.business/api/webhooks/rustdesk/devices/{source_uuid}
```

Each has its own exact-raw-body HMAC contract. **Never send browser cookies to these routes.** See [the signed device webhook](device-webhook.md) and [billing and payments](billing-and-payments.md).

## Public, unauthenticated endpoints

```text
GET /api/v1/public/releases/            (optionally ?code=<download-code>)
GET /api/v1/public/releases/{id}/download/
GET /api/v1/health/
GET /api/v1/plans/
```

These are what [safedesk.org](https://safedesk.org/) consumes. The public feed lists only releases that operators marked public and that passed the ClamAV scan.

## Compatibility and hardening

API evolution follows **expand → migrate → contract**, so clients must ignore additive response fields. On the integrator's side: validate TLS, content type, schemas, URLs and identifiers; cap response size and timeouts; redact cookies and headers from logs; and keep production and test organizations — and their secrets — strictly separate.

## Related

- [The signed device webhook](device-webhook.md) — the inbound integration contract
- [Billing and payments](billing-and-payments.md) — orders, checkout, ledger
- [Security model](security-model.md) — the full error contract and control list
