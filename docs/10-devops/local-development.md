# docs/10-devops/local-development.md

> **Status:** Current (target) — stack not yet implemented | **Owner:** DevOps
> **Last Updated:** 2026-10-02 | **Source:** `ARCHITECTURE.md` §11; `IMPLEMENTATION_PLAN.md` Phase 0
> **Goal:** a new contributor reaches a healthy stack in under 30 minutes (`NFR`/G-09).

## 1. Prerequisites

| Tool | Version | Why |
|------|---------|-----|
| Node.js | LTS (20.x or newer) | `api`, `worker`, `web` |
| pnpm | 9.x or newer | workspace package manager (ADR-002) |
| Docker + Docker Compose | Current stable | Postgres/PostGIS, Redis, MinIO, proxy |
| Git | Current | source control |
| Python | 3.11+ | **optional** — only for the AI sidecar profile |

Node and pnpm are needed to run the apps locally; Postgres, Redis, and MinIO run in
Compose so no host services are required.

## 2. First Run

```bash
git clone <repository> yngo-cms
cd yngo-cms
cp .env.example .env                 # then fill the required values (never commit .env)
pnpm install
docker compose up -d postgres redis minio
pnpm --filter api migration:run      # apply migrations (one-shot)
pnpm bootstrap:org                   # create the FIRST tenant + org + admin (ADR-012)
pnpm dev                             # api + worker + web in watch mode
```

Then open:

```text
http://localhost:3000        web (ar is the default locale)
http://localhost:4000/health api liveness
http://localhost:4000/ready  api readiness (DB + Redis + storage)
http://localhost:4000/api/docs OpenAPI UI (dev only)
http://localhost:9001        MinIO console
```

The stack starts **empty**: no demo tenant, no sample content (`ADR-013`). Populate
it through the admin UI, imports, or the API.

## 3. Environment Contract

```text
required at boot (validated, fail-fast):
  NODE_ENV · APP_URL · API_PORT · WEB_PORT · DATABASE_URL · REDIS_URL
  SESSION_SECRET · JWT_SECRET · STORAGE_ENDPOINT · STORAGE_BUCKET
  STORAGE_ACCESS_KEY · STORAGE_SECRET_KEY · DEFAULT_LOCALE · SUPPORTED_LOCALES
optional (features stay disabled without them):
  SMTP_* · SMS_* · WHATSAPP_* · OIDC_* · SAML_* · MAP_TILE_URL · AI_* · AV_*
```

Rules (`AGENTS.md` §10): `.env.example` documents every key with a safe placeholder
and a comment; secrets never appear in source, tests, logs, or CI output; the app
**fails fast** naming the missing variable; no production URL is hard-coded.

## 4. Common Commands

```bash
pnpm dev                       # api + worker + web (watch)
pnpm build                     # production build of web + api
pnpm lint                      # eslint + prettier check
pnpm typecheck                 # tsc --noEmit across workspaces
pnpm test                      # unit + integration
pnpm test:api                  # API/contract
pnpm test:authz                # authorization + tenant isolation
pnpm test:e2e                  # Playwright
pnpm --filter api migration:run
pnpm --filter api migration:revert
pnpm --filter api openapi:generate
docker compose logs -f api
docker compose down            # stop (keeps volumes)
docker compose down -v         # stop and DELETE local data (destructive)
```

## 5. Services and Ports (local)

| Service | Port | Notes |
|---------|:----:|-------|
| `web` | 3000 | Next.js dev server |
| `api` | 4000 | NestJS + Fastify |
| `postgres` | 5432 | PostgreSQL + PostGIS (volume-backed) |
| `redis` | 6379 | cache, rate limits, BullMQ |
| `minio` | 9000 / 9001 | S3 API / console |
| `proxy` | 8080 | optional local TLS-terminating proxy |
| `ai` (profile) | 8000 | FastAPI sidecar, `--profile ai` |
| `monitoring` (profile) | 9090 / 3000 | Prometheus / Grafana, `--profile monitoring` |

Compose profiles keep optional services out of the default footprint:

```bash
docker compose --profile monitoring up -d
docker compose --profile ai up -d
```

## 6. Database Workflow

```text
create migration     pnpm --filter api migration:create --name <verb-subject>
apply                pnpm --filter api migration:run
roll back one        pnpm --filter api migration:revert
inspect              docker compose exec postgres psql -U <user> -d <db>
reset locally only   docker compose down -v && docker compose up -d postgres
```

Rules: migrations are append-only, never edited after being applied, and never
carry business data (`docs/04-data/migrations.md`). Integration tests run against a
**real** database, created and migrated by CI — never against a mock
(`docs/09-testing/strategy.md`).

## 7. Bootstrap (first org + admin)

```bash
pnpm bootstrap:org
# prompts for: organization name (ar/en), admin email, admin password (typed)
# creates: tenant + organization + admin user + default role bundle
# writes an audit record; the route self-disables after success
```

Rules: no default credentials, no automatic demo organization, password supplied at
runtime and never logged; re-running without a token fails (`ADR-012`, `FR-003`).

## 8. Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| API exits at boot naming a variable | missing required env var | add it to `.env` (see `.env.example`) |
| `/ready` fails with a DB error | Postgres not up or migrations not applied | `docker compose up -d postgres` then `migration:run` |
| `/ready` fails on storage | MinIO not reachable or bucket missing | start MinIO; create the bucket per `STORAGE_BUCKET` |
| Uploads return 503 | storage unavailable | check storage container and credentials |
| Login fails with `AUTH_INVALID_CREDENTIALS` | no tenant/admin created yet | run `pnpm bootstrap:org` |
| Requests rejected with `TENANT_REQUIRED` | missing tenant context | sign in properly or send `X-Tenant-Id` per `ADR-005` |
| Rate limits firing locally | shared Redis keys from tests | flush the dev Redis namespace or use a separate DB index |
| Jobs never run | worker not started | `pnpm dev` includes the worker, or run `pnpm --filter worker dev` |

## 9. Definition of Done for an Environment Change

```text
[ ] .env.example updated with the new key, placeholder, and comment
[ ] boot-time validation updated and tested (missing var fails fast)
[ ] docs/10-devops/* updated in the same change
[ ] no secret committed; CI secret scan passes
[ ] compose file remains valid and documented
[ ] local-development instructions still reproduce a healthy stack from scratch
```

## Related

- Deployment and environments: `deployment.md` · Monitoring: `monitoring.md`
- Backup/restore: `backup-recovery.md` · Ops runbook: `../12-operations/runbook.md`
- Bootstrap decision: `../13-decisions/ADR-0012-bootstrap.md`

*End of docs/10-devops/local-development.md*