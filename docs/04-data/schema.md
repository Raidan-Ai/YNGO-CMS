# docs/04-data/schema.md

> **Status:** Current (catalogue) — implementation `PLANNED` | **Owner:** DB Architect
> **Last Updated:** 2026-10-02 | **Source:** `Engineering_Documentation_Package_v2.md`,
> ADR-004 | **Depends on:** `database.md`, `../02-domain/domain-model.md`

This file is the **table catalogue**: which tables exist, in which context, and the
columns that matter. It is not the machine schema — the schema is created by
migrations (`migrations.md`) and the authoritative column list is the generated
migration set once Phase 0 exists.

## 1. Column Contract

Every tenant-owned table (from `../02-domain/domain-model.md`):

```text
id          uuid          PK default gen_random_uuid()
tenant_id   uuid          NOT NULL FK -> tenant(id)     -- omitted only on global tables
created_at  timestamptz   NOT NULL default now()
updated_at  timestamptz   NOT NULL default now()
created_by  uuid          NULL FK -> user(id)
updated_by  uuid          NULL FK -> user(id)
status      text|enum     only where a lifecycle exists
deleted_at  timestamptz   only where soft delete is a genuine business rule
```

Rules:

1. Enum-like columns use a Postgres enum or a `CHECK` constraint — never free text.
2. Money is `numeric(18,2)` with a sibling `currency_code` — never `float`.
3. Dates are `date`; instants are `timestamptz` (stored UTC).
4. `jsonb` only for schema-variable data (custom fields, form schema, workflow
   conditions, capture snapshots) and always validated on write.
5. Every FK is indexed with `tenant_id` as the leading column.

## 2. Platform-Global Tables (no `tenant_id` — intentional)

| Table | Purpose | Notes |
|-------|---------|-------|
| `tenant` | Tenant registry (the isolation root) | `slug`, `status`, `region`, `created_at` |
| `tenant_settings` | Per-tenant configuration values | keyed `(tenant_id, key)` — the only listed table that *does* carry `tenant_id` |
| `tenant_domain` | Verified hostnames per tenant | used when subdomain resolution is enabled (ADR-005) |
| `permission` | Permission catalog (`resource:action`) | platform reference data |
| `master_data_entry` | Reference lists owned by the platform | governorates, districts, sectors |
| `currency`, `country` | Reference data | read-only |
| `integration_provider` | Adapter catalog | configuration is per tenant |
| `ai_provider`, `ai_model`, `ai_prompt_version` | AI registry | optional; Phase 10 |

Any omission of `tenant_id` outside this table is a defect; this list is the
authoritative allow-list and mirrors `database.md`.

## 3. Tables by Bounded Context

Column lists below show the **domain-defining** columns only; the universal
contract in §1 is implicit on every tenant-owned table.

### 3.1 Tenancy, Identity, Access

| Table | Key columns | Rules |
|-------|-------------|-------|
| `user` | `email` (unique per tenant), `password_hash`, `status`, `mfa_enabled` | Argon2id only; never a default password |
| `user_profile` | `user_id`, `full_name_ar`, `full_name_en`, `job_title`, `locale` | locale default `ar` |
| `user_tenant_membership` | `user_id`, `tenant_id`, `status`, `joined_at` | a user reaches a tenant only through a membership |
| `session` | `user_id`, `token_hash`, `expires_at`, `revoked_at`, `ip`, `user_agent` | tokens stored hashed |
| `refresh_token` | `session_id`, `token_hash`, `rotated_from`, `expires_at` | rotation invalidates the predecessor (BR-011) |
| `invitation` | `email`, `role_id`, `token_hash`, `expires_at`, `accepted_at` | single-use, expiring (BR-012) |
| `password_reset_token` | `user_id`, `token_hash`, `expires_at`, `used_at` | single-use |
| `mfa_factor` | `user_id`, `type`, `secret_encrypted`, `confirmed_at` | Phase 7 |
| `login_event` | `user_id`, `result`, `ip`, `user_agent`, `created_at` | security audit input |
| `role` | `code`, `name_ar`, `name_en`, `is_system` | per tenant; system roles not editable |
| `role_permission` | `role_id`, `permission_id` | join |
| `group`, `group_member` | `group_id`, `user_id` | ABAC inputs |
| `user_role` | `user_id`, `role_id`, `scope_type`, `scope_id` | optional attribute scope on assignment |
| `api_key` | `name`, `key_hash`, `prefix`, `scopes`, `expires_at`, `revoked_at`, `last_used_at` | hash at rest; raw secret shown once (BR-006) |

### 3.2 Organization

| Table | Key columns |
|-------|-------------|
| `organization` | `legal_name_ar`, `legal_name_en`, `registration_no`, `logo_media_id`, `default_locale` |
| `organization_unit` | `parent_id`, `type` (`branch`/`department`/`office`/`team`), `path` |
| `position` | `title_ar`, `title_en`, `unit_id` |
| `contact_point` | `owner_type`, `owner_id`, `type` (`email`/`phone`/`address`), `value`, `is_primary` |

`organization_unit.path` (materialised path or closure table) exists so ABAC
subtree scoping is a single indexed predicate.

### 3.3 Governance

| Table | Key columns |
|-------|-------------|
| `board`, `board_member` | `board_id`, `user_id`, `role`, `term_start`, `term_end` |
| `committee`, `committee_member` | `committee_id`, `user_id` |
| `board_meeting` | `board_id`, `scheduled_at`, `minutes_document_id`, `status` |
| `board_resolution` | `meeting_id`, `number`, `text`, `status`, `decided_at` |
| `policy`, `bylaw` | `code`, `version`, `effective_from`, `classification` |
| `decision_record` | `subject_type`, `subject_id`, `decided_by`, `decided_at`, `rationale` |
| `delegation_of_authority` | `from_user_id`, `to_user_id`, `scope`, `valid_from`, `valid_to` |
| `conflict_of_interest_declaration` | `user_id`, `subject_type`, `subject_id`, `declared_at` |

### 3.4 CMS, Media, Documents

| Table | Key columns | Rules |
|-------|-------------|-------|
| `content` | `type`, `slug` (unique per tenant+type), `locale`, `status`, `current_version_id`, `published_at`, `scheduled_at` | `status` changes only via workflow (BR-021) |
| `content_version` | `content_id`, `version_no`, `title`, `summary`, `body`, `blocks` (jsonb), `status`, `approved_by` | one version is immutable once approved |
| `content_revision` | `content_id`, `version_id`, `field`, `old_value`, `new_value`, `actor_id` | change history (BR-025) |
| `content_lock` | `content_id`, `locked_by`, `expires_at` | prevents concurrent edits (BR-026) |
| `category`, `tag`, `content_category`, `content_tag` | `slug`, `name_ar`, `name_en` | taxonomy per tenant |
| `navigation_menu`, `menu_item` | `menu_id`, `parent_id`, `target_type`, `target_id` | `target_type` drives rendering |
| `redirect` | `from_path`, `to_path`, `status_code` | 301/302 |
| `media_asset` | `storage_key`, `mime_type`, `size_bytes`, `checksum`, `width`, `height`, `folder_id`, `status` | object key is normalised, never user-supplied (R-07) |
| `rendition` | `media_asset_id`, `preset`, `storage_key`, `format`, `width`, `height` | generated by the worker |
| `folder`, `collection`, `media_tag`, `asset_usage` | hierarchy; usage links an asset to its owning record |
| `document` | `title`, `category_id`, `classification_label_id`, `retention_policy_id`, `current_version_id` | classification drives search/export eligibility (BR-024) |
| `document_version` | `document_id`, `version_no`, `storage_key`, `checksum`, `approved_by` | |
| `document_access` | `document_id`, `subject_type`, `subject_id`, `level` | explicit grants |
| `retention_policy`, `classification_label` | `code`, `name_ar`, `name_en`, `retention_months`, `rank` | per-tenant reference data |

### 3.5 Programs, Projects, MEAL, GIS

| Table | Key columns | Rules |
|-------|-------------|-------|
| `program` | `code`, `name_ar`, `name_en`, `sector_id`, `start_date`, `end_date`, `status` | |
| `project` | `program_id`, `organization_unit_id`, `code`, `name_ar`, `name_en`, `status`, `budget_amount`, `currency_code` | exactly one program + one unit in the same tenant (BR-030) |
| `project_location` | `project_id`, `location_id`, `precision`, `is_public` | `is_public=false` excludes it from public layers (BR-034) |
| `project_team`, `project_partner` | `project_id`, `user_id` / `partner_id`, `role` | |
| `milestone` | `project_id`, `title`, `due_date`, `completed_at`, `status`, `is_mandatory` | open mandatory milestones block completion (BR-033) |
| `activity` | `project_id`, `milestone_id`, `title`, `planned_start`, `planned_end`, `status` | |
| `workplan` | `project_id`, `period_id`, `rows` (jsonb, validated) | |
| `indicator` | `project_id`, `code`, `name_ar`, `name_en`, `unit`, `direction`, `baseline_value`, `target_value` | |
| `reporting_period` | `project_id`, `code`, `start_date`, `end_date`, `is_closed` | closed periods reject new values |
| `indicator_value` | `indicator_id`, `period_id`, `value`, `disaggregation` (jsonb), `verified_by` | same project as the indicator (BR-031); edits keep history (BR-032) |
| `log_frame`, `log_frame_row`, `result_chain`, `target`, `baseline` | `project_id` (+ `period_id` on targets) | |
| `assessment`, `evaluation`, `evaluation_finding` | `project_id`, `type`, `conducted_at`, `findings` | |
| `location` | `name`, `admin_level`, `geom` (geography Point), `precision`, `is_sensitive` | `geo` schema; GiST indexed |
| `boundary` | `name`, `admin_level`, `geom` (geography MultiPolygon) | |
| `map_layer` | `code`, `source`, `visibility`, `privacy_rule` | visibility enforced server-side |
| `project_map`, `geofence` | `project_id`, `layer_id`, `geom` | |

### 3.6 Funding, Relationships, Forms

| Table | Key columns | Rules |
|-------|-------------|-------|
| `grant` | `donor_id`, `funding_source_id`, `code`, `amount`, `currency_code`, `status`, `start_date`, `end_date` | must reference a donor or funding source (BR-041) |
| `grant_milestone` | `grant_id`, `title`, `due_date`, `completed_at`, `required` | |
| `grant_report` | `grant_id`, `period`, `status`, `submitted_at`, `approved_by` | blocked while required milestones are open (BR-040) |
| `grant_amendment` | `grant_id`, `version_no`, `amount_delta`, `reason`, `approved_by` | awarded-amount history preserved (BR-042) |
| `opportunity`, `proposal`, `proposal_version`, `award`, `funding_source` | opportunity → proposal → award chain | |
| `donor`, `donor_contact` | `type`, `name`, `country_code`, `preferred_locale` | confidential |
| `partner`, `partner_agreement` | `type`, `agreement_document_id`, `valid_from`, `valid_to` | |
| `stakeholder`, `contact`, `interaction`, `meeting`, `call`, `crm_task`, `communication_log` | polymorphic `subject_type`/`subject_id` | interactions append-only (BR-043) |
| `form` | `code`, `status`, `current_version_id`, `purpose` | |
| `form_version` | `form_id`, `version_no`, `schema` (jsonb, validated), `published_at` | immutable once published |
| `form_field` | `form_version_id`, `key`, `type`, `required`, `conditional_logic`, `pii_class` | `pii_class` drives field policy |
| `form_submission` | `form_version_id`, `submitted_by`, `submitted_at`, `values` (jsonb), `status`, `location_id` | keeps its form version (BR-070) |
| `form_review`, `form_submission_file` | `submission_id`, `reviewer_id`, `decision` / `media_asset_id` | |

### 3.7 People (privacy-critical)

| Table | Key columns | Rules |
|-------|-------------|-------|
| `beneficiary` | `reference_code`, `full_name`, `national_id_encrypted`, `sex`, `year_of_birth`, `household_id`, `consent_recorded_at` | `sensitive`; direct identifiers encrypted; pseudonymous `reference_code` in lists |
| `household` | `reference_code`, `head_beneficiary_id`, `location_id`, `size` | `sensitive` |
| `household_member`, `enrollment`, `assistance`, `referral` | `household_id` / `beneficiary_id` links | `restricted` |
| `case` | `beneficiary_id`, `case_worker_id`, `status`, `opened_at`, `closed_at`, `priority` | `restricted`; never returned by an unscoped query (BR-060) |
| `case_note`, `case_assessment`, `follow_up` | `case_id`, `author_id`, `body` / `outcome` | access audited with a captured reason (BR-061) |
| `safeguarding_case` | `reference_code`, `status`, `reported_at`, `assigned_investigator_id`, `severity` | own audit stream (BR-063); excluded from search/export (BR-062) |
| `safeguarding_incident`, `safeguarding_action`, `investigator` | `safeguarding_case_id`, ... | `sensitive` |
| `complaint` | `channel`, `is_anonymous`, `category`, `status`, `submitted_at` | anonymity preserved in audit |
| `complaint_message`, `complaint_resolution` | `complaint_id`, `direction`, `body` / `outcome` | |
| `staff`, `staff_contract`, `leave_request`, `training_record` | `user_id` / `staff_id` links | `confidential` |
| `volunteer`, `skill`, `availability`, `volunteer_assignment`, `attendance`, `volunteer_hours`, `certificate` | `volunteer_id` links | |
| `membership`, `membership_due` | `member_id`, `status`, `valid_to`, `amount`, `paid_at` | |

### 3.8 Assets, Procurement, Finance, Fleet, Travel

| Table | Key columns | Rules |
|-------|-------------|-------|
| `asset` | `code`, `category`, `serial_no`, `acquired_at`, `value`, `status`, `location_id` | |
| `asset_assignment`, `maintenance_record` | `asset_id`, `user_id` / `vendor_id`, dates | |
| `inventory_item`, `stock_movement` | `item_id`, `warehouse_id`, `quantity`, `direction`, `reference` | movements append-only |
| `vehicle`, `trip`, `itinerary`, `per_diem`, `travel_report` | `vehicle_id` / `trip_id` links | |
| `procurement_request` | `requested_by`, `status`, `justification`, `budget_line_id` | |
| `rfq`, `quotation`, `procurement_evaluation`, `purchase_order`, `delivery`, `vendor` | RFQ → quotation → award chain | PO requires an awarded quotation (BR-050); minimum quotations enforced (BR-051) |
| `budget`, `cost_center`, `expense_import`, `exchange_rate` | `project_id`, `cost_center_id`, `amount`, `currency_code` | budgeting/utilisation only — no GL (BR-053) |

### 3.9 Workflow, Automation, Notifications, Platform Services

| Table | Key columns | Rules |
|-------|-------------|-------|
| `workflow_definition`, `workflow_version`, `workflow_step`, `workflow_transition`, `workflow_condition` | `definition_id`, `version_no`, `from_state`, `to_state`, `guard`, `permission` | versions immutable; in-flight instances keep theirs (BR-080) |
| `workflow_instance`, `workflow_task`, `approval`, `escalation` | `workflow_version_id` (pinned), `entity_type`, `entity_id`, `state`, `assignee_id`, `due_at` | state changes only via transitions (BR-021) |
| `automation_rule`, `automation_execution`, `automation_log` | `trigger`, `conditions` (jsonb), `actions` (jsonb) | runs with the configuring actor's permissions (BR-082) |
| `notification`, `notification_template`, `notification_preference`, `delivery_attempt`, `notification_channel_config` | `recipient_id`, `channel`, `template_key`, `attempt_no`, `result` | every attempt recorded (BR-083) |
| `search_document` | `entity_type`, `entity_id`, `title`, `body`, `tsv` (tsvector), `permission_scope` | **projection** — rebuildable, never authoritative |
| `import_job`, `import_row`, `export_job` | `entity_type`, `status`, `total_rows`, `failed_rows`, `error_report_key` | validate-then-commit (BR-071/072) |
| `report_template`, `report_run`, `report_snapshot` | `template_id`, `parameters` (jsonb), `output_key`, `status` | permission-checked at create **and** download (BR-073) |
| `dashboard`, `widget`, `saved_view` | `scope`, `layout` (jsonb), `query` (jsonb, validated) | |
| `webhook_endpoint`, `webhook_subscription`, `webhook_delivery`, `webhook_attempt`, `webhook_secret` | `event`, `url`, `secret_hash`, `attempt_no`, `response_status` | HMAC-signed, retried, logged (BR-084) |
| `integration_config` | `provider_id`, `credentials_encrypted`, `enabled` | credentials encrypted at rest |
| `setting` | `key`, `value` (jsonb), `scope` | tenant configuration |
| `data_quality_rule`, `data_quality_issue` | `entity_type`, `rule`, `severity`, `resolution` | issues surfaced, never silently corrected (BR-074) |
| `ai_request`, `ai_response`, `ai_job`, `ai_review` | `provider_id`, `model_id`, `prompt_version_id`, `actor_id`, `review_status` | a draft until human approval (BR-090) |

### 3.10 Audit (schema `audit`)

`audit.audit_log` is defined in `database.md` §"Audit Schema". It is append-only:
no `UPDATE`/`DELETE` grants are issued, and safeguarding writes go to a separate
stream readable only by designated roles (BR-063).

## 4. Tenancy Verification Checklist

```text
[ ] every tenant-owned table has tenant_id NOT NULL + FK + leading index
[ ] every FK has a matching (tenant_id, fk_id) index
[ ] no tenant-owned table is missing from §1's contract
[ ] the only tables without tenant_id appear in §2
[ ] every list path filters deleted_at IS NULL where soft delete applies
[ ] every unique business key is scoped by tenant_id
[ ] restricted/sensitive tables are excluded from unscoped queries and search projections
```

## Related

- Storage strategy + audit schema: `database.md`
- Migrations: `migrations.md`
- Entities and ownership: `../02-domain/entities.md`
- Rules: `../02-domain/business-rules.md`

*End of docs/04-data/schema.md*