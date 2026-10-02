# docs/10-devops/deployment.md

> **Status:** Current (target) — no deployment exists yet | **Owner:** DevOps
> **Last Updated:** 2026-10-02 | **Source:** `ARCHITECTURE.md` §11; `IMPLEMENTATION_PLAN.md` Phase 11
> **Purpose:** environments, release process, and rollback.

## 1. Environments

| Environment | Purpose | Data | Access |
|-------------|---------|------|--------|
| `dev` | local development | disposable, never production data | developer machine |
| `test` | CI runs (unit/integration/API/authz/e2e) | created and destroyed per run | CI only |
| `staging` | pre-production verification | a copy/refresh of production-shaped data or none | internal team |
| `prod` | live service | real tenant data | restricted, audited |

Rules (`AGENTS.md` §10, `NFR-010`): **identical images** across environments with
different configuration; no environment-specific code branches; credentials and
endpoints only from configuration; production data never leaves `prod` except
through an approved, audited export.

## 2. Reference Deployment

```text
Ubuntu LTS host
  ├── Docker Engine + Docker Compose
  ├── proxy (TLS termination, routing, header hygiene)
  ├── web (Next.js)  ──┐
  ├── api (NestJS)   ──┼── internal network (not publicly reachable)
  ├── worker (BullMQ)──┘
  ├── postgres + PostGIS   (volume)
  ├── redis                (volume/cache)
  └── minio (S3 API)       (volume)

Volumes          named volumes for DB, Redis, and object data
Secrets          .env with chmod 600, or an external secret store
TLS              managed by the proxy (ACME or provided certificates)
Backups          scheduled jobs + off-site copy (backup-recovery.md)
```

Scaling path (`ARCHITECTURE.md` §13): vertical first → additional `api` replicas
behind the proxy → read replicas + PgBouncer → extract proven hotspots (worker,
search, reporting) without changing API contracts.

## 3. Build and Release Pipeline

```text
1  merge to the default branch (protected; PR + review required)
2  CI: lint → typecheck → unit → integration → API/contract → authz/tenant
       → build images → secret scan → dependency scan
3  images tagged by immutable version (semantic version + git sha)
4  migration job runs as a ONE-SHOT step before application rollout
5  deploy: rolling replace of api, then worker, then web (proxy keeps upstreams healthy)
6  smoke check: /health, /ready, a read of a public endpoint
7  release notes + version tag + migration instructions published (CHANGELOG)
```

Health gating: `api`/`worker` define `healthcheck`; the proxy only routes to healthy
upstreams; a failing readiness check aborts the rollout before traffic shifts.

## 4. Configuration Discipline

```text
required-at-boot vars          validated; app fails fast naming the missing key
secrets                        never in the image, never in git, never logged
per-tenant configuration       in the database (integration_config, settings) — encrypted
feature flags                  configuration-driven; disabled features return FEATURE_DISABLED
no hard-coded domains          APP_URL and provider endpoints come from configuration
```

## 5. Migration Deployment Rules

1. Migrations run **before** the new application version starts serving traffic.
2. Migrations must be backward-compatible with the currently running version
   (expand → deploy → contract) so a rollback of the app remains possible.
3. A destructive migration requires a verified backup and an ADR
   (`docs/04-data/migrations.md` §7).
4. Migration failure aborts the rollout; the previous version keeps serving.

## 6. Rollback

| Situation | Action |
|-----------|--------|
| Bad application release | redeploy the previous image tag (no schema change needed if migrations were expand-only) |
| Bad migration (additive) | run the migration's `down()` in the one-shot job, then deploy the previous version |
| Bad migration (destructive) | restore from the pre-migration backup (documented in `backup-recovery.md`), then deploy the previous version |
| Dependency failure (external provider) | disable the integration; core flows must keep operating |
| Data corruption discovered | stop writes for the affected tenant, restore, replay from audit where possible |

Every rollback is recorded with what happened, why, and what changed afterwards.

## 7. Release Gate (Phase 11 exit)

```text
[ ] load test meets SLOs (API p95 < 300 ms; search p95 < 500 ms) at target scale
[ ] restore-from-backup verified on a clean environment within the documented RTO
[ ] no Critical/High risk remains OPEN without a signed ADR acceptance
[ ] all documentation sets current and cross-linked
[ ] release notes + version tag + migration instructions published
[ ] no secrets in images, logs, or the repository
[ ] security review completed with no open critical finding
```

## 8. Post-Deploy Verification

```text
1  /health returns 200
2  /ready returns 200 (DB + Redis + storage all reachable)
3  a public page renders the published content it did before the release
4  an authenticated read succeeds against the versioned API
5  a background job completes (report/export) to prove the queue is healthy
6  key metrics are flat (error rate, latency, queue depth) for a monitoring window
```

Any failure triggers rollback rather than "watch and hope".

## Related

- Local environment: `local-development.md` · Monitoring and SLOs: `monitoring.md`
- Backup and restore: `backup-recovery.md` · Runbook: `../12-operations/runbook.md`
- Root architecture: `../../ARCHITECTURE.md` §11, §13

*End of docs/10-devops/deployment.md*