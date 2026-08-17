# services-billing

**Status:** Not yet implemented — first ADR: TBD (introduced when monetization work starts;
not mapped to an MVP/v1.0/v1.1 epic in [`INITIAL_PLAN.md`](../../INITIAL_PLAN.md) yet)

## What This Service Owns

The `billing` domain:

- Stripe subscription/plan state
- Invoicing and payment event history

## Events Published

| Event | When |
| --- | --- |
| `billing/subscription.created` / `-updated` / `-canceled` | Stripe subscription lifecycle changes |

## Events Consumed

| Event | From | Why |
| --- | --- | --- |
| `crm/organization.created` | `services-crm-core` | Provisions a default (free) plan for a new organization |

## Inngest Functions

| Function | Purpose |
| --- | --- |
| `billing/reconcile-stripe-webhook` | Durable processing of inbound Stripe webhooks — verifies signature, updates subscription state, retries on transient failure |

## Dependencies

- `packages/core`, `packages/otel`
- Stripe SDK (server-side only, never exposed to the frontend beyond publishable keys)

## Development

```bash
npx nx run services-billing:test        # available once scaffolded
npx nx run services-billing:typecheck
npx nx run services-billing:lint
```
