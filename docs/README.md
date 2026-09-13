<div align="center">

<img src="../assets/icon.png" width="72" alt="SafeDesk"/>

# SafeDesk documentation

**A self-contained description of how SafeDesk works, for readers who have not signed in.**

<a href="https://safedesk.business/docs"><img src="https://img.shields.io/badge/Live%20documentation%20%E2%80%94%2011%20guides-0A6CFF?style=for-the-badge" alt="Live documentation"/></a>

</div>

<img src="../assets/divider.svg" width="100%" height="4" alt=""/>

## What this folder is

SafeDesk's authoritative documentation is published at **[safedesk.business/docs](https://safedesk.business/docs)** and is kept in step with the running platform. This folder is a condensed companion to the [showcase README](../README.md): enough detail to evaluate the architecture, the security model and the integration contract without creating an account.

It contains **no source code**. The SafeDesk application and the SafeDesk Engine deployment code are private.

## Read in this order

| # | Document | Read it if you want to know… |
|---|---|---|
| 1 | [Architecture](architecture.md) | how the three surfaces fit together and where every trust boundary sits |
| 2 | [Devices and limits](devices-and-limits.md) | how a device gets into SafeDesk, how presence expires, what the allowance does |
| 3 | [The signed device webhook](device-webhook.md) | the exact integration contract: headers, signing input, idempotency, responses |
| 4 | [Roles and permissions](roles-and-permissions.md) | who can do what, how agent credentials work, how offboarding works |
| 5 | [IP restrictions](ip-restrictions.md) | the master switch, safe rollout, and why locking yourself out is hard |
| 6 | [Billing and payments](billing-and-payments.md) | plans, terms, orders, balance, the immutable ledger, webhook-only fulfilment |
| 7 | [Security model](security-model.md) | the controls, and what SafeDesk deliberately does not claim |
| 8 | [Developer API](api.md) | the REST surface, session bootstrap, pagination, error codes |
| 9 | [SafeDesk Engine](engine.md) | the private self-hosted source-server deployment |
| 10 | [Client experience](client-experience.md) | what the person receiving help actually sees |
| 11 | [FAQ](faq.md) | straight answers for buyers, engineers and end users |

## The one-paragraph version

A company creates an organization on [safedesk.business](https://safedesk.business/), buys a plan for a 1–12 month term and pays in cryptocurrency through [DrogoPay](https://drogopay.com/)'s hosted checkout. SafeDesk operations then create a **device source** for that organization — a unique webhook URL and a 256-bit HMAC secret, shown exactly once. The company's own source server pushes signed `device.created` / `device.updated` / `device.deleted` events to that URL. SafeDesk verifies the signature over the exact raw bytes, deduplicates on the delivery ID, rejects stale events by their source timestamp, and commits the device, the allowance flag and the audit entry in a single transaction. The workspace then shows the fleet, lets admins issue agent credentials and lock access to trusted networks, and appends every privileged action to an immutable ledger. Meanwhile the person whose screen is being controlled downloads the client from [safedesk.org](https://safedesk.org/), shares a short ID, and accepts — or declines — each request.

## Conventions in these documents

- Example IPs use the documentation ranges `192.0.2.0/24`, `198.51.100.0/24` and `2001:db8::/32`. Never copy them into production.
- No real secret, token, credential or customer identifier appears anywhere in this folder.
- Where a value is installation-specific it is written as `<placeholder>`.
- Where the live documentation is more detailed, each page links to it.

<div align="center">
<br/>
<a href="../README.md"><img src="https://img.shields.io/badge/%E2%86%90%20Back%20to%20the%20SafeDesk%20overview-0B1F4D?style=flat-square" alt="Back"/></a>
<a href="https://safedesk.business/docs"><img src="https://img.shields.io/badge/Live%20docs-0A6CFF?style=flat-square" alt="Live docs"/></a>
</div>
