# TECHNICAL_DECISIONS.md — Architecture Decision Records

> **Status:** Active | **Owner:** Lead Architect | **Last Updated:** 2026-10-02
> **Rule:** A decision marked `PROPOSED` is not binding. `ACCEPTED` decisions are binding and may only be changed by a superseding ADR.
> **Individually filed copies:** `docs/13-decisions/ADR-NNNN-*.md`

| ADR | Decision | Status |
|-----|----------|--------|
| ADR-001 | Backend framework: NestJS + Fastify (primary), FastAPI optional AI sidecar | ACCEPTED |
| ADR-002 | Monorepo with pnpm workspaces, `apps/*` + `packages/*` | PROPOSED |
| ADR-003 | ORM & migrations: TypeORM native Nest integration (Prisma considered) | ACCEPTED |
| ADR-004 | Multi-tenancy: shared schema + mandatory `tenant_id` + RLS defence | ACCEPTED |
| ADR-005 | Tenant resolution: `X-Tenant-Id` header first, subdomain mapping later | ACCEPTED |
| ADR-006 | API style: REST `/api/v1`, OpenAPI generated, snake_case JSON | PROPOSED |
| ADR-007 | Auth: SPA session cookies + API bearer tokens + rotating refresh; MFA later | PROPOSED |
| ADR-008 | Background jobs: BullMQ on Redis, separate worker entrypoint | PROPOSED |
| ADR-009 | File storage: S3-compatible MinIO behind `StoragePort` | ACCEPTED |
| ADR-010 | Search: PostgreSQL FTS behind `SearchPort` (OpenSearch later) | PROPOSED |
| ADR-011 | Frontend i18n/RTL: next-intl + logical CSS properties | PROPOSED |
| ADR-012 | Bootstrap: CLI/bootstrap-token provisioning of first org + admin | ACCEPTED |
| ADR-013 | No runtime mock data; professional empty states; fixtures in tests only | ACCEPTED |
| ADR-014 | Optional Python AI/data sidecar with bounded contract, human approval | ACCEPTED |
| ADR-015 | Validation: shared Zod contracts at UI edge + class-validator DTOs at API | PROPOSED |

---

## ADR-001 — Backend Framework

**Context:** Two frozen specs conflict. `Engineering_Documentation_Package_v2`
states FastAPI/Python; `TECHNICAL_ARCHITECTURE_INSTRUCTIONS` states NestJS +
Fastify/Node/TS as mandatory. 40+ bounded contexts must be enforced as modules.

**Options:**
- **A. NestJS + Fastify (TypeScript).** First-class module/DI system; shares
  language and validation patterns with the frontend; native OpenAPI generation;
  mature guards/interceptors for tenant + authz + audit; large hiring pool.
- **B. FastAPI (Python).** Excellent OpenAPI and validation; strong data/AI
  ecosystem; but no equivalent opinionated module boundary system, and forces
  duplicated contract modelling across two languages.
- **C. Hybrid.** NestJS for transactional domain; Python only for AI/OCR/NLP.

**Decision (recommended):** **A + C** — NestJS/Fastify is the primary API and
domain runtime; Python is permitted *only* as an optional bounded sidecar.

**Reason:** The product's core difficulty is not HTTP performance but enforcing
50+ module boundaries, tenant isolation, RBAC/ABAC, and audit at every layer —
NestJS's module/guard system maps 1:1 to the mandatory requirements, and a single
language across UI and API keeps contracts consistent. Python's strengths
(NLP, OCR, embeddings) are isolated in a sidecar that never owns canonical data.

**Trade-offs:** Node single-threaded CPU-bound work must go to workers; Python
skills are still required for the AI sidecar; NestJS DI adds boilerplate.

**Consequences:** All backend docs/tasks must reference NestJS. FastAPI mentions
in the frozen Package must be read as applying to the optional AI service only.

**Migration impact:** None (greenfield). **Status:** PROPOSED — requires Q-01 sign-off.

---

## ADR-002 — Repository / Monorepo Layout

**Context:** Blueprint shows `yngo-cms/apps/*` + `modules/*`; Technical
Instructions show `apps/api/src/{core,modules,common}`. Both are valid.

**Decision:** Single monorepo, `pnpm` workspaces:

```text
yngo-cms/
├── apps/      web · api · worker · field (later) · docs · developer-portal
├── packages/  ui · types · sdk · config · i18n · auth · api-client
├── infra/     docker · compose · proxy · monitoring
├── migrations/
├── scripts/
├── tests/     e2e
└── docs/
```

**Reason:** One install, one lockfile, shared types/SDK, atomic cross-cutting
changes; matches both source docs. **Trade-offs:** larger repo, CI must use
affected-project filtering. **Migration impact:** none. **Status:** PROPOSED.

---

## ADR-003 — ORM and Migrations

**Context:** Canonical store is PostgreSQL + PostGIS; tenancy requires reliable
scoped queries; migrations must be append-only and rollback-tested.

**Options:** TypeORM (native Nest integration, PostGIS geometry support, query
builder, migrations) · Prisma (excellent DX/types, weaker PostGIS + RLS story) ·
Drizzle (lightweight, SQL-first, newer ecosystem).

**Decision (recommended):** **TypeORM** for entities + migrations, with raw SQL
escape hatches for PostGIS and performance-critical queries.

**Reason:** Native NestJS integration, direct `geometry`/`geography` support and
fine-grained query control are all required by this product; Prisma's PostGIS
gaps would force raw SQL for core GIS features.

**Trade-offs:** TypeORM's typing is weaker than Prisma's; discipline required to
avoid lazy-loading N+1. **Migration impact:** none. **Status:** PROPOSED (Q-02).

## ADR-004 — Multi-Tenancy Data Model

**Context:** Specs require that Tenant A can never access Tenant B data across
records, lists, search, exports, reports, documents, webhooks, jobs, and AI
retrieval — but do not prescribe a mechanism.

**Options:**
- **A. Shared schema + `tenant_id` column** (app-layer scoping, optional RLS).
- **B. Database-per-tenant.** Strong isolation; heavy operations and migrations.
- **C. Schema-per-tenant.** Moderate isolation; migration and connection complexity.

**Decision:** **A**, with PostgreSQL Row-Level Security enabled as
defence-in-depth on tenant-owned tables.

**Reason:** Best fit for a self-hostable platform hosting many small NGOs:
one migration path, simple backups, efficient pooling. RLS adds a second,
database-level guarantee so an application bug cannot silently cross tenants.

**Consequences:**
- Every tenant-owned table: `tenant_id uuid NOT NULL REFERENCES tenant(id)`, indexed.
- All data access goes through a tenant-scoped repository base (no raw entity
  manager access in modules).
- Composite indexes lead with `tenant_id`.
- Negative tests required for each surface (list, search, export, file, webhook, job).

**Trade-offs:** RLS adds a small query cost and requires session variable
handling; noisy-neighbour risk at extreme scale (mitigated later by extraction).

**Migration impact:** none. **Status:** PROPOSED (Q-04).

---

## ADR-005 — Tenant Resolution

**Context:** Multiple clients (admin SPA, public site, portals, PWA, external
APIs) must resolve a tenant deterministically without client trust.

**Decision:** Resolution order — (1) authenticated session/token's bound tenant;
(2) `X-Tenant-Id` header validated against the caller's memberships; (3) host/
subdomain mapping (`org.example.org`) once tenant domains are configured; (4)
path prefix only for public microsites. A request that resolves no tenant for a
tenant-scoped route is rejected with `403 TENANT_REQUIRED`.

**Reason:** Header-first keeps v1 simple and works for headless/external clients;
subdomain mapping is additive and does not change API contracts.

**Consequences:** `TenantGuard` runs after auth and before RBAC; tenant is never
taken from a request body; tenant id is attached to logs, audit, and job payloads.

**Trade-offs:** Header approach requires clients to be correct; mitigated by
verification against membership. **Migration impact:** none. **Status:** PROPOSED (Q-03).

## ADR-006 — API Style, Versioning, and Contracts

**Context:** Specs mandate REST/OpenAPI, `/api/v1`, consistent errors, pagination,
filtering, sorting, audit behaviour per endpoint, and no hand-written divergence.

**Decision:** REST + JSON under `/api/v1`. OpenAPI 3.1 is **generated** from
NestJS decorators and published in CI. JSON uses `snake_case` (matching the frozen
specs' examples). Uniform error envelope `{ error: { code, message, details, request_id } }`.
Cursor pagination on collections; whitelisted filtering/sorting; `Idempotency-Key`
on state-critical POSTs. Breaking change ⇒ `/api/v2` with `Deprecation`/`Sunset`.

**Reason:** Matches every frozen spec; generated contracts prevent drift; stable
error codes make clients (and agents) deterministic.

**Consequences:** Every controller declares auth, authz, validation, audit, and
error codes; contract tests run in CI. **Trade-offs:** naming convention
(`snake_case`) differs from TS idiom — a mapper sits at the DTO boundary.

**Migration impact:** none. **Status:** PROPOSED.

---

## ADR-007 — Authentication Strategy

**Context:** Clients: SPA, public site, portals, external API consumers, future
mobile. Specs require email/password, MFA, OIDC/SAML later, and never
`admin/admin` or shared default passwords.

**Decision:** Phase 2 = email/password with server-side sessions for the SPA
(httpOnly, SameSite=Lax, secure in TLS) + short-lived bearer access tokens with
rotating refresh tokens for API clients. API keys for machine integrations.
Phase 7 adds TOTP MFA and OIDC/SAML adapters. Passwords hashed with Argon2id.
All login, reset, MFA, and permission-denial events are audited.

**Reason:** Simplest secure mechanism first (per specs), with an explicit upgrade
path; cookie sessions avoid storing tokens in browser memory for the SPA.

**Trade-offs:** Two token mechanisms to maintain; requires CSRF protection for
cookie flows. **Migration impact:** none. **Status:** PROPOSED.

---

## ADR-008 — Background Jobs and Queue

**Context:** Exports, renditions, notifications, webhooks, report generation, and
automation must run asynchronously and survive process restarts.

**Decision:** BullMQ on Redis with a separate `worker` entrypoint from the same
codebase. Jobs are idempotent, carry `tenant_id` + `request_id`, use exponential
backoff, and move to a dead-letter queue after max attempts. Scheduled jobs use
repeatable queues. `pg-boss` remains the fallback if Redis must be removed.

**Reason:** Matches the mandated Redis infrastructure and the modular-monolith
constraint (no external broker at v1). **Trade-offs:** Redis becomes operationally
important; jobs must be written to tolerate Redis flush. **Status:** PROPOSED (Q-09).

---

## ADR-009 — Object Storage

**Context:** Media, documents, exports, and restricted attachments must not live
in the database or on the app filesystem.

**Decision:** `StoragePort` abstraction with an S3-compatible MinIO adapter as
the v1 implementation (also the reference self-hosted deployment). Keys are
server-generated, tenant-prefixed (`tenants/<tenant_id>/<class>/<uuid>.<ext>`),
original filenames never used. Private objects are served by short-lived presigned
URLs; downloads of sensitive classes are audited. Azure Blob/AWS S3 adapters are
additive.

**Reason:** Provider independence plus a legitimate self-hostable default for
NGO deployments. **Trade-offs:** MinIO must be operated and backed up; presigned
URL lifetimes must be tuned. **Migration impact:** none. **Status:** PROPOSED (Q-06).

---

## ADR-010 — Search Architecture

**Context:** Search must cover content and operational records, support Arabic
first-class, respect permissions and tenancy, and be swappable later.

**Decision:** `SearchPort` interface (index, update, delete, query with facets/
filters). v1 implementation = PostgreSQL full-text search (`tsvector` + GIN
indexes, Arabic configuration/dictionary strategy) plus JSONB filterable
attributes. OpenSearch is a drop-in later adapter with no API-contract change.
Search results are always tenant-scoped and permission-filtered, and sensitive
contexts (Beneficiary, Case, Safeguarding) are excluded for unauthorised actors
by policy, not by UI hiding.

**Reason:** Avoids operating a second data engine at v1 while keeping the exit
path explicit. **Trade-offs:** FTS tuning for Arabic requires care; scaling
beyond Postgres is deferred. **Status:** PROPOSED (Q-11).

## ADR-011 — Frontend i18n and RTL

**Context:** Arabic-first RTL is a first-class product principle; English LTR must
also work; per-tenant branding must not be hard-coded.

**Decision:** Next.js App Router with locale segments
(`/[locale]/...`), `next-intl` for messages/formatting, and Tailwind with
**logical properties** (`ms-*`, `me-*`, `ps-*`, `pe-*`, `text-start/end`) instead
of hard-coded left/right. `dir` is set per locale at the document level. Arabic
typography uses a dedicated font stack. Numbers/currency/dates use locale-aware
formatters. Tenant branding comes from design tokens stored per tenant, never from
compiled CSS constants.

**Reason:** Logical properties prevent RTL regressions; server-side locale
negotiation keeps public pages SEO-correct in both languages.

**Trade-offs:** Developers must never write physical direction utilities;
enforced by lint rule and code review. **Status:** PROPOSED (Q-10).

---

## ADR-012 — First-Run Bootstrap

**Context:** No mock data and no default credentials are allowed, yet a fresh
deployment must be able to create the first organisation and administrator.

**Decision:** A one-shot bootstrap command (`pnpm bootstrap:org`) that reads
operator-provided values from environment/prompt and creates the first Tenant,
Organization, and Administrator with a password supplied at runtime (never
defaulted, never logged). An optional guarded web setup wizard is enabled only
while the platform has zero tenants and requires a one-time bootstrap token from
the environment. The bootstrap endpoint self-disables after first success and is
audited.

**Reason:** Satisfies "no `admin/admin`" and "no automatic demo organisation"
while keeping first-run usable for non-technical operators.

**Trade-offs:** Requires a CLI step or a token for the wizard; documented in
`docs/10-devops/local-development.md` and the runbook. **Status:** PROPOSED (Q-05).

---

## ADR-013 — No Runtime Mock Data (ACCEPTED)

**Context:** Multiple frozen specs forbid fabricated business data presented as
real, while requiring usable empty states.

**Decision:** Runtime code MUST NOT contain or import mock/demo/sample/fake data.
A clean install is genuinely empty. Every collection view implements loading,
empty, error, populated, unauthorized, and (where relevant) partial states, with
professional empty-state copy and a clear next action (create or import).
Synthetic data is permitted **only** inside `tests/**` fixtures and is never
imported by application code. CI scans source (excluding `tests/`) for
`mockData|demoData|sampleData|fakeData|dummyData`, hardcoded statistics, and
`admin/admin`.

**Status:** ACCEPTED (binding). Any deviation requires a superseding ADR.

---

## ADR-014 — Optional Python AI/Data Sidecar (ACCEPTED)

**Context:** AI is an optional platform capability and must never be the source of
truth or overrule human decisions; Python is valuable for OCR/NLP/embeddings.

**Decision:** AI is accessed only through `AiPort`. The optional sidecar is a
separate FastAPI service with no database ownership and no credentials to the
canonical store beyond read-only, permission-scoped retrieval. Every AI output is
a **draft** requiring explicit human review before it becomes a record. AI
requests store model/provider, prompt version, timestamp, actor, source refs,
review status, and usage metadata. AI inherits the requesting user's permissions
and may never broaden them.

**Status:** ACCEPTED. Implementation deferred to Phase 10.

---

## ADR-015 — Validation Strategy

**Context:** Specs require Zod on the frontend and schema-aligned validation on
the backend, with the backend remaining authoritative.

**Decision:** The API owns validation with `class-validator` DTOs (the source of
truth for accepted input). The frontend uses Zod schemas aligned to those
contracts for immediate UX feedback and form ergonomics. Client validation is
never treated as a security control; server responses (including
`VALIDATION_ERROR` details) drive the final form state.

**Reason:** One authoritative validator, one UX-friendly mirror, no duplicated
trust. **Trade-offs:** Two schema definitions must be kept aligned; mitigated by
generated types from OpenAPI. **Status:** PROPOSED.

---

## Decision Log Conventions

- One ADR per decision; superseding an old ADR means creating a new one that
  references it and flipping the old status to `SUPERSEDED BY ADR-NNN`.
- ADRs are immutable once `ACCEPTED` except for a status line update.
- Every `PROPOSED` ADR maps to an open question in `OPEN_QUESTIONS.md`.
- Architectural changes during implementation are invalid until an ADR exists.

*End of TECHNICAL_DECISIONS.md*
