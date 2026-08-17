# Data Guide

Conventions for PostgreSQL + Drizzle ORM, used identically from every NestJS service and
from the Next.js BFF where it needs direct reads (e.g. server-rendered dashboards, if that
pattern is chosen over calling a service — decide per-ADR, document the choice).

## Schema Conventions

- **snake_case** for every table and column name.
- **`organization_id`** on every table (except a small set of truly global tables, e.g.
  `organizations` itself) — this is the multi-tenancy boundary, enforced at the query layer,
  not just documented. See [`security-limits.md`](./security-limits.md).
- **`id`**: UUID primary key (`gen_random_uuid()` default).
- **`created_at` / `updated_at`**: `timestamptz`, defaulted server-side, never client-supplied.
- Each domain owns its schema file: `services/<domain>/src/db/schema.ts`. No table is defined
  in more than one place.

```typescript
// services/crm-core/src/db/schema.ts
import { pgTable, uuid, text, timestamp, numeric } from 'drizzle-orm/pg-core';

export const leads = pgTable('leads', {
  id: uuid('id').primaryKey().defaultRandom(),
  organizationId: uuid('organization_id').notNull(),
  stageId: uuid('stage_id').notNull(),
  email: text('email'),
  dealValue: numeric('deal_value'),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
});
```

Export the inferred type alongside the table, same convention as the Zod schemas used at the
API boundary:

```typescript
export type Lead = typeof leads.$inferSelect;
export type NewLead = typeof leads.$inferInsert;
```

## Migrations

- Drizzle Kit generates migrations from schema diffs: `drizzle-kit generate` →
  `services/<domain>/src/db/migrations/`.
- Migrations are committed, reviewed like any other code change, and run as an explicit
  deploy step (never auto-applied on service boot in production).
- One migration per ADR that changes a schema — never bundle unrelated schema changes.

## Indexing

- Every query filtering by `organization_id` + one more field needs a composite index on
  `(organization_id, <field>)` — list these explicitly in the ADR's Data Model section, not
  discovered after a slow-query alert.
- No unbounded scans: list queries always paginate (`limit`/cursor), never
  `select * from leads where organization_id = $1` with no bound.

## Seed Data

- `services/<domain>/src/db/seed.ts` — idempotent, safe to run repeatedly, scoped to a
  clearly-fake demo organization (never seeds into a real `organization_id`).
