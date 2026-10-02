# docs/00-overview/goals.md

> **Status:** Current | **Owner:** Product Owner | **Last Updated:** 2026-10-02
> **Source:** YNGO-CMS_Blueprint.md, PRD inputs | **Depends on:** vision.md

## 1. Primary Goals

| ID | Goal | Success indicator |
|----|------|-------------------|
| G-01 | Run an NGO's public presence and internal content on one platform | Public site and admin CMS served from the same API |
| G-02 | Manage programs, projects, and results with evidence | Program → project → milestone → indicator → report chain works on real data |
| G-03 | Manage funding relationships end to end | Grant → milestone → report → donor linkage is queryable |
| G-04 | Serve people safely | Beneficiary/case/safeguarding data accessible only to authorized roles, with field-level rules and audit |
| G-05 | Guarantee tenant isolation | Every data surface proven isolated by negative tests |
| G-06 | Be genuinely configurable | Custom fields, forms, workflows, permissions, dashboards, themes without code where practical |
| G-07 | Be Arabic-first | Arabic RTL is the default experience, not a translation layer |
| G-08 | Be API-first and integrable | All capabilities available via `/api/v1` + webhooks + generated SDKs |
| G-09 | Be operable by a small team | One-command local stack, documented deploy, backup/restore drill |
| G-10 | Use AI responsibly | AI output is a draft requiring human approval; never canonical |

## 2. Non-Goals (explicit)

| ID | Not a goal (at V1–V3) | Rationale |
|----|----------------------|-----------|
| NG-01 | Full ERP / general ledger accounting | Finance is a budgeting/utilization layer plus integrations |
| NG-02 | Microservices architecture at V1 | Modular monolith is cheaper to operate and refactor |
| NG-03 | Being a WordPress-compatible clone | Different tenancy, permissions, and API model |
| NG-04 | Real-time collaborative editing (CRDT) | Not required by the specified workflows |
| NG-05 | Native mobile apps before the API is frozen | Field PWA + generated SDKs first |
| NG-06 | Fabricated demo data in any environment | ADR-013 — clean installs are empty |
| NG-07 | Multi-region active-active deployment at V1 | Single-region with tested backup/restore |

## 3. Success Metrics (measured on real data only)

| Metric | Target | Source |
|--------|--------|--------|
| API latency | p95 < 300 ms | `docs/10-devops/monitoring.md` |
| Search latency | p95 < 500 ms | same |
| Standard report job | < 60 s | same |
| Availability | 99.5 % monthly | same |
| Cross-tenant leakage | 0 incidents; proven by negative tests | `docs/09-testing/strategy.md` |
| Sensitive-data leakage | 0 incidents; field-level policy tests | `docs/11-security/privacy.md` |
| Test coverage of security boundaries | 100 % of modules have an authz negative test | `AGENTS.md` §12 |
| Local onboarding time | < 30 minutes to a healthy stack | `docs/10-devops/local-development.md` |
| Restore drill | Verified within documented RTO | `docs/10-devops/backup-recovery.md` |

> Targets above are **initial SLO proposals** carried from `ARCHITECTURE.md` §12;
> they are confirmed at Phase 11 (TASK-110/112) and are not yet measured because
> no code exists.

## 4. Constraints

1. **No code before decisions** — BLOCKING questions in `OPEN_QUESTIONS.md` gate Phase 0.
2. **Single canonical stack** — ADR-001 (NestJS/Fastify) removes the FastAPI/Python
   contradiction for the API tier.
3. **Sequential phase gates** — a phase is done only when its exit gate passes
   (`IMPLEMENTATION_PLAN.md`).
4. **Yemen/offline context** — deployment must work on modest infrastructure and
   tolerate intermittent connectivity (influences self-hosting and future PWA work).
5. **Documentation is a deliverable**, not an afterthought (`AGENTS.md` §6).

## Related

- Vision: `vision.md` · Scope: `scope.md` · Requirements: `../01-product/requirements.md`
- Risks: `../../RISK_REGISTER.md` · Plan: `../../IMPLEMENTATION_PLAN.md`

*End of docs/00-overview/goals.md*