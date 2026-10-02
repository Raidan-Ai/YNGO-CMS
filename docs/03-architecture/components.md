# docs/03-architecture/components.md

> **Status:** Current (inventory) — implementation `PLANNED` | **Owner:** Lead Architect
> **Last Updated:** 2026-10-02 | **Source:** `ARCHITECTURE.md` §4–§5
> **Purpose:** container and component inventory. Module-level detail lives in
> `../07-backend/modules.md`; this file covers deployable units and shared packages.

## 1. Containers (deployable units)

| Container | Tech | Entry | Responsibility | Stateful |
|-----------|------|-------|----------------|:--------:|
| `proxy` | Caddy/nginx | HTTPS 443 | TLS termination, routing, header hygiene, gzip/brotli, health-based upstream selection | No |
| `web` | Next.js (Node) | HTTP 3000 | Public site, admin SPA, portals, auth pages; SSR/ISR; no business logic | No |
| `api` | NestJS + Fastify | HTTP 4000 | The only write path to canonical data; all validation, guards, audit | No |
| `worker` | Node + BullMQ | Queue consumer | Async jobs: exports, renditions, notifications, webhooks, reports, automation, imports, search indexing, maintenance | No |
| `postgres` | PostgreSQL + PostGIS | TCP 5432 | Canonical transactional + geospatial store, audit schema | **Yes** |
| `redis` | Redis | TCP 6379 | Cache, rate-limit counters, BullMQ queues | Rebuildable |
| `minio` | MinIO (S3 API) | HTTP 9000 | Object storage for media/documents/exports/import reports | **Yes** (source objects) |
| `ai` (optional profile) | FastAPI (Python) | HTTP 8000 | Bounded AI/OCR/NLP capabilities behind `AiPort`; no DB ownership | No |
| `monitoring` (optional profile) | Prometheus + Grafana | HTTP 9090/3000 | Metrics scrape, dashboards, alerts | Yes (metrics) |

`api` and `worker` are **the same image**, different entrypoints — one codebase,
one dependency set, identical configuration contract (`NFR-010`).

## 2. Component Breakdown of `api`

```text
api
├── core/            config · database · logging · errors · health · events
│                    security · pagination · validation · idempotency
├── core/ports/      StoragePort · SearchPort · NotifierPort · IdpPort
│                    AiPort · AntivirusPort · MapPort · PaymentPort
├── common/          decorators (@RequirePermissions, @PublicRoute, @Audit)
│                    guards · interceptors · filters · pipes · base repository
├── modules/         52 bounded contexts (see ../07-backend/modules.md)
└── jobs/            queue producers (job payload builders, not consumers)
```

Components and their responsibilities are the module anatomy in
`../07-backend/architecture.md`; the pipeline (guards → interceptors → controller →
service → domain → repository) is defined there and is identical for every module.

## 3. Component Breakdown of `web`

```text
web
├── app/[locale]/(site)/     public website — ISR/SSR, SEO, hreflang
├── app/[locale]/(admin)/    authenticated admin SPA
├── app/[locale]/(portal)/   donor/partner/applicant/volunteer/member portals
├── app/[locale]/(auth)/     login, MFA, reset, invitation acceptance
├── components/              composed feature components (per module folder)
├── lib/api/                 generated typed client + query hooks
├── lib/i18n/                next-intl setup, ar default, logical CSS only
└── lib/permissions/         UX-only permission gating helpers
```

Rules: no business logic in components, no direct data access, query keys
namespaced by tenant + resource, and every data view implements loading / empty /
error / unauthorized / populated / partial states (`docs/06-frontend/*`).

## 4. Shared Packages

| Package | Contents | Consumed by |
|---------|----------|-------------|
| `packages/ui` | shadcn/Radix primitives, DataTable, EmptyState, ErrorState, Skeleton, StatusBadge, Timeline, Chart, Map | `web` |
| `packages/types` | Generated/shared domain types (from OpenAPI) | `web`, `api` (contract types only), `sdk` |
| `packages/sdk` | Published SDK surface (generated) | external integrators |
| `packages/api-client` | Generated typed client + query hooks | `web`, `tests/e2e` |
| `packages/config` | ESLint/TS/Tailwind configs shared across workspaces | all |
| `packages/i18n` | Message catalogs (ar/en), formatting helpers | `web` |
| `packages/auth` | Permission-check helpers (UX only), session utilities | `web` |

`packages/*` never import from `apps/*`. Shared code that needs database or domain
behaviour belongs in `api`, not in a package.

## 5. Worker Components

```text
worker
├── consumers/     one processor per queue (jobs/exports, renditions, ...)
├── schedulers/    cron-style repeatable jobs (deadline scans, retention, backups)
├── runtime/       tenant + request context hydration from job payload
└── observability/ job metrics, dead-letter visibility
```

Every processor resumes tenant context from the payload, opens its own transaction,
and is idempotent (`docs/07-backend/services.md` §5).

## 6. Dependency Direction (enforced)

```text
web ──▶ api-client ──▶ api  (HTTP only, generated contract)
web ──▶ ui · i18n · auth · types      (no cycle back into web)
api ──▶ core · common · modules        (modules never import each other's repositories)
worker ──▶ api core (shared libraries) — never the reverse at runtime
api/worker ──▶ ports ──▶ adapters → external systems
```

Violations (a package importing an app, a module importing another module's
repository, a component calling the database) are architecture defects and require
an ADR plus refactor — they are never "temporary".

## Related

- Root architecture: `../../ARCHITECTURE.md` §4–§5
- Context and boundaries: `system-context.md` · Flows: `data-flow.md`
- Modules: `../07-backend/modules.md` · Frontend: `../06-frontend/architecture.md`

*End of docs/03-architecture/components.md*