# docs/07-backend/modules.md

> **Status:** Current (inventory) — implementation `PLANNED` | **Owner:** Backend Lead
> **Last Updated:** 2026-10-02 | **Source:** `ARCHITECTURE.md` §5–§6, `IMPLEMENTATION_PLAN.md`
> **Single source of truth for the module count** (resolves contradiction **C-03**:
> the frozen specs quote 40 / 52 / 53 modules).

## 1. Canonical Module Count

```text
Canonical bounded contexts: 52
├── Platform Core                    12
├── Content & Communication           6
├── NGO Operations                   15
├── People & Sensitive                7
├── Insight & Platform Services       8
└── Operational Platform Services     4
```

**Rule (`AGENTS.md` §13):** adding a **53rd** module or any new bounded context
requires an ADR **before** code is written. The numbers quoted in the frozen
specifications are superseded by this table.

## 2. Module Anatomy (mandatory for every row below)

```text
apps/api/src/modules/<module>/
├── <module>.module.ts      exports ONLY the public service
├── <module>.controller.ts  thin HTTP boundary
├── dto/                    validation at the edge
├── <module>.service.ts     application use-cases
├── domain/                 entities, value objects, invariants, policies
├── repositories/           tenant-scoped data access
├── events/                 domain events
└── policies/               authorization + field-level rules
```

Layering and the request pipeline are defined in `architecture.md`; a module may
never import another module's repository (`ARCHITECTURE.md` P4).

## 3. Platform Core (12)

| # | Module | Path | Phase | V1 | Notes |
|---|--------|------|:-----:|:--:|-------|
| 1 | `tenancy` | `modules/tenancy` | 0 | ✅ | Tenant registry, settings resolution, tenant context, bootstrap hooks (ADR-004/005/012) |
| 2 | `identity` | `modules/identity` | 0–1 | ✅ | Users, invitations, sessions, refresh rotation, password reset, MFA prep |
| 3 | `access` | `modules/access` | 1 | ✅ | Roles, permissions catalog, groups, ABAC policy, API keys |
| 4 | `organization` | `modules/organization` | 1 | ✅ | Org profile, unit tree (ABAC scope source), positions, contact points |
| 5 | `governance` | `modules/governance` | 6 | ⬜ | Boards, committees, meetings, resolutions, policies, delegations, COI |
| 6 | `audit` | `modules/audit` | 0–1 | ✅ | Append-only audit writes, query API, separate safeguarding stream |
| 7 | `settings` | `modules/settings` | 1 | ✅ | Scoped configuration; typed accessors; no hard-coded tenant values |
| 8 | `files` | `modules/files` | 2 | ✅ | `StoragePort` adapter, signed URLs, quarantine, object lifecycle |
| 9 | `search` | `modules/search` | 2 | ✅ | `SearchPort` (PG FTS), projections, permission-filtered queries |
| 10 | `notifications` | `modules/notifications` | 3 | ✅ | In-app + SMTP via `NotifierPort`, templates, preferences, delivery attempts |
| 11 | `workflow` | `modules/workflow` | 7 | ⬜ | Versioned definitions, instances, tasks, approvals, escalation (BR-080/021) |
| 12 | `automation` | `modules/automation` | 7 | ⬜ | Trigger → conditions → actions, execution log, permission inheritance (BR-082) |

## 4. Content & Communication (6)

| # | Module | Path | Phase | V1 | Notes |
|---|--------|------|:-----:|:--:|-------|
| 13 | `content` | `modules/content` | 2 | ✅ | All content types, versions, revisions, locks, editorial transitions, scheduling, SEO |
| 14 | `taxonomy` | `modules/taxonomy` | 2 | ✅ | Categories, tags, hierarchy, per-tenant slugs |
| 15 | `media` | `modules/media` | 2 | ✅ | DAM: upload validation, renditions, folders, collections, usage links |
| 16 | `documents` | `modules/documents` | 2 | ✅ | DMS: versions, categories, retention, classification, explicit access |
| 17 | `navigation` | `modules/navigation` | 2 | ✅ | Menus, menu items, redirects, public route mapping |
| 18 | `communications` | `modules/communications` | 3 | ✅ | Outbound messaging beyond system notifications: campaigns, audience segments, send logs |

Themes, microsites, and white-labelling are **configuration surfaces** of
`settings` + `content` + `web` at V1 (tokens and templates per tenant), not
modules. Promoting either to a bounded context requires an ADR (§1).

## 5. NGO Operations (15)

| # | Module | Path | Phase | V1 | Notes |
|---|--------|------|:-----:|:--:|-------|
| 19 | `programs` | `modules/programs` | 3 | ✅ | Programs, sectors, program-level roll-ups |
| 20 | `projects` | `modules/projects` | 3 | ✅ | Projects, locations, team, partners, milestones, lifecycle (BR-030/033) |
| 21 | `activities` | `modules/activities` | 3 | ✅ | Activities, workplan rows, activity locations |
| 22 | `meal` | `modules/meal` | 5 | ⬜ | Indicators, values per period, logframes, baselines, targets, assessments, evaluations |
| 23 | `grants` | `modules/grants` | 4 | ⬜ | Grants, milestones, donor reports, amendments (BR-040/041/042) |
| 24 | `fundraising` | `modules/fundraising` | 4 | ⬜ | Opportunities, proposals, versions, awards |
| 25 | `donors` | `modules/donors` | 4 | ⬜ | Donor records, contacts, funding sources |
| 26 | `partners` | `modules/partners` | 4 | ⬜ | Partners, agreements/MoU, collaboration documents |
| 27 | `crm` | `modules/crm` | 4 | ⬜ | Stakeholders, contacts, interactions, meetings, calls, CRM tasks (append-only) |
| 28 | `procurement` | `modules/procurement` | 3 | ⬜ | Requests → RFQ → quotations → evaluation → PO → delivery; vendors (BR-050/051) |
| 29 | `finance` | `modules/finance` | 3 | ⬜ | Budgets, cost centres, utilisation, expense imports, FX rates — **no GL** (BR-053) |
| 30 | `assets` | `modules/assets` | 3 | ⬜ | Fixed assets, assignments, maintenance, inventory, stock movements |
| 31 | `fleet` | `modules/fleet` | 3 | ⬜ | Vehicles, drivers, trips, itineraries, per diem, travel reports |
| 32 | `travel` | `modules/travel` | 3 | ⬜ | Travel requests/approvals and reconciliation (splits from `fleet` when approved) |
| 33 | `events` | `modules/events` | 3 | ⬜ | Events, agendas, registrations, attendance, post-event reports |

## 6. People & Sensitive (7)

| # | Module | Path | Phase | V1 | Sensitivity | Notes |
|---|--------|------|:-----:|:--:|-------------|-------|
| 34 | `beneficiaries` | `modules/beneficiaries` | 6 | ⬜ | **sensitive** | Beneficiaries, households, enrolment, assistance, referrals; field-level policy (BR-060/066) |
| 35 | `cases` | `modules/cases` | 6 | ⬜ | **restricted** | Cases, assessments, notes, follow-ups; reason-captured access (BR-061) |
| 36 | `safeguarding` | `modules/safeguarding` | 6 | ⬜ | **sensitive** | Incidents, actions, investigators; separate audit stream, excluded from search/exports (BR-062/063) |
| 37 | `complaints` | `modules/complaints` | 6 | ⬜ | restricted | Public intake, triage, assignment, resolution; anonymity preserved |
| 38 | `volunteers` | `modules/volunteers` | 6 | ⬜ | confidential | Volunteers, skills, availability, assignments, hours, certificates |
| 39 | `people` | `modules/people` | 6 | ⬜ | confidential | HR-lite: staff, contracts, leave, training (payroll stays external) |
| 40 | `membership` | `modules/membership` | 6 | ⬜ | confidential | Members, dues, renewals, status |

Every module in this group MUST: exclude itself from unscoped queries, search
projections, activity feeds, and generic exports; and ship negative tests per data
surface (read, list, search, export, report) as part of its Definition of Done.

## 7. Insight & Platform Services (8)

| # | Module | Path | Phase | V1 | Notes |
|---|--------|------|:-----:|:--:|-------|
| 41 | `reporting` | `modules/reporting` | 3 | ✅ | Templates, async runs, snapshots, permission-checked downloads (BR-073) |
| 42 | `analytics` | `modules/analytics` | 8 | ⬜ | KPI queries, dashboards, widgets, saved views |
| 43 | `gis` | `modules/gis` | 3 | ✅ | Locations, boundaries, layers, geofences, privacy rules via PostGIS + MapLibre |
| 44 | `import-export` | `modules/import-export` | 3 | ✅ | Validate-then-commit imports, async exports, per-row error reports (BR-071/072) |
| 45 | `data-governance` | `modules/data-governance` | 5 | ⬜ | Quality rules, issue queue, controlled merges, retention enforcement (BR-074) |
| 46 | `integrations` | `modules/integrations` | 9 | ⬜ | Provider adapters (M365/Google/SMS/WhatsApp/payments/identity), credential handling |
| 47 | `developer-platform` | `modules/developer-platform` | 9 | ⬜ | OpenAPI publication, SDK generation, developer portal content, sandbox keys |
| 48 | `ai` | `modules/ai` | 10 | ⬜ | `AiPort` (no-op by default), request/response records, human review workflow (ADR-014) |

## 8. Operational Platform Services (4)

| # | Module | Path | Phase | V1 | Notes |
|---|--------|------|:-----:|:--:|-------|
| 49 | `tasks` | `modules/tasks` | 3 | ✅ | Assignable tasks, due dates, reminders, linkage to any entity |
| 50 | `api-platform` | `modules/api-platform` | 1 | ✅ | API keys, rate-limit policy, webhook endpoints/subscriptions/deliveries, versioning rules |
| 51 | `observability` | `modules/observability` | 8 | ✅ | Health/readiness, metrics, tracing hooks, admin status surfaces (not the log pipeline) |
| 52 | `system-admin` | `modules/system-admin` | 8 | ⬜ | Cross-tenant platform operations, explicitly audited, distinct code path (ADR-004 rule 6) |

## 9. Cross-Module Rules

1. A module exposes **only** its public service; cross-context reads use that
   service or a domain-event projection.
2. Every module declares its permissions in the catalog; deny-by-default applies
   (`BR-003`).
3. Every module registers its `TenantGuard` + `PermissionsGuard` coverage tests.
4. Sensitive modules additionally register a field-policy test and an
   export/search exclusion test.
5. Any module added later shifts `system-admin` numbering — the **count** stays
   authoritative until an ADR changes it.
6. Deferred candidates (advocacy, plugins/marketplace, knowledge base, microsites)
   are listed in `../99-project-management/backlog.md` and require an ADR to
   become modules.

## Related

- Architecture and pipeline: `architecture.md` · Cross-cutting services: `services.md`
- Boundaries: `ARCHITECTURE.md` §5–§6 · Endpoints: `../05-api/endpoints.md`
- Plan: `../../IMPLEMENTATION_PLAN.md` · Contradiction record: `../../PROJECT_AUDIT.md`

*End of docs/07-backend/modules.md*