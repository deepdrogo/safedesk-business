# FAQ

[← Documentation index](README.md) · [SafeDesk overview](../README.md) · [Live FAQ](https://safedesk.business/#faq)

Answers for three different readers: the person evaluating SafeDesk, the engineer integrating it, and the person about to accept a support session.

<img src="../assets/business-faq.png" width="100%" alt="The SafeDesk Business FAQ"/>

## For buyers

**What exactly am I buying?**
A multi-tenant control plane for remote device fleets: signed device events, delegated agent identities with generated credentials, an IP restriction switch with trusted networks, non-custodial cryptocurrency billing, in-app support, and an immutable audit ledger behind all of it. The plan you choose sets a **device allowance**, not a feature tier — every plan includes the whole platform.

**How much is it?**
Starter is $1,500/month for 1,000 devices, Growth $2,000 for 2,000, Enterprise $4,000 for 5,000. Terms of 1–12 months are available and **only longer terms are discounted: 3 % at 2 months, 5 % at 3, 7 % at 6, and a maximum of 10 % at 12.** Full breakdown in [billing and payments](billing-and-payments.md).

**How do payments work?**
Choosing a plan and a term creates an order, which is paid either through DrogoPay's hosted cryptocurrency checkout (BTC, ETH, USDC, USDT, BNB, SOL, TRX on their respective networks) or from the organization's internal balance. **SafeDesk never holds your funds, and a redirect alone never marks an order paid** — only the signed confirmation webhook does.

**Can I upgrade later?**
Yes. Selecting a bigger plan creates a new order paid the same way, and the new allowance applies once the payment is verified. Renewing the *same* plan **adds** the paid months to your current end date, so paying early never costs you days you already bought.

**Is there a trial?**
Trials exist but are opened by SafeDesk operations, not requested self-service: three days, up to 20 devices, no payment. **One per company, ever**, and it cannot be extended.

**What happens when I hit the device limit?**
An amber warning at 90 % of the allowance, in the dashboard and by notification. Devices over the limit are **flagged in the fleet table rather than silently hidden**, so an upgrade is a decision you make, not a surprise you discover.

**What happens if my subscription lapses?**
The workspace keeps sign-in, its own organization record, purchasing, balance funding, notifications and support. Everything else answers `402 subscription_required`. **Nothing is destroyed** — devices, notes and agents are preserved, and paying again restores them untouched.

**Where is my data stored?**
Data location is as configured by your operator. SafeDesk publishes exactly how the platform is protected — encrypted secrets at rest, `HttpOnly` session cookies, immutable audit — and **does not claim certifications it does not hold.** The design follows GDPR principles.

**Are you SOC 2 or ISO 27001 certified?**
No, and SafeDesk does not imply otherwise. What it offers instead is a published control list and an audit trail you can check yourself. See [the security model](security-model.md).

**How do I get support?**
Every workspace page has a support chat in the corner, going straight to SafeDesk operators who reply in the same thread, with a notification. **Threads are per member, not per workspace** — the owner, each admin, each billing or viewer account and each agent identity hold separate conversations, and no member can read a colleague's.

## For engineers

**How does my source server connect?**
SafeDesk operations create a **device source** for your organization: a unique webhook URL and a 256-bit HMAC secret shown exactly once. Your server pushes signed device events to it. **SafeDesk never asks for your server's API token** — and refuses configuration payloads that contain one.

**What is the signing input?**
`HMAC_SHA256(secret, timestamp + "." + exactRawBody)`, hex-encoded, compared in constant time. Sign the exact bytes you transmit — re-serialising the JSON after signing breaks the signature. Full contract in [the signed device webhook](device-webhook.md).

**Do I have to handle retries and duplicates?**
SafeDesk deduplicates on `delivery_id` and rejects stale events by the source timestamp, so **retrying the same delivery ID is safe and correct**. Retry the same ID with backoff; do not mint a new one for a retry.

**Does a `200` mean the device was created?**
It means the delivery was accepted and processed by the documented rules. For payments the distinction matters more: a `2xx` on the payment webhook means *accepted*, **not** *fulfilled*. Fulfilment happens only on the specific verified events listed in [billing and payments](billing-and-payments.md).

**Is `online` a network probe?**
No. It is the latest accepted value from your source, and it **expires after three minutes** without an event. Treat it as freshness, not reachability.

**Is there an API?**
Yes — the dashboard is built on it. REST under `/api/v1/`, same-origin `HttpOnly` session cookie plus CSRF, `X-Organization-ID` on every tenant request, cursor pagination, a live OpenAPI schema and Swagger UI. See [the developer API](api.md).

**Can I write devices through the API?**
No. Devices are read-only except for the notes action; they enter SafeDesk only through a signed source webhook. Subscriptions, balances, ledger, provisioning, notifications and audit are read-only too.

**What about real-time updates?**
A notifications WebSocket at `wss://safedesk.business/ws/v1/notifications/`, authenticated by the same session cookie, **with the organization IP allowlist enforced on it**.

**Can I lock the workspace to our office network?**
Yes. Owners and admins add trusted CIDR networks and flip one master switch. **Enabling requires at least one enabled network**, so an empty allowlist cannot lock everyone out; disabling is always allowed and keeps the networks saved. Rules apply to every member, agents included. See [IP restrictions](ip-restrictions.md).

**What are agents?**
Delegated identities inside your organization with generated, shown-once credentials. They work with devices and notes but **never receive administrator, billing or settings controls**, and they inherit the IP restriction automatically. See [roles and permissions](roles-and-permissions.md).

**Do you run the server my clients connect through?**
No — you do. SafeDesk Engine is the private deployment engine that turns a clean VPS into that source server. SafeDesk Business is the record of which devices exist and who may reach them; the connections themselves happen on your infrastructure. See [SafeDesk Engine](engine.md).

## For people receiving help

**Is it free?**
Yes. The app for the person receiving help is free to download and use; the technician's company covers the service.

**Do I need an account?**
No account, email address or phone number. Open the app, share your device ID with your technician, accept their request.

**Does it run in the background?**
**Only while it is open.** Quit the app and nobody can connect. To remove it entirely, uninstall it like any other app.

**Can the technician see my files or passwords?**
During a session they see your screen and can control your mouse and keyboard — exactly like sitting next to you. **Nothing is shared before you accept and nothing after you disconnect.** Close sensitive documents before accepting, and never type a password while someone is connected.

**How do I end a session?**
Click Disconnect, or just quit SafeDesk. The connection stops immediately and the technician **cannot reconnect without a new request that you accept.** You do not need anyone's permission to disconnect.

**Which platforms are supported?**
Windows, macOS and Android. If a platform is not listed on the downloads page, it is not available yet.

**Why does macOS ask for Screen Recording and Accessibility?**
macOS requires every remote-support app to ask before it can show your screen or move your mouse. Grant them in System Settings → Privacy & Security, **for SafeDesk only**, and revoke them there whenever you like.

**How do I check the download is genuine?**
Every installer is published with its SHA-256 checksum. Run `sha256sum` (or `Get-FileHash` on Windows) and compare all 64 characters through a channel you trust. A checksum proves the file has not changed; the publisher signature proves who built it.

**Someone called me out of the blue and asked for my ID.**
Hang up. **Legitimate technicians do not phone you unprompted asking for codes or IDs.** A connection request should never be a surprise — if you did not arrange the session yourself, decline it. More in [the client experience](client-experience.md).

**My company gave me a code.**
Enter it on the downloads page and you will get your company's installers instead of the standard public ones.

## Related

- [Client experience](client-experience.md) — the full client-side walkthrough
- [Billing and payments](billing-and-payments.md) — plans, terms, ledger
- [Security model](security-model.md) — controls, and what is deliberately not claimed
- [Developer API](api.md) — the integration surface
