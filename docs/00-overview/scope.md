# docs/00-overview/scope.md

> **Status:** Current | **Owner:** Product Owner | **Last Updated:** 2026-10-02
> **Source:** YNGO-CMS_Blueprint.md, Engineering_Documentation_Package_v2.md
> **Purpose:** define what is in scope, out of scope, and when

## In Scope — Platform

Tenancy · Identity · Access (RBAC/ABAC) · Organization · Governance · Audit ·
Settings · Media/DAM · Documents/DMS · Search · Notifications · Files · Workflow ·
Automation · API platform (keys, webhooks, versioning) · Reporting/Analytics ·
GIS · Import/Export · Data Governance/Quality · Observability · Backup/DR ·
Developer platform (OpenAPI, SDKs).

## In Scope — CMS

Pages · Articles · Posts · News · Stories · Announcements · Press releases ·
Publications · Research · Policy papers · Case studies · FAQs · Campaigns ·
Vacancies · Tenders · Events · Categories/Tags · SEO · Media library · Page
builder/blocks · Themes · Localization (ar/en) · Editorial workflow · Versioning ·
Scheduling.

## In Scope — NGO Operations

Programs · Projects · Activities · Indicators/MEAL · Grants · Donors ·
Opportunities · Proposals · Partners · Stakeholders/CRM · Beneficiaries ·
Households · Cases · Referrals · Safeguarding · Complaints/Feedback · Forms/Surveys ·
Volunteers · HR/People-lite · Procurement · Finance Layer (budgets, not GL) ·
Assets/Inventory · Fleet · Travel · Events · Tasks · Communications · Advocacy ·
Membership.

## In Scope — Portals & Integrations

Public website · Partner portal · Donor portal · Applicant portal · Volunteer
portal · Member portal · Field portal/PWA · Board portal · Developer portal ·
Microsites · Theme engine · White-label. Integrations: M365, Google Workspace,
Teams, Slack, email, SMS, WhatsApp, Telegram, accounting, cloud storage, identity
providers, GIS services, payments, AI providers — all behind adapters.

## Out of Scope (explicit)

| Item | Reason |
|------|--------|
| General ledger / double-entry accounting | Finance is a budgeting/utilisation layer; accounting is an integration boundary. |
| Payroll processing | Integration only unless explicitly enabled later. |
| Full ERP replacement (v1) | Explicit product decision — avoid monolithic ERP creep. |
| Distributed microservices at v1 | Modular monolith first; extraction only when justified. |
| Graph database as canonical store | PostgreSQL + PostGIS is canonical. |
| Redis or object storage as primary data store | Non-canonical, rebuildable infrastructure. |
| Real-time collaboration/co-editing | Not a stated requirement; defer. |
| Mobile native apps (v1) | PWA/field later; API is ready. |
| Offline field capability (v1) | Deferred to Phase 10; must be real, not simulated. |
| AI as an authoritative record producer | AI output is always a human-reviewed draft. |

## Release Scoping

**V1** = Phase 0 → Phase 3: Foundation (repo, env, compose, CI, tenancy, auth
skeleton, migrations, health) + Core/Identity/Access + Content & Media + Programs/
Projects/Operations. Everything else extends V1 without breaking API contracts.

**Post-V1** = Phases 4–11: Grants/Donors/CRM, MEAL/Forms, Sensitive Domains,
Workflow/Automation/Notifications, Reporting/Observability, Portals/Integrations,
Optional AI & Field/Offline, Production Hardening.

## Success Criteria (platform level)

- A new NGO can be onboarded and populated through real entry, imports, and APIs —
  starting from a genuinely empty system.
- Tenant isolation and authorization are provably enforced on every data surface.
- Arabic RTL and English LTR are both production quality.
- Every capability is reachable through the versioned API.
- The system is self-hostable on a single Linux server with Docker Compose.

## Related

- Requirements: `docs/01-product/prd.md`, `docs/01-product/requirements.md`
- Phases and gates: `IMPLEMENTATION_PLAN.md`