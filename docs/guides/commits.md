# Commits Guide

Enforced by the `git-manager` (Niobe) skill on every commit it makes.

## Format

Conventional commits, prefixed with the matching [gitmoji](https://gitmoji.dev):

```
<emoji> <type>(<scope>): <short description>

<optional body — only if the WHY is non-obvious>
```

- **Emoji:** the literal character (not the `:code:` shortcode), from the table below —
  exactly one, matching `<type>`.
- **Short description:** imperative mood, no trailing period, under 72 characters.
- **Body:** only when there's a hidden constraint, workaround, or non-obvious reason — never
  a restatement of the diff.

## Types

| Emoji | Type | When |
| --- | --- | --- |
| ✨ | `feat` | New feature or behavior |
| 🐛 | `fix` | Bug fix |
| ✅ | `test` | Test additions/modifications only |
| ♻️ | `refactor` | Restructuring without behavior change |
| 🔧 | `chore` | Tooling, dependencies, config |
| 📝 | `docs` | Documentation only |
| 🧱 | `infra` | Terraform/Helm/ArgoCD changes |
| ⚡️ | `perf` | Performance improvement |

## Scope

The domain name (`crm-core`, `ingestion`, `automation`, `analytics`, `billing`,
`notifications`), `web` for frontend-only changes, `infra-aws` / `infra-k8s` for
infra-only changes, or omitted if a change genuinely spans multiple domains.

## Splitting Commits

If a diff contains more than one logical change, split it — one commit per logical unit.
`git-manager` proposes the split before committing rather than deciding unilaterally.

## Example

```
✨ feat(ingestion): add dedup check on lead-capture endpoint

Dedup key is (organization_id, email) — collapsing on email alone would
merge leads across orgs that happen to share a contact address.
```

## Rules

- Never `git add -A` / `git add .` — stage specific files by name.
- Never commit with a failing typecheck/lint/test for the affected project.
- Never force-push without explicit human confirmation.
- Never skip hooks with `--no-verify`.
