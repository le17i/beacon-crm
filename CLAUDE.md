# CLAUDE.md — Beacon CRM Agent Guide

This file is the entry point for any Claude Code agent working in this repo. Read it first,
then follow the linked guide(s) for the area you're touching. Guides are written as rules an
agent must follow, not background reading — treat every "must"/"never" as a hard constraint.

## What this repo is

Beacon CRM: an open-source, API-first CRM (Kanban pipeline, lead ingestion API, automation
engine) that is also a portfolio project. Product spec: [`INITIAL_PLAN.md`](./INITIAL_PLAN.md).
Human-facing overview: [`README.md`](./README.md).

**Current state:** no application code exists yet. This repo holds the product plan, the
engineering guides, and the agent workflow that will build it. Don't assume any
`apps/`/`services/*` source exists until you've checked — the domain map below describes
services that are *planned*, not present.

## Hard rules (apply everywhere, no exceptions)

1. **Org-scoping.** Every table has `organization_id`; every query/mutation is scoped to the
   caller's verified org. Never trust an `org_id` from a path/body param at face value.
2. **Service ownership.** A service only reads/writes its own tables. Cross-domain data comes
   from events, never a direct DB read into another service's schema.
3. **Infra portability.** Application code never branches on which infra flavor (AWS
   serverless vs Kubernetes) it's running on. Config is env-var driven (12-factor). Only
   Terraform/Helm/ArgoCD know which flavor is active.
4. **No secrets in code.** Ever. Secrets Manager / External Secrets Operator only.
5. **Human gates are mandatory.** No PR self-merges. PRD approval and pre-push review are
   both human steps the agent workflow stops for — never skip them.
6. **PRD → ADR → TDD, always.** No implementation starts without an approved PRD and an ADR
   with a TDD plan and an explicit Definition of Done.

## Domain map

| Domain | Owns | Future service |
| --- | --- | --- |
| `crm-core` | Organizations, users, pipelines, stages, leads, contacts, custom fields, timeline | `services-crm-core` |
| `ingestion` | API keys, public lead-capture endpoint, dedup | `services-ingestion` |
| `automation` | Trigger rules, outgoing webhook delivery, retries, audit log | `services-automation` |
| `analytics` | KPI aggregation, funnel, dashboard reads | `services-analytics` |
| `billing` | Stripe subscription/plan state | `services-billing` |
| `notifications` | Resend email sending/templates | `services-notifications` |

Each has a stub at `services/<domain>/README.md` describing its planned ownership, events,
and dependencies. Full rationale and the event-backbone (RabbitMQ/Redis/Inngest) role split:
[`docs/guides/architecture.md`](./docs/guides/architecture.md).

## Guides

| Guide | Covers |
| --- | --- |
| [`architecture.md`](./docs/guides/architecture.md) | Domain map, service boundaries, event backbone, sequence-diagram conventions |
| [`frontend.md`](./docs/guides/frontend.md) | Next.js/Tailwind/shadcn conventions, BFF data-fetching, Clerk integration |
| [`backend.md`](./docs/guides/backend.md) | NestJS module/DTO conventions, RabbitMQ/Inngest patterns, logging |
| [`data.md`](./docs/guides/data.md) | Postgres schema conventions, Drizzle workflow, indexing |
| [`testing.md`](./docs/guides/testing.md) | Unit/integration/e2e conventions + AI-agent regression testing |
| [`observability.md`](./docs/guides/observability.md) | OpenTelemetry conventions across services |
| [`commits.md`](./docs/guides/commits.md) | Conventional commit rules |
| [`pull-requests.md`](./docs/guides/pull-requests.md) | PR process and review gates |
| [`security-limits.md`](./docs/guides/security-limits.md) | Auth, rate limits, secrets, IaC security rules |
| [`infra-aws-serverless.md`](./docs/guides/infra-aws-serverless.md) | Terraform for the AWS serverless flavor |
| [`infra-kubernetes.md`](./docs/guides/infra-kubernetes.md) | Terraform + Helm + ArgoCD for the Kubernetes flavor |

## Templates

- [`docs/templates/PRD.md`](./docs/templates/PRD.md) — filled by `product-prd` (Oracle)
- [`docs/templates/ADR.md`](./docs/templates/ADR.md) — filled by `staff-adr` (Neo)
- [`.github/PULL_REQUEST_TEMPLATE.md`](./.github/PULL_REQUEST_TEMPLATE.md) — filled by
  `git-manager` (Niobe)

## Agent workflow

Every feature moves through the same loop. Full mechanics, gates, and the PR-comment
iteration mode live in the `staff-adr` skill; summary:

1. **`product-prd`** (Oracle) — writes the PRD → **human approves**.
2. **`staff-adr`** (Neo) — writes the ADR (TDD plan + implementation checklist +
   Definition of Done) from the approved PRD, then orchestrates the loop below. Also accepts
   a free-text instruction or `--pr <number>` (resume from PR review comments) instead of a
   PRD.
3. **Loop**, dispatched by Neo until the ADR's Definition of Done is met:
   - `backend-implementation` (Trinity) / `frontend-implementation` (Link) implement.
   - `infra-aws-serverless` (Tank) / `infra-kubernetes` (Architect) implement matching infra
     changes when the checklist item touches infra.
   - `qa-test-planner` (Merovingian) runs/extends tests against the TDD plan.
   - `code-review` (Morpheus) and `security-review` (Smith) review in parallel; any P1 sends
     the item back to the implementer.
4. **Human reviews** the result.
5. **`git-manager`** (Niobe) — commits, pushes, opens the PR; in `--resolve-comment` mode,
   replies to each addressed review comment with the fix commit SHA.

Skills live under [`.claude/skills/<name>/SKILL.md`](./.claude/skills/). Invoke by skill
`name`, not by persona name.

## Monorepo commands

Nx + pnpm workspaces (`apps/*`, `services/*`, `packages/*`). Once projects exist:

```bash
npx nx run <project>:test
npx nx run <project>:typecheck
npx nx run <project>:lint
npx nx run-many -t test        # everything
npx nx show projects           # list discovered projects
```
