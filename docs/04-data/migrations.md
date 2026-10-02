# docs/04-data/migrations.md

> **Status:** Current (policy) — no migrations exist yet | **Owner:** DB Architect
> **Last Updated:** 2026-10-02 | **Source:** `AGENTS.md` §9, ADR-003, ADR-004
> **Tooling:** TypeORM migrations (`ADR-0003`), run as a one-shot step before rollout.

## 1. Policy (binding)

```text
Append-only        a migration that has been applied is never edited
Reversible         every migration ships a tested down path (or an ADR explaining why not)
Pre-rollout        migrations run as a separate one-shot job before application rollout
Tenant-first       new tenant-owned tables include tenant_id NOT NULL + leading index
Audited            schema changes update docs/04-data/* in the same change
Backed up          destructive/irreversible changes require a backup and an ADR
```

A migration is **never** the place for business data. Seed data is limited to
platform reference data (permission catalog, master data, enum rows) — never
fabricated business records (`ADR-013`).

## 2. Naming and Location

```text
migrations/
  0001-init-tenancy-and-audit
  0002-init-identity
  0003-init-access
  ...
NNNN-<verb>-<subject>
```

- Four-digit, strictly increasing, never reused, never renumbered.
- `verb` is a single lowercase word (`create`, `add`, `alter`, `backfill`, `drop`).
- Each migration exports `up()` and `down()` and contains no environment-conditional
  logic (dev/test/staging/prod run identical DDL).

## 3. Planned Migration Sequence (target, not yet created)

| # | Migration | Contents | Phase |
|---|-----------|----------|-------|
| 0001 | `init-tenancy-and-audit` | `tenant`, `tenant_settings`, `tenant_domain`, `audit.audit_log`; extensions `postgis`, `pgcrypto`, `pg_trgm`, `unaccent`; RLS helper function | 0 |
| 0002 | `init-identity` | `user`, `user_profile`, `user_tenant_membership`, `session`, `refresh_token`, `invitation`, `password_reset_token`, `login_event` | 0–1 |
| 0003 | `init-access` | `permission` (catalog seed), `role`, `role_permission`, `group`, `group_member`, `user_role`, `api_key` | 1 |
| 0004 | `init-organization` | `organization`, `organization_unit`, `position`, `contact_point` | 1 |
| 0005 | `init-cms` | `content`, `content_version`, `content_revision`, `content_lock`, `category`, `tag`, joins, `navigation_menu`, `menu_item`, `redirect` | 2 |
| 0006 | `init-media-documents` | `media_asset`, `rendition`, `folder`, `collection`, `asset_usage`, `document`, `document_version`, `document_access`, `retention_policy`, `classification_label` | 2 |
| 0007 | `init-search` | `search_document` + `tsvector` column, GIN indexes, refresh function | 2 |
| 0008 | `init-programs-projects` | `program`, `project`, `project_location`, `project_team`, `project_partner`, `milestone`, `activity`, `workplan` | 3 |
| 0009 | `init-gis` | `geo.location`, `geo.boundary`, `geo.map_layer`, `geo.project_map`, `geo.geofence` + GiST indexes | 3 |
| 0010 | `init-platform-services` | `import_job`, `import_row`, `export_job`, `report_template`, `report_run`, `report_snapshot`, `dashboard`, `widget`, `saved_view`, `webhook_*`, `notification*`, `setting`, `integration_config` | 3 |
| 0011 | `init-funding-relationships` | `grant*`, `donor*`, `partner*`, `opportunity`, `proposal*`, `award`, CRM tables | 4 |
| 0012 | `init-meal-forms` | `indicator*`, `log_frame*`, `target`, `baseline`, `assessment`, `evaluation*`, `form*` | 5 |
| 0013 | `init-sensitive-domains` | `beneficiary`, `household*`, `case*`, `safeguarding*`, `complaint*`, `referral`, `assistance`, `enrollment`, `membership*` | 6 |
| 0014 | `init-workflow-automation` | `workflow_*`, `automation_*`, `mfa_factor` | 7 |
| 0015 | `init-ops-assets` | `asset*`, `inventory*`, `vehicle`, `trip`, `itinerary`, `per_diem`, `travel_report`, `procurement*`, `vendor`, `budget`, `cost_center`, `exchange_rate` | 3/6 |
| 0016 | `init-ai` | `ai_provider`, `ai_model`, `ai_prompt_version`, `ai_request`, `ai_response`, `ai_job`, `ai_review` | 10 |

Numbers are illustrative until Phase 0 creates the first migration; the **order**
is binding because later tables reference earlier ones.

## 4. RLS and Tenant Conventions in Migrations

Every tenant-owned table created by a migration follows this order:

```text
1  CREATE TABLE ... (tenant_id uuid NOT NULL REFERENCES tenant(id), ...)
2  CREATE INDEX ... ON <table> (tenant_id, ...)          -- tenant_id leads
3  ALTER TABLE <table> ENABLE ROW LEVEL SECURITY
4  CREATE POLICY tenant_isolation ON <table>
     USING (tenant_id = current_setting('app.tenant_id')::uuid)
5  GRANT SELECT, INSERT, UPDATE, DELETE ON <table> TO app_role
   -- audit tables receive INSERT/SELECT only
```

The `app.tenant_id` session variable is set per transaction by the tenant-scoped
repository base (`docs/07-backend/architecture.md`). RLS is defence-in-depth; the
application predicate remains mandatory (`ADR-0004`).

## 5. Rollback Expectations

| Change type | Down path |
|-------------|-----------|
| `create table` | `drop table` |
| `add column` (nullable) | `drop column` |
| `add column NOT NULL` | backfill → set default → add constraint; down drops the column |
| `add index` | `drop index` |
| enum extension | not reversible in place — requires a new type + column swap, documented in the header |
| `drop column` / `drop table` | **irreversible** — requires a backup, an ADR, and a `down()` that fails loudly |

Every migration header states: purpose, forward effect, rollback path, and whether
a backup is required.

## 6. Validation Before Merge

```text
[ ] `pnpm --filter api migration:run` succeeds on a clean database
[ ] `pnpm --filter api migration:revert` returns the database to the prior state
[ ] integration tests pass against the migrated database
[ ] tenant-leading indexes verified (`EXPLAIN` shows the tenant predicate used)
[ ] no business/demo data inserted by the migration
[ ] docs/04-data/schema.md updated when a table or column changes
```

These commands become runnable once Phase 0 (`TASK-006`) is implemented.

## 7. Forbidden

```text
X  editing, renaming, or deleting an applied migration
X  inserting business records as part of a migration
X  destructive change without backup + ADR + tested restore
X  environment-conditional DDL
X  creating a tenant-owned table without tenant_id and a leading index
X  granting UPDATE/DELETE on audit tables
```

## Related

- Storage strategy + conventions: `database.md`
- Table catalogue: `schema.md`
- Decision: `../13-decisions/ADR-0003-orm-migrations.md`
- Rules: `../../AGENTS.md` §9

*End of docs/04-data/migrations.md*