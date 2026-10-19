<div align="center">

<img src="assets/banner.png" width="100%" alt="SafeDesk — Remote access your security team will sign off on."/>

<br/>

<img src="assets/icon.png" width="96" alt="SafeDesk icon"/>

# SafeDesk

### Remote access your security team will sign off on.

<img src="https://img.shields.io/badge/Category-Remote%20access-0A6CFF?style=for-the-badge&labelColor=0B1F4D" alt="Category: Remote access"/>
<img src="https://img.shields.io/badge/Status-Live-ffffff?style=for-the-badge&labelColor=0B1F4D" alt="Status: Live"/>
<img src="https://img.shields.io/badge/Source-Private-6B7A99?style=for-the-badge&labelColor=0B1F4D" alt="Source: Private"/>

<a href="https://safedesk.business/"><img src="https://img.shields.io/badge/Business-safedesk.business-0A4FD6?style=for-the-badge&labelColor=0B1F4D&logo=googlechrome&logoColor=white" alt="safedesk.business"/></a>
<a href="https://safedesk.org/"><img src="https://img.shields.io/badge/Clients-safedesk.org-0A4FD6?style=for-the-badge&labelColor=0B1F4D&logo=googlechrome&logoColor=white" alt="safedesk.org"/></a>

<br/>

<a href="https://safedesk.business/register"><img src="https://img.shields.io/badge/%F0%9F%8F%A2%20%20Create%20your%20organization-0A6CFF?style=for-the-badge" alt="Create your organization"/></a> <a href="https://safedesk.business/docs"><img src="https://img.shields.io/badge/%F0%9F%93%98%20%20Documentation%20%E2%80%94%2011%20guides-ffffff?style=for-the-badge" alt="Documentation"/></a> <a href="https://safedesk.org/downloads"><img src="https://img.shields.io/badge/%E2%AC%87%EF%B8%8F%20%20Get%20help%20on%20your%20computer-0B1F4D?style=for-the-badge" alt="Downloads"/></a>

<br/>

<sub><b>This repository is a public showcase.</b> It documents what SafeDesk is, how it works and what it looks like.<br/>The SafeDesk source code is private and is not published here.</sub>

</div>

<br/>

<img src="assets/divider.svg" width="100%" height="4" alt=""/>

## 📋 Table of contents

- [Overview](#-overview)
- [One product, two front doors](#-one-product-two-front-doors)
- [Key features](#-key-features)
- [Screenshots](#%EF%B8%8F-screenshots)
- [How it works](#%EF%B8%8F-how-it-works)
- [Architecture](#-architecture)
- [The signed device webhook](#-the-signed-device-webhook)
- [Roles and permissions](#-roles-and-permissions)
- [Security](#-security)
- [Who it is for](#-who-it-is-for)
- [Plans and billing](#-plans-and-billing)
- [For the people receiving help](#-for-the-people-receiving-help)
- [Documentation](#-documentation)
- [Free live demonstration](#-free-live-demonstration)
- [Links](#-links)

<img src="assets/divider.svg" width="100%" height="4" alt=""/>

## 🧭 Overview

**SafeDesk is a remote-support product built around a single promise: every session and every privileged action can be proven after the fact.**

Most remote-access tools are built for the technician. SafeDesk is built for the two people who have to live with the technician's access — the person whose screen is being controlled, and the person who has to answer for it in a security review. That is why the product ships as two front doors over one platform: [safedesk.business](https://safedesk.business/) is the multi-tenant control plane where a company runs its device fleet, and [safedesk.org](https://safedesk.org/) is the plain-language site where the customer downloads the app, shares an ID and accepts — or declines — a connection request.

The control plane never reaches into your infrastructure. SafeDesk learns about your devices **only** from signed webhook events that your own device source server pushes to it, and it never asks for, accepts or stores a source-server API token. Agents get generated credentials that are shown once and inherit the organization's IP allowlist automatically. Plans are paid in cryptocurrency through [DrogoPay](https://drogopay.com/)'s hosted checkout, and a browser redirect never marks an order paid — only a verified signed webhook does. Every privileged action is appended to an immutable audit ledger with actor, organization, target and source IP.

<table>
<tr>
<td align="center" width="25%"><b>Category</b><br/><sub>Remote access</sub></td>
<td align="center" width="25%"><b>Status</b><br/><sub>Live</sub></td>
<td align="center" width="25%"><b>Plans</b><br/><sub>From $1,500 / month</sub></td>
<td align="center" width="25%"><b>Payment</b><br/><sub>Crypto · non-custodial</sub></td>
</tr>
<tr>
<td align="center" width="25%"><b>Tenancy</b><br/><sub>Multi-tenant, per-organization</sub></td>
<td align="center" width="25%"><b>Clients</b><br/><sub>Windows · macOS · Android</sub></td>
<td align="center" width="25%"><b>Integration</b><br/><sub>HMAC-signed webhooks</sub></td>
<td align="center" width="25%"><b>Operator</b><br/><sub>SafeDesk Corporation</sub></td>
</tr>
</table>

<img src="assets/divider.svg" width="100%" height="4" alt=""/>

## 🚪 One product, two front doors

One platform, three surfaces — each aimed at a different person, all sharing the same brand, the same identity model and the same audit trail.

| Surface | Who opens it | What it does |
|---|---|---|
| **[safedesk.business](https://safedesk.business/)** | The company that provides support | The control plane. Device fleet with notes and limits, agents with generated credentials, IP restriction, plans and orders, balance and ledger, provisioning, notifications, audit log, support chat and an 11-guide documentation site. |
| **[safedesk.org](https://safedesk.org/)** | The person receiving help | The client site. Download for Windows, macOS and Android with published SHA-256 checksums, a company code that swaps the public installers for your branded ones, three-step instructions, safety tips and an FAQ. Free, no account needed. |
| **SafeDesk Engine** | The company's own infrastructure | The private deployment engine that turns a clean VPS into a SafeDesk device source server: containerised rendezvous and relay services, HTTPS console behind Nginx with automatic certificate renewal, strict firewall rules, near-real-time device synchronisation, encrypted backups and one-command restore. See [docs/engine.md](docs/engine.md). |

> **Why this split matters.** The person whose screen is shared should never have to read a B2B feature page, and a security reviewer should never have to guess what the end user was shown. Splitting the surfaces lets each audience read copy written for them — while the device, the session and the audit record stay in one system.

<img src="assets/divider.svg" width="100%" height="4" alt=""/>

## ✨ Key features

### 🖥️ Device fleet, notes and limits

- **Organization-scoped inventory** — every device your source servers report, searchable by device key, source device ID, name, user, group and notes; filterable by online, enabled, platform, source and over-limit state.
- **Expiring presence** — a device shows *Online* only while events keep arriving. Three minutes without an event and the panel reads *Offline*, even if the last payload claimed otherwise. No status is taken on faith.
- **Shared per-device notes** — any member, agents included, can annotate a device (`Reception notebook, 2nd floor`, a ticket reference). Notes live only in SafeDesk, are never pushed back to the source and are never overwritten by an incoming event.
- **Device allowance that warns instead of cutting off** — a live ring turns amber at 90 % of the plan limit. A device arriving over the allowance is still stored, just flagged `over_limit` — never silently dropped, and the source still receives `200 {"ok":true}`.
- **The flag does not stick** — buy or upgrade a plan and over-limit devices are pulled back into the allowance automatically, on their next event and by a reconciliation sweep every five minutes, oldest first.
- **Deleted devices keep their history** — a `device.deleted` event marks the record deleted and leaves it auditable instead of erasing it.

### 🔐 Signed device webhooks, no tokens exchanged

- **SafeDesk never calls your server** — the direction of trust is one-way. Your source server pushes; SafeDesk verifies and records.
- **No source-server API token, ever** — SafeDesk does not need it, does not store it and refuses configuration payloads that contain one.
- **Two values, one of them shown once** — a per-source webhook URL and a 256-bit secret, created by SafeDesk operations and displayed exactly once; the stored copy is encrypted and cannot be shown again.
- **HMAC-SHA256 over the exact raw bytes**, compared in constant time, with a ±5-minute timestamp window and a 2 MB body cap.
- **Idempotent and order-safe** — deliveries deduplicate on `delivery_id`, and an event older than the stored state is acknowledged as `stale` instead of overwriting newer data.
- **All-or-nothing persistence** — delivery record, device row, limit flag and audit entry commit in one transaction, or the sender gets a `500` and retries the identical body.

### 🧑‍🔧 Agents, roles and IP restriction

- **Five roles, least privilege by default** — owner, admin, billing, agent and viewer, with SafeDesk staff acting only through a separate Operations workspace.
- **Agents with generated credentials** — agents never choose their own password. Owners and admins issue, read back, replace or rotate it; every disclosure is rate-limited and written to the audit log as `agent.password_revealed`.
- **Rotation ends sessions** — setting or rotating an agent password signs that identity out of every active session immediately.
- **One IP restriction switch** — per-organization IPv4 and IPv6 CIDR allowlists behind a single master switch. Enabling requires at least one enabled network, so an empty list cannot lock the company out; disabling is always allowed and keeps the networks saved.
- **Agents inherit the allowlist** — no separate agent policy to forget. Requests from outside every enabled network are refused with `403 ip_not_allowed`.
- **Private support threads** — every member and every agent holds their own conversation with SafeDesk operators. Colleagues cannot read it; a reply notifies only the person who wrote.

### 💳 Balance, immutable ledger and crypto checkout

- **Prepaid balance with an append-only ledger** — top-ups, subscriptions, upgrades and staff adjustments are appended as `credit`, `debit` or `adjustment`. Entries cannot be edited or deleted; a mistake is corrected with a compensating entry.
- **Row-locked money movement** — balance mutation and ledger insertion happen under database row locks, and a debit that would go negative is refused.
- **Hosted crypto checkout** — BTC (Bitcoin), ETH / USDC / USDT (Ethereum), BNB / USDT (BSC), SOL / USDC (Solana) and TRX / USDT (Tron), through DrogoPay. SafeDesk never holds your funds.
- **Webhook-only fulfilment** — a redirect, a screenshot, a user-supplied transaction hash or a frontend message never marks an order paid. Only a verified, deduplicated `payment.confirmed` or `payment.manually_confirmed` event does.
- **Terms that never cost you days** — renew early and the paid months are added to the current end date; upgrade and the new plan starts immediately.
- **Honest edge cases** — underpaid, overpaid or review-flagged payments park the order in `review` rather than guessing.

### 🛠️ Provisioning, notifications and support

- **Operator-run provisioning** — customers never wire up their own device source. Orders move `paid → queued → provisioning → ready`, with the status message and the ETA written by a SafeDesk operator.
- **Live countdown** — the workspace shows the operator's estimate against order state instead of an empty spinner.
- **Real-time notifications** — provisioning, billing and security events arrive over a WebSocket, with the organization's IP allowlist enforced on the socket too.
- **Support chat on every page** — messages go straight to SafeDesk operators and reply in the same thread; website visitors get a guest thread keyed to a token their browser alone holds.
- **Immutable audit stream** — sign-ins, plan changes, secret rotations, IP rules and role edits, each with actor, organization, target, source IP, status and time.

### 📦 Releases, checksums and company codes

- **Every installer publishes its SHA-256** on both the workspace Downloads page and the public site, so the file can be verified before it runs.
- **Per-company builds behind a short code** — a 2–3 character company code routes end users to your branded installers instead of the public builds; unknown codes simply return *not found*, and requests are rate-limited.
- **Client and Agent builds are separated** — Windows, macOS and Android clients for operators and unattended agents for managed endpoints, each with its own release notes.
- **Link-only releases** — a release can point at the customer's own download page instead of hosting a file, so an existing distribution channel keeps working.
- **Server-side scanning** — uploaded artifacts are hashed and virus-scanned before they are ever published.

<img src="assets/divider.svg" width="100%" height="4" alt=""/>

## 🖼️ Screenshots

Captured from the live [safedesk.business](https://safedesk.business/) and [safedesk.org](https://safedesk.org/) sites. No mock-ups.

### SafeDesk Business — the control plane

<table>
<tr>
<td colspan="2" align="center"><img src="assets/business-hero.png" alt="SafeDesk Business — remote access your security team will sign off on"/><br/><sub><b>The promise, stated up front</b> — signed device events, generated agent credentials, an IP restriction switch, non-custodial crypto billing and an immutable audit ledger</sub></td>
</tr>
<tr>
<td colspan="2" align="center"><img src="assets/business-platform-1.png" alt="Device fleet and device limit gauge"/><br/><sub><b>Fleet and allowance</b> — searchable inventory with over-limit rows flagged rather than hidden, and an allowance ring that warns at 90 %</sub></td>
</tr>
<tr>
<td colspan="2" align="center"><img src="assets/business-platform-2.png" alt="Signed webhook, provisioning ETA, balance and ledger, crypto checkout, IP restriction and audit stream"/><br/><sub><b>The platform, tile by tile</b> — signed device webhook, provisioning ETA, balance and immutable ledger, crypto checkout, IP restriction with delegated agents, and the live audit stream</sub></td>
</tr>
<tr>
<td colspan="2" align="center"><img src="assets/business-platform-3.png" alt="Roles and MFA, provisioning queue, notifications and the end-to-end flow"/><br/><sub><b>Roles, queue and the whole loop</b> — least-privilege roles with TOTP for operators, the provisioning queue, real-time notifications, and one signed event updating every view</sub></td>
</tr>
<tr>
<td width="50%" align="center"><img src="assets/business-how-it-works.png" alt="From sign-up to a managed fleet in four steps"/><br/><sub>Four steps: organization, plan and term, operator-connected source, then day-to-day management</sub></td>
<td width="50%" align="center"><img src="assets/business-audience.png" alt="Who SafeDesk Business is for"/><br/><sub>IT teams and MSPs, regulated finance, healthcare-style environments and field operations</sub></td>
</tr>
<tr>
<td width="50%" align="center"><img src="assets/business-roles.png" alt="Roles and permissions matrix"/><br/><sub>Five roles spelled out capability by capability — "view" means read-only</sub></td>
<td width="50%" align="center"><img src="assets/business-security.png" alt="Security posture"/><br/><sub>Hardened by default: session hardening, Argon2, operator MFA, rate limits, edge-only exposure, secrets at rest, immutable audit, tested backups</sub></td>
</tr>
<tr>
<td width="50%" align="center"><img src="assets/business-pricing.png" alt="Plans priced per device allowance, 12-month term selected"/><br/><sub>Three plans priced per device allowance — shown here on the 12-month term with the maximum 10 % discount</sub></td>
<td width="50%" align="center"><img src="assets/business-pricing-page.png" alt="Full pricing page on the standard one-month term"/><br/><sub>The full pricing page on the standard one-month term, with what every plan includes</sub></td>
</tr>
<tr>
<td width="50%" align="center"><img src="assets/business-sign-in.png" alt="Sign in to a SafeDesk workspace"/><br/><sub>Sign-in — 12-character minimum, Argon2 at rest</sub></td>
<td width="50%" align="center"><img src="assets/business-create-account.png" alt="Create a SafeDesk workspace"/><br/><sub>Workspace creation — organization name, slug and an explicit policy agreement</sub></td>
</tr>
<tr>
<td colspan="2" align="center"><img src="assets/business-docs.png" alt="SafeDesk documentation index"/><br/><sub><b>Eleven operator-grade guides</b> — getting started, downloads, organization administration, agents, IP restrictions, devices and limits, the device webhook, billing, the DrogoPay lifecycle, the developer API and security</sub></td>
</tr>
<tr>
<td width="50%" align="center"><img src="assets/business-webhook-guide.png" alt="Device webhook integration guide"/><br/><sub>The webhook guide an integrating engineer actually needs: endpoint, headers, exact signing input and tolerances</sub></td>
<td width="50%" align="center"><img src="assets/business-ip-restrictions.png" alt="IP restrictions guide"/><br/><sub>IP restrictions, including how the master switch refuses to lock you out</sub></td>
</tr>
<tr>
<td width="50%" align="center"><img src="assets/business-getting-started.png" alt="Getting started guide"/><br/><sub>Getting started, written per role rather than as one generic tour</sub></td>
<td width="50%" align="center"><img src="assets/business-faq.png" alt="Business FAQ"/><br/><sub>The questions security teams actually ask, answered on the page</sub></td>
</tr>
</table>

### SafeDesk — for the person receiving help

<table>
<tr>
<td colspan="2" align="center"><img src="assets/client-hero.png" alt="Get help on your computer — safely"/><br/><sub><b>One sentence, no jargon</b> — a technician you trust connects only when you allow it, you see every action, and you can disconnect at any moment</sub></td>
</tr>
<tr>
<td width="50%" align="center"><img src="assets/client-how-it-works.png" alt="Three steps the customer controls"/><br/><sub>Three steps, each one the customer's decision</sub></td>
<td width="50%" align="center"><img src="assets/client-downloads.png" alt="Downloads section"/><br/><sub>Downloads with published SHA-256 checksums — and an honest empty state while installers are being prepared</sub></td>
</tr>
<tr>
<td width="50%" align="center"><img src="assets/client-safety.png" alt="Safety tips before accepting a session"/><br/><sub><b>Anti-scam guidance built into the product</b> — never accept a surprise request, never read out a code from an unexpected call</sub></td>
<td width="50%" align="center"><img src="assets/client-faq.png" alt="Client FAQ"/><br/><sub>Can the technician see my files? How do I end a session? Why does macOS ask for permissions?</sub></td>
</tr>
<tr>
<td colspan="2" align="center"><img src="assets/client-downloads-page.png" alt="Download page with company code and per-platform install steps"/><br/><sub><b>The download page</b> — a company code swaps the public builds for your branded installers, with per-platform install steps and the permissions each OS will ask for</sub></td>
</tr>
<tr>
<td width="50%" align="center"><img src="assets/client-mobile-hero.png" width="300" alt="SafeDesk on a phone"/><br/><sub>The client site on a phone</sub></td>
<td width="50%" align="center"><img src="assets/client-mobile-how.png" width="300" alt="Three steps on a phone"/><br/><sub>Three steps on a phone</sub></td>
</tr>
</table>

<img src="assets/divider.svg" width="100%" height="4" alt=""/>

## ⚙️ How it works

**From sign-up to a managed fleet**

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#EEF4FF', 'primaryTextColor': '#0B1F4D', 'primaryBorderColor': '#0A6CFF', 'lineColor': '#0A6CFF', 'secondaryColor': '#F5F8FF', 'tertiaryColor': '#ffffff', 'clusterBkg': '#F5F8FF', 'clusterBorder': '#CFDCF7', 'edgeLabelBackground': '#ffffff', 'fontFamily': 'Inter, Segoe UI, sans-serif' } } }%%
flowchart LR
    A["1 · Create the organization<br/>username · work email · 12-char password<br/>name + slug"] --> B["Appoint a second owner<br/>least-privileged role for everyone else"]
    B --> C["2 · Choose a plan and term<br/>1, 2, 3, 6 or 12 months"]
    C --> D["Hosted crypto checkout<br/>or pay from internal balance"]
    D --> E{"Verified DrogoPay webhook"}
    E -- "payment.confirmed" --> F["Subscription activated<br/>device allowance snapshotted"]
    E -- "underpaid · overpaid<br/>requires_review" --> R["Order parked in review<br/>nothing activated"]
    F --> G["3 · Operators connect the source<br/>webhook URL + 256-bit secret, shown once"]
    G --> H["4 · Fleet, agents and access<br/>notes · credentials · IP restriction"]
    H --> I["Every privileged action<br/>appended to the audit ledger"]
```

**A support session, from the customer's side**

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#EEF4FF', 'primaryTextColor': '#0B1F4D', 'primaryBorderColor': '#0A6CFF', 'lineColor': '#0A6CFF', 'secondaryColor': '#F5F8FF', 'tertiaryColor': '#ffffff', 'actorBkg': '#EEF4FF', 'actorBorder': '#0A6CFF', 'actorTextColor': '#0B1F4D', 'signalColor': '#0B1F4D', 'signalTextColor': '#0B1F4D', 'labelBoxBkgColor': '#F5F8FF', 'labelBoxBorderColor': '#CFDCF7', 'noteBkgColor': '#FFF4E0', 'noteBorderColor': '#B26A00', 'fontFamily': 'Inter, Segoe UI, sans-serif' } } }%%
sequenceDiagram
    autonumber
    actor C as Customer
    participant App as SafeDesk app
    actor T as Technician
    participant SD as SafeDesk Business

    C->>App: Download from safedesk.org and open
    Note over C,App: No account, no email, no phone number
    App-->>C: Shows a short device ID
    C->>T: Shares the ID through a channel they already trust
    T->>App: Requests a connection
    App-->>C: "Alex from IT Support wants to view and control your screen"
    alt Customer accepts
        C->>App: Accept
        App-->>C: Session visible on screen the whole time
        T->>App: Works on the device
        C->>App: Disconnect at any moment, no permission needed
    else Customer declines
        C->>App: Decline
        App-->>T: No session starts
    end
    App->>SD: Source server reports device state
    SD-->>SD: Signature verified, device and audit record written
```

<img src="assets/divider.svg" width="100%" height="4" alt=""/>

## 🗺️ Architecture

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#EEF4FF', 'primaryTextColor': '#0B1F4D', 'primaryBorderColor': '#0A6CFF', 'lineColor': '#0A6CFF', 'secondaryColor': '#F5F8FF', 'tertiaryColor': '#ffffff', 'clusterBkg': '#F5F8FF', 'clusterBorder': '#CFDCF7', 'edgeLabelBackground': '#ffffff', 'fontFamily': 'Inter, Segoe UI, sans-serif' } } }%%
flowchart TB
    subgraph Endpoints["Endpoints"]
        AG["SafeDesk Agent<br/>unattended · policy-locked"]
        CL["SafeDesk Client<br/>operator · Windows · macOS · Android"]
    end

    subgraph Engine["SafeDesk Engine — customer infrastructure (private)"]
        RV["Rendezvous + ID service"]
        RL["Relay fallback"]
        CN["HTTPS console and API<br/>behind Nginx · automatic certificates"]
        DS["Device sync service<br/>durable outbox · signed delivery"]
    end

    subgraph Cloud["SafeDesk Business — safedesk.business"]
        WH["Webhook receiver<br/>HMAC · timestamp · idempotency"]
        API["REST API /api/v1/<br/>session + CSRF + organization scope"]
        DB[("Devices · orders · ledger<br/>audit events · encrypted secrets")]
        WS["Notifications WebSocket"]
        UI["Workspace + Operations<br/>SolidJS"]
    end

    subgraph Public["safedesk.org"]
        DL["Public release feed<br/>installers + SHA-256"]
    end

    PAY["DrogoPay<br/>hosted crypto checkout"]

    CL <-. "direct peer-to-peer where possible" .-> AG
    AG --> RV
    CL --> RV
    RV -. "relay when direct fails" .-> RL
    RV --- CN
    RL --- CN
    CN --> DS
    DS -- "signed device events" --> WH
    WH --> DB
    API --> DB
    DB --> WS
    UI --> API
    WS --> UI
    API -- "create checkout" --> PAY
    PAY -- "signed payment webhook" --> WH
    API -- "public releases" --> DL

    classDef cloud fill:#EEF4FF,stroke:#0A6CFF,color:#0B1F4D
    classDef edge fill:#ffffff,stroke:#CFDCF7,color:#3B4A6B
    class WH,API,DB,WS,UI cloud
    class AG,CL,RV,RL,CN,DS,DL,PAY edge
```

**The trust boundaries, stated plainly**

| Boundary | Direction | Credential |
|---|---|---|
| Endpoint ↔ endpoint | Direct peer-to-peer, relayed only when a direct path fails | Session-scoped |
| Engine → SafeDesk Business | Outbound only. SafeDesk never dials into customer infrastructure. | Per-source 256-bit HMAC secret |
| Browser → SafeDesk Business | Same-origin `HttpOnly` session cookie plus CSRF token on every write | Session + `X-Organization-ID` |
| DrogoPay → SafeDesk Business | Inbound signed webhook; the redirect carries no authority | DrogoPay webhook secret |
| SafeDesk Business → your server | **Never happens.** No source-server API token is requested, accepted or stored. | — |

<img src="assets/divider.svg" width="100%" height="4" alt=""/>

## 🔗 The signed device webhook

The whole integration is one endpoint and one secret. [The public guide](https://safedesk.business/docs/device-webhook-integration) is written for the engineer configuring the sender; a condensed version lives in [docs/device-webhook.md](docs/device-webhook.md).

| Item | Value |
|---|---|
| **Endpoint** | `POST https://safedesk.business/api/webhooks/rustdesk/devices/{source_uuid}` |
| **Events** | `device.created` · `device.updated` · `device.deleted` |
| **Headers** | `X-SafeDesk-Event` · `X-SafeDesk-Delivery` · `X-SafeDesk-Timestamp` · `X-SafeDesk-Signature` |
| **Signing input** | `HMAC_SHA256(secret, timestamp + "." + rawBody)` → `sha256=<lowercase hex>` |
| **Timestamp tolerance** | ± 5 minutes (300 s) |
| **Maximum body** | 2 MB |
| **Upsert key** | `(source, device_key)` |
| **Idempotency** | `(source, delivery_id)` with the SHA-256 of the raw body |
| **Ordering** | `occurred_at` — an older event is acknowledged as `stale` and changes nothing |

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#EEF4FF', 'primaryTextColor': '#0B1F4D', 'primaryBorderColor': '#0A6CFF', 'lineColor': '#0A6CFF', 'secondaryColor': '#F5F8FF', 'tertiaryColor': '#ffffff', 'edgeLabelBackground': '#ffffff', 'fontFamily': 'Inter, Segoe UI, sans-serif' } } }%%
flowchart TD
    S["Source server serialises the body once<br/>and signs those exact bytes"] --> V{"Signature, timestamp<br/>and headers valid?"}
    V -- no --> E1["401 invalid_signature<br/>or 400 invalid_request"]
    V -- yes --> D{"delivery_id already seen<br/>for this source?"}
    D -- yes --> E2["200 ok · duplicate<br/>nothing applied twice"]
    D -- no --> O{"Stored occurred_at<br/>newer than this event?"}
    O -- yes --> E3["200 ok · stale<br/>newer state preserved"]
    O -- no --> T["One transaction:<br/>delivery + device upsert<br/>+ limit flag + audit"]
    T --> L{"New device beyond<br/>the plan allowance?"}
    L -- yes --> F["Stored and flagged over_limit<br/>source still receives 200 ok"]
    L -- no --> K["200 ok"]
    F --> K
```

**Responses the sender must handle**

| Status | Body | Sender action |
|---|---|---|
| `200` | `{"ok":true}` | Stored. |
| `200` | `{"ok":true,"duplicate":true}` | Intentional no-op. Do not resend. |
| `200` | `{"ok":true,"stale":true}` | Older than stored state. No-op. |
| `400` | `invalid_request` | Fix the sender. Do not retry unchanged. |
| `401` | `invalid_signature` | Wrong secret, re-serialised body, uppercase hex, missing `sha256=` prefix, or clock skew. |
| `404` | `not_found` | Unknown or deactivated source. |
| `413` | `payload_too_large` | Body above 2 MB. |
| `429` | — | Edge rate limit. Back off. |
| `500` | `internal_error` | Retry the identical body with the same `delivery_id`. |

> **Never mint a new `delivery_id` to force a retry through.** It defeats duplicate detection and writes duplicate history.

<img src="assets/divider.svg" width="100%" height="4" alt=""/>

## 👥 Roles and permissions

Five roles inside every organization. Owners can do everything; every other role is a strict subset. SafeDesk staff never act as a customer role — they work through a separate Operations workspace with MFA and re-authentication on sensitive actions.

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

<sub>👁️ = read-only. Agents receive generated, shown-once credentials and inherit the organization's IP restriction automatically. Full detail in [docs/roles-and-permissions.md](docs/roles-and-permissions.md).</sub>

<img src="assets/divider.svg" width="100%" height="4" alt=""/>

## 🛡️ Security

SafeDesk claims no certifications it does not hold. What it publishes instead is exactly how the platform is protected — and the audit trail to check it.

| Control | What it means |
|---|---|
| **Session hardening** | `HttpOnly`, `Secure` session cookies with CSRF protection on every write. Sessions last 12 hours and are revoked server-side when a password changes. |
| **Password storage** | Argon2 hashing with a 12-character minimum. Owner, admin, billing and viewer passwords are not recoverable. |
| **Operator MFA** | TOTP required for every SafeDesk operations account; optional for organization users. |
| **Rate limits** | Per-route limits with `Retry-After` guidance instead of silent drops. Sign-in is limited at the edge. |
| **Edge only** | Cloudflare-fronted edge; the origin application listener is never exposed directly. `/admin/` deliberately returns 404 and staff administration lives on an environment-configured random path. |
| **Secrets at rest** | Webhook, source and agent secrets are encrypted at rest with the installation's field-encryption key and shown exactly once where applicable. |
| **Tenant isolation** | The server, not the interface, is the authority. Every organization request carries `X-Organization-ID` and membership is checked server-side. |
| **Immutable audit** | Actor, organization, target, source IP, status and time on every privileged action. Ledger entries can never be updated or deleted. |
| **Transactional money** | Balance mutation and ledger insertion under row locks; a debit that would go negative is refused. |
| **Backups** | Encrypted, tested backups, with transactional processing for billing events. |
| **Data handling** | Designed around GDPR principles. Data location is as configured by the operator. |

See [docs/security-model.md](docs/security-model.md) and the live [security and troubleshooting guide](https://safedesk.business/docs/security-and-troubleshooting).

<img src="assets/divider.svg" width="100%" height="4" alt=""/>

## 🎯 Who it is for

<table>
<tr>
<td width="50%" valign="top"><b>🔧 IT teams and MSPs</b><br/><sub>One workspace per client, agents with scoped credentials, and every device your source servers report — without sharing server tokens between tenants. Per-device notes survive a handover.</sub></td>
<td width="50%" valign="top"><b>🏦 Regulated finance</b><br/><sub>Every privileged action written to an immutable ledger with actor, target and source IP. The IP restriction switch turns a workspace into an allowlist-only zone.</sub></td>
</tr>
<tr>
<td width="50%" valign="top"><b>🩺 Healthcare-style environments</b><br/><sub>Least-privilege roles keep support staff away from billing and settings, while owners keep a full trail of who accessed what and when. Secrets are shown once.</sub></td>
<td width="50%" valign="top"><b>🚚 Field operations</b><br/><sub>Kiosks, warehouses and vehicles report through your own source server. Online state, platform and last-seen time per endpoint, with a provisioning ETA per order.</sub></td>
</tr>
<tr>
<td width="50%" valign="top"><b>🧑‍💼 Teams that answer to a security review</b><br/><sub>The questions a reviewer asks — who connected, from where, under whose credential, and who approved it — are answers the product already stores.</sub></td>
<td width="50%" valign="top"><b>🏠 The people receiving help</b><br/><sub>Free, no account, no phone number. A connection request they can decline, a session they can watch, and a disconnect button that needs nobody's permission.</sub></td>
</tr>
</table>

<img src="assets/divider.svg" width="100%" height="4" alt=""/>

## 💰 Plans and billing

Three plans, priced per device allowance. Every plan includes the full platform — you choose how many devices you need and for how long. Prices are set by SafeDesk operations and derived server-side; a price sent by a client is ignored.

| Plan | Devices | Monthly | 12-month total | Saving |
|---|---|---|---|---|
| **Starter** | 1,000 | $1,500 | $16,200 | −10 % |
| **Growth** *(most popular)* | 2,000 | $2,000 | $21,600 | −10 % |
| **Enterprise** | 5,000 | $4,000 | $43,200 | −10 % |

**Term discounts** — only longer terms are discounted, and 10 % is the maximum:

| Term | Discount | Starter | Growth | Enterprise |
|---|---|---|---|---|
| 1 month | — | $1,500 | $2,000 | $4,000 |
| 2 months | 3 % | $2,910 | $3,880 | $7,760 |
| 3 months | 5 % | $4,275 | $5,700 | $11,400 |
| 6 months | 7 % | $8,370 | $11,160 | $22,320 |
| 12 months | 10 % | $16,200 | $21,600 | $43,200 |

**Included in every plan** — unlimited agents and administrators, the signed device source webhook connected by SafeDesk operations, the IP restriction switch with allowlists inherited by agents, provisioning status with an operator ETA, the internal balance and immutable ledger, in-app support chat, and a 90 % allowance warning instead of a silent cut-off.

**How paying works** — choose a plan and term to create an order, then pay through DrogoPay's hosted crypto checkout or from your internal balance. Renewing the same plan adds the paid months to your current end date. Upgrading closes the running term and starts the new plan immediately. Details in [docs/billing-and-payments.md](docs/billing-and-payments.md) and on the [full pricing page](https://safedesk.business/pricing).

<img src="assets/divider.svg" width="100%" height="4" alt=""/>

## 🏠 For the people receiving help

[safedesk.org](https://safedesk.org/) exists because the hardest part of remote support is not the connection — it is trust. The whole site is written for someone who did not choose this software and does not want to learn it.

- **Free, and no account** — no email, no phone number, nothing to configure. Open the app, read out the short ID, accept the request.
- **Three steps, each one theirs** — download and open, share the ID with the technician they already agreed to work with, accept the request and watch every step.
- **Anti-scam guidance in the product, not in a policy PDF** — a connection request should never be a surprise; legitimate technicians do not phone out of the blue asking for codes; a session can be ended at any time without permission.
- **Verifiable downloads** — every installer is published with its SHA-256 checksum, and the site says plainly: download only from safedesk.org.
- **Honest about permissions** — why macOS asks for Screen Recording and Accessibility, why Android asks to capture the screen, and how to revoke both.
- **A company code, when there is one** — end users of a SafeDesk customer enter a short code and get that company's branded installers instead of the public builds.
- **It stops when they quit** — SafeDesk runs only while it is open. Quit the app and nobody can connect.

More in [docs/client-experience.md](docs/client-experience.md).

<img src="assets/divider.svg" width="100%" height="4" alt=""/>

## 📚 Documentation

**Live documentation — 11 operator-grade guides** at [safedesk.business/docs](https://safedesk.business/docs):

| Area | Guides |
|---|---|
| Getting started | [Getting Started](https://safedesk.business/docs/getting-started) · [Downloads](https://safedesk.business/docs/downloads) |
| Organization | [Organization Administration](https://safedesk.business/docs/organization-administration) · [Agents and Access](https://safedesk.business/docs/agents-and-access) · [IP Restrictions](https://safedesk.business/docs/ip-restrictions) |
| Devices | [Devices and Limits](https://safedesk.business/docs/devices-and-limits) · [Device Webhook Integration](https://safedesk.business/docs/device-webhook-integration) |
| Billing | [Billing, Subscriptions, and Balance](https://safedesk.business/docs/billing-subscriptions-and-balance) · [DrogoPay Payment Lifecycle](https://safedesk.business/docs/drogopay-payment-lifecycle) |
| Developers | [Developer API](https://safedesk.business/docs/developer-api) · [OpenAPI 3 schema](https://safedesk.business/api/schema/) · [Swagger UI](https://safedesk.business/api/docs/) |
| Support | [Security and Troubleshooting](https://safedesk.business/docs/security-and-troubleshooting) |

**In this repository** — a condensed, self-contained set for readers who want the shape of the system without signing in:

| Document | What it covers |
|---|---|
| [docs/architecture.md](docs/architecture.md) | The three surfaces, trust boundaries and the path a device event takes |
| [docs/device-webhook.md](docs/device-webhook.md) | The signed webhook contract end to end, with the decision tree and every response |
| [docs/devices-and-limits.md](docs/devices-and-limits.md) | Device lifecycle, expiring presence, notes and how the allowance behaves |
| [docs/roles-and-permissions.md](docs/roles-and-permissions.md) | Five roles, the Operations workspace, agent credentials and offboarding |
| [docs/ip-restrictions.md](docs/ip-restrictions.md) | The master switch, safe rollout and why lockout is hard by design |
| [docs/billing-and-payments.md](docs/billing-and-payments.md) | Plans, terms, orders, balance, the immutable ledger and webhook-only fulfilment |
| [docs/security-model.md](docs/security-model.md) | Controls, trust boundaries, what is deliberately not claimed |
| [docs/api.md](docs/api.md) | REST surface, session bootstrap, organization scope, pagination and errors |
| [docs/engine.md](docs/engine.md) | SafeDesk Engine — the private self-hosted source-server deployment |
| [docs/client-experience.md](docs/client-experience.md) | What the person receiving help actually sees |
| [docs/faq.md](docs/faq.md) | Straight answers for buyers, engineers and end users |

<img src="assets/divider.svg" width="100%" height="4" alt=""/>

## 🎥 Free live demonstration

<div align="center">

**See SafeDesk running before you pay a cent.**<br/>
A live walkthrough of the control plane, a signed device event arriving in real time, and the client experience end to end.

<a href="https://t.me/drogoz"><img src="https://img.shields.io/badge/Request%20a%20free%20demonstration%20on%20Telegram-0A6CFF?style=for-the-badge&logo=telegram&logoColor=white" alt="Request a demonstration"/></a>
<a href="https://safedesk.business/register"><img src="https://img.shields.io/badge/Or%20create%20your%20organization-0B1F4D?style=for-the-badge" alt="Create your organization"/></a>

<sub>Trial terms are granted by SafeDesk operations, not requested self-service. Ask in the support chat or on Telegram.</sub>

</div>

<img src="assets/divider.svg" width="100%" height="4" alt=""/>

## 🔗 Links

| Channel | Link |
|---|---|
| 🏢 **Business site** | [safedesk.business](https://safedesk.business/) · [Platform](https://safedesk.business/#platform) · [Security](https://safedesk.business/#security) · [Pricing](https://safedesk.business/pricing) · [FAQ](https://safedesk.business/#faq) |
| ⬇️ **Client site** | [safedesk.org](https://safedesk.org/) · [Downloads](https://safedesk.org/downloads) · [How it works](https://safedesk.org/#how-it-works) · [Safety tips](https://safedesk.org/#safety) |
| 📘 **Documentation** | [All guides](https://safedesk.business/docs) · [Developer API](https://safedesk.business/docs/developer-api) · [OpenAPI schema](https://safedesk.business/api/schema/) |
| 🔐 **Workspace** | [Sign in](https://safedesk.business/login) · [Create account](https://safedesk.business/register) |
| 💳 **Payments** | [DrogoPay](https://drogopay.com/) · [Payment guide](https://drogopay.com/guide) |
| ✉️ **Contact** | [support@safedesk.business](mailto:support@safedesk.business) · [legal@safedesk.org](mailto:legal@safedesk.org) · [privacy@safedesk.org](mailto:privacy@safedesk.org) · [abuse@safedesk.org](mailto:abuse@safedesk.org) |
| ⚖️ **Legal** | [Terms of Service](https://safedesk.business/legal/terms) · [Acceptable Use](https://safedesk.business/legal/acceptable-use) · [Privacy](https://safedesk.business/legal/privacy) · [Refunds](https://safedesk.business/legal/refund) · [Disclaimer](https://safedesk.business/legal/disclaimer) · [Compliance](https://safedesk.business/legal/compliance) |
| 🐙 **GitHub** | [deepdrogo](https://github.com/deepdrogo/deepdrogo) |

<br/>

> **Zero tolerance for fraud and unlawful access.** SafeDesk reviews reported or detected misuse, may suspend or terminate a business account, and cooperates with competent law-enforcement, regulatory and government authorities worldwide in response to valid lawful requests. Each business customer is solely responsible for unlawful or malicious use by its users, employees, contractors and agents. SafeDesk is a remote-support product for **authorised** access only — see the [Acceptable Use Policy](https://safedesk.business/legal/acceptable-use).

<img src="assets/divider.svg" width="100%" height="4" alt=""/>

<div align="center">
  <br/>
  <img src="assets/wordmark-on-white.png" width="180" alt="SafeDesk"/>
  <br/><br/>
  <sub>SafeDesk is operated by <b>SafeDesk Corporation</b>, State of Delaware, United States.<br/>Payments are processed by <a href="https://drogopay.com/">DrogoPay</a>. SafeDesk never holds customer funds.</sub>
  <br/><br/>
  <a href="https://drogoz.network"><img src="assets/drogoz-logo.svg" width="52" alt="Drogoz Network"/></a>
  <br/><br/>
  <sub>Built and maintained by <a href="https://drogoz.network">Drogoz</a></sub>
  <br/><br/>
  <a href="https://github.com/deepdrogo/deepdrogo"><img src="https://img.shields.io/badge/%E2%86%90%20Back%20to%20the%20profile-0B1F4D?style=flat-square" alt="Profile"/></a>
  <a href="https://safedesk.business/"><img src="https://img.shields.io/badge/safedesk.business-0A6CFF?style=flat-square&logo=googlechrome&logoColor=white" alt="safedesk.business"/></a>
  <a href="https://safedesk.org/"><img src="https://img.shields.io/badge/safedesk.org-0A4FD6?style=flat-square&logo=googlechrome&logoColor=white" alt="safedesk.org"/></a>
  <a href="mailto:support@safedesk.business"><img src="https://img.shields.io/badge/support%40safedesk.business-6B7A99?style=flat-square&logo=gmail&logoColor=white" alt="Email"/></a>
  <br/><br/>
  <sub><b>More from the network:</b> <a href="https://github.com/deepdrogo/drog-ai">Drog AI</a> · <a href="https://github.com/deepdrogo/mymask-ai">MyMask AI</a> · <a href="https://github.com/deepdrogo/replika">Replika</a> · <a href="https://github.com/deepdrogo/lidaro-ai">Lidaro AI</a> · <a href="https://github.com/deepdrogo/imperatori">IMPERATORI</a> · <a href="https://github.com/deepdrogo/zoi-talks">ZOI Talks</a> · <a href="https://github.com/deepdrogo/riderswap-io">RiderSwap</a> · <a href="https://github.com/deepdrogo/hyperblast-ai">HyperBlast AI</a> · <a href="https://github.com/deepdrogo/verifhub-ai">VerifHub AI</a> · <a href="https://github.com/deepdrogo/vadira-net">Vadira</a> · <a href="https://github.com/deepdrogo/skriper-io">Skriper</a> · <a href="https://github.com/deepdrogo/drogopay-com">DrogoPay</a> · <a href="https://github.com/deepdrogo/drogscan">DrogScan</a> · <a href="https://github.com/deepdrogo/mamont-tech">Mamont</a> · <a href="https://github.com/deepdrogo/mytasker">MyTasker</a></sub>
  <br/><br/>
  <sub>© 2026 SafeDesk Corporation. All rights reserved. Provided strictly for lawful, authorised use.<br/>Each customer is solely responsible for how they use the software.</sub>
</div>
