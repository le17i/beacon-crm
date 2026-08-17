# Backend Guide

Conventions for the NestJS domain services (`services/*`). Applies primarily to the
`backend-implementation` (Trinity) skill.

## Project Structure

Every domain service follows the same shape:

```
services/<domain>/src/
├── modules/
│   └── <feature>/
│       ├── <feature>.controller.ts
│       ├── <feature>.service.ts
│       ├── <feature>.module.ts
│       ├── dto/
│       └── <feature>.service.spec.ts
├── events/
│   ├── publishers/      # RabbitMQ producers
│   └── consumers/       # RabbitMQ consumers
├── inngest/              # Inngest function definitions, if this domain has any
├── db/
│   ├── schema.ts         # Drizzle schema for this domain's tables
│   └── migrations/
├── common/                # guards, interceptors, filters shared within the service
└── main.ts
```

## Rules

- **Validation:** every controller method validates its input with a DTO + `class-validator`
  decorators (NestJS's native pattern) — never touch `req.body`/`req.query` unvalidated.
- **Org-scoping:** every service method that touches data takes a verified `organizationId`
  from the authenticated request context — never from an unchecked path/body param. See
  [`security-limits.md`](./security-limits.md).
- **Dependency injection:** constructor-injected `private readonly` deps, standard Nest DI —
  no manual instantiation, no service locators.
- **Arrow functions are not required for Nest class methods** (Nest binds via DI, not `this`
  capture in callbacks) — but callbacks passed to `.map()`/`.forEach()`/etc. always use full
  descriptive parameter names: `leads.map((lead) => ...)`, never `(l) => ...`.
- **Logging:** structured `{ context, data }` shape via the shared `packages/otel` logger —
  never `console.log`/`warn`/`error`.

  ```typescript
  logger.info('Lead created', {
    context: { actor: 'services-crm-core/leads.service', action: 'create-lead' },
    data: { organization_id: organizationId, lead_id: lead.id },
  });
  ```

- **Error handling:** `try/catch` with structured logging on every catch; re-throw or handle
  explicitly, never swallow silently. Use Nest exception filters for consistent HTTP error
  shapes — never leak an internal error's `message`/stack to the client.
- **RabbitMQ producer/consumer pattern:** publishers live in `events/publishers/`, one
  function per event type; consumers live in `events/consumers/`, one handler per subscribed
  event, each wrapped in the same structured logging + error handling as any other handler.
  Event payloads are validated with the same DTO/Zod pattern as HTTP input — a message off
  the bus is still untrusted input.
- **Inngest functions:** used only for the multi-step/durable workflows described in
  [`architecture.md`](./architecture.md#event-backbone) — not as a general-purpose queue.
  Every function sets `idempotency`, `concurrency`, and (when relevant) `rateLimit`/
  `singleton` explicitly, even when the answer is "not needed, because X" (document why).
- **API-key auth** (ingestion domain's public endpoint): key validated + scope-checked in a
  guard before the controller method runs; rate-limited via Redis (see
  [`security-limits.md`](./security-limits.md)).
- **SDK clients** (DB pool, Rabbit channel, Redis client) initialized at module scope, not
  per-request.

## Testing

Unit tests colocated as `*.spec.ts`, one per service/controller. Mock RabbitMQ/Inngest/DB at
the boundary — see [`testing.md`](./testing.md) for the full mocking pattern and what belongs
in integration vs unit tests.
