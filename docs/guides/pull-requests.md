# Pull Requests Guide

## Process

Every PR is opened by `git-manager` (Niobe) only after: the ADR's Definition of Done is met,
`code-review` (Morpheus) and `security-review` (Smith) both report zero P1s, and a **human**
has reviewed the result. No PR is opened before that human review step, and no PR self-merges
— a second human approval on GitHub is required regardless of what the agent loop concluded.

## Size & Scope

- One PR per ADR. If an ADR's implementation checklist is large enough to warrant splitting,
  split the ADR first (into sequenced ADRs), not the PR after the fact.
- A PR's diff should map cleanly onto the ADR's implementation checklist — a reviewer should
  be able to check off checklist items against files changed.

## Required Checklist (from `.github/PULL_REQUEST_TEMPLATE.md`)

- [ ] Linked PRD and ADR
- [ ] Tests pass (`nx run <project>:test`)
- [ ] Typecheck passes (`nx run <project>:typecheck`)
- [ ] Lint passes (`nx run <project>:lint`)
- [ ] No secrets committed
- [ ] Infra changes (if any) applied to **both** flavors, or explicitly marked N/A for one
- [ ] Screenshots attached for any frontend-visible change

## Review Comment Iteration

When review comments need addressing, don't re-run the full loop from scratch:
`staff-adr --pr <number>` scopes the loop to just the unresolved comments. `git-manager`
pushes fix commits and replies to each comment with the commit SHA — see
[`CLAUDE.md`](../../CLAUDE.md#agent-workflow) for the full mechanics.

## What Blocks Merge vs What Doesn't

- **Blocks:** any unresolved P1 from `code-review`/`security-review`, failing CI, missing
  ADR/PRD link, an infra change applied to only one flavor without an explicit N/A rationale.
- **Doesn't block:** P3s (nice-to-fix), accepted-with-rationale P2s (rationale must be visible
  in the PR description, not just implied).
