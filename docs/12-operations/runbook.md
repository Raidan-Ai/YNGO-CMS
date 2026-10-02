# docs/12-operations/runbook.md

> **Status:** Current (planned procedures) — none rehearsed yet | **Owner:** DevOps
> **Last Updated:** 2026-10-02 | **Source:** `ARCHITECTURE.md` §11–§13, `../10-devops/*`
> **Rule:** procedures are `PLANNED` until exercised on a real environment; record the
> drill result in §9 when run.

## 1. Purpose & Scope

Day-2 operational procedures for a running YNGO-CMS deployment: startup/shutdown,
health verification, common incidents, backup/restore, upgrades, and access reviews.
Environment specifics (URLs, hosts, secrets) come from configuration only — never
hard-coded (`AGENTS.md` §10).

## 2. Service Inventory

| Service | Role | Health | Notes |
|---------|------|--------|-------|
| `proxy` | TLS, routing, headers | upstream health | Only public entry point |
| `web` | Next.js (public + admin) | HTTP 200 | SSR/ISR for public pages |
| `api` | NestJS + Fastify | `/health`, `/ready` | Stateless; horizontally scalable |
| `worker` | BullMQ jobs | process + queue depth | Idempotent jobs; retries |
| `postgres` | PG + PostGIS (canonical) | `pg_isready` | WAL archiving on |
| `redis` | cache/queue/rate-limit | `PING` | Non-canonical; rebuildable |
| `minio` | object storage | S3 health | Non-canonical; versioned |
| migrations | one-shot job | exit 0 | Runs before app rollout |

## 3. Standard Procedures

### 3.1 Start / verify the stack

```text
1  docker compose up -d
2  wait for postgres healthy (pg_isready)
3  run the migration one-shot job → expect exit 0
4  verify api /health (liveness) and /ready (DB+Redis+storage) return 200
5  verify worker is consuming (queue depth drops to steady state)
6  verify proxy routes web + api; TLS certificate valid
```

### 3.2 Stop / restart

```text
1  drain: stop sending traffic at the proxy (or scale api to 0 upstream)
2  stop worker first (finish in-flight jobs; no new jobs accepted)
3  stop api replicas, then web
4  leave postgres/redis/minio running unless doing maintenance
5  restart in reverse order; re-run readiness checks
```

### 3.3 Bootstrap a new environment (ADR-012)

```text
1  Ensure the database is genuinely empty (no tenants)
2  Run `pnpm bootstrap:org`; supply org name, admin email, admin password
3  Confirm tenant + organization + admin exist and an audit record was written
4  Confirm the bootstrap path is self-disabled (re-run without token fails)
5  NEVER create demo data — an empty system is expected (ADR-013)
```

## 4. Incident Playbooks

| Symptom | Likely cause | First actions |
|---------|--------------|---------------|
| `/ready` fails, API 503 | Postgres unavailable | Check postgres health/disk/connections; do not force writes; verify workers paused/retrying |
| Elevated 5xx rate | API error spike | Check logs by `request_id`; inspect recent deploy; roll back to previous image if correlated |
| Queue backlog growing | worker down / failure loop | Check worker process + dead-letter queue; requeue after fix; jobs are idempotent |
## 5. Backup, Restore & DR

- Baseline: daily DB backup + WAL archiving, weekly full backup, object versioning,
  off-site copy, last-success surfaced in admin (`ARCHITECTURE.md` §13).
- Restore drill steps and RTO/RPO targets: `../10-devops/backup-recovery.md`.
- A restore is not proven until it runs on a **clean** environment and the API passes
  `/ready` against the restored data (TASK-112).

## 6. Upgrade / Release Procedure

```text
1  Read release notes + migration instructions (CHANGELOG.md)
2  Take a fresh backup and verify it completed
3  Deploy identical images with new configuration (dev/test/staging/prod parity)
4  Run the migration one-shot job; confirm exit 0 and documented rollback path
5  Roll out api replicas, then web, then worker
6  Verify /health, /ready, queue depth, and a smoke flow (login → list → audit)
7  On failure: roll back images; apply the tested migration rollback if schema changed
```

## 7. Routine Access & Secret Reviews

| Cadence | Action |
|---------|--------|
| Monthly | Review platform-admin and cross-tenant accounts; confirm each is justified |
| Per phase exit | Re-score risks; confirm no Critical risk is open without ADR acceptance |
| Quarterly | Rotate API keys and webhook secrets; verify old secrets are rejected |
| Before release | Confirm no secrets in source/logs; dependency scan has no Critical findings |

## 8. On-Call Checklist

```text
[ ] Confirm scope: which environment, which tenant(s), customer-visible?
[ ] Capture request_id / timestamps / error codes before changing anything
[ ] Check health endpoints and recent deploys first
[ ] Prefer reversible actions; never force-write canonical data
[ ] Log the incident: start time, actions, evidence, resolution
[ ] If sensitive data is involved, follow privacy.md (reason capture, audit)
[ ] Open a follow-up task for the root cause; add a regression test
```

## 9. Drill Record

| Date | Procedure | Environment | Result | Notes |
|------|-----------|-------------|--------|-------|
| — | none run yet (greenfield) | — | — | All procedures `PLANNED` until first rehearsal |

## Related

- Local dev: `../10-devops/local-development.md` · Deploy: `../10-devops/deployment.md`
- Monitoring + SLOs: `../10-devops/monitoring.md` · Backup/DR: `../10-devops/backup-recovery.md`
- Security + privacy: `../11-security/security.md`, `../11-security/privacy.md`
- Risks: `../../RISK_REGISTER.md` (R-02, R-10, R-11)

*End of docs/12-operations/runbook.md*
| Redis unavailable | cache/queue outage | Rate limits degrade in-process, jobs delay, cache misses fall through; canonical data unaffected |
| Object storage errors | MinIO down/full | Uploads/list return 503; metadata not committed without a stored object; free space/restart |
| Login failures spike | brute force / IdP outage | Check throttling + login events; local admin credentials remain valid if IdP is down |
| Webhook/notification failures | provider down | Retries with backoff then dead-letter; visible in admin; verify HMAC secrets |
| Permission-denial surge | role/policy misconfiguration | Denials are security events; review recent access changes; never broaden roles to "fix" |
| Cross-tenant suspicion | isolation bug | Treat as **Critical** (R-02): capture `request_id`, freeze deploys, open incident, add negative test |