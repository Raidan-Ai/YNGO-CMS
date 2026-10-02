# docs/06-frontend/design-system.md

> **Status:** Current (spec) — implementation `PLANNED` | **Owner:** Frontend Lead
> **Last Updated:** 2026-10-02 | **Source:** `ARCHITECTURE.md` §4; ADR-011, ADR-013
> **Purpose:** tokens, RTL rules, and the mandatory states that make the UI feel
> finished without inventing data.

## 1. Design Tokens

```text
tokens/
├── color/       semantic roles: bg, surface, border, text, muted, primary,
│                success, warning, danger, info (+ hover/active/subtle variants)
├── typography/  Arabic and Latin stacks, scale (xs→3xl), weights, line heights
├── spacing/     logical scale (0–12) applied with ms/me/ps/pe
├── radius/      sm · md · lg · full
├── shadow/      sm · md · lg · focus ring
├── motion/      duration + easing tokens; reduced-motion alternative mandatory
└── breakpoints/ sm · md · lg · xl · 2xl
```

Rules:

1. Components consume **semantic** tokens, never raw hex values.
2. Tokens are per-tenant overridable (branding) at the theme layer — no component
   hard-codes one organization's colour, logo, or wording.
3. Motion respects `prefers-reduced-motion`; no animation is required to understand
   state.

## 2. Typography and Locale

| Concern | Rule |
|---------|------|
| Arabic | dedicated Arabic font stack first; no faux-bold synthesis |
| Latin | system/Inter stack; numerals and units in Latin form where appropriate |
| Mixed content | bidi isolation for embedded Latin tokens (IDs, codes, URLs) |
| Numbers | locale-aware grouping and decimal separators |
| Dates | locale-aware; ISO in payloads, localized in the UI |
| Currency | amount + explicit currency code, never a bare symbol |
| Line length | readable measure for article/Arabic body text |

## 3. RTL Rules (binding)

```text
[ ] layout uses logical properties: ms/me/ps/pe, start/end, text-start/text-end
[ ] icons that imply direction (arrows, chevrons) mirror in RTL
[ ] icons that are not directional (logo, clock, search) never mirror
[ ] `dir` set per locale at the document level; `<html lang dir>` correct
[ ] focus order and tab sequence verified in `ar`
[ ] charts and timelines read right-to-left where sequence matters
[ ] no physical `left`/`right`/`ml`/`mr`/`pl`/`pr` utilities anywhere
```

RTL regressions are treated as defects, not polish (`R-20`), and a RTL checklist is
part of every UI task.

## 4. Status and Semantic Colour

Status colours map from a single state→token table shared with `StatusBadge`:

```text
neutral   draft · archived · closed
info      planned · scheduled · submitted · in review
success   active · approved · published · completed · verified
warning   changes requested · awaiting approval · suspension risk
danger    rejected · cancelled · suspended · overdue · incident
```

Rules: never colour-only meaning (icon/text accompanies it); contrast meets
WCAG 2.1 AA in both themes; dark mode is a token variant, not a separate design.

## 5. Mandatory States (Definition of Done for any data view)

| State | Required behaviour |
|-------|--------------------|
| Loading | shape-accurate skeleton; no layout jump; no fake content |
| Empty | honest message + one primary next action (create or import); secondary link to help |
| Error | human message from the API error code, `request_id` for support, retry action |
| Unauthorized | explains the missing permission; suggests who to ask; leaks no data |
| Populated | real rows only; correct pagination, sorting, and totals |
| Partial | masked/omitted restricted fields clearly indicated, never silently blank |
| Read-only | explains why (permission, workflow state, or lock) and offers the valid action |

**Empty states are never simulated with fabricated rows** (`ADR-013`). A demo or
screenshot may illustrate layout, but runtime code contains no sample records.

## 6. Forms

```text
[ ] React Hook Form + Zod schema aligned to the API contract (ADR-015)
[ ] field-level inline validation on blur, summary at submit
[ ] server `VALIDATION_ERROR.details` mapped back to fields by name
[ ] unsaved-changes warning on navigation away
[ ] destructive actions require ConfirmDialog; restricted access/export requires ReasonDialog
[ ] submit button shows progress and prevents double submission (idempotency key)
[ ] error text is programmatically associated with its input
[ ] RTL: labels and inputs align with text-start; numeric inputs stay LTR where needed
```

## 7. Data Display Rules

| Content type | Rule |
|--------------|------|
| Money | `amount` + currency code; right-aligned per locale; no bare symbols |
| Percentages | explicit precision; no invented precision beyond source data |
| Totals/aggregates | labelled with the filter scope and "as of" timestamp |
| Charts | axis labels, a table alternative, and a "no data" state instead of an empty chart |
| Maps | privacy-filtered layers, legend, and an unavailable state when tiles fail |
| Timelines/history | ordered, labelled actor + timestamp, immutable record references |

Aggregates never fabricate or extrapolate: if there is no data, the UI says so
(`BR-035`).

## 8. Accessibility Baseline

WCAG 2.1 AA for public and admin core flows (`NFR-007`): keyboard reachability,
visible focus, announced errors, logical reading order in both directions, labelled
controls, sufficient contrast, no motion-only meaning, and no colour-only status.

## 9. Component Contribution Checklist

```text
[ ] uses tokens, not raw values
[ ] RTL verified (logical properties; mirrored directional icons only)
[ ] all applicable states implemented (loading/empty/error/unauthorized/partial)
[ ] keyboard + screen-reader verified
[ ] no data fetching or business logic inside the component
[ ] permission gating is UX only and clearly marked
[ ] story/test coverage added where the component is reused
[ ] no fabricated data in examples
```

## Related

- Architecture and data layer: `architecture.md`
- Component taxonomy: `components.md` · Routes: `routes.md`
- Rules: `../../AGENTS.md` §4 · Decision: `../13-decisions/ADR-0011-i18n-rtl.md`

*End of docs/06-frontend/design-system.md*