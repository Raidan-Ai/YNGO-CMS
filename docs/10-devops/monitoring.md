# docs/10-devops/monitoring.md

> **Status:** Current (target) — no metrics collected yet | **Owner:** DevOps
> **Last Updated:** 2026-10-02 | **Source:** `ARCHITECTURE.md` §12; `NFR-003`, `NFR-008`
> **Purpose:** signals, SLOs, and alert thresholds. These SLO targets are the
> authoritative numbers other docs cite (e.g. `../00-overview/goals.md`).

## 1. Signals

| Signal | Implementation | Retention |
|--------|----------------|-----------|
| Logs | Structured JSON: `ts, level, msg, request_id, tenant_id, user_id, module, action, duration_ms, error.code` | per policy (no PII/secrets) |
| Traces | `X-Request-Id` propagated UI → API → worker → provider; spans for DB, queue, storage, provider calls | short, sampled |
| Metrics | HTTP latency/error rate, DB pool, queue depth/failures, storage ops, webhook delivery rate, report duration | per retention policy |
| Health | `/health` (liveness) and `/ready` (DB + Redis + storage) | live |
| Audit | Business audit log + security events (login failures, permission denials, exports) | append-only, policy-driven |
| Errors | Error-reporting adapter; payloads carry no PII | per adapter |

Logs, metrics, and traces share `request_id`, so any user-visible failure traces to
the exact request, job, and tenant.

## 2. Service Level Objectives

| SLO | Target | Measured | Source of truth |
|-----|--------|----------|-----------------|
| API latency | p95 < 300 ms | rolling window on the request metric | this file |
| Search latency | p95 < 500 ms | rolling window on the search metric | this file |
| Standard report job | < 60 s | job duration histogram | this file |
| Availability | 99.5 % monthly | `/ready` success ratio | this file |
| Error rate | < 1 % of requests (5xx) | request metric | this file |
| Queue health | dead-letter count = 0 for > 1 h | queue metric | this file |
| Webhook delivery | ≥ 99 % of deliveries succeed within retry budget | delivery metric | this file |
| Backup success | last successful backup within 24 h | backup job metric | `backup-recovery.md` |

Targets are **initial SLO proposals** confirmed at Phase 11 (`TASK-110`/`TASK-112`)
and are not yet measured because no code exists (`docs/00-overview/goals.md` §3).

## 3. Alert Thresholds

| Alert | Threshold | Severity | First action |
|-------|-----------|----------|--------------|
| API down | `/health` fails 2 consecutive checks | Critical | check container + proxy upstreams; roll back if a deploy caused it |
| Not ready | `/ready` failing > 2 min | Critical | identify which dependency failed (DB/Redis/storage) |
| DB unavailable | connection errors > 5 in 1 min | Critical | verify DB container/host; pause workers; do not retry-storm |
| Error rate spike | 5xx > 5 % for 5 min | Critical | inspect recent deploy; consider rollback |
| Latency breach | api p95 > 600 ms for 10 min | High | check DB pool, slow queries, queue backlog |
| Queue backlog | depth > 500 or age > 10 min | High | check worker health; scale workers |
| Dead-letter growth | any dead-letter > 0 for 1 h | High | inspect failed jobs; fix then redeliver |
| Webhook failures | delivery failure rate > 5 % for 15 min | High | check endpoint health; notify tenant admin |
| Storage unavailable | upload/list errors > 5 in 1 min | High | verify storage service + credentials |
| Backup missed | no successful backup in 24 h | High | run a manual backup; investigate the job |
| Auth failures | login failures > 20/min from one IP | Medium | rate-limit/block; investigate brute force |
| Permission denials | spike > 50/min for one actor | Medium | possible misconfigured role or attack; review security events |
| Retention job failure | job error | Medium | re-run; check object/permission issues |

Alert rules never include sensitive payloads — they reference metric names,
`request_id`, and identifiers only.

## 4. Dashboards (minimum)

```text
1  Service health      health/readiness, uptime, version, deploy markers
2  API performance     p50/p95/p99 latency, error rate, top routes, rate-limit hits
3  Database            connections, pool saturation, slow queries, replication/backup state
4  Queues              depth per queue, processing rate, failures, oldest job age
5  Storage             request rate, errors, bytes stored, signed-URL issuance
6  Security            failed logins, permission denials, export attempts, blocked uploads
7  Business-safe KPIs  request volume per tenant (aggregate only, never PII)
```

## 5. Health Check Contract

```text
GET /health   → 200 { status: "ok", version, uptime_s }
                no tenant context, no dependency checks, no sensitive detail

GET /ready    → 200 { status: "ready", checks: { database: "ok", redis: "ok", storage: "ok" } }
              → 503 { status: "not_ready", checks: { ... failing dependency } }

Rules  liveness never depends on a dependency (so the proxy does not restart a
       healthy process during a DB blip); readiness fails closed, so traffic is not
       routed to an instance that cannot serve correctly.
```

## 6. Logging Rules

```text
[ ] structured JSON only; no console.log in application code
[ ] every entry carries request_id, and tenant_id/user_id where known
[ ] no secrets, tokens, passwords, raw API keys, or PII
[ ] sensitive request/response bodies are never logged
[ ] audit before/after values are redacted per classification
[ ] log levels: error = actionable failure · warn = degraded · info = business event ·
    debug = development only (never enabled in prod by default)
[ ] logs are shipped/rotated per policy; disk growth is monitored
```

## 7. Continuity of Evidence

Metrics and logs support the SLOs claimed elsewhere, but they never replace audit:
audit records are the authoritative history of who changed what, and they are
append-only with no `UPDATE`/`DELETE` grants (`docs/04-data/database.md`).

## Related

- Root architecture: `../../ARCHITECTURE.md` §12
- Deployment and release checks: `deployment.md` · Backup signals: `backup-recovery.md`
- Ops procedures: `../12-operations/runbook.md` · SLO references: `../00-overview/goals.md`

*End of docs/10-devops/monitoring.md*