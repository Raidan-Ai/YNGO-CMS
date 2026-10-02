# docs/09-testing/strategy.md

> **Status:** Current | **Owner:** QA Lead | **Last Updated:** 2026-10-02
> **Source:** Technical_Architecture_Instructions §41; Eng_Package §6; AGENTS.md §5
> **Purpose:** how quality is proven in this project.

## Principles

1. Testing is mandatory, not a later phase.
2. Every security boundary ships a **negative test** (proving denial).
3. Fixtures exist only under `tests/**`; never imported by runtime code.
4. A failing test is never weakened or deleted to make a build pass.
5. CI blocks merges when lint, typecheck, or any suite fails.

## Test Pyramid

```text
        E2E (Playwright)                 critical user journeys
      Integration (real DB)              repositories, migrations, jobs
    API / Contract tests                 endpoints, validation, error codes
  Unit (services, domain, policies)      rules, invariants, calculations
```

Plus two cross-cutting suites that are mandatory:

```text
Authorization tests     deny-by-default proven for every protected route
Tenant-isolation tests  cross-tenant read/list/search/export/report/file/webhook
```

## Suites by Layer

### Backend
| Suite | Scope | Example |
|-------|-------|---------|
| Unit | services, domain rules, policies, calculators | indicator aggregation |
| Integration | repositories + real Postgres (tenant scoping, constraints, transactions) | project repository scoping |
| API | request/response contracts, validation, error codes, pagination | `GET /api/v1/projects` |
| Authorization | permission matrix enforcement per route | `projects:approve` denied without role |
| Tenant isolation | attempt access to another tenant's entity on every surface | cross-tenant `GET` returns 404 |
| Workflow | guarded transitions, no bypass via CRUD | status change rejected |
| Job/queue | idempotency, retry, dead-letter | export retried on failure |

### Frontend
| Suite | Scope |
|-------|-------|
| Component | primitives, DataTable, EmptyState/ErrorState, forms |
| Form | validation, server-error mapping, unsaved-changes warning |
| State | query key isolation, cache invalidation behaviour |
| RTL | layout mirroring, logical spacing, focus order in `ar` |

### End-to-End (Playwright)
Critical flows only — keep them stable and meaningful:

```text
Auth         invitation → login → logout → password reset
CMS          create page → review → approve → publish → public view
Projects     create program → project → activity → indicator → report
Media        upload → validate → signed URL access → restricted denial
Security     cross-tenant access attempt denied; unauthorized export denied
```

## Acceptance Criteria Convention

Every feature documents criteria as Given/When/Then so they become executable
tests directly:

```text
Given   a user without `beneficiaries:export`
When    they call POST /api/v1/beneficiaries/export
Then    the response is 403 with code PERMISSION_DENIED
And     an audit record exists for the denied attempt
```

Feature-level criteria live in `docs/09-testing/acceptance-criteria.md`.

## Test Data Policy

- Fixtures/factories under `tests/fixtures/**` and `tests/factories/**`.
- Integration tests run against a **real** test database created/migrated in CI,
  never against production and never against mocks pretending to be the DB.
- Each test creates only the data it needs and cleans up.
- No shared mutable global fixtures between tests.

## Required Commands (target)

```bash
pnpm lint                # eslint + prettier check
pnpm typecheck           # tsc --noEmit across workspaces
pnpm test                # unit + integration
pnpm test:api            # API + contract
pnpm test:authz          # authorization + tenant isolation
pnpm test:e2e            # Playwright
pnpm build               # production build of web + api
```

These become runnable once Phase 0 is implemented; each task in
`IMPLEMENTATION_PLAN.md` names the exact commands it must pass.

## Coverage Expectations

- Domain/service logic: high, tested by unit tests (no numeric gate that
  encourages meaningless tests).
- Every protected route: covered by an authorization test.
- Every tenant-scoped resource: covered by a tenant-isolation test.
- Critical journeys: covered by E2E.

Coverage percentage is reported, but the **mandatory** gates are the
authorization and tenant-isolation suites — not a raw percentage.

## Related

- Definition of Done: `AGENTS.md` §12
- Acceptance criteria: `docs/09-testing/acceptance-criteria.md`
- Security boundaries to test: `docs/11-security/security.md`
- CI pipeline: `docs/10-devops/deployment.md`