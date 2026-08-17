# Security & Limits Guide

Hard rules enforced by the `security-review` (Smith) skill on every loop pass. A P1 here
blocks Definition of Done — no exceptions, no "accepted with rationale."

## Auth

- **Human sessions:** Clerk. Every route under the dashboard is behind Clerk middleware; every
  server action/route handler re-verifies the session server-side — a client-supplied
  `organizationId` is never trusted without cross-checking the verified Clerk org membership.
- **External ingestion (Epic 2):** API-key scheme. Keys are scoped (permissions +
  organization), hashed at rest, revocable, and rate-limited via Redis
  (`ingestion:ratelimit:<key-id>`, sliding window).
- **Admin-only endpoints** (if/when they exist) are gated by an explicit role check, never by
  URL obscurity.

## Org-Scoping

- Every query/mutation is scoped to the caller's **verified** `organization_id` — never one
  read from `req.params`/`req.body` and trusted at face value.
- No `collection`/table scan without a `WHERE organization_id = $1` (or narrower).

## Input Validation

- Every HTTP body/query/param passes through a DTO (`class-validator`) or Zod schema before
  touching business logic — no raw `req.body.field` reads.
- Free-text fields stored in Postgres have explicit `max()`/length constraints.
- Webhook/event payloads from RabbitMQ are treated as untrusted input and validated the same
  way as HTTP input.

## Injection

- Postgres queries go through Drizzle's parameterized query builder — no raw string
  concatenation into SQL.
- Any outbound `fetch()`/HTTP call whose URL includes user-supplied data (webhook delivery,
  Epic 5) is validated against `https://` only, no `localhost`/private IP ranges — SSRF
  prevention for the automation engine.

## Secrets

- No API keys, tokens, or credentials as string literals anywhere in source.
- **AWS serverless flavor:** AWS Secrets Manager / SSM Parameter Store.
- **Kubernetes flavor:** External Secrets Operator syncing from Secrets Manager into k8s
  Secrets — never a committed value in a Helm `values.yaml`.
- Stripe webhook signatures are verified before the payload is trusted.

## Data Exposure

- API responses never leak: another org's data, internal error stack traces, internal-only
  fields not in the OpenAPI/DTO response shape.
- `logger.error()` never logs PII or tokens — log the error, not the payload that caused it,
  unless the payload has been scrubbed.
- Resend email templates never interpolate more PII than the specific email requires.

## Rate Limits

- Public ingestion endpoint: per-API-key rate limit (Redis), returns `429` with `Retry-After`.
- Dashboard-facing endpoints: per-org rate limit as a backstop against runaway frontend
  polling, not a primary control.

## Infrastructure-as-Code

- **IAM (AWS):** one execution role per Lambda, least privilege — no `"Resource": "*"`, no
  `"Action": "*"`.
- **Kubernetes:** no privileged/root containers; every Deployment sets resource
  requests/limits; RBAC is least-privilege per ServiceAccount, no cluster-admin bindings for
  application workloads.
- **Terraform state** is encrypted at rest (S3 + SSE, or the Kubernetes flavor's equivalent
  backend) and locked (DynamoDB) — no local state files committed, ever.
