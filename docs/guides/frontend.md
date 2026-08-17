# Frontend Guide

Conventions for `apps/web` (Next.js). Applies to any agent implementing UI —
primarily the `frontend-implementation` (Link) skill.

## Stack

Next.js (App Router) + Tailwind CSS + shadcn/ui. Next.js is the **BFF**: it is the only
thing the browser talks to. It calls backend domain services server-side (route handlers /
server actions) — the browser never calls a `services-*` API directly.

## Project Structure

```
apps/web/src/
├── app/                # App Router routes (server components by default)
├── components/
│   ├── ui/             # shadcn primitives (generated, don't hand-edit)
│   └── <feature>/      # feature-specific components
├── lib/
│   ├── api/            # typed clients for each backend service, server-only
│   └── auth/           # Clerk helpers
├── hooks/               # client-side hooks (state, optimistic updates)
└── styles/
```

## Rules

- **Server Components by default.** Add `"use client"` only where interactivity requires it
  (drag-and-drop, forms with local state, optimistic updates).
- **Never call a backend service from the browser.** Route handlers or server actions proxy
  the request; this is what keeps API keys/service credentials server-side and lets the BFF
  enforce the Clerk session before forwarding.
- **Auth:** Clerk middleware protects all routes under `app/(dashboard)/`. Server
  actions/route handlers re-verify the session — never trust a client-supplied org context.
- **Data fetching:** server components fetch on the server; client components mutate via
  server actions, not direct `fetch()` to a service URL.
- **Optimistic updates** (Kanban drag-and-drop, Epic 1 US1.2): update local state
  immediately, reconcile on the server action's response, roll back on error with a toast.
  Use a single source of truth for pipeline state (no duplicated state between the board and
  card components).
- **Styling:** Tailwind utility classes; shared design tokens in `tailwind.config` via
  `packages/config`. No inline `style={}` except for values that must be computed at runtime
  (e.g. drag-transform offsets).
- **Components:** shadcn primitives stay unmodified in `components/ui/`; wrap/compose them in
  feature folders instead of editing generated files.
- **No single-letter or abbreviated variable names**, matching the backend convention —
  `(lead) => ...`, not `(l) => ...`.

## Testing

- **Unit tests** for non-trivial hooks/logic (optimistic-update reducers, KPI formatting,
  form validation) — colocated `*.test.ts(x)`.
- **Component tests** only for components with real logic (conditional rendering, derived
  state) — not for pure presentational shadcn wrappers.
- **E2E (Playwright):** one flow per epic acceptance criterion that's user-facing — see
  [`testing.md`](./testing.md) for the full pattern and directory layout.

## Environment

All config via `NEXT_PUBLIC_*` (client-safe) or server-only env vars — never hardcode a
service URL or API key. See [`security-limits.md`](./security-limits.md) for what must never
be `NEXT_PUBLIC_*`.
