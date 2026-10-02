# docs/99-project-management/backlog.md

> **Status:** Current | **Owner:** Product Owner | **Last Updated:** 2026-10-02
> **Source:** `../../PROJECT_AUDIT.md`, `../07-backend/modules.md` §9
> **Rule:** an item here is **not** a module and **not** scheduled work. Promoting an
> item requires an ADR (it would change the 52-module count or the architecture).

## 1. Deferred Module Candidates

Named in the frozen specifications or audit, but deliberately outside the canonical
inventory. Each requires an ADR before any code.

| Candidate | Source | Why deferred | Promotion requires |
|-----------|--------|---------------|--------------------|
| Advocacy | Blueprint | Overlaps governance + CRM; scope undefined | ADR + entity/rules definition |
| Knowledge base | Blueprint | Overlaps content + DMS; search semantics undecided | ADR + search/scoping rules |
| Microsites | Blueprint | Phase 9 covers them via theme engine; separate module unnecessary | ADR if split out |
| Plugin / marketplace | Blueprint | Conflicts with enforceability of tenant scoping | ADR + security model |
| Asset/lock controller | Blueprint | Only justified at large organizations | ADR + real-organization validation |
| Attendance biometrics | Blueprint | High privacy risk, not needed | ADR + privacy review |
| Full ERP / general ledger | Blueprint | Explicitly out of scope (`../00-overview/scope.md`) | ADR + product decision |

## 2. Deferred Capability Decisions

| Item | Deferred because | Revisit when |
|------|------------------|--------------|
| Real-time collaborative editing (CRDT) | Non-goal NG-04; not required by any documented workflow | A concrete workflow requires concurrent authorship |
| Native mobile apps | API + generated SDKs first (NG-05) | Field offline (Phase 10) proves a mobile need |
| OpenSearch | PG FTS sufficient at V1 scale (ADR-010) | Search p95 breaches the SLO at real scale |
| Microservices extraction | Modular monolith is cheaper to operate (NG-02) | A proven hotspot justifies extraction |
| Multi-region / active-active | Single-region + tested restore is the V1 posture (NG-07) | Availability target exceeds 99.5 % |

## 3. Known Documentation Work Not Yet Done

These are documentation gaps recorded honestly rather than silently skipped.

| Item | Status | Note |
|------|--------|------|
| Per-entity field-level specification | `PARTIAL` | `../02-domain/entities.md` is a catalogue; field detail is added per module as its phase runs, so it cannot go stale |
| Endpoint catalogue detail beyond planned surfaces | `PARTIAL` | `../05-api/endpoints.md` lists planned surfaces; final paths are confirmed when each module is implemented and OpenAPI is generated |
| `ACCESSIBILITY.md` / concrete CI workflow YAML | Deferred | Concrete config arrives with Phase 0 (TASK-009); criteria already captured in `../09-testing/acceptance-criteria.md` |
| `SECURITY.md` disclosure policy | Deferred | Publication channel is a distribution decision (Q-13 licensing) |

## 4. Promotion Rules

1. A backlog item is **not** implemented directly from this file.
2. Promotion requires an ADR in `../13-decisions/` that either adds a bounded context
   (changing the canonical count, currently **52**) or changes architecture.
3. The ADR is added to the register in `../13-decisions/index.md` **and** the canonical
   table in `../../TECHNICAL_DECISIONS.md`.
4. `../07-backend/modules.md`, `../05-api/endpoints.md`, and `tasks.md` are updated in
   the same change so no document contradicts the new inventory.
5. Any risk the item carries is added to `../../RISK_REGISTER.md` before code starts.

## Related

- Canonical module inventory: `../07-backend/modules.md` · Scope: `../00-overview/scope.md`
- Audit: `../../PROJECT_AUDIT.md` · Open questions: `../../OPEN_QUESTIONS.md`
- Decisions: `../../TECHNICAL_DECISIONS.md`, `../13-decisions/index.md`

*End of docs/99-project-management/backlog.md*