# services-notifications

**Status:** Not yet implemented — first ADR: TBD

## What This Service Owns

The `notifications` domain:

- Resend email sending
- Email templates (digests, automation-triggered emails)

Feeds [Epic 5 (Automation Engine & Outgoing Webhooks)](../../INITIAL_PLAN.md#epic-5-automation-engine--outgoing-webhooks) —
email is one of the destinations a trigger rule can target.

## Events Published

| Event | When |
| --- | --- |
| `notifications/email.sent` / `-failed` | An email delivery attempt completes |

## Events Consumed

| Event | From | Why |
| --- | --- | --- |
| `automation/webhook.failed` | `services-automation` | Alerts the org owner when a configured webhook starts failing |
| `crm/lead.stage-changed` | `services-crm-core` | Feeds the (future) daily/weekly digest email |

## Inngest Functions

| Function | Purpose |
| --- | --- |
| `notifications/send-digest` | Scheduled (cron) digest email compilation and send |

## Dependencies

- `packages/core`, `packages/otel`
- Resend SDK (server-side only)
- **PII rule:** templates only interpolate the minimum PII required for that specific email —
  see [`security-limits.md`](../../docs/guides/security-limits.md#data-exposure).

## Development

```bash
npx nx run services-notifications:test        # available once scaffolded
npx nx run services-notifications:typecheck
npx nx run services-notifications:lint
```
