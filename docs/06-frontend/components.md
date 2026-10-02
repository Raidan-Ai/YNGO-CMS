# docs/06-frontend/components.md

> **Status:** Current (taxonomy) — implementation `PLANNED` | **Owner:** Frontend Lead
> **Last Updated:** 2026-10-02 | **Source:** `ARCHITECTURE.md` §4; ADR-011
> **Purpose:** the component taxonomy so feature work composes existing primitives
> instead of inventing new ones.

## 1. Layers

```text
packages/ui        primitives + shared patterns (presentational, no data access)
components/        feature components per module (compose primitives + query hooks)
app/**/page.tsx    route shells: layout, data wiring, state orchestration
```

Rules: no component performs data fetching directly (hooks do); no business logic in
components; server components by default and client components explicitly marked.

## 2. Primitives — `packages/ui`

| Component | Notes |
|-----------|-------|
| `Button`, `IconButton` | variants; loading state; disabled reason via tooltip |
| `Input`, `Textarea`, `Select`, `Combobox`, `Checkbox`, `RadioGroup`, `Switch`, `DatePicker` | all RTL-safe, labelled, error-announcing |
| `Form`, `FormField`, `FormError` | React Hook Form + Zod wiring; server errors mapped by code |
| `Dialog`, `Sheet`, `Drawer`, `Popover`, `Tooltip`, `DropdownMenu` | Radix-based; focus trap and restore |
| `Tabs`, `Accordion`, `Breadcrumb`, `Stepper` | navigation and multi-step forms |
| `Card`, `Panel`, `Section`, `Toolbar` | layout composition |
| `Table`, `DataTable` | see §3 |
| `Badge`, `StatusBadge`, `Chip`, `Tag` | status rendering from a state→token map |
| `Avatar`, `AvatarGroup` | with fallback initials |
| `Progress`, `Spinner`, `Skeleton` | loading feedback |
| `Timeline`, `ActivityFeed` | audit/history presentation |
| `Chart` (wraps the chart lib), `Map` (MapLibre wrapper) | theme/RTL-aware |
| `Toast`, `Alert`, `InlineMessage` | transient and persistent feedback |
| `Pagination`, `CursorPager`, `FilterBar`, `SortMenu` | list controls |
| `EmptyState`, `ErrorState`, `UnauthorizedState`, `NotFoundState` | see §4 |
| `ConfirmDialog`, `ReasonDialog` | destructive actions; reason capture for restricted access/exports |
| `FileUpload`, `Dropzone`, `UploadProgress` | validation feedback per file |
| `CopyField`, `SecretRevealOnce` | API key/signed URL display with copy-once semantics |
| `LocaleSwitcher`, `DirectionProvider` | ar/en switching and `dir` handling |

## 3. `DataTable` Requirements

```text
[ ] server-driven pagination (cursor) with stable keys
[ ] search, filter bar, whitelisted sort columns
[ ] column visibility + per-user saved views
[ ] bulk selection with permission-aware bulk actions
[ ] export action (permission-checked, async when large)
[ ] row-level actions that respect field and record policy
[ ] loading (skeleton rows) · empty (with next action) · error (retryable) ·
    unauthorized · partial (some rows masked) states
[ ] responsive mode: table → stacked cards on narrow screens
[ ] RTL mirroring verified; no physical left/right utilities
```

## 4. State Components (mandatory on every data surface)

| State | Component | Content rules |
|-------|-----------|---------------|
| Loading | `Skeleton` / `Progress` | shape-accurate placeholders, never a fake table of rows |
| Empty | `EmptyState` | honest copy + exactly one primary next action (create/import) — **never fabricated rows** (`ADR-013`) |
| Error | `ErrorState` | message mapped from the API error code + retry; `request_id` shown for support |
| Unauthorized | `UnauthorizedState` | explains the missing permission in plain language; no partial data leak |
| Partial | inline masked cells | masked value + reason, never an empty cell that looks like missing data |
| Read-only | `ReadOnlyBanner` | explains why editing is unavailable (permission, state, lock) |

## 5. Feature Component Pattern

```text
components/projects/
├── ProjectTable.tsx          DataTable wiring + filters + bulk actions
├── ProjectForm.tsx           RHF + Zod; server errors mapped by code
├── ProjectStatusActions.tsx  transition buttons from the workflow's valid actions
├── ProjectMilestoneList.tsx  milestone completion with guard messaging
├── ProjectMapPanel.tsx       MapLibre + privacy-filtered layers
└── ProjectDetailHeader.tsx   identity, status badge, breadcrumbs
```

Conventions:

1. Feature components receive data via hooks (`useProjectsQuery`, `useProjectMutation`).
2. Actions are gated by `usePermissions()` — **UX only**; the API still decides.
3. Status changes call transition endpoints, never a generic PATCH.
4. Optimistic updates only where the server cannot reject the change on a business
   rule; otherwise await the response and reconcile.
5. Mutation success invalidates only the affected query keys (tenant-scoped).

## 6. Accessibility and RTL Obligations

```text
[ ] Radix primitives retained (keyboard, focus, ARIA) — not re-implemented
[ ] every input has an associated label and an announced error
[ ] colour is never the only status signal (icon/text accompanies it)
[ ] logical properties only (ms/me/ps/pe/start/end)
[ ] focus order verified in both `ar` and `en`
[ ] numbers, dates, and currency formatted per locale
[ ] icon-only buttons carry an accessible name
```

## 7. Anti-Patterns (rejected in review)

```text
X  fetch/axios calls inside a component
X  duplicating server state into Zustand
X  hiding a feature as "security" without a server-side check
X  building a bespoke table instead of DataTable
X  hard-coding one organization's branding or wording
X  mock rows used to "show the layout"
X  physical left/right spacing utilities
X  rendering a restricted field the API masked
```

## Related

- Architecture and data layer: `architecture.md` · Routes: `routes.md`
- Design tokens and states: `design-system.md`
- API errors to render: `../05-api/errors.md`

*End of docs/06-frontend/components.md*