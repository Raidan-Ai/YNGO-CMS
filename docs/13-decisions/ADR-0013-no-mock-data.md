# ADR-0013 — No Runtime Mock Data; Professional Empty States

> **Status:** ACCEPTED (binding) | **Owner:** Lead Architect | **Date:** 2026-10-02
> **Canonical rationale:** `../../TECHNICAL_DECISIONS.md` (ADR-013)
> **Blocks:** all runtime code, all environments

## Context

Multiple frozen specifications forbid fabricated business data presented as real,
while still requiring the product to be usable and demonstrable before an NGO has
entered its own data. A previous temptation — shipping demo records so screens look
populated — directly conflicts with the "no fabricated statistics / no seeding demo
business records" rule (`AGENTS.md` §13) and would corrupt trust in every report,
indicator, and dashboard.

## Options

| Option | Pros | Cons |
|--------|------|------|
| A. Seed demo business data at install | Screens look complete immediately | Fabricated data; violates product integrity; contaminates reporting |
| B. Empty install + honest empty states | Truthful; forces real onboarding | Requires deliberate empty-state design per view |
| C. Empty install + fixtures only in tests | Truthful; tests still have data | Test fixtures must never be imported by runtime |

## Decision

**Runtime code MUST NOT contain, import, or generate mock/demo/sample/fake business
data. A clean install is genuinely empty. Every collection view implements loading,
empty, error, unauthorized, and populated states, with professional empty-state copy
and a single clear next action (create or import). Synthetic data is permitted only
inside `tests/**` fixtures and is never imported by application code.**

## Mandatory Rules

1. No `mockData|demoData|sampleData|fakeData|dummyData` identifiers or hardcoded
   statistics anywhere outside `tests/`; CI fails the build if found.
2. No `admin/admin` or any default credential is ever shipped (see ADR-012).
3. Every list/table view renders a real empty state with one primary action — never
   placeholder rows, lorem ipsum, or fake counters.
4. Fixtures live only in `tests/**` and are excluded from production bundles.
5. Where a screen would previously show "sample" figures, it shows an honest
   zero/empty state with guidance to create or import data.

## Consequences

- Positive: reports, indicators, and dashboards can be trusted; onboarding is real.
- Negative / accepted cost: more design work on empty states and first-run guidance.
- Follow-up work: the CI scan pattern is defined in Phase 0 (TASK-009); the empty-state
  primitives are part of the web skeleton (TASK-005).

## Migration Impact

None — greenfield. This is a binding constraint from the first commit.

## Status & Approval

- Status: `ACCEPTED` (binding). Any deviation requires a superseding ADR.
- Register update: `index.md` and `TECHNICAL_DECISIONS.md`.

## Related

- Canonical rationale: `../../TECHNICAL_DECISIONS.md`
- Empty-state requirements: `../06-frontend/design-system.md`, `../09-testing/acceptance-criteria.md` (X-4)
- Risk: R-05

*End of ADR-0013*
