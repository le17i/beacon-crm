# services-automation

**Status:** Not yet implemented — first ADR: TBD

## What This Service Owns

The `automation` domain — reacting to pipeline events and delivering them onward:

- Trigger rules (e.g. "when a lead moves to 'Proposal Sent', send a webhook to URL X")
- Outgoing webhook delivery, with retries
- Delivery audit log (HTTP status codes, manual retry)

Feeds [Epic 5 (Automation Engine & Outgoing Webhooks)](../../INITIAL_PLAN.md#epic-5-automation-engine--outgoing-webhooks).

## Events Published

| Event | When |
| --- | --- |
| `automation/rule.created` / `-updated` / `-deleted` | A user changes their trigger rules |
| `automation/webhook.delivered` | A webhook succeeds |
| `automation/webhook.failed` | A webhook exhausts retries |

## Events Consumed

| Event | From | Why |
| --- | --- | --- |
| `crm/lead.stage-changed`, `crm/lead.created`, `crm/lead.updated` | `services-crm-core` | Evaluated against active trigger rules to decide whether a webhook fires |

## Inngest Functions

| Function | Purpose |
| --- | --- |
| `automation/deliver-webhook` | The durable delivery workflow: attempts the HTTP call, retries with backoff on failure, writes the audit log entry, emits `automation/webhook.delivered` or `-failed`. This is *the* canonical example of "why Inngest" in this repo — see [`architecture.md`](../../docs/guides/architecture.md#event-backbone). |

## Dependencies

- `packages/core`, `packages/otel`
- Reacts only to `crm-core`'s published events — never reads `crm-core`'s tables directly.

## Development

```bash
npx nx run services-automation:test        # available once scaffolded
npx nx run services-automation:typecheck
npx nx run services-automation:lint
```
