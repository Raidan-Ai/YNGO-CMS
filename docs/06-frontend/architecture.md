# docs/06-frontend/architecture.md

> **Status:** Current | **Owner:** Frontend Lead | **Last Updated:** 2026-10-02
> **Source:** Technical_Architecture_Instructions §2; **Authoritative:** `ARCHITECTURE.md`
> **Purpose:** frontend implementation detail under the approved architecture.

## Stack (binding)

```text
Next.js (App Router) + React + TypeScript
Tailwind CSS + shadcn/ui + Radix UI
TanStack Query (server state)
Zustand (genuine client state only)
React Hook Form + Zod (forms + validation UX)
MapLibre (maps)
next-intl (i18n, Arabic-first RTL)
```

## Application Surfaces

```text
apps/web/
├── app/[locale]/(site)/       # public website (ISR/SSR, SEO, hreflang)
├── app/[locale]/(admin)/      # admin SPA (authenticated)
├── app/[locale]/(portal)/     # partner/donor/applicant/volunteer/member portals
└── app/[locale]/(auth)/       # login, reset, MFA, invitation acceptance
```

Route groups keep public, admin, portal, and auth concerns separate while sharing
the same typed API client. Portals do not get separate backend logic.

## Rendering Strategy

| Surface | Strategy |
|---------|----------|
| Public content pages | ISR/SSG with revalidation on publish; SSR for personalised edges |
| Admin dashboards | Client rendering with TanStack Query + server components for shells |
| Portals | SSR for first paint + client hydration for interaction |
| Auth pages | Static shell + client logic |

Server components are the default; client components are opt-in and explicit.

## Data Layer

```text
Typed API client (generated from OpenAPI)
  ↓
TanStack Query hooks per module (query keys namespaced by tenant + resource)
  ↓
Components consume hooks; no fetch calls inside components
```

- Query keys include tenant id to prevent cache leakage across tenants.
- Mutations invalidate precisely-scoped keys; optimistic updates only where safe.
- Errors map to the API error contract and are surfaced with stable code handling.
- No business logic in components; no direct database access ever.

## State Management

- **Server state:** TanStack Query (authoritative).
- **Client state:** Zustand only for genuine UI state (sidebar, theme, drafts).
- **Form state:** React Hook Form; Zod for UX validation; server remains authoritative.
- Avoid duplicating server state into Zustand.

## i18n and RTL

- Locale-prefixed routes: `/[locale]/...` with `ar` default and `en` supported.
- `dir` set per locale at the document level; **logical properties only**
  (`ms/me/ps/pe/start/end`) — physical `left/right` utilities are forbidden.
- Arabic typography uses a dedicated font stack; numbers/dates/currency are
  locale-aware.
- `hreflang` alternates for public pages.

## Design System

```text
packages/ui                 shared primitives (Button, Input, Select, Dialog, ...)
DataTable                   search, filters, sorting, pagination, column visibility,
                            bulk selection/actions, export, saved views, responsive mode
EmptyState / ErrorState / Skeleton / StatusBadge / Timeline / Chart / Map
```

Every data-heavy page implements: **loading · empty · error · unauthorized ·
populated · partial**.

Design tokens (colour, typography, spacing, radius) are per-tenant configurable —
never hard-coded to one organization's branding.

## Accessibility

- Radix primitives for keyboard/focus/ARIA correctness.
- Every interactive element is keyboard reachable with visible focus.
- Form errors are announced; colour is never the only signal.
- RTL and LTR both tested for layout mirroring and focus order.

## Permissions in the UI

The UI hides or disables actions the user lacks permission for — **as UX only**.
It is never the security boundary; the backend enforces everything.

## Related

- Routes: `docs/06-frontend/routes.md`
- Components: `docs/06-frontend/components.md`
- Design system: `docs/06-frontend/design-system.md`
- API: `docs/05-api/api.md`
- Architecture: `ARCHITECTURE.md` §4, §8