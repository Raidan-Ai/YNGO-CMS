# docs/01-product/requirements.md

> **Status:** Current (all requirements `PLANNED`) | **Owner:** Product Owner
> **Last Updated:** 2026-10-02 | **Source:** frozen specs + ADRs
> **Rule:** `Implemented` may only be used when code + tests exist.

## 1. How Requirements Are Used

- IDs are stable and referenced by tasks (`../99-project-management/tasks.md`),
  tests (`../09-testing/acceptance-criteria.md`), and risks.
- A requirement without an acceptance criterion is not actionable.
- New requirements require a source citation or `DECISION REQUIRED`.

## 2. Functional Requirements (V1 scope)

| ID | Requirement | Phase | Acceptance |
|----|-------------|-------|-----------|
| FR-001 | The platform is multi-tenant; every tenant-owned row carries `tenant_id` | 0 | Cross-tenant read/list/search/export returns 403/404/empty (negative tests) |
| FR-002 | Tenant context is resolved on every request and is mandatory on data paths | 0 | Requests without valid tenant context are rejected |
| FR-003 | First tenant, organization, and admin are created only via bootstrap | 0 | Re-running bootstrap without token fails; password supplied at runtime |
| FR-004 | Users authenticate and receive sessions/tokens with rotation | 0–1 | Refresh rotation invalidates the old token; replay is rejected |
| FR-005 | Authorization is deny-by-default with `resource:action` permissions | 1 | Every handler declares permissions; unauthorized returns 403 |
| FR-006 | ABAC scoping restricts records by org unit / project assignment | 1 | Out-of-scope records are invisible and non-mutable |
| FR-007 | API keys are hashed at rest; raw secret shown once | 1 | Secret is unresolvable after creation; revocation is immediate |
| FR-008 | Every mutation produces an append-only audit record | 1 | Audit row has actor, entity, before/after, `request_id` |
| FR-009 | Content supports draft→review→publish lifecycle with versions | 2 | Published content always has an approved revision |
| FR-010 | Media uploads enforce type allow-list, magic bytes, size, quarantine | 2 | Forged MIME is rejected; downloads use signed URLs |
| FR-011 | Documents support versions, categories, retention, classification | 2 | Restricted documents never appear in public search |
| FR-012 | Search is tenant-scoped and permission-filtered via `SearchPort` | 2 | Cross-tenant search returns nothing |
| FR-013 | Programs → projects → milestones → indicators → reports chain works | 3 | Chain is queryable on real data with reports |
| FR-014 | Project locations render on a map with privacy rules | 3 | Restricted locations excluded from public layers |
| FR-015 | Reports generate asynchronously with progress and audit | 3 | Job is durable, cancellable, and permission-checked |
| FR-016 | Notifications and webhooks deliver with retry and delivery logs | 3 | Failures retry with backoff then dead-letter visibly |
| FR-017 | Import is validate-then-commit with per-row failure reporting | 3 | Partial failure does not corrupt committed rows |
| FR-018 | Sensitive domains enforce field-level policy | 6 | Restricted field denied even when the record is readable |
| FR-019 | Safeguarding rows never appear in generic search or exports | 6 | Proven by negative tests per surface |
| FR-020 | Workflow-managed status changes are only possible via transitions | 7 | Direct CRUD status edit is rejected |

## 3. Non-Functional Requirements

| ID | Requirement | Target | Verified by |
|----|-------------|--------|-------------|
| NFR-001 | API latency | p95 < 300 ms | TASK-110 load test |
| NFR-002 | Search latency | p95 < 500 ms | TASK-110 |
| NFR-003 | Availability | 99.5 % monthly | Monitoring (TASK-084) |
| NFR-004 | Arabic RTL + English LTR quality | Both production-grade | RTL checks in CI |
| NFR-005 | Type safety | TS `strict`; no `any` in domain code | Lint/typecheck gate |
| NFR-006 | Test coverage of security boundaries | 100 % of modules | CI |
| NFR-007 | Accessibility | WCAG 2.1 AA on public + admin core flows | TASK-110 audit |
| NFR-008 | Observability | Structured logs, metrics, tracing, `/health`, `/ready` | TASK-084 |
| NFR-009 | Backup/restore | Documented RTO/RPO with a verified drill | TASK-112 |
| NFR-010 | Portability | Same images across dev/test/staging/prod | Compose + env contract |
| NFR-011 | Data protection | Password hashing Argon2id; secrets never logged | Security review |
| NFR-012 | No fabricated data | Clean install is empty with professional empty states | ADR-013 + CI scan |

## 4. Traceability Rules

```text
Requirement (FR/NFR)
   → Phase (IMPLEMENTATION_PLAN.md)
   → Task (docs/99-project-management/tasks.md)
   → Acceptance criterion (docs/09-testing/acceptance-criteria.md)
   → Risk (RISK_REGISTER.md) where applicable
```

## Related

- Product intent: `prd.md` · Personas: `personas.md` · Flows: `user-flows.md`
- Plan: `../../IMPLEMENTATION_PLAN.md` · Testing: `../09-testing/strategy.md`

*End of docs/01-product/requirements.md*