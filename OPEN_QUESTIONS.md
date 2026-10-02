# OPEN_QUESTIONS.md — YNGO-CMS / YemenNGO-CMS

> **Status:** Active | **Owner:** Lead Architect
> **Last Updated:** 2026-10-02 | **Source:** PROJECT_AUDIT.md §19
> **Rule:** BLOCKING before code. IMPORTANT before affected phase. OPTIONAL may be deferred.

## Summary

| Class | Count | IDs |
|-------|-------|-----|
| BLOCKING | 6 | Q-01 .. Q-06 |
| IMPORTANT | 5 | Q-07 .. Q-11 |
| OPTIONAL | 3 | Q-12 .. Q-14 |
| **Total** | **14** | |

---

## BLOCKING — Must resolve before Phase 0 (Foundation)

### Q-01 — Canonical backend stack

**Question:** Which backend stack is canonical? `Engineering_Package_v2` says FastAPI/Python; `TECHNICAL_ARCHITECTURE_INSTRUCTIONS` says NestJS/Fastify/Node/TS is mandatory.

**Recommended:** ADR-001 — NestJS + Fastify (primary); FastAPI/Python only as optional bounded AI sidecar behind `AiPort`. No Python in `apps/api`.

**Decides:** ADR-001 | **Blocks:** repo init, all backend tasks.

### Q-02 — ORM and migration tool

**Question:** Which ORM / migration toolchain for Postgres+PostGIS?

**Options:** A. Prisma (best DX, weaker PostGIS/RLS) · B. TypeORM (native NestJS, strong PostGIS) · C. Drizzle (SQL-close, small NestJS surface).

**Recommended:** ADR-003 — **TypeORM** default; Prisma admissible via later ADR if parity proven.

**Decides:** ADR-003 | **Blocks:** DB schema, migrations, `apps/api` scaffolding.

### Q-03 — Tenant resolution mechanism

**Question:** How does the platform resolve the current tenant on each request?

**Options:** Subdomain (`acme.yngo.ye`), header (`X-Tenant-Id`), path prefix (`/t/acme`).

**Recommended:** ADR-005 — header `X-Tenant-Id` as v1 canonical; subdomain mapping later behind same `TenantContext`. Path prefix rejected.

**Decides:** ADR-005 | **Blocks:** TenantGuard, routing, auth.

### Q-04 — Tenancy data isolation model

**Question:** Shared schema + `tenant_id`, row-level security, or schema-per-tenant / DB-per-tenant?

**Recommended:** ADR-004 — **Shared schema + mandatory `tenant_id`** on every tenant-owned table, tenant-scoped repository base, optional Postgres RLS as defence-in-depth.

**Decides:** ADR-004 | **Blocks:** every migration, every query, security review.

### Q-05 — Bootstrap of the first organization and admin

**Question:** How is the very first tenant/org/admin created when the DB is empty, without violating "no `admin/admin`" and "no automatic demo org"?

**Recommended:** ADR-012 — CLI `pnpm bootstrap:org` (primary) + optional one-time-token web wizard that self-disables after first success. Password at runtime, never defaulted.

**Decides:** ADR-012 | **Blocks:** local dev, first deploy, onboarding.

### Q-06 — Reference object storage for v1

**Question:** Which S3-compatible storage is the v1 reference?

**Recommended:** ADR-009 — **MinIO self-hosted** behind `StoragePort`; swap to AWS S3 / Azure Blob via adapter. Runs in compose for dev and reference prod.

**Decides:** ADR-009 | **Blocks:** Media, Documents, any file upload.

---

## IMPORTANT — Must resolve before the affected phase

### Q-07 — GIS base map source

**Question:** Default base map tiles — public OSM, self-hosted, or vendor?

**Default:** Public OSM tiles for v1; self-hosted tile server behind `MapPort` when offline Yemen deployments require it. No vendor lock-in at v1. **Phase:** GIS (Phase 6).

### Q-08 — Package manager

**Question:** pnpm vs npm vs yarn?

**Recommended:** ADR-002 — **pnpm workspaces** (fastest, strictest, best monorepo). CI/docs assume pnpm.

### Q-09 — Background queue

**Question:** BullMQ (Redis) vs pg-boss (Postgres-only)?

**Recommended:** ADR-008 — **BullMQ on Redis** + separate `worker`. Redis already required for cache/rate-limit.

### Q-10 — Frontend i18n framework

**Question:** `next-intl` vs `next-i18next` / `i18next` for Arabic-first RTL?

**Recommended:** ADR-011 — **next-intl** (first-class App Router + RSC, ICU, RTL `dir` handling).

### Q-11 — Search abstraction

**Question:** What is the `SearchPort` interface so PG FTS can swap for OpenSearch without domain changes?

**Default:** `SearchPort { index(doc), remove(id), search(query, filters, pagination) }` with tenant + permission predicates mandatory on every call. Full contract in `docs/03-architecture/`.

---

## OPTIONAL — May be deferred with a documented default

### Q-12 — AI provider(s) for Phase 10

**Question:** Which LLM / embedding / OCR providers for the optional AI sidecar, and what data-privacy routing rules apply?

**Default:** No provider at v1. AI disabled by default; when enabled, ADR-014 governs.

### Q-13 — Licensing model for the platform itself

**Question:** Under which licence will YNGO-CMS be distributed (AGPL, MIT, proprietary)?

**Default:** Undetermined — no public distribution until decided. Record before external release.

### Q-14 — SMS / WhatsApp provider for Yemen context

**Question:** Preferred SMS/WhatsApp gateway(s) for Yemen?

**Default:** `NotifierPort` with SMTP + in-app at v1; SMS/WhatsApp adapters added when a provider is selected.

---

## Decision Log

| Date | Question | Answer | ADR | Approved by |
|------|----------|--------|-----|-------------|
| 2026-10-02 | — | Initial 14 questions recorded | — | Lead Architect (draft) |
| 2026-10-02 | Q-01 | **NestJS + Fastify** (primary); FastAPI Python only as optional bounded AI sidecar behind `AiPort`. No Python in `apps/api`. | ADR-001 → **ACCEPTED** | Lead Architect |
| 2026-10-02 | Q-02 | **TypeORM** with native NestJS integration; Prisma reconsiderable via later ADR once PostGIS parity is proven. | ADR-003 → **ACCEPTED** | Lead Architect |
| 2026-10-02 | Q-03 | **`X-Tenant-Id` header** as v1 canonical; subdomain mapping enabled by config behind the same `TenantContext`. Path prefix rejected. | ADR-005 → **ACCEPTED** | Lead Architect |
| 2026-10-02 | Q-04 | **Shared schema + mandatory `tenant_id`** on every tenant-owned table; tenant-scoped repository base; optional Postgres RLS as defence-in-depth. | ADR-004 → **ACCEPTED** | Lead Architect |
| 2026-10-02 | Q-05 | **CLI `pnpm bootstrap:org`** (primary) + optional one-time-token web wizard that self-disables after first success. Password at runtime, never defaulted. | ADR-012 → **ACCEPTED** | Lead Architect |
| 2026-10-02 | Q-06 | **MinIO self-hosted** behind `StoragePort`; swap to AWS S3 / Azure Blob via adapter. Runs in Compose for dev and reference prod. | ADR-009 → **ACCEPTED** | Lead Architect |

*End of OPEN_QUESTIONS.md*
