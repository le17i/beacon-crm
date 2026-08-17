<!--
Template for docs/prds/<slug>.md — filled by the `product-prd` skill (Oracle).
Keep it clean and straight, but detailed enough that `staff-adr` (Neo) can write an ADR from
it without asking clarifying questions. Every Acceptance Criterion must be testable.
-->

# PRD: <Feature Name>

**Slug:** `<kebab-case-slug>`
**Status:** Draft | In Review | Approved
**Author:** Oracle (`product-prd`)
**Date:** <YYYY-MM-DD>
**Related epic:** <link to the epic in INITIAL_PLAN.md, or "new">

## Problem Statement

What problem exists today, for whom, and why it matters. 2-4 sentences — no solutioning here.

## Goals

- Goal 1
- Goal 2

## Non-Goals

- Explicitly out of scope for this PRD — prevents scope creep during the ADR/implementation
  phase.

## Personas Affected

Reference the personas in [`INITIAL_PLAN.md`](../../INITIAL_PLAN.md#2-personas--use-cases) —
add a new one here only if this PRD introduces one.

## User Stories

- As a `<persona>`, I want `<capability>` so that `<outcome>`.

## Acceptance Criteria

Numbered, each independently testable:

1. Given `<context>`, when `<action>`, then `<expected result>`.
2. ...

## Out of Scope

What a reader might reasonably assume is included but isn't.

## Success Metrics / KPIs

How we'll know this worked, in numbers.

## Open Questions

Anything still undecided that the human reviewer needs to resolve before approval.

## Dependencies

Other PRDs/ADRs/services this depends on, or that depend on it.
