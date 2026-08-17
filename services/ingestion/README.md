# services-ingestion

**Status:** Not yet implemented — first ADR: TBD

## What This Service Owns

The `ingestion` domain — the public-facing entry point for external systems:

- API key issuance, scopes, and revocation
- The public lead-capture endpoint (`POST /api/v1/leads`)
- Request-shape validation for inbound payloads

Feeds [Epic 2 (Lead Ingestion & API Open-First)](../../INITIAL_PLAN.md#epic-2-lead-ingestion--api-open-first).

Deliberately **does not** own dedup logic or the `leads` table itself — see below.

## Events Published

| Event | When |
| --- | --- |
| `ingestion/lead.received` | A validated payload arrives via a valid API key; forwarded for `services-crm-core` to dedup and persist |

## Events Consumed

None. This service only validates and forwards — it doesn't react to domain state.

## Inngest Functions

None currently.

## Dependencies

- `packages/core` — shared Zod schemas/types (the lead-capture request schema is shared with
  `crm-core` so both sides agree on shape without a direct dependency on each other's code)
- `packages/otel`
- Redis — API-key rate limiting only (see
  [`docs/guides/security-limits.md`](../../docs/guides/security-limits.md#rate-limits))

## Development

```bash
npx nx run services-ingestion:test        # available once scaffolded
npx nx run services-ingestion:typecheck
npx nx run services-ingestion:lint
```
