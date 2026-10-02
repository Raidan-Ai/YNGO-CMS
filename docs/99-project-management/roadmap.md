# docs/99-project-management/roadmap.md

> **Status:** Current (sequencing) — all work `PLANNED` | **Owner:** Product Owner
> **Last Updated:** 2026-10-02 | **Source:** `../../IMPLEMENTATION_PLAN.md`
> **Rule:** phases are sequential in their prerequisites; each phase has an explicit
> exit gate. Release dates are deliberately not committed (see §5).

## 1. Release Model

| Release | Phases | Content |
|---------|--------|---------|
| **V1** | 0 → 3 | Foundation, Core/Identity/Access, Content & Media, Programs/Projects/Ops + platform services (Tenancy, Identity, Access, Media, Search, Audit) |
| **V1.1** | 4 → 5 | Grants/Donors/CRM, MEAL/Forms/Surveys |
| **V2** | 6 → 7 | Sensitive domains, Workflow/Automation/Notifications |
| **V2.1** | 8 | Reporting/Analytics/Observability |
| **V3** | 9 | Portals, Integrations, Developer platform, theme engine |
| **V3.1** (opt-in) | 10 | Optional AI sidecar, Field/Offline PWA |

Every later release **extends** earlier API contracts; no release may break a
published `/api/v1` contract (ADR-006).

## 2. Phase Sequence

```text
Phase 0  Foundation (BLOCKING — gated on ADR-001/003/004/005/009/012 + Q-01…Q-06)
    ↓
Phase 1  Core, Identity & Access
    ↓
Phase 2  Content & Media Platform
    ↓
Phase 3  Programs, Projects, Operations
    ↓
Phase 4  Grants, Donors, Partners & CRM
    ↓
Phase 5  MEAL, Forms & Surveys
    ↓
Phase 6  Sensitive Domains (People, Safeguarding, Complaints, Volunteers)
    ↓
Phase 7  Workflow, Automation & Notifications
    ↓
Phase 8  Reporting, Analytics & Observability
    ↓
Phase 9  Portals, Integrations & Developer Platform
    ↓
Phase 10 Optional AI & Field/Offline (opt-in, ADR-014)
    ↓
Phase 11 Production Hardening & Release
```

Full phase scope, tasks, and exit gates: `../../IMPLEMENTATION_PLAN.md`.
Task-level index: `tasks.md`.

## 3. Gate Summary

| Phase | Gate highlights |
|-------|-----------------|
| 0 | Compose healthy, CI green, OpenAPI served, tenant negative test, bootstrap without defaults |
| 1 | Permission catalog covers every route, deny-by-default proven, append-only audit grants, API-key lifecycle |
| 2 | Content transitions guarded, upload allow-list + quarantine + signed URLs, tenant-scoped search |
| 3 | Program→project→indicator→report on real data, durable queues, async reports, GIS privacy |
| 4 | Grant→milestone→report→donor queryable, real deadline notifications, append-only CRM |
| 5 | Logframe/indicator/values, versioned forms, data-quality issues surfaced not hidden |
| 6 | Field-level policy, no safeguarding in generic search/export, separate audit stream, reason capture |
| 7 | Workflow versioning, transition-only status changes, automation permission inheritance |
| 8 | Real metrics (no fabricated), `/ready` reflects dependencies, durable cancellable report jobs |
| 9 | Public site SSR/ISR + `hreflang` + RTL, per-portal guards, generated SDKs, swappable adapters |
| 10 | AI draft-only + permission inheritance negative tests, evaluation gates, deterministic offline sync |
| 11 | SLOs met, restore drill verified, no unaccepted Critical/High risk, docs current, release published |

## 4. Critical Path & Parallelism

- **Critical path:** Phase 0 → 1 → 3 (tenancy, access, and the program/project chain
  unlock everything downstream).
- Phases are sequential in *prerequisites* but may overlap in *staffing* once a gate
  passes — e.g. frontend empty-state/design work can proceed alongside backend modules
  once Phase 0 is green.
- Phases 6–7 are the highest-risk for privacy and integrity; both require security
  review before their gates are signed.

## 5. Estimation Posture

This roadmap deliberately commits **no dates**. Sizing in `IMPLEMENTATION_PLAN.md` §
"Phase Sizing" is relative (S/M/L), not a schedule. Dates are set only after Phase 0
completes, when real velocity exists — a date committed now would be fabricated
precision on a greenfield project.

## Related

- Plan + gates (canonical): `../../IMPLEMENTATION_PLAN.md`
- Tasks: `tasks.md` · Deferred: `backlog.md` · Risks: `../../RISK_REGISTER.md`

*End of docs/99-project-management/roadmap.md*