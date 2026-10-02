# ADR-0011 — Frontend i18n and RTL

> **Status:** PROPOSED | **Owner:** Frontend Lead | **Date:** 2026-10-02
> **Canonical rationale:** `../../TECHNICAL_DECISIONS.md` (ADR-0011)
> **Decides:** `../../OPEN_QUESTIONS.md` Q-10 | **Blocks:** every UI surface, public site, portals

## Context

Arabic is the primary audience language and must be a first-class experience, not a
translation overlay. The web surface includes an authenticated admin SPA, a public
website with SEO needs, and several portals, all on Next.js App Router with server
components. Copy must be authorable per tenant and switchable without code.

## Options

| Option | Pros | Cons |
|--------|------|------|
| A. `next-intl` | First-class App Router/RSC support, ICU messages, locale routing, `dir` handling | Younger ecosystem |
| B. `next-i18next`/`i18next` | Mature, huge ecosystem | Pages-router-era idioms; more plumbing for RSC |
| C. Custom dictionary loader | Full control | Reinvents routing, formatting, plurals |

## Decision

**`next-intl` with locale-segmented routes (`ar` default, `en` additional), ICU
message formatting, and direction driven by locale. Styling uses Tailwind with
logical properties/`dir`-aware variants only.**

## Mandatory Rules

1. No physical-direction CSS (`ml-*`, `mr-*`, `left-*`, `right-*`) in components;
   use logical equivalents (`ms-*`, `me-*`, `start-*`, `end-*`).
2. All user-visible strings come from message catalogues; no literal user-facing
   text in components (lint-enforced).
3. Numbers, dates, currency, and pluralisation use ICU/intl formatters, never string
   concatenation.
4. Arabic layout requires an explicit RTL review step for every UI task
   (checklist in `../06-frontend/design-system.md`).
5. Public pages declare `hreflang` and set `dir="rtl"` correctly for `ar`.
6. Tenant-level branding/theme tokens never override locale correctness.
7. `en` LTR must remain fully usable — RTL support must not break LTR.

## Consequences

- Positive: Arabic-first with correct SEO and RSC-friendly translations.
- Negative: contributors must learn Intl message conventions; RTL regressions are a
  tracked risk (R-20) mitigated by CI checks and review.

## Related

- Frontend docs: `../06-frontend/architecture.md`, `../06-frontend/design-system.md`
- Public site: Phase 9 (TASK-090) · Risk: R-20

*End of ADR-0011*