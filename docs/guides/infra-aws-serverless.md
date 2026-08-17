# Infra Guide — AWS Serverless

Conventions for the AWS serverless deployment flavor. Applies to the `infra-aws-serverless`
(Tank) skill. Provisioning tool: **Terraform**. See
[`architecture.md`](./architecture.md#deployment-portability) for how this flavor relates to
the Kubernetes one — same application code, different runtime.

## Module Layout

```
infra/aws-serverless/terraform/
├── modules/
│   ├── lambda-service/     # one NestJS domain service → Lambda + API Gateway route
│   ├── networking/         # VPC, subnets, security groups
│   ├── data/                # RDS Aurora Serverless v2, ElastiCache Serverless, Amazon MQ
│   └── observability/       # ADOT Lambda layer wiring, log groups
└── envs/
    ├── dev/
    ├── staging/
    └── prod/
```

Each `envs/<name>` composes the modules with environment-specific variables — no
environment-specific logic inside a module itself.

## State

- Remote backend: **S3** (versioned, SSE-encrypted) + **DynamoDB** table for locking. One
  state file per environment (`envs/<name>/terraform.tfstate`).
- No local state ever committed.

## Compute Mapping

| App | Runtime |
| --- | --- |
| NestJS domain services | AWS Lambda behind API Gateway (HTTP API), one Lambda per service via a Lambda-HTTP adapter (e.g. `serverless-http`) |
| Next.js BFF | OpenNext build → Lambda@Edge/CloudFront, or Amplify Hosting (pick one per the first frontend ADR, document the choice there) |

## Data Layer

| Need | Service |
| --- | --- |
| Postgres | Aurora PostgreSQL Serverless v2 |
| Redis (cache/rate-limit) | ElastiCache Serverless |
| RabbitMQ (domain event bus) | Amazon MQ (managed RabbitMQ broker) |
| Durable workflows | **Inngest Cloud** (SaaS) — self-hosting Inngest on Lambda isn't practical |

## Networking

- Stateful resources (RDS, ElastiCache, Amazon MQ) live in private subnets, no public
  ingress.
- Lambdas only join the VPC when they need to reach a private-subnet resource — VPC-attached
  Lambdas cold-start slower, so `ingestion`'s public endpoint (latency-sensitive) should be
  evaluated for VPC-free alternatives (e.g. RDS Proxy / Data API) if cold starts become an
  issue; document the decision in that service's first infra ADR.

## IAM

One execution role per Lambda, scoped to exactly the resources that Lambda touches — no
shared "do everything" role. Enforced by `security-review` (Smith) per
[`security-limits.md`](./security-limits.md#infrastructure-as-code).

## Observability

ADOT (AWS Distro for OpenTelemetry) Lambda layer attached to every function, exporting to the
same collector target used by the Kubernetes flavor — see
[`observability.md`](./observability.md#exporting).

## CI/CD

1. On PR: `terraform plan` for the affected environment, posted as a PR comment.
2. On merge to `main`: `terraform apply` to `dev` automatically; `staging`/`prod` require a
   manual approval gate — no auto-apply to `prod` under any circumstance.
