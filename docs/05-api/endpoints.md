# docs/05-api/endpoints.md

> **Status:** Current (catalogue) — all endpoints `PLANNED` | **Owner:** API Owner
> **Last Updated:** 2026-10-02 | **Source:** `ARCHITECTURE.md` §8, `api.md`
> **Machine contract:** generated OpenAPI (`openapi.md`) — this file is the human
> catalogue of intent, not the contract.

## 1. Conventions Used Below

```text
Base              /api/v1
IDs               uuid in the path
Listing           GET /<resources>?limit&cursor&sort&<filters>
Detail            GET /<resources>/{id}
Create            POST /<resources>
Update            PATCH /<resources>/{id}          (partial; never changes status)
Delete            DELETE /<resources>/{id}         (soft/hard per entity rule)
Transitions       POST /<resources>/{id}/<action>  (status changes ONLY here)
Permissions       resource:action, enforced by PermissionsGuard (BR-003)
```

Rules that apply to every row below:

1. Every path is tenant-scoped; the tenant comes from context, never the body.
2. Every listing endpoint is cursor-paginated unless noted, and whitelists its
   filters and sort fields (`api.md`).
3. `status` is never accepted by `PATCH`; only transition endpoints change it
   (`WORKFLOW_TRANSITION_INVALID`).
4. Denial returns 403/404 per `errors.md`; every protected row requires an authz
   test plus a cross-tenant negative test (`AGENTS.md` §5).

## 2. Identity, Access, Organization

| Method | Path | Purpose | Permission | Phase |
|--------|------|---------|-----------|-------|
| POST | `/auth/login` | Password (+MFA) login, issues session/tokens | public | 0 |
| POST | `/auth/logout` | Revoke the current session | authenticated | 0 |
| POST | `/auth/refresh` | Rotate refresh token (BR-011) | refresh token | 0 |
| POST | `/auth/password/forgot` | Start reset (always returns 202) | public | 1 |
| POST | `/auth/password/reset` | Complete reset with a single-use token | public | 1 |
| POST | `/auth/invitations/accept` | Accept invitation, set password | invitation token | 1 |
| GET | `/auth/me` | Effective user, memberships, permissions | authenticated | 0 |
| GET | `/users` | List members of the tenant | `users:read` | 1 |
| POST | `/users/invitations` | Invite a user with a role | `users:invite` | 1 |
| GET | `/users/{id}` | User detail (field policy applies) | `users:read` | 1 |
| PATCH | `/users/{id}` | Update profile/roles | `users:update` | 1 |
| POST | `/users/{id}/suspend` · `/activate` | Status transitions | `users:manage` | 1 |
| DELETE | `/users/{id}` | Deactivate + revoke sessions | `users:manage` | 1 |
| GET/POST | `/roles` | List/create roles | `roles:read` / `roles:create` | 1 |
| PATCH/DELETE | `/roles/{id}` | Update/delete a non-system role | `roles:update` / `roles:delete` | 1 |
| PUT | `/roles/{id}/permissions` | Replace the permission set | `roles:update` | 1 |
| GET | `/permissions` | Permission catalog (platform reference) | `roles:read` | 1 |
| GET/POST/DELETE | `/groups` · `/groups/{id}/members` | Groups for ABAC and bulk assignment | `groups:*` | 1 |
| GET/POST | `/api-keys` | List/create API keys (secret shown once) | `api_keys:read` / `api_keys:create` | 1 |
| POST | `/api-keys/{id}/revoke` | Immediate revocation (BR-006) | `api_keys:manage` | 1 |
| GET/PATCH | `/organization` | Profile, branding, default locale | `organization:read` / `organization:update` | 1 |
| GET/POST | `/organization/units` | Org tree (branches/departments/offices/teams) | `organization:read` / `organization:manage` | 1 |
| PATCH/DELETE | `/organization/units/{id}` | Move/rename/remove a unit | `organization:manage` | 1 |
| GET/POST | `/organization/positions` | Positions | `organization:read` / `organization:manage` | 1 |

## 3. Audit, Settings, Master Data

| Method | Path | Purpose | Permission | Phase |
|--------|------|---------|-----------|-------|
| GET | `/audit` | Append-only audit search (actor, entity, action, date) | `audit:read` | 1 |
| GET | `/audit/{id}` | Single audit record with before/after (redacted) | `audit:read` | 1 |
| POST | `/audit/export` | Asynchronous audit export (audited, rate-limited) | `audit:export` | 8 |
| GET/PATCH | `/settings` | Tenant settings by scope | `settings:read` / `settings:update` | 1 |
| GET | `/master-data/{registry}` | Reference lists (governorates, districts, sectors) | authenticated | 1 |
| POST | `/master-data/{registry}` | Extend a tenant-owned registry | `settings:update` | 1 |

## 4. CMS, Media, Documents, Search

| Method | Path | Purpose | Permission | Phase |
|--------|------|---------|-----------|-------|
| GET | `/content` | List/filter by type, status, category, locale, author | `content:read` | 2 |
| POST | `/content` | Create draft | `content:create` | 2 |
| GET/PATCH | `/content/{id}` | Read/update the working version | `content:read` / `content:update` | 2 |
| DELETE | `/content/{id}` | Archive-or-delete per rule | `content:delete` | 2 |
| GET | `/content/{id}/versions` | Version history | `content:read` | 2 |
| POST | `/content/{id}/versions/{v}/restore` | New version from an older one | `content:update` | 2 |
| POST | `/content/{id}/submit` · `/request-changes` · `/approve` · `/reject` | Editorial transitions | `content:update` / `content:approve` | 2 |
| POST | `/content/{id}/publish` · `/unpublish` · `/schedule` | Publication transitions | `content:publish` | 2 |
| POST | `/content/{id}/lock` · `/unlock` | Editor locking (BR-026) | `content:update` | 2 |
| GET | `/content/{id}/revisions` | Field-level change history | `content:read` | 2 |
| GET/POST/PATCH/DELETE | `/categories` · `/tags` | Taxonomy | `content:read` / `content:manage-taxonomy` | 2 |
| GET/POST/PATCH/DELETE | `/navigation` · `/navigation/{id}/items` | Menus | `content:read` / `content:manage-navigation` | 2 |
| GET/POST/PATCH/DELETE | `/redirects` | Redirect rules | `content:manage-navigation` | 2 |
| GET/POST | `/media` | List/upload assets (multipart; validated + quarantined) | `media:read` / `media:upload` | 2 |
| GET | `/media/{id}` | Asset metadata + renditions | `media:read` | 2 |
| GET | `/media/{id}/signed-url` | Time-limited download URL (BR-023) | `media:read` | 2 |
| PATCH/DELETE | `/media/{id}` | Update metadata / delete object + record | `media:update` / `media:delete` | 2 |
| GET/POST | `/media/folders` · `/media/collections` | DAM organisation | `media:*` | 2 |
| GET/POST | `/documents` | List/register documents | `documents:read` / `documents:create` | 2 |
| GET/PATCH/DELETE | `/documents/{id}` | Detail / metadata / remove | `documents:read` / `documents:update` / `documents:delete` | 2 |
| POST | `/documents/{id}/versions` | Add a version | `documents:update` | 2 |
| GET | `/documents/{id}/signed-url` | Controlled download (audited) | `documents:read` | 2 |
| PUT | `/documents/{id}/access` | Explicit access grants | `documents:manage-access` | 2 |
| GET/POST | `/document-categories` · `/retention-policies` · `/classification-labels` | DMS reference data | `documents:read` / `documents:manage` | 2 |
| GET | `/search` | Permission-filtered tenant search (facets, Arabic) | `search:read` | 2 |
| POST | `/search/reindex` | Rebuild projections for an entity type | `search:reindex` | 2 |

## 5. Programs, Projects, MEAL, GIS

| Method | Path | Purpose | Permission | Phase |
|--------|------|---------|-----------|-------|
| GET/POST | `/programs` | List/create programs | `programs:read` / `programs:create` | 3 |
| GET/PATCH/DELETE | `/programs/{id}` | Program detail/update/remove | `programs:read` / `programs:update` / `programs:delete` | 3 |
| GET/POST | `/projects` | List/create projects (filters: program, status, unit, donor) | `projects:read` / `projects:create` | 3 |
| GET/PATCH/DELETE | `/projects/{id}` | Project detail/update/remove | `projects:read` / `projects:update` / `projects:delete` | 3 |
| GET/POST/DELETE | `/projects/{id}/locations` | Project locations (privacy-flagged) | `projects:read` / `projects:update` | 3 |
| GET/POST/DELETE | `/projects/{id}/team` · `/partners` | Staffing and partners | `projects:update` | 3 |
| GET/POST | `/projects/{id}/milestones` | Milestones | `projects:read` / `projects:update` | 3 |
| POST | `/projects/{id}/complete` · `/suspend` · `/close` | Lifecycle transitions | `projects:approve` | 3 |
| GET/POST | `/projects/{id}/activities` | Activities | `activities:read` / `activities:update` | 3 |
| GET/POST | `/projects/{id}/workplan` | Workplan rows per period | `projects:read` / `projects:update` | 3 |
| GET/POST | `/indicators` | Indicators (per project) | `indicators:read` / `indicators:create` | 5 |
| PATCH/DELETE | `/indicators/{id}` | Update/remove an indicator | `indicators:update` / `indicators:delete` | 5 |
| GET/POST | `/indicators/{id}/values` | Values per reporting period (auditable) | `indicators:read` / `indicators:record` | 5 |
| POST | `/indicator-values/{id}/verify` | Verification transition | `indicators:verify` | 5 |
| GET/POST | `/reporting-periods` | Periods (closing blocks new values) | `indicators:read` / `indicators:manage` | 5 |
| GET/POST | `/logframes` · `/logframes/{id}/rows` | Logframe structure | `indicators:read` / `indicators:manage` | 5 |
| GET/POST | `/baselines` · `/targets` · `/assessments` · `/evaluations` | MEAL records | `meal:*` | 5 |
| GET | `/gis/locations` | Location lookup (precision per role) | `gis:read` | 3 |
| GET | `/gis/layers/{code}` | Map layer GeoJSON (privacy-filtered, BR-034) | `gis:read` or public for public layers | 3 |
| POST | `/gis/layers` | Publish a map layer | `gis:manage` | 6 |
| GET/POST | `/gis/boundaries` | Administrative boundaries | `gis:read` / `gis:manage` | 6 |

## 6. Funding, Donors, Partners, CRM

| Method | Path | Purpose | Permission | Phase |
|--------|------|---------|-----------|-------|
| GET/POST | `/grants` | List/create grants | `grants:read` / `grants:create` | 4 |
| GET/PATCH/DELETE | `/grants/{id}` | Grant detail/update/remove | `grants:read` / `grants:update` / `grants:delete` | 4 |
| POST | `/grants/{id}/award` · `/activate` · `/close` | Grant transitions | `grants:approve` | 4 |
| GET/POST | `/grants/{id}/milestones` | Grant milestones | `grants:read` / `grants:update` | 4 |
| GET/POST | `/grants/{id}/reports` | Donor reports (blocked while milestones open, BR-040) | `grants:read` / `grants:report` | 4 |
| POST | `/grant-reports/{id}/submit` · `/approve` | Report transitions | `grants:report` / `grants:approve` | 4 |
| GET/POST | `/grants/{id}/amendments` | Amendments (versioned, BR-042) | `grants:update` | 4 |
| GET/POST | `/opportunities` · `/proposals` | Funding pipeline | `grants:read` / `grants:create` | 4 |
| GET/POST | `/donors` | Donor records | `donors:read` / `donors:create` | 4 |
| GET/PATCH/DELETE | `/donors/{id}` (+ `/contacts`) | Donor detail and contacts | `donors:*` | 4 |
| GET/POST | `/partners` | Partner organisations | `partners:read` / `partners:create` | 4 |
| GET/POST | `/partners/{id}/agreements` | Agreements/MoU with documents | `partners:read` / `partners:update` | 4 |
| GET/POST | `/stakeholders` · `/contacts` | CRM entities | `crm:read` / `crm:create` | 4 |
| GET/POST | `/interactions` · `/meetings` · `/calls` · `/crm-tasks` | Relationship activity (append-only, BR-043) | `crm:read` / `crm:write` | 4 |
| GET/POST | `/events` | Events with registration tracking | `events:read` / `events:create` | 4 |

## 7. Sensitive Domains (People, Cases, Safeguarding, Complaints)

| Method | Path | Purpose | Permission | Phase |
|--------|------|---------|-----------|-------|
| GET/POST | `/beneficiaries` | Pseudonymous list; full detail requires field grant | `beneficiaries:read` / `beneficiaries:create` | 6 |
| GET/PATCH/DELETE | `/beneficiaries/{id}` | Detail (field-level policy, access audited) | `beneficiaries:read` / `beneficiaries:update` | 6 |
| GET/POST | `/households` · `/households/{id}/members` | Household units | `beneficiaries:read` / `beneficiaries:update` | 6 |
| POST | `/beneficiaries/{id}/assistance` · `/enrollments` · `/referrals` | Service records | `beneficiaries:update` | 6 |
| POST | `/beneficiaries/export` | Sensitive export (permission + reason + audit, BR-064) | `beneficiaries:export` | 6 |
| GET/POST | `/cases` | Cases (never in unscoped queries, BR-060) | `cases:read` / `cases:create` | 6 |
| GET/PATCH | `/cases/{id}` | Case detail/update (reason required, BR-061) | `cases:read` / `cases:update` | 6 |
| GET/POST | `/cases/{id}/notes` · `/assessments` · `/follow-ups` | Case records | `cases:read` / `cases:update` | 6 |
| POST | `/cases/{id}/refer` · `/resolve` · `/close` | Case transitions | `cases:update` | 6 |
| GET/POST | `/safeguarding/cases` | Safeguarding register (separate audit stream) | `safeguarding:read` / `safeguarding:create` | 6 |
| GET/PATCH | `/safeguarding/cases/{id}` | Detail/update with justification | `safeguarding:read` / `safeguarding:update` | 6 |
| POST | `/safeguarding/cases/{id}/triage` · `/assign` · `/action` · `/close` | Safeguarding transitions | `safeguarding:manage` | 6 |
| POST | `/safeguarding/cases/{id}/export` | Controlled export (elevated + audited, BR-062/064) | `safeguarding:export` | 6 |
| POST | `/complaints` | Public intake (rate-limited, anonymous allowed) | public | 6 |
| GET | `/complaints` | Internal queue (never includes anonymous identity) | `complaints:read` | 6 |
| POST | `/complaints/{id}/triage` · `/assign` · `/respond` · `/resolve` | Complaint transitions | `complaints:manage` | 6 |
| GET/POST | `/volunteers` · `/volunteers/{id}/assignments` · `/hours` | Volunteer management | `volunteers:*` | 6 |
| GET/POST | `/staff` · `/staff/{id}/contracts` · `/leave` · `/training` | HR-lite | `staff:*` | 6 |
| GET/POST | `/memberships` · `/memberships/{id}/dues` | Membership and dues | `memberships:*` | 6 |
| GET/POST | `/governance/boards` · `/meetings` · `/resolutions` · `/policies` | Governance records | `governance:*` | 6 |

## 8. Forms, Workflow, Automation, Notifications, Reporting

| Method | Path | Purpose | Permission | Phase |
|--------|------|---------|-----------|-------|
| GET/POST | `/forms` | List/create forms | `forms:read` / `forms:create` | 5 |
| GET/PATCH/DELETE | `/forms/{id}` | Form definition | `forms:read` / `forms:update` | 5 |
| GET/POST | `/forms/{id}/versions` | Versioned schema (immutable once published) | `forms:update` | 5 |
| POST | `/forms/{id}/publish` · `/close` · `/archive` | Form transitions | `forms:publish` | 5 |
| GET/POST | `/forms/{id}/submissions` | Submissions (keeps form version, BR-070) | `forms:read` / `forms:submit` | 5 |
| POST | `/form-submissions/{id}/review` | Review decision | `forms:review` | 5 |
| GET/POST | `/workflows` | Workflow definitions | `workflows:read` / `workflows:manage` | 7 |
| POST | `/workflows/{id}/versions` · `/publish` | Version and activate a definition | `workflows:manage` | 7 |
| GET | `/workflow-instances` | In-flight instances (own version pinned, BR-080) | `workflows:read` | 7 |
| POST | `/workflow-instances/{id}/transition` | The **only** way to change status (BR-021) | `workflows:execute` | 7 |
| GET | `/workflow-tasks` | My pending tasks | authenticated | 7 |
| POST | `/workflow-tasks/{id}/complete` | Complete a task | `workflows:execute` | 7 |
| POST | `/approvals/{id}/approve` · `/reject` · `/delegate` | Approval decisions (four-eyes aware) | `approvals:decide` | 7 |
| GET/POST | `/automations` | Automation rules | `automations:read` / `automations:manage` | 7 |
| GET | `/automations/{id}/executions` | Execution log | `automations:read` | 7 |
| GET | `/notifications` | My notifications | authenticated | 7 |
| POST | `/notifications/{id}/read` | Mark read | authenticated | 7 |
| GET/PATCH | `/notification-preferences` | Channel preferences | authenticated | 7 |
| GET/POST | `/notification-templates` | Templates | `notifications:manage` | 7 |
| GET/POST | `/reports/templates` | Report templates | `reports:read` / `reports:manage` | 3 |
| POST | `/reports/run` | Start an async report run | `reports:run` | 3 |
| GET | `/reports/runs` · `/reports/runs/{id}` | Run status and progress | `reports:read` | 3 |
| GET | `/reports/runs/{id}/download` | Permission-checked download (BR-073) | `reports:read` | 3 |
| POST | `/reports/runs/{id}/cancel` | Cancel a run | `reports:run` | 3 |
| GET/POST | `/dashboards` · `/dashboards/{id}/widgets` | Dashboards | `dashboards:read` / `dashboards:manage` | 8 |
| GET | `/analytics/{metric}` | KPI/analytics queries (permission-scoped) | `analytics:read` | 8 |
| GET/POST | `/saved-views` | Saved filters for list pages | authenticated | 3 |

## 9. Operations, Integrations, Developer, Platform

| Method | Path | Purpose | Permission | Phase |
|--------|------|---------|-----------|-------|
| GET/POST | `/assets` · `/assets/{id}/assignments` · `/maintenance` | Fixed assets | `assets:*` | 3 |
| GET/POST | `/inventory` · `/inventory/movements` | Stock items and movements | `inventory:*` | 3 |
| GET/POST | `/procurement/requests` · `/rfqs` · `/quotations` · `/evaluations` · `/purchase-orders` · `/deliveries` | Procurement chain (BR-050/051) | `procurement:*` | 3 |
| GET/POST | `/vendors` | Vendor registry | `vendors:*` | 3 |
| GET/POST | `/finance/budgets` · `/cost-centers` · `/expense-imports` · `/exchange-rates` | Finance layer (no GL, BR-053) | `finance:*` | 3 |
| GET/POST | `/fleet/vehicles` · `/trips` · `/travel/reports` | Fleet and travel | `fleet:*` / `travel:*` | 3 |
| GET/POST | `/tasks` · `/tasks/{id}/complete` | Task management | `tasks:*` | 3 |
| GET/POST | `/imports` | Duplicate-safe import (validate-then-commit, BR-071) | `imports:run` | 3 |
| GET | `/imports/{id}` · `/imports/{id}/errors` | Progress and per-row failures | `imports:read` | 3 |
| GET/POST | `/exports` | Async export jobs (audited, BR-073) | `exports:run` | 3 |
| GET | `/exports/{id}/download` | Signed, permission-checked download | `exports:read` | 3 |
| GET/POST | `/webhooks/endpoints` | Webhook endpoints | `webhooks:read` / `webhooks:manage` | 3 |
| GET/POST | `/webhooks/subscriptions` | Event subscriptions | `webhooks:manage` | 3 |
| GET | `/webhooks/deliveries` · `/webhooks/deliveries/{id}/attempts` | Delivery log and retries (BR-084) | `webhooks:read` | 3 |
| POST | `/webhooks/deliveries/{id}/redeliver` | Manual redelivery | `webhooks:manage` | 3 |
| GET/POST | `/integrations` | Adapter configuration (credentials encrypted) | `integrations:read` / `integrations:manage` | 9 |
| POST | `/integrations/{id}/test` | Connectivity test with a redacted result | `integrations:manage` | 9 |
| GET | `/data-quality/issues` | Data-quality review queue (BR-074) | `data_quality:read` | 5 |
| POST | `/data-quality/merges` | Controlled merge with explicit confirmation | `data_quality:manage` | 5 |
| GET | `/ai/providers` · `/ai/models` | AI registry (optional, disabled by default) | `ai:read` | 10 |
| POST | `/ai/requests` | Create a bounded AI request → draft output | `ai:use` | 10 |
| POST | `/ai/responses/{id}/review` | Human approval or rejection of a draft | `ai:review` | 10 |
| GET | `/health` | Liveness (no auth, no tenant context) | public | 0 |
| GET | `/ready` | Readiness: DB + Redis + object storage | public (restricted detail) | 0 |
| GET | `/api/docs` | Generated OpenAPI UI/JSON | per environment policy | 0 |

## 10. Public (Unauthenticated) Surface

| Method | Path | Purpose | Notes |
|--------|------|---------|-------|
| GET | `/public/content/{type}` · `/public/content/{type}/{slug}` | Published content only | cacheable; never exposes drafts |
| GET | `/public/projects` · `/public/projects/{slug}` | Published project/impact data | privacy-filtered locations only (BR-034) |
| GET | `/public/vacancies` · `/public/tenders` · `/public/events` · `/public/publications` | Published items per type | locale-aware |
| POST | `/public/complaints` | Feedback/complaint intake | rate-limited, optional anonymity |
| POST | `/public/forms/{code}/submissions` | Public form submission where enabled | reCAPTCHA/honeypot per config |
| GET | `/public/search` | Search over published content only | never returns internal entities |

Public routes are additionally protected by: strict rate limits, response caching
with publish-time invalidation, and a hard rule that only `Published` content is
reachable.

## 11. Coverage Rules for This Catalogue

```text
[ ] every module in docs/07-backend/modules.md has its endpoints listed here
[ ] every listed path maps to a permission in the catalog (or is explicitly public)
[ ] every protected path appears in the authz test matrix
[ ] status-changing operations are transition routes, never PATCH
[ ] restricted/sensitive modules expose no unscoped list/search/export route
```

## Related

- Conventions (authoritative): `ARCHITECTURE.md` §8
- Conventions in practice: `api.md` · Errors: `errors.md` · Generation: `openapi.md`
- Modules: `../07-backend/modules.md` · Permissions: `../11-security/security.md`

*End of docs/05-api/endpoints.md*