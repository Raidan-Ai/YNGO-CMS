# docs/01-product/prd.md

> **Status:** Current | **Owner:** Product Owner | **Last Updated:** 2026-10-02
> **Source:** Engineering_Documentation_Package_v2.md §1 (PRD)
> **Scope:** V1 + declared roadmap; supersedes prose duplicates in frozen specs

## 1. Product

**YNGO-CMS** (YemenNGO-CMS): a configurable digital operating platform for NGO
content, programs, projects, people, funding, knowledge, compliance, reporting,
and public engagement.

## 2. Problem

NGOs operate across disconnected systems for website content, documents, projects,
grants, contacts, reporting, forms, field collection, and communication. Data is
duplicated, reporting is manual, and accountability evidence is fragmented.
YNGO-CMS centralises these capabilities while preserving modularity and permission
boundaries.

The product **MUST NOT** become a generic ERP disguised as a CMS: the core
prioritises NGO communication/content and operational coordination, with
finance/accounting as an integration boundary unless explicitly enabled.

## 3. Users

**Internal:** platform administrators, organization administrators, executive
leadership, program directors/managers, project managers, MEAL staff,
communications teams, HR staff, finance staff, procurement teams, field officers,
case workers, editors/authors/reviewers, volunteers.

**External:** donors, implementing partners, suppliers/vendors, members,
applicants, beneficiaries, the general public, media, regulators.

## 4. Jobs To Be Done

| Actor | Job |
|-------|-----|
| Communications editor | Publish accurate Arabic/English content with an approval workflow. |
| Program manager | Track programs, projects, activities, and indicators against plans. |
| MEAL officer | Define indicators, collect values, and produce verified results. |
| Grants officer | Track awards, milestones, deadlines, and donor reporting. |
| Partnerships lead | Manage donors, partners, and stakeholder relationships. |
| Case worker | Manage beneficiary cases with strict confidentiality. |
| Field officer | Collect data (later: offline) and sync reliably. |
| Executive | See portfolio health, funding, and impact in dashboards. |
| Donor/Partner | View progress and reports through a portal. |
| Applicant | Apply to opportunities and track status. |
| Member | Manage membership status and renewals. |
| Platform admin | Operate tenants, integrations, and system health safely. |
| External system | Consume and push data through the versioned API. |

## 5. Functional Requirements (grouped)

### 5.1 Platform
- **FR-P1** Multi-tenant isolation across every data surface (records, lists, search, exports, reports, files, webhooks, jobs, AI).
- **FR-P2** Identity: invitations, login, password reset, sessions, MFA (later), OIDC/SAML adapters (later).
- **FR-P3** Access: roles, permissions (`resource:action`), groups, ABAC attribute rules, API keys.
- **FR-P4** Organization: profile, departments, branches, offices, teams, positions.
- **FR-P5** Audit: append-only record of every mutation and security event.
- **FR-P6** Media/DAM with upload validation, renditions, and signed access.
- **FR-P7** Documents/DMS with versions, categories, retention, and classification.
- **FR-P8** Search over permitted resources with Arabic support and facets.
- **FR-P9** Notifications: in-app, email, SMS, push, webhook, with templates and preferences.
- **FR-P10** Workflow engine with versioned definitions and guarded transitions.
- **FR-P11** Automation: trigger → conditions → actions, logged and permission-aware.
- **FR-P12** API platform: versioned REST, generated OpenAPI, scopes, rate limits, webhooks.
- **FR-P13** Reporting/Analytics/Dashboards over real data only.
- **FR-P14** GIS: PostGIS-backed locations, boundaries, layers, and privacy rules.
- **FR-P15** Import/Export: CSV/XLSX/JSON with validation, preview, and reporting.
- **FR-P16** Data governance: classification, retention, access reviews, export/deletion workflows.
- **FR-P17** Observability: structured logs, request ids, health/readiness, metrics.
- **FR-P18** Backup/DR: scheduled backups, off-site copy, restore verification.

### 5.2 CMS
- **FR-C1** Content types: pages, articles, posts, news, stories, announcements, press releases, publications, research, policy papers, case studies, FAQs, campaigns, vacancies, tenders, events.
- **FR-C2** Lifecycle: draft → in review → changes requested → approved → scheduled → published → unpublished → archived.
- **FR-C3** Editorial: calendar, assignments, reviewers, due dates, comments, locking, version comparison.
- **FR-C4** Media library, blocks/page builder, themes, SEO, localization (ar/en), scheduling.

### 5.3 Operations
- **FR-O1** Programs, projects, activities, milestones, locations, team.
- **FR-O2** MEAL: logframes, indicators, values, baselines, assessments, evaluations.
- **FR-O3** Grants, opportunities, proposals, milestones, donor reports.
- **FR-O4** Donors, partners, stakeholders/CRM interactions.
- **FR-O5** Beneficiaries, households, enrolment, assistance, referrals (privacy-critical).
- **FR-O6** Cases with assessment, notes, referrals, follow-ups (restricted).
- **FR-O7** Safeguarding cases (highly restricted, separate audit).
- **FR-O8** Complaints/feedback with public intake and anonymous option.
- **FR-O9** Forms/surveys with conditional logic, submissions, review, exports.
- **FR-O10** Volunteers, membership, HR-lite, governance.
- **FR-O11** Procurement (request → RFQ → evaluation → PO → delivery).
- **FR-O12** Finance layer (budgets, cost centres, utilisation) — not a GL.
- **FR-O13** Assets/inventory, fleet, travel, events, tasks.

## 6. Non-Functional Requirements

| Area | Requirement |
|------|-------------|
| Security | Secure by default; deny-by-default authorization; no default credentials; secrets never committed. |
| Isolation | Tenant A can never access Tenant B data on any surface. |
| Privacy | Beneficiary/Case/Safeguarding data has stricter defaults and field-level controls. |
| i18n | Arabic-first RTL and English LTR both first-class. |
| Performance | API p95 < 300 ms; search p95 < 500 ms; async report < 60 s (standard templates). |
| Availability | 99.5 % monthly target for the reference deployment. |
| Auditability | Every mutation and security event is recorded and immutable. |
| Portability | Self-hostable on one Linux host with Docker Compose; no vendor lock-in. |
| Maintainability | Modular monolith with enforced boundaries; testable modules. |
| Accessibility | WCAG-oriented components; keyboard navigation; focus management. |
| Data integrity | Referential integrity, constraints, transactions; migrations reversible. |

## 7. Acceptance Criteria (product level)

1. A clean installation is empty and shows professional empty states everywhere.
2. An operator can create the first organization + administrator without defaults.
3. Cross-tenant attempts fail on read, list, search, export, report, file, webhook, and job surfaces.
4. Unauthorized users cannot reach beneficiary/case/safeguarding data through any route.
5. Content can be created, reviewed, approved, published, and viewed publicly in both languages.
6. A program → project → activity → indicator → report chain works with real data.
7. Every capability is reachable through `/api/v1` with generated OpenAPI.
8. No mock, demo, or fabricated business data exists in runtime code.

## 8. Related

- Scope: `docs/00-overview/scope.md`
- Requirements detail: `docs/01-product/requirements.md`
- Personas and flows: `docs/01-product/personas.md`, `docs/01-product/user-flows.md`
- Domain model: `docs/02-domain/domain-model.md`