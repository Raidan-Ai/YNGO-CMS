# docs/00-overview/glossary.md

> **Status:** Current | **Owner:** Lead Architect | **Last Updated:** 2026-10-02
> **Source:** consolidated from all specs | **Purpose:** one canonical meaning per term

| Term | Meaning in this project |
|------|-------------------------|
| **Tenant** | The isolation boundary. One organization (or project office) whose data is never visible to another tenant. |
| **Organization** | The NGO entity that owns a tenant: profile, structure (departments, branches, teams), branding. |
| **Branches, Departments, Offices, Teams** | Organizational units used for ABAC scoping and reporting roll-ups. |
| **Platform (System) Admin** | Cross-tenant operator role for platform maintenance; heavily audited; never a business-data editor by default. |
| **Module** | An independently understandable bounded context (e.g. `projects`, `grants`, `meal`) inside the modular monolith. |
| **Bounded Context** | A module's boundary; owns its entities and exposes only a public service + events. |
| **Port / Adapter** | Interface (e.g. `StoragePort`) plus a swappable implementation (e.g. MinIO, Azure Blob). |
| **Content** | Editorial material: pages, articles, news, stories, publications, FAQs, campaigns, vacancies, tenders, events. |
| **Media / DAM** | Digital assets (images, video, audio, files) with renditions and folders. |
| **Document / DMS** | Managed business documents with versions, categories, retention, and classification. |
| **Program** | A long-running thematic area of work containing projects. |
| **Project** | A time-bound funded body of work with locations, team, milestones, and indicators. |
| **Activity** | A discrete action within a project that consumes resources and produces outputs. |
| **Indicator** | A measurable variable used to track results (with values per reporting period). |
| **Logframe** | Logical framework linking goals → outcomes → outputs → activities → indicators. |
| **MEAL** | Monitoring, Evaluation, Accountability, and Learning. |
| **Grant** | Funding awarded to the organization, tracked with milestones and reports. |
| **Donor** | Funding provider (individual, institutional, corporate, government). |
| **Partner** | Implementing or strategic partner organization. |
| **Stakeholder** | Any person/organization with an interest; managed via CRM. |
| **Beneficiary** | A person or household receiving services. **Privacy-critical.** |
| **Household** | A beneficiary group treated as one assistance unit. |
| **Case** | A managed service/support case with assessment, notes, referrals, follow-ups. **Restricted.** |
| **Safeguarding Case** | A confidential incident/protection record. **Highly restricted, separate audit.** |
| **Complaint / Feedback** | Accountability intake item, optionally anonymous, with triage → resolution. |
| **Volunteer** | Unpaid contributor with skills, availability, assignments, hours, certificates. |
| **Asset / Inventory** | Registered equipment/property and consumable stock. |
| **RFQ / PO** | Request for Quotation / Purchase Order in procurement. |
| **Finance Layer** | Budgeting, cost centres, funding sources, utilization, expense imports. **Not a general ledger.** |
| **GIS** | Geographic information: locations, boundaries, layers, maps, spatial queries (PostGIS + MapLibre). |
| **Workflow** | Versioned definition of steps/transitions/conditions/assignments/approvals/SLA. |
| **Automation** | Trigger → conditions → actions rule with execution logging. |
| **RBAC** | Role-based access control on `resource:action` permissions. |
| **ABAC** | Attribute-based rules (e.g. same branch/department, allowed project ids). |
| **Field-level policy** | Rules that restrict individual attributes even when a record is readable. |
| **Audit Log** | Append-only record of who did what, when, to which entity, with before/after and request id. |
| **Public Portal** | Unauthenticated/limited external surface consuming the same API. |
| **Field PWA** | Offline-capable client for field data collection (later phase). |
| **SDK** | Generated client library (TypeScript, Python) from the OpenAPI contract. |
| **Plugin** | Future third-party extension with a manifest; must never bypass tenant boundaries. |
| **Empty state** | The honest UI shown when no real data exists — never filled with fabricated records. |
| **DoD** | Definition of Done; see `AGENTS.md` §12. |
| **ADR** | Architecture Decision Record; see `TECHNICAL_DECISIONS.md`. |

## Naming Conventions (canonical)

- Database columns and JSON payloads: `snake_case` (e.g. `tenant_id`, `created_at`).
- TypeScript identifiers: `camelCase` / `PascalCase`.
- API paths: plural, kebab/lowercase (`/api/v1/programs`).
- Permissions: `resource:action` (e.g. `projects:approve`).
- Avoid synonyms: use **Tenant** (not "account/workspace"), **Organization** (not
  "company"), **Beneficiary** (not "client/patient").