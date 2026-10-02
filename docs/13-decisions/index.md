# docs/13-decisions/index.md — ADR Index

> **Status:** Current | **Owner:** Lead Architect | **Last Updated:** 2026-10-02
> **Canonical record:** `TECHNICAL_DECISIONS.md` (repo root) holds full rationale.
> **Rule:** a decision marked `PROPOSED` is **not binding**. Only `ACCEPTED` bindings
> may be relied upon when implementing. `ACCEPTED` decisions change only via a
> superseding ADR.

## 1. Decision Register

| ADR | Decision | Status | Blocks | Record |
|-----|----------|--------|--------|--------|
| ADR-0001 | Backend framework: NestJS + Fastify primary; FastAPI sidecar only for AI | ACCEPTED | All backend work | `ADR-0001-backend-framework.md` |
| ADR-0002 | Monorepo with pnpm workspaces (`apps/*`, `packages/*`) | PROPOSED | Repo layout, CI | `ADR-0002-monorepo-pnpm.md` |
| ADR-0003 | ORM & migrations: TypeORM (Prisma reconsiderable by later ADR) | ACCEPTED | Data model, migrations | `ADR-0003-orm-migrations.md` |
| ADR-0004 | Multi-tenancy: shared schema + mandatory `tenant_id` (+RLS defence) | ACCEPTED | Every table/query | `ADR-0004-tenancy-isolation.md` |
| ADR-0005 | Tenant resolution: `X-Tenant-Id` header at v1; subdomain later | ACCEPTED | Guards, routing | `ADR-0005-tenant-resolution.md` |
| ADR-0006 | API style: REST `/api/v1`, snake_case JSON, generated OpenAPI | PROPOSED | All endpoints | `ADR-0006-api-style.md` |
| ADR-0007 | Auth: SPA session cookie + API bearer tokens + rotating refresh | PROPOSED | Identity module | `ADR-0007-authentication.md` |
| ADR-0008 | Background jobs: BullMQ on Redis + dedicated worker | PROPOSED | Async anything | `ADR-0008-background-jobs.md` |
| ADR-0009 | File storage: S3-compatible MinIO behind `StoragePort` | ACCEPTED | Media, documents | `ADR-0009-object-storage.md` |
| ADR-0010 | Search: PostgreSQL FTS behind `SearchPort` | PROPOSED | Search, indexes | `ADR-0010-search.md` |
| ADR-0011 | Frontend i18n/RTL: next-intl + logical CSS properties | PROPOSED | Every UI surface | `ADR-0011-i18n-rtl.md` |
| ADR-0012 | Bootstrap: CLI + one-time-token wizard for first org/admin | ACCEPTED | First run, deploy | `ADR-0012-bootstrap.md` |
| ADR-0013 | No runtime mock data; professional empty states | **ACCEPTED** | All runtime code | `ADR-0013-no-mock-data.md` |
| ADR-0014 | Optional bounded Python AI sidecar; AI output is a draft | **ACCEPTED** | Phase 10 | `ADR-0014-ai-sidecar.md` |
| ADR-0015 | Validation: class-validator DTOs authoritative + aligned Zod on UI | PROPOSED | DTOs, forms | `ADR-0015-validation.md` |

## 2. Phase-Gating ADRs

Phase 0 may not start until ADR-0001, 0003, 0004, 0005, 0009, 0012 are moved to
`ACCEPTED` and Q-01…Q-06 in `../../OPEN_QUESTIONS.md` are answered.

## 3. Related (non-ADR) Decisions Recorded Elsewhere

| Subject | Where |
|---------|-------|
| SLO targets | `../10-devops/monitoring.md` |
| Sensitive-data classes | `../11-security/privacy.md` |
| Privacy/retention matrix | `../11-security/privacy.md` |
| Offline sync deferral | `../../RISK_REGISTER.md` R-21 |
| Licensing (open) | `../../OPEN_QUESTIONS.md` Q-13 |

## 4. How to Add an ADR

1. Ask: does this change module boundaries, tenancy, data model, API version,
   security flow, storage/search choice, or deployment topology? If yes → ADR.
2. Copy `ADR-0000-template.md`, assign the next number, never reuse numbers.
3. Add the row to the register above **and** the canonical table in
   `TECHNICAL_DECISIONS.md`.
4. If the decision affects phases, update `IMPLEMENTATION_PLAN.md`.
5. If it supersedes an ADR, set the old record to `SUPERSEDED BY ADR-NNNN` and
   never edit its body again.

## Related

- Canonical rationale: `../../TECHNICAL_DECISIONS.md`
- System design: `../../ARCHITECTURE.md` · Change control: `ARCHITECTURE.md` §14

*End of docs/13-decisions/index.md*
