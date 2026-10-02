# docs/99-project-management/tasks.md — Task Index (TASK-001 … TASK-115)

> **Status:** Current (all tasks `PLANNED`) | **Owner:** Lead Architect
> **Last Updated:** 2026-10-02 | **Source:** `../../IMPLEMENTATION_PLAN.md`
> **Rule:** TASK ids are stable and are referenced by ADRs, risks, acceptance criteria,
> and reports. A task is `Done` only when every Definition-of-Done box in
> `../../AGENTS.md` §12 holds **and** the validation commands were actually run.
> **Scope rule:** this file is the **index** (id, title, goal, dependencies). Phase
> scope, deliverables, and exit gates stay in the canonical `IMPLEMENTATION_PLAN.md`
> and are deliberately not duplicated here.

## 1. How to Use This Index

1. Look up the task id below (ids for the "Key tasks" of each phase come verbatim from
   `../../IMPLEMENTATION_PLAN.md`; phases without an explicit key-task line have their
   ids allocated here inside that phase's reserved block).
2. Read the phase scope + exit gate in `../../IMPLEMENTATION_PLAN.md`.
3. Read the module placement (`../07-backend/modules.md`), endpoints
   (`../05-api/endpoints.md`), and expected tests
   (`../09-testing/acceptance-criteria.md`).
4. Expand the task using the template in §3 **before** writing code.
5. Run the `AGENTS.md` §11 loop and report with the §14 format.

## 2. Task Index

### Phase 0 — Foundation (BLOCKING)

Ids `TASK-001 … TASK-010` · Gate: ADR-001/003/004/005/009/012 `ACCEPTED` and
Q-01…Q-06 answered.

| ID | Task | Goal | Depends on |
|----|------|------|-----------|
| TASK-001 | Repo + workspaces | pnpm monorepo skeleton (`apps/*`, `packages/*`) installs, builds, lints | ADR-002 (Q-08) |
| TASK-002 | Environment contract | `.env.example` documents every var; boot fails fast when one is missing | TASK-001 |
| TASK-003 | Compose stack | proxy/web/api/worker/postgres+PostGIS/redis/minio all report healthy | TASK-001, TASK-002 |
| TASK-004 | API skeleton | NestJS+Fastify with global pipes/filters/interceptors, `/health`, `/ready`, OpenAPI | TASK-001, ADR-001, ADR-003 |
| TASK-005 | Web skeleton | Next.js App Router + next-intl (ar/en), RTL `dir`, Tailwind+shadcn, empty-state primitives | TASK-001, ADR-011 |
| TASK-006 | DB + migrations | Migration tool wired; `tenant`, `user`, `audit_log` + `tenant_id` convention | TASK-003, TASK-004, ADR-003 |
| TASK-007 | Tenancy primitives | `TenantContext`, `TenantGuard`, tenant-scoped repository base | TASK-006, ADR-004, ADR-005 |
| TASK-008 | Auth skeleton | Session cookie + bearer + refresh rotation, Argon2id, login audit | TASK-004, TASK-007, ADR-007 |
| TASK-009 | CI | Lint + typecheck + unit + secret scan + no-mock-data scan (ADR-013); branch protection | TASK-001 |
| TASK-010 | Bootstrap CLI | `pnpm bootstrap:org` creates first tenant/org/admin, no defaults, self-disables | TASK-006, TASK-008, ADR-012 |

### Phase 1 — Core, Identity & Access

Ids `TASK-011 … TASK-020` (allocated in this index) · Test focus: deny-by-default
authz, ABAC scope, token rotation, audit immutability
(`../09-testing/acceptance-criteria.md` §3).

| ID | Task | Goal | Depends on |
|----|------|------|-----------|
| TASK-011 | Tenancy lifecycle | Tenant CRUD, settings resolution, tenant admin surface | TASK-007 |
| TASK-012 | Identity module | Users, profiles, sessions, login events, MFA preparation | TASK-008 |
| TASK-013 | Invitations | Single-use expiring invitations; accept → set password → active | TASK-012 |
| TASK-014 | Password reset | Single-use expiring reset tokens, audited | TASK-012 |
| TASK-015 | Permission catalog | Platform `resource:action` catalog + reviewed change process | TASK-011 |
| TASK-016 | RBAC roles | Roles, role-permission, user-role, effective permissions (BR-014) | TASK-015 |
| TASK-017 | ABAC policy | Org-unit subtree and record assignment scope; never widens (BR-004) | TASK-016 |
| TASK-018 | Groups | Groups + membership feeding effective permissions | TASK-016 |
| TASK-019 | API keys | Hashed at rest, prefix shown, raw secret once, scoped, revocable (BR-006) | TASK-015 |
| TASK-020 | Organization module | Org profile, unit tree (ABAC source), positions, contact points | TASK-011 |
### Phase 2 — Content & Media Platform

Ids `TASK-021 … TASK-026` (allocated in this index) · Test focus: guarded
transitions, upload hardening, tenant-scoped search
(`../09-testing/acceptance-criteria.md` §4).

| ID | Task | Goal | Depends on |
|----|------|------|-----------|
| TASK-021 | Audit hardening | Append-only DB grants, query API, redaction, separate safeguarding stream | TASK-006, TASK-012 |
| TASK-022 | Settings module | Scoped configuration, typed accessors, no hard-coded tenant values | TASK-011 |
| TASK-023 | Content module | All content types with versions, revisions, locks (BR-025/026) | TASK-015, TASK-021 |
| TASK-024 | Media/DAM + files | Upload validation, renditions, folders, `StoragePort`, signed URLs, quarantine | TASK-003, ADR-009 |
| TASK-025 | Taxonomy | Categories, tags, hierarchy, per-tenant slugs | TASK-023 |
| TASK-026 | Search module | PG FTS behind `SearchPort`; tenant-scoped and permission-filtered | TASK-023, ADR-010 |

### Phase 3 — Programs, Projects, Operations

Ids `TASK-030 … TASK-036` (verbatim from `IMPLEMENTATION_PLAN.md`) · Test focus: the
program→project→indicator→report chain on real data, durable queues, GIS privacy
(`../09-testing/acceptance-criteria.md` §5).

| ID | Task | Goal | Depends on |
|----|------|------|-----------|
| TASK-030 | Programs + activities | Program, sector, activity, activity locations | TASK-015 |
| TASK-031 | Projects + milestones | Project, locations, team, partners, milestones, workplan | TASK-030 |
| TASK-032 | Project locations (GIS) | PostGIS locations/boundaries, MapLibre rendering, layer privacy | TASK-031 |
| TASK-033 | Reporting engine (async) | Templates, async runs, progress, snapshots, permission-checked download | TASK-034 |
| TASK-034 | MEAL | Indicators, indicator values, reporting periods, logframe | TASK-031 |
| TASK-035 | Notifications baseline | In-app + SMTP via `NotifierPort`, templates, preferences, attempts | TASK-008 |
| TASK-036 | Import/export baseline | Validate-then-commit imports, async exports, per-row error reports | TASK-006 |

### Phase 4 — Grants, Donors, Partners & CRM

Ids `TASK-040 … TASK-046` (verbatim from `IMPLEMENTATION_PLAN.md`).

| ID | Task | Goal | Depends on |
|----|------|------|-----------|
| TASK-040 | Grants core | Grant lifecycle, amendments, funding source (BR-040/042) | TASK-031 |
| TASK-041 | Grant milestones + reports | Milestones and grant reports with completeness guard | TASK-040 |
| TASK-042 | Donors | Donor profiles, contacts, funding links | TASK-040 |
| TASK-043 | Partners + MoU | Partner profiles, agreements | TASK-031 |
| TASK-044 | CRM interactions | Contacts, interactions, meetings, calls, CRM tasks (append-only, BR-043) | TASK-043 |
| TASK-045 | Events | Events, registration, attendance, certificates | TASK-031 |
| TASK-046 | Notifications wiring for deadlines | Real dated records → notifications (BR-044) | TASK-041, TASK-035 |

### Phase 5 — MEAL, Forms & Surveys

Ids `TASK-050 … TASK-055` (verbatim from `IMPLEMENTATION_PLAN.md`) · Test focus:
indicator aggregation, form versioning, import partial-failure.

| ID | Task | Goal | Depends on |
|----|------|------|-----------|
| TASK-050 | Logframe + indicators | Logframe rows, indicators, baselines, targets, periods | TASK-034 |
| TASK-051 | Indicator values + aggregation | Period-scoped auditable values; correct aggregation (BR-031/032) | TASK-050 |
| TASK-052 | Form schema engine | Reusable field schema, conditional logic, versioning (BR-070) | TASK-024 |
| TASK-053 | Submissions + review | Submissions, attachments, review workflow, submissions export | TASK-052 |
| TASK-054 | Import pipeline | CSV/XLSX/JSON mapping → preview → confirm → report (BR-071/072) | TASK-036 |
| TASK-055 | Data-quality rules | Duplicate/missing detection, issue queue, retention (BR-074) | TASK-054 |

### Phase 6 — Sensitive Domains (highest scrutiny)

Ids `TASK-060 … TASK-066` (verbatim from `IMPLEMENTATION_PLAN.md`) · Requires security
review before the gate is signed · Test focus: field-level policy, exclusion from
generic search/export, reason capture, separate audit stream
(`../09-testing/acceptance-criteria.md` §6).

| ID | Task | Goal | Depends on |
|----|------|------|-----------|
| TASK-060 | Beneficiaries + field-level policy | Beneficiary/household/enrollment/assistance/referral with field policy (BR-066) | TASK-017, TASK-021 |
| TASK-061 | Cases | Case lifecycle, notes, assessments, follow-ups (BR-060/061) | TASK-060 |
| TASK-062 | Safeguarding + separate audit | Restricted cases, incidents, investigators, controlled export (BR-062/063) | TASK-061 |
| TASK-063 | Complaints | Public intake, anonymous option, triage, escalation, resolution | TASK-062 |
| TASK-064 | Volunteers + membership | Volunteers, assignments, hours, certificates; members, dues | TASK-060 |
| TASK-065 | Governance | Boards, committees, resolutions, policies, delegations, COI | TASK-031 |
| TASK-066 | GIS layers + privacy rules | Public/private layer policy, precision reduction, exclusion tests (BR-034) | TASK-032 |

### Phase 7 — Workflow, Automation, Notifications & MFA

Ids `TASK-070 … TASK-075` (verbatim from `IMPLEMENTATION_PLAN.md`) · Test focus:
transition-only status changes, definition versioning, permission inheritance.

| ID | Task | Goal | Depends on |
|----|------|------|-----------|
| TASK-070 | Workflow engine | Versioned definitions, steps, transitions, conditions (BR-080/021) | TASK-023 |
| TASK-071 | Approval + escalation | Approvals, delegation, SLA escalation, explicit timeouts | TASK-070 |
| TASK-072 | Automation rules | Trigger → conditions → actions, permission-aware, execution log (BR-082) | TASK-071 |
| TASK-073 | Notification center + templates | Multi-channel delivery, tenant-aware localized templates, attempts | TASK-046 |
| TASK-074 | MFA (TOTP) | Enrolment, verification, recovery — all audited | TASK-012 |
| TASK-075 | Security headers + CSRF/CSP | Headers, CSRF, CSP, rate-limit tightening | TASK-004 |

### Phase 8 — Reporting, Dashboards, Analytics & Observability

Ids `TASK-080 … TASK-085` (verbatim from `IMPLEMENTATION_PLAN.md`) · Test focus:
metrics computed from real queries, honest empty states (no fabricated numbers).

| ID | Task | Goal | Depends on |
|----|------|------|-----------|
| TASK-080 | Report templates + async runs | Parameterized reports, charts, maps, snapshots, async generation | TASK-033 |
| TASK-081 | Exports | PDF/HTML/CSV/XLSX export; permission-checked + audited downloads (BR-073) | TASK-080 |
| TASK-082 | KPI + analytics queries | KPI engine, portfolio/funding/geographic analysis, drill-down | TASK-080 |
| TASK-083 | Dashboard builder | Widgets (KPI, chart, table, map, progress, timeline); tenant + permission aware | TASK-082 |
| TASK-084 | Metrics + tracing | Metrics, tracing, error-reporting adapter, SLO dashboards, alerting | TASK-004 |
| TASK-085 | Search facets + Arabic tuning | Facets, Arabic tokenization, cross-entity search | TASK-026 |

### Phase 9 — Portals, Integrations & Developer Platform

Ids `TASK-090 … TASK-094` (verbatim from `IMPLEMENTATION_PLAN.md`) · Test focus:
per-portal guards, generated SDKs, swappable adapters.

| ID | Task | Goal | Depends on |
|----|------|------|-----------|
| TASK-090 | Public site | Public pages from the API, ISR/SSR, `hreflang`, RTL for `ar` | TASK-083 |
| TASK-091 | Portals + guards | Partner/Donor/Applicant/Volunteer/Member/Board portals with independent guards | TASK-090 |
| TASK-092 | Developer portal + SDK | OpenAPI publication, guides, keys, webhooks, CI-generated SDKs | TASK-019 |
| TASK-093 | Integration adapters | M365, Google, Teams, Slack, SMS/WhatsApp, accounting, IdP, GIS, payments | TASK-075 |
| TASK-094 | Theme engine + white-label | Design tokens, microsites, tenant theming that never breaks RTL | TASK-090 |

### Phase 10 — Optional AI & Field/Offline (opt-in, ADR-014)

Ids `TASK-100 … TASK-105` (verbatim from `IMPLEMENTATION_PLAN.md`) · Gated by Phase 9
**and** an explicit operational decision to enable AI · Test focus: draft-only output,
permission-inheritance negative tests, deterministic offline sync.

| ID | Task | Goal | Depends on |
|----|------|------|-----------|
| TASK-100 | AI port + registry | `AiPort` (no-op by default), provider/model registry, request records | TASK-093 |
| TASK-101 | AI governance + review UI | Draft → human review → approval; never a silent overwrite (BR-090/092) | TASK-100 |
| TASK-102 | AI evaluation harness | Datasets, metrics, thresholds, regression tests gating feature enablement | TASK-101 |
| TASK-103 | Python sidecar | Optional FastAPI OCR/NLP/embeddings sidecar with no canonical ownership | TASK-100 |
| TASK-104 | Field offline store + sync | Offline forms, local encrypted store, queue, retry, GPS capture | TASK-052 |
| TASK-105 | Conflict resolution + local encryption | Deterministic sync that never loses accepted data | TASK-104 |

### Phase 11 — Production Hardening & Release

Ids `TASK-110 … TASK-115` (verbatim from `IMPLEMENTATION_PLAN.md`).

| ID | Task | Goal | Depends on |
|----|------|------|-----------|
| TASK-110 | Performance pass | Query budgets, index review, N+1 elimination, caching, load test | TASK-083 |
| TASK-111 | Security review | Threat-model review, dependency scan, penetration checklist, rotation drill | TASK-075 |
| TASK-112 | Backup/restore + DR drill | Restore verified on a clean environment within the documented RTO | TASK-084 |
| TASK-113 | Runbook + on-call | Operational runbook, alerting, on-call checklist, upgrade procedure | TASK-110 |
| TASK-114 | Compliance artefacts | Retention + privacy matrix, access reviews, export/deletion workflows | TASK-062 |
| TASK-115 | Release + changelog | Release notes, version tag, migration instructions, `CHANGELOG.md` | TASK-111 |

## 3. Task Template (expand before coding)

```text
TASK-NNN — <title>
Phase:          <n>
Goal:           <one sentence outcome>
Context:        <why it exists; links to ARCHITECTURE / domain / API docs>
Dependencies:   <TASK-IDs that must be complete>
Files:          <expected paths touched>
Implementation: <precise steps, no ambiguity>
Acceptance:     <testable criteria>
Tests:          <unit / integration / authz / tenant-isolation / E2E>
Validation:     <exact commands, e.g. pnpm --filter api test>
Do not:         <guardrails for this task>
```

Required tests per task type (`../../AGENTS.md` §5, §12):

| Task touches | Must include |
|---------------|--------------|
| Any data entity | Cross-tenant negative test (read/list/search/export) |
| Any protected route | Authorization test proving deny-by-default |
| Any mutation | Audit-record assertion |
| Sensitive fields | Field-level policy test + search/export exclusion test |
| Any UI list | Loading / empty / error / unauthorized / populated states |
| Any localized surface | RTL review (logical properties only) |

## 4. Status Rules

```text
PLANNED     defined in this index; not started
IN PROGRESS branch opened; work underway
BLOCKED     cannot proceed — record the blocking id (ADR or Q-NN) per AGENTS.md §11
DONE        every DoD box satisfied and validation commands actually run
```

A task is never marked `DONE` without observed command output. If validation did not
run, the status is `PARTIAL` and the report must say so (`AGENTS.md` §14).

## 5. Id Allocation Note

Ids for Phases 0 and 3–11 are **verbatim** from `../../IMPLEMENTATION_PLAN.md` §"Key
tasks". Ids for Phases 1 and 2 (`TASK-011 … TASK-020`, `TASK-021 … TASK-026`) are
allocated here in the gaps between the plan's key tasks, so no id is ever reused or
renumbered. If the plan later adds key tasks to those phases, the plan's ids win and
this file is updated to match.

## Related

- Canonical plan + gates: `../../IMPLEMENTATION_PLAN.md` · Roadmap: `roadmap.md`
- Acceptance criteria: `../09-testing/acceptance-criteria.md` · Risks: `../../RISK_REGISTER.md`
- Modules: `../07-backend/modules.md` · Execution contract: `../../AGENTS.md`

*End of docs/99-project-management/tasks.md*