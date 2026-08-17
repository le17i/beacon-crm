# 🔦 Beacon CRM [WIP]

> **Clarity in Pipeline & Developer-First Automation**

An open-source, API-first CRM for freelancers, startups, and SMBs who want pipeline
visibility and frictionless automation without the weight of legacy CRMs.

Beacon CRM is also a **portfolio project**: it exists to demonstrate how I build software end
to end, both as a product manager and as an engineer — real architecture decisions, real
trade-offs, built in public.

---

## 🎯 Motivation

Most CRMs are either too simple (a spreadsheet with a UI) or too heavy (weeks of
configuration before the first lead lands). Beacon aims for the middle: a Kanban-first
pipeline, an API you can `curl` a lead into in minutes, and automation that reacts to your
sales flow instead of getting in its way.

Building it in public is the point — every architectural decision below, and every feature
that ships, is meant to be readable by someone evaluating how I think about product and
engineering, not just what I shipped.

## ✨ Objectives

- **Product:** real-time visibility into leads and pipeline health, with automation that
  removes manual busywork (see [`INITIAL_PLAN.md`](./INITIAL_PLAN.md) for the full epic
  breakdown).
- **Portfolio:** demonstrate API-first design, event-driven architecture, resilient async
  processing, and a disciplined PRD → ADR → implementation workflow — including how that
  workflow is executed by a team of AI agents with human approval gates.

## 🧱 Stack

| Layer | Choice | Why |
| --- | --- | --- |
| Frontend / BFF | Next.js, Tailwind CSS, shadcn/ui | Fast, modern, server-first UI; Next.js doubles as the BFF in front of the backend services |
| Backend APIs | NestJS (domain microservices) | Structured, testable, DI-first — good showcase for service boundaries |
| Pipelines / workers | Node.js | Lightweight processing outside the request/response cycle |
| Events & async | RabbitMQ, Redis, Inngest | Three tools, three distinct jobs — see [`docs/guides/architecture.md`](./docs/guides/architecture.md#event-backbone) |
| Data | PostgreSQL + Drizzle ORM | Relational integrity for pipeline/deal data, typed queries |
| Observability | OpenTelemetry | Vendor-agnostic tracing/metrics/logs across every service |
| Email | Resend | Transactional + automation-triggered email |
| Payments | Stripe | Subscription/plan billing |
| Auth | Clerk (managed) | Human/dashboard sessions; a separate API-key scheme handles external lead ingestion |

## ☁️ Infra — two deployable flavors

Beacon ships as the **same application code**, deployable two different ways — a deliberate
part of the portfolio story:

| Flavor | Cloud provisioning | Runtime | Event backbone |
| --- | --- | --- | --- |
| **AWS Serverless** | Terraform | NestJS services on Lambda + API Gateway; Next.js via OpenNext/Amplify | Amazon MQ, ElastiCache, Inngest Cloud (managed) |
| **Kubernetes** | Terraform (cluster) + Helm (workloads) + ArgoCD (GitOps) | Everything containerized | Self-hosted RabbitMQ, Redis, and Inngest in-cluster |

Details in [`docs/guides/infra-aws-serverless.md`](./docs/guides/infra-aws-serverless.md) and
[`docs/guides/infra-kubernetes.md`](./docs/guides/infra-kubernetes.md).

## 🗺️ Roadmap

See the full Release Matrix and Epics/User Stories in
[`INITIAL_PLAN.md`](./INITIAL_PLAN.md):

- **MVP (v0.1):** Kanban pipeline + public lead-ingestion API
- **v1.0:** Lead profile, timeline, custom fields
- **v1.1:** Analytics dashboard + outgoing webhook automation engine

## 🤖 How this repo is built

Every feature moves through the same agentic loop, styled as a crew of Matrix personas —
each one a Claude Code skill under [`.claude/skills/`](./.claude/skills/):

1. **Oracle** (`product-prd`) writes the PRD.
2. A human reviews and approves the PRD.
3. **Neo** (`staff-adr`) turns the PRD into an ADR with a TDD plan and implementation
   checklist, then coordinates the loop below.
4. **Trinity** (backend), **Link** (frontend), **Tank**/**The Architect** (infra), **the
   Merovingian** (QA), **Morpheus** (code review), and **Smith** (security review) loop until
   the ADR's Definition of Done is met.
5. A human reviews the result.
6. **Niobe** (`git-manager`) commits, opens the PR, and — if this run started from PR review
   comments instead of a PRD — replies to each one once it's fixed.

Full workflow, agent roster, and guides: [`CLAUDE.md`](./CLAUDE.md).

## 🚀 Getting Started

No application code exists yet — this repo currently holds the product plan, engineering
guides, and the agent workflow that will build it. The first PRD to flow through that
pipeline scaffolds the actual apps and services.

## 📄 License

MIT
