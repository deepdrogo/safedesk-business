# Roles and permissions

[← Documentation index](README.md) · [SafeDesk overview](../README.md) · [Live guide](https://safedesk.business/docs/agents-and-access)

An **organization** is SafeDesk's tenant boundary. Devices, sources, members, subscriptions, balances, ledger entries, orders and audit activity all belong to exactly one organization. The server enforces that boundary — hiding an item in the interface is not an authorization control.

<img src="../assets/business-roles.png" width="100%" alt="The SafeDesk roles and permissions matrix"/>

## The five roles

| Role | Intended use |
|---|---|
| `owner` | Account ownership, recovery and full organization administration |
| `admin` | Day-to-day organization and access administration |
| `billing` | Commercial review and billing operations |
| `agent` | Authorised device work without administrator access |
| `viewer` | Read-only oversight |

Owners can do everything; every other role is a strict subset.

| Capability | Owner | Admin | Billing | Agent | Viewer |
|---|:---:|:---:|:---:|:---:|:---:|
| View devices, status and notes | ✅ | ✅ | 👁️ | ✅ | 👁️ |
| Edit device notes | ✅ | ✅ | ✅ | ✅ | ✅ |
| Create agents and issue credentials | ✅ | ✅ | ❌ | ❌ | ❌ |
| Plans, orders, balance and checkout | ✅ | ✅ | ✅ | ❌ | ❌ |
| IP restriction switch and trusted networks | ✅ | ✅ | ❌ | ❌ | ❌ |
| Members and roles | ✅ | ✅ | ❌ | ❌ | ❌ |
| Audit log and activity | ✅ | ✅ | 👁️ | ❌ | 👁️ |
| Support chat with operators | ✅ | ✅ | ✅ | ✅ | ✅ |

<sub>👁️ = read-only.</sub>

## SafeDesk staff are not a customer role

Platform staff never hold a customer role. They act through a separate **Operations** workspace with:

- a staff identity distinct from any customer account;
- mandatory TOTP multi-factor authentication;
- explicit, deliberate organization scope on every cross-tenant action;
- re-authentication on sensitive actions;
- full audit recording.

Staff are exempt from a customer's IP allowlist — which is what makes operator-assisted lockout recovery possible — and they are the only party who can create device sources, publish releases, grant trial terms or post a balance adjustment.

## Agents

Agents are organization members intended for **delegated device work**. They are not administrators and must never receive shared administrator credentials.

**Credentials are issued, not chosen.** An agent cannot pick its own password. On creation, SafeDesk returns a generated password exactly once. Owners and admins can afterwards:

- read the password back from **Dashboard → Agents → Credentials**;
- replace it with a chosen one;
- rotate it to a fresh random value.

Agent passwords are encrypted at rest with the installation's field-encryption key and belong to the workspace that issued them. Every disclosure is **rate-limited**, written to the audit log as `agent.password_revealed`, and restricted to owner and admin roles — an agent can never read another agent's credential.

**Rotation ends sessions.** Setting or rotating an agent password signs that identity out of every active session immediately.

**Sign-in address.** Agents sign in with an e-mail address. If none is supplied at creation, one is derived from the username on the workspace domain (for example `support-eu-01@safedesk.business`). Supply a real, verified, unique address whenever the agent needs notifications or self-service recovery.

**Scope.** An agent sees every device of the organization it belongs to, including the source-reported device ID, and can edit notes. It cannot create sources, manage billing or change members. Per-device assignment inside one organization is deliberately **not** offered — if you need hard separation, use separate organizations.

**Owner, admin, billing and viewer passwords are not recoverable.** Those accounts are stored one-way hashed; only a new password can be set.

## Session behaviour

| Property | Value |
|---|---|
| Session transport | Same-origin `HttpOnly`, `Secure` cookie |
| Writes | Require `X-CSRFToken` in addition to the session |
| Organization scope | `X-Organization-ID: <organization-uuid>` on every authenticated tenant request |
| Lifetime | 12 hours |
| Revocation | A password change, password reset, agent password rotation or agent removal increments the user's server-side session version, invalidating every existing session for that user immediately |
| MFA | Mandatory TOTP for platform operators; optional for organization users |
| IP policy | Agents inherit the organization allowlist; staff may override per organization |

An agent request from outside every enabled allowlist network is refused:

```json
{ "error": { "code": "ip_not_allowed", "message": "IP address is not allowed.", "details": {} } }
```

## Private support threads

Every account has its own support conversation with SafeDesk operators, reached from the chat in the workspace.

- A thread is **private to one member**. The owner, each administrator, each billing or viewer account and each agent identity of the same company hold separate conversations.
- A colleague can never read another member's thread.
- An operator reply is delivered only to the member who wrote, and also arrives as a notification.

The practical consequence: agents report device and connection problems in their own chat, and ask their own administrator for anything that changes access — new agents, plan limits or trusted networks.

## Membership hygiene

Recommended process for granting access:

1. Verify the person's identity and business need.
2. Select the **narrowest** role that works.
3. Confirm the intended organization before saving.
4. Require a separate account — never share an owner or agent credential.
5. Review the resulting audit event.
6. Revoke access immediately when responsibilities end.

Never leave a workspace with only one recoverable owner. Treat role elevation, owner changes and access restoration as sensitive actions that deserve independent verification.

Review agent identities at least quarterly and after any personnel or supplier change, confirming individual ownership, current business need, expected organization, last activity and source network, and the absence of dormant, duplicate or shared identities.

## Offboarding checklist

1. Remove or downgrade the membership.
2. Revoke active sessions — every password change or agent rotation does this server-side.
3. Rotate any source secret the person could have accessed.
4. Review recent audit events and device changes.
5. Transfer billing and recovery ownership.
6. Preserve required evidence without retaining unnecessary personal data.

For an urgent offboarding, add: disable access, rotate exposed source secrets, check audit events, investigate access from unknown IPs, preserve evidence and notify security.

## Session-safety rules for agents

- Connect only to systems the organization is authorised to manage.
- Verify device identity and user consent where required.
- Request only the permissions the task needs.
- Never ask a user for passwords, one-time codes, wallet transfers, or to disable a security tool.
- Stop if the displayed target or policy differs from the work order.
- End the session explicitly and document the outcome without secrets.
- If a device appears that should not be visible, **stop work and report it** rather than testing other devices.

## Audit

Mutating requests are recorded with actor, organization scope, HTTP action, target, source IP, status and time. Filters cover action and target type. Investigate unexpected role, IP rule, source, billing or download changes promptly.

Two cautions worth stating: audit records describe **control-plane** actions and are not a substitute for endpoint forensic logs, and they are only comparable across systems if clocks are synchronised.

## Troubleshooting

| Symptom | What it means |
|---|---|
| `401` | Session missing or expired. Sign in again; never copy session cookies between machines. |
| `403` | Confirm membership, the owner/admin requirement for management actions, the CSRF token, and IP policy. |
| Organization appears empty | Verify `X-Organization-ID` and the user's membership in that organization. |
| Write denied although reads work | Read access does not imply write access. Confirm the role. |
| Agent not listed | Confirm the active organization and that the membership role is `agent`. |
| Generated password lost | Open the agent in Dashboard → Agents and reveal it; if the agent predates credential storage, issue a new password. |
| `402 subscription_required` | The workspace has no active plan. Agents, devices and downloads stay locked until one is paid. |
| Unexpected cross-organization result | Stop the workflow, preserve request IDs and timestamps, and escalate as a tenant-isolation incident. |

## Related

- [IP restrictions](ip-restrictions.md) — the network layer agents inherit
- [Security model](security-model.md) — the controls behind these rules
- [Billing and payments](billing-and-payments.md) — what an inactive subscription locks
