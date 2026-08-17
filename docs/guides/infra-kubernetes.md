# Infra Guide — Kubernetes

Conventions for the Kubernetes deployment flavor. Applies to the `infra-kubernetes`
(The Architect) skill. **Terraform** provisions the cluster and cloud layer; **Helm** defines
the workloads; **ArgoCD** deploys them via GitOps. See
[`architecture.md`](./architecture.md#deployment-portability) for how this flavor relates to
the AWS-serverless one.

## Cloud Layer (Terraform)

Same tool as the AWS-serverless flavor, narrower scope — it provisions only what the cluster
needs, not the application workloads:

```
infra/kubernetes/terraform/
├── modules/
│   ├── cluster/        # EKS cluster + node groups
│   ├── networking/      # VPC, subnets, security groups
│   └── data/             # RDS (managed Postgres stays managed in this flavor too)
└── envs/{dev,staging,prod}/
```

Same state backend convention as [`infra-aws-serverless.md`](./infra-aws-serverless.md#state)
(S3 + DynamoDB, one state file per environment).

## Workload Layer (Helm)

```
infra/kubernetes/charts/
├── common/              # library chart: Deployment/Service/Ingress/HPA templates
├── crm-core/
│   ├── Chart.yaml        # depends on `common`
│   ├── values.yaml
│   └── values-{dev,staging,prod}.yaml
├── ingestion/
├── automation/
├── analytics/
├── billing/
├── notifications/
└── web/                  # Next.js BFF
```

Each service chart is thin — it supplies values to the shared `common` library chart rather
than redefining Deployment/Service/Ingress boilerplate six times. New services extend
`common`'s templates instead of forking them.

## GitOps (ArgoCD)

- **App-of-apps pattern:** one root `ApplicationSet` generates one child `Application` per
  (service × environment) pair, each pointing at `infra/kubernetes/charts/<service>` with the
  matching `values-<env>.yaml`.
- **Sync policy:** `automated: { prune: true, selfHeal: true }` for `dev`/`staging`; **manual
  sync only** for `prod` — mirrors the mandatory human-review principle in
  [`CLAUDE.md`](../../CLAUDE.md#hard-rules).
- **CI never runs `kubectl apply` or `helm install` directly.** CI's job ends at pushing a
  chart/values change to git; ArgoCD is the only thing that touches the cluster.

## Namespaces

One namespace per environment (`beacon-dev`, `beacon-staging`, `beacon-prod`), domain-prefixed
resource names within it — simpler RBAC surface than per-domain namespaces at this scale.

## Stateful Services

| Need | Choice | Why |
| --- | --- | --- |
| Postgres | Managed RDS | Stateful data isn't worth self-hosting for a demo, in either flavor |
| RabbitMQ | Self-hosted in-cluster (RabbitMQ Cluster Operator) | Deliberate contrast with the AWS flavor's managed Amazon MQ — demonstrates operating the broker directly |
| Redis | Self-hosted in-cluster (Helm chart) | Same rationale |
| Durable workflows | Self-hosted Inngest server (in-cluster) | Contrast with the AWS flavor's Inngest Cloud |

## Networking & TLS

- Ingress: `ingress-nginx` or the AWS Load Balancer Controller (pick one per the cluster's
  first infra ADR).
- TLS: `cert-manager`, automated certificate issuance.

## Secrets

External Secrets Operator syncs from AWS Secrets Manager into native k8s Secrets — same
source of truth as the AWS-serverless flavor, never a literal value in a chart's
`values.yaml`. See [`security-limits.md`](./security-limits.md#infrastructure-as-code).

## Observability

OpenTelemetry Collector deployed as an in-cluster Deployment/DaemonSet, exporting to the same
backend target as the AWS-serverless flavor so telemetry from both is directly comparable —
see [`observability.md`](./observability.md#exporting).
