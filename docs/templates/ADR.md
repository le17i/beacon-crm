<!--
Template for docs/adrs/<slug>.md — filled by the `staff-adr` skill (Neo).
This is the loop's contract: the Implementation Checklist and Definition of Done below are
what the backend/frontend/infra/qa/review agents actually execute against and check off.
Keep it clean and straight, but detailed enough that no agent in the loop has to guess.
-->

# ADR: <Decision Title>

**Status:** Proposed | Accepted | Superseded by <link>
**Date:** <YYYY-MM-DD>
**Linked PRD:** [`docs/prds/<slug>.md`](../prds/<slug>.md)

## Context

What the PRD requires, in engineering terms. Relevant constraints (performance, cost,
consistency). Link the [`architecture.md`](../guides/architecture.md) domain(s) this touches.

## Decision

The chosen approach, stated plainly.

### Alternatives Considered

| Alternative | Why not |
| --- | --- |
| <option> | <reason rejected> |

## Domain / Service Ownership

Which service(s) this touches, and confirmation none of them requires a cross-service DB
read (per [`architecture.md`](../guides/architecture.md#service-boundary-rules)).

## Data Model

New/changed tables (Drizzle schema outline), migration summary, new composite indexes and
the query pattern that needs them. `N/A` if this ADR has no data changes.

## API Contract

New/changed REST endpoints, RabbitMQ event contracts (`<domain>/<model>.<action>`), or
Inngest function signatures (with `idempotency`/`concurrency`/`singleton`/`rateLimit`
decisions stated explicitly). `N/A` if this ADR is infra-only.

## Infra Impact

New Terraform resources / Helm values needed, called out for **both** flavors:

- **AWS Serverless:** <changes, or "N/A">
- **Kubernetes:** <changes, or "N/A">

## TDD Plan

Failing tests to write first, mapped to each acceptance criterion from the PRD:

| AC # | Test | Type | File |
| --- | --- | --- | --- |
| 1 | <description> | unit / integration / e2e | `services/<domain>/src/.../<name>.spec.ts` |

## Implementation Checklist

Ordered by dependency. One line per loop handoff — the agent that owns each line is implied
by its prefix:

- [ ] `backend:` ...
- [ ] `frontend:` ...
- [ ] `infra-aws:` ...
- [ ] `infra-k8s:` ...
- [ ] `qa:` write/extend tests per the TDD Plan above
- [ ] `review:` code review passes with zero P1
- [ ] `security:` security review passes with zero P1

## Definition of Done

The loop's objective exit condition — all of the following, no partial credit:

- [ ] Every Implementation Checklist item above is checked
- [ ] Every TDD Plan test passes
- [ ] `code-review` reports zero P1 (P2s fixed or accepted with written rationale below)
- [ ] `security-review` reports zero P1 (P2s fixed or accepted with written rationale below)
- [ ] Infra Impact applied to both flavors, or explicitly marked N/A for one with rationale

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- |
| ... | ... | ... | ... |

## Rollback Plan

How to revert if this ships broken — migration down path, feature flag, or revert commit.
