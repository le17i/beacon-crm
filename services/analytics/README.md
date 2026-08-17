# services-analytics

**Status:** Not yet implemented — first ADR: TBD

## What This Service Owns

The `analytics` domain — turning raw pipeline events into strategic intelligence:

- KPI aggregation (conversion rate, total pipeline value, average deal size)
- Conversion funnel / bottleneck computation
- Dashboard read APIs, with time/source filtering

Feeds [Epic 4 (Commercial Visibility & Analytics)](../../INITIAL_PLAN.md#epic-4-commercial-visibility--analytics).

## Events Published

| Event | When |
| --- | --- |
| `analytics/kpi.updated` | A KPI rollup completes, for cache invalidation on the dashboard |

## Events Consumed

| Event | From | Why |
| --- | --- | --- |
| `crm/lead.created`, `crm/lead.stage-changed`, `crm/lead.deleted` | `services-crm-core` | Feeds its own denormalized aggregation tables — never reads `crm-core`'s tables directly |
| `automation/webhook.delivered`, `-failed` | `services-automation` | Webhook delivery success rate is itself a tracked metric |

## Inngest Functions

| Function | Purpose |
| --- | --- |
| `analytics/rollup-kpis` | Scheduled (cron) rollup of the aggregation tables that back the dashboard's KPI/funnel views |

## Dependencies

- `packages/core`, `packages/otel`
- Read-only consumer of other domains' events — never a writer into another service's data.

## Development

```bash
npx nx run services-analytics:test        # available once scaffolded
npx nx run services-analytics:typecheck
npx nx run services-analytics:lint
```
