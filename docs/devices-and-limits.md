# Devices and limits

[← Documentation index](README.md) · [SafeDesk overview](../README.md) · [Live guide](https://safedesk.business/docs/devices-and-limits)

SafeDesk maintains an **organization-scoped projection** of the devices pushed by configured source servers. It does not poll. It does not hold a source-server API token. The fleet view is a read model built entirely from signed events.

<img src="../assets/business-platform-1.png" width="100%" alt="Device fleet and the device allowance gauge"/>

## Lifecycle

Three events drive everything:

| Event | Effect |
|---|---|
| `device.created` | Upsert the full device under `(source, device_key)` |
| `device.updated` | Upsert the full device, preserving newer state |
| `device.deleted` | Mark the device deleted and keep its history |

A device is identified by the combination of **source** and **`device_key`** — never by a name or an IP, both of which change. SafeDesk projects searchable columns (source device ID, name, user, group, platform, enabled, online, last-online) and retains the complete authenticated device object as `raw_payload` for forward compatibility.

A delete event marks the record deleted. It does **not** authorise any destructive action on the source server, and it does not remove the audit history. Older events can never overwrite newer state.

## Presence expires

`online` is the latest accepted source value, not a continuous network probe — and it is not trusted indefinitely:

- A device reads **Online** only while events keep arriving for it.
- Three minutes with no event and the panel reads **Offline**, even if the last payload claimed `is_online: true`.
- `?online=true|false` filters on the same expiring rule, and lists sort connected devices first.

`last_online` is source-reported where available, `enabled` is source-reported state, and `source_occurred_at` determines ordering. Unknown nested fields stay in `raw_payload`; consumers must ignore what they do not understand.

## Notes

Every member of the organization — **including agents** — can attach free-text notes to a device. A note might be `Reception notebook, 2nd floor` or a ticket reference.

- Notes live only in SafeDesk.
- They are never sent back to the source server.
- They are never overwritten by an incoming device event.

That last property is what makes notes usable during a handover: the source can keep re-reporting the device all day without erasing what a technician wrote about it.

## The device allowance

Each active subscription **snapshots** its plan's `device_limit`. Editing a plan does not retroactively rewrite an existing subscription's stored allowance.

**Counting policy.** Devices that are not deleted and not already flagged over-limit count toward the active subscription's limit.

**What happens at the limit.** When a new device arrives and the count is already at the allowance:

1. SafeDesk **still stores** the device.
2. It flags it `over_limit: true`.
3. The webhook is acknowledged with `200 {"ok":true}` — **no HTTP error is returned to the source.**
4. The dashboard shows the device as over limit until the plan has room.

**The flag does not stick.** Once a plan is bought or upgraded, over-limit devices are brought back into the allowance automatically — on their next reported event, and independently of that by a reconciliation sweep every five minutes, oldest devices first.

**The warning.** The dashboard summary exposes `total`, `online`, `managed`, `over_limit`, `limit`, `usage_percent` and `limit_warning`, which turns true at 90 % usage. A live allowance ring turns amber at the same threshold.

> **Why flag instead of reject?** Rejecting the event would make the source server look broken and hide a device that physically exists. Flagging keeps the inventory honest and turns an upgrade into a decision the owner makes on purpose, rather than an outage they discover later.

## Operational practice

- Monitor active device usage before a bulk enrolment.
- Leave capacity for replacement and emergency devices.
- Upgrade or renew before reaching the allowance.
- Retire stale devices through the authoritative source and send a delete event.
- Never delete database records manually to evade a limit.

## Browsing the fleet

Inventory is read-only through the API. Filters include `online`, `enabled`, `platform`, `source` and `over_limit`. Search matches `device_key`, source device ID, name, user, group and notes. Ordering supports `source_occurred_at`, `name`, `last_online` and `created_at`.

Lists use cursor pagination, default page size 50, maximum requested 200. Cursor URLs are opaque: follow `page.next` or `page.previous` exactly and do not construct or store them permanently.

Device API operations are read-only except for the notes action. Inventory mutations arrive **only** through the signed per-source webhook — a separation that prevents a browser user from impersonating a source.

## Integration is operator-run

Customers do not configure device sources or webhook URLs themselves. When a subscription is provisioned, SafeDesk operators connect the source server to the workspace and devices begin appearing automatically. A new source or a secret rotation is requested through the in-workspace support chat.

## Troubleshooting

| Symptom | What to check |
|---|---|
| Device does not appear | Source active; correct source UUID in the URL; timestamp within five minutes; all SafeDesk headers present; signed bytes exactly equal to the sent bytes; body ≤ 2 MB; supported schema and event. |
| Device is stale | Compare source `occurred_at` values and clock synchronisation. An older event returns success with `"stale": true` and does not overwrite. |
| Duplicate update | A retry with the same delivery ID returns `"duplicate": true`. Expected; the mutation is not applied twice. |
| Search misses a device | Clear filters, verify organization and source, then search the stable `device_key` or source device ID. Deleted devices stay listed with `last_event: "device.deleted"`. |
| Limit warning is unexpected | Compare the active subscription's snapshotted allowance with the counting policy. Do not create duplicate sources as a workaround. |
| Device under the wrong organization | **Stop processing and escalate.** Treat it as a possible source or tenant-scoping issue, not a display bug. |

## Related

- [The signed device webhook](device-webhook.md) — how a device gets here
- [Billing and payments](billing-and-payments.md) — how the allowance is bought
- [Roles and permissions](roles-and-permissions.md) — who can read and annotate devices
