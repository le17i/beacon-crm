# Observability Guide

OpenTelemetry conventions, shared across every service via `packages/otel` so traces are
comparable regardless of which infra flavor is running them.

## Tracing

- **Span naming:** `<domain>.<action>` (e.g. `crm-core.create-lead`,
  `automation.deliver-webhook`) — stable, low-cardinality, greppable.
- **Trace-context propagation across async hops:** every RabbitMQ message carries the trace
  context in its headers (W3C `traceparent`); every Inngest function extracts it from the
  triggering event and continues the trace, not opens a new root span. A lead's full journey
  — HTTP request → event → Inngest workflow → DB write — must be visible as one trace.
- **Correlation ID:** every request gets one at the BFF edge, threaded through as a span
  attribute (`correlation_id`) on every hop, independent of the trace ID — useful for
  correlating logs when tracing isn't enough.

## Logging

- Structured `{ context, data }` shape (see [`backend.md`](./backend.md)) — logs are never
  free-text interpolation of variables.
- **Never log:** raw PII (email/phone/name bodies beyond an ID), auth tokens/API keys,
  full request/response bodies, internal error stack traces sent externally.
- Every log line includes `organization_id` when the operation is org-scoped — makes
  per-tenant debugging possible without a stack trace.

## Metrics

- RED metrics (rate, errors, duration) per service, auto-instrumented via OTel's HTTP/Nest
  instrumentation — don't hand-roll counters for things the instrumentation already covers.
- Business metrics that matter to the product (Epic 4 KPIs — conversion rate, pipeline
  value) are computed by `services-analytics` from domain events, not derived from traces.

## Exporting

- One OTel Collector config, one backend target, regardless of infra flavor — Terraform
  (AWS) and Helm (Kubernetes) both point every service at the same collector shape so the two
  flavors' telemetry is directly comparable. See
  [`infra-aws-serverless.md`](./infra-aws-serverless.md#observability) and
  [`infra-kubernetes.md`](./infra-kubernetes.md#observability).
