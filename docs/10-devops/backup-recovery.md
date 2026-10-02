# docs/10-devops/backup-recovery.md

> **Status:** Current (policy) — no backup job exists yet | **Owner:** DevOps
> **Last Updated:** 2026-10-02 | **Source:** `ARCHITECTURE.md` §13; `NFR-009`, `R-10`
> **Purpose:** what is backed up, how, and how restore is proven.

## 1. Backup Scope

| Data | Method | Frequency | Retention |
|------|--------|-----------|-----------|
| PostgreSQL (canonical + audit + PostGIS) | WAL archiving (PITR) | continuous | per retention policy |
| PostgreSQL base | logical or physical base backup | daily | 30 days |
| PostgreSQL full | full backup + integrity verification | weekly | 12 weeks |
| Object storage (media, documents, exports, import reports) | bucket versioning + replication | continuous / daily | lifecycle per retention policy |
| Configuration | `.env`/secret store snapshot (encrypted) | on change | 12 weeks |
| Infrastructure definitions | git (compose, proxy config, scripts) | on change | indefinite |
| Metrics/monitoring data | provider retention | continuous | per provider |

Rules: backups are **encrypted at rest** and copied **off-site** to an independent
location; backup jobs are monitored (`monitoring.md` §3 "Backup missed"); the last
successful backup is surfaced in the admin console so failure is never silent.

## 2. Schedule

```text
continuous   WAL archiving → point-in-time recovery to any moment in the window
daily        base backup + object storage snapshot verification
weekly       full backup + restore-ability verification (integrity check)
monthly      restore DRILL into an isolated environment (not production)
on-change    configuration/secret snapshot when deployment config changes
```

## 3. Recovery Objectives

| Scenario | RPO (max data loss) | RTO (max downtime) | Method |
|----------|--------------------:|-------------------:|--------|
| Database corruption / bad migration | ≤ 5 min (WAL) | ≤ 2 h | PITR to just before the event |
| Accidental row/table deletion | ≤ 5 min | ≤ 2 h | PITR to a point before deletion |
| Failed application release | 0 (no data change) | ≤ 30 min | roll back the previous image |
| Host loss (single node) | ≤ 24 h (or ≤ 5 min with WAL off-site) | ≤ 4 h | rebuild host, restore DB + objects |
| Object storage loss | ≤ 24 h | ≤ 4 h | restore bucket from replication/versioning |
| Regional loss | ≤ 24 h | ≤ 8 h | restore off-site copy on new infrastructure |

Targets are confirmed by the Phase 11 restore drill (`TASK-112`) and become binding
once proven; until then they are documented targets, not measured guarantees.

## 4. Restore Procedure (database)

```text
1  Announce: notify the team; declare the incident (runbook.md)
2  Freeze writes: stop api/worker, or put the platform in read-only mode
3  Identify the target point in time (last known good before the event)
4  Restore the base backup into a clean database
5  Replay WAL to the target time (PITR)
6  Verify: row counts for critical tables, audit continuity, PostGIS objects valid
7  Re-run pending migrations if the target precedes the current schema version
8  Start api/worker; confirm /health and /ready
9  Smoke test: login, read a published page, run one job
10 Reconcile: replay audit-driven actions performed after the restore point
11 Record: timeline, data loss window (actual RPO), duration (actual RTO), follow-ups
```

Rules: **never** restore over the live database — restore to a new instance, verify,
then switch; keep the pre-incident database intact until the incident is closed.

## 5. Restore Procedure (objects)

```text
1  Verify which bucket/prefix is affected
2  Restore the affected keys from versioning/replication (do not blanket-overwrite)
3  Reconcile metadata rows: any row whose object is missing is flagged as inconsistent
4  Regenerate derivatives (renditions) via the `renditions` queue — never hand-copy
5  Verify a sample: signed-URL access for a restricted and a public object
6  Record the outcome and any orphaned metadata
```

## 6. Restore Drill (mandatory)

```text
Frequency     monthly (isolated environment) and once before each release
Evidence      timestamps, restore target, actual RPO/RTO achieved, verifier name
Success       /ready healthy · critical row counts match · audit chain intact ·
              a public page renders · one job completes
Failure       drill is a blocking finding; fix before the next release
```

A backup that has never been restored is treated as **unverified**, not as a
backup.

## 7. Verification Queries (examples)

```sql
-- audit continuity: no gaps in the expected range
SELECT min(created_at), max(created_at), count(*) FROM audit.audit_log
 WHERE created_at >= :since;

-- tenant scoping sanity: no row without a tenant outside the global allow-list
SELECT count(*) FROM <tenant_table> WHERE tenant_id IS NULL;

-- orphaned objects vs metadata (run against the app, not raw storage)
SELECT count(*) FROM media_asset WHERE storage_key IS NULL OR status = 'quarantined';
```

## 8. Responsibilities and Rules

```text
Owner            DevOps owns the schedule and the drill; DB Architect owns restore
                 correctness for schema and data integrity
Access           restore credentials are restricted and audited; no shared accounts
Secrets          backup encryption keys are stored separately from backups
Retention        enforced by policy; deletion of backups is logged
Compliance       retention windows may not be shortened without a documented decision
```

## Related

- Root architecture: `../../ARCHITECTURE.md` §13
- Monitoring and alerts: `monitoring.md` · Deployment/rollback: `deployment.md`
- Incident procedures: `../12-operations/runbook.md` · Migration rules: `../04-data/migrations.md`

*End of docs/10-devops/backup-recovery.md*