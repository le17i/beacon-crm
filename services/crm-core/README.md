# services-crm-core

**Status:** Not yet implemented — first ADR: TBD

## What This Service Owns

The `crm-core` domain — the source of truth for the sales pipeline itself:

- Organizations, users
- Pipelines, stages
- Leads, contacts
- Custom fields
- Activity timeline (stage changes, manual notes, completed tasks)

Feeds [Epic 1 (Visual Pipeline Management)](../../INITIAL_PLAN.md#epic-1-visual-pipeline-management-core-pipeline)
and [Epic 3 (Context & Timeline)](../../INITIAL_PLAN.md#epic-3-context--timeline-lead-journey).

## Events Published

| Event | When |
| --- | --- |
| `crm/lead.created` | A new lead lands in the pipeline (including via ingestion) |
| `crm/lead.updated` | Lead fields change |
| `crm/lead.stage-changed` | A lead moves between pipeline stages |
| `crm/lead.deleted` | A lead is removed |
| `crm/pipeline.stage-created` / `-updated` / `-deleted` | Pipeline column customization (US1.4) |
| `crm/contact.created` / `-updated` | Contact record changes |

## Events Consumed

| Event | From | Why |
| --- | --- | --- |
| `ingestion/lead.received` | `services-ingestion` | Performs the dedup check (by email/phone) against its own `leads` table, then creates or updates the lead and publishes `crm/lead.created`/`crm/lead.updated`. Dedup logic lives here, not in `ingestion`, because it needs read access to data only this service owns. |

## Inngest Functions

None currently — CRUD and dedup are synchronous, no multi-step durable workflow needed yet.

## Dependencies

- `packages/core` — shared Zod schemas/types
- `packages/otel` — shared observability setup
- No dependency on any other service's code or database — only on events it consumes above.

## Development

```bash
npx nx run services-crm-core:test        # available once scaffolded
npx nx run services-crm-core:typecheck
npx nx run services-crm-core:lint
```
