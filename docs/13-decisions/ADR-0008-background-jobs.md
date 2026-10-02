# ADR-0008 — Background Jobs and Queues

> **Status:** PROPOSED | **Owner:** Backend Lead | **Date:** 2026-10-02
> **Canonical rationale:** `../../TECHNICAL_DECISIONS.md` (ADR-008)
> **Decides:** `../../OPEN_QUESTIONS.md` Q-09 | **Blocks:** reports, exports, notifications, imports, webhooks

## Context

Reports, exports, imports, notification delivery, webhook delivery, media
processing, scheduled automations, and (later) AI jobs must not block HTTP
requests. Redis is already required for caching and rate limiting, and the
deployment reference is a small number of containers on one host.

## Options

| Option | Pros | Cons |
|--------|------|------|
| A. BullMQ on Redis | Mature: retries, backoff, repeatables, priorities, concurrency, dashboard | Adds Redis as a hard dependency for async work |
| B. pg-boss (Postgres) | No extra infrastructure; transactional with data | Extra load on the primary DB; less mature tooling |
| C. In-process worker | Zero infrastructure | Not durable; dies with the API process; unacceptable for exports/reports |

## Decision

**BullMQ on Redis with a dedicated `apps/worker` process. Job producers live in the
API; consumers live in the worker; the API never performs long-running work
inline.**

## Mandatory Rules

1. Every job type declares: payload schema, tenant id, max attempts, backoff,
   timeout, idempotency key, and dead-letter behaviour.
2. Jobs are idempotent — retries must not duplicate business records or
   notifications (`Idempotency-Key` per delivery).
3. Every job run is attributable: `request_id`, `tenant_id`, `actor_id`, job id.
4. Long jobs report progress and are cancellable by the requester or an admin.
5. Failures surface in the admin console (queue depth, failed jobs, dead letters);
   silent failure is a bug.
6. Scheduling (repeatable jobs, SLA timers, digest notifications) is configuration,
   never hard-coded cron strings in source.
7. Redis loss degrades gracefully: reads fall through to the DB; queues delay and
   recover; canonical data is never stored only in Redis.

## Consequences

- Positive: durable async pipeline, one worker entrypoint to scale independently.
- Negative: Redis becomes operationally significant (persistence, memory policy).
- Follow-up: document queue names and retention in `../12-operations/runbook.md`.

## Related

- Monitoring: `../10-devops/monitoring.md` · Runbook: `../12-operations/runbook.md`
- Failure behaviour: `../../ARCHITECTURE.md` §13

*End of ADR-0008*