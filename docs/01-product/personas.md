# docs/01-product/personas.md

> **Status:** Current | **Owner:** Product Owner | **Last Updated:** 2026-10-02
> **Source:** Engineering_Documentation_Package_v2.md (audiences), Blueprint.md
> **Purpose:** who uses the platform, what they may do, and what constrains them.

## 1. Internal Personas

| ID | Persona | Primary need | Default access posture |
|----|---------|--------------|------------------------|
| P-01 | Organization Administrator | Configure the tenant, org structure, roles, users | Full tenant admin; cannot touch platform-global settings |
| P-02 | Content Editor / Publisher | Create, review, publish multilingual content | `content:*` per role; publish requires approval |
| P-03 | Communications Officer | News, press, campaigns, media room | Content + media write; no finance/people data |
| P-04 | Program Manager | Programs, projects, milestones, workplans | Project scope via ABAC (assigned projects) |
| P-05 | MEAL Officer | Indicators, logframe, values, assessments | Read projects, write MEAL data |
| P-06 | Grants / Fundraising Officer | Grants, milestones, reports, opportunities | Grant + donor scope; no safeguarding |
| P-07 | Field Officer / Data Collector | Forms, submissions, beneficiary intake | Field-scoped; uploads; offline-capable later |
| P-08 | Case Worker | Cases, assessments, referrals, follow-ups | Case scope only; restricted fields per policy |
| P-09 | Safeguarding Officer | Confidential incidents | Highest restriction; separate audit stream |
| P-10 | Finance Officer | Budgets, cost centres, utilization, imports | Finance-lite; ledger stays external |
| P-11 | Procurement / Logistics Officer | Requests, RFQs, quotations, POs, delivery | Procurement scope; vendor data |
| P-12 | HR / People Officer | Records, contracts, leave, training | People scope; payroll external |
| P-13 | Volunteer Coordinator | Volunteers, skills, assignments, hours | Volunteer scope only |
| P-14 | Board / Governance Member | Meetings, resolutions, policies | Read-heavy; board scope |
| P-15 | Auditor / Compliance Reviewer | Audit trail, evidence, exports | Broad read + audit log read; no edit |
| P-16 | Platform (System) Admin | Cross-tenant operations, health, integrations | Platform-only; every action heavily audited |
| P-17 | Developer / Integrator | API keys, webhooks, SDK | API-key scoped to granted permissions |

## 2. External Personas

| ID | Persona | Primary need | Access posture |
|----|---------|--------------|----------------|
| X-01 | Public Visitor | Read published content, projects, impact, tenders | Anonymous; published data only |
| X-02 | Donor (external) | See agreements, reports, impact | Portal; own records only |
| X-03 | Partner Organization | Coordinate projects, documents, reports | Portal; own collaboration records |
| X-04 | Applicant / Beneficiary (self-service, where enabled) | Apply, submit documents, track status | Portal; own submission only |
| X-05 | Volunteer (external) | Apply, track assignments, certificates | Portal; own records only |
| X-06 | Complainant / Feedback Sender | Submit feedback, optionally anonymous | Public form; rate-limited; no account required |

## 3. Persona → Permission Pattern

- Roles are **compositions of `resource:action` permissions**, created per tenant
  (see `../07-backend/services.md` §access).
- ABAC narrows, never widens: e.g. a Program Manager role plus "assigned projects
  only" attribute scope.
- Field-level policy is independent of record readability — a Case Worker may read
  a case while specific attributes stay hidden (`../11-security/privacy.md`).
- External personas never receive internal permissions; portals call the same API
  with restricted tokens.

## 4. Persona Constraints that Shape the UI

| Constraint | Consequence |
|------------|-------------|
| Arabic-first, RTL | Every surface must be reviewed in `ar` and `en`; logical CSS only |
| Low-bandwidth field use | Small payloads, pagination, optimistic-free flows, later offline PWA |
| Non-technical operators | Clear empty states with a single primary next action; no ambiguous errors |
| Multiple roles per user | Effective-permission display; role switcher is UX only, never security |
| Sensitive work (P-08, P-09) | Explicit "why am I accessing this" reason for restricted exports |
| Auditors (P-15) | Read-only export of audit trail with filters and immutable record ids |

## 5. Anti-Personas (explicitly not designed for)

- Someone wanting a general-purpose website builder without NGO operations.
- Someone wanting a full ERP/GL replacement (see `../00-overview/scope.md` §4).
- Someone wanting anonymous unrestricted bulk data export.
- Someone wanting AI to make final decisions about people or funding.

## Related

- Requirements: `requirements.md` · Flows: `user-flows.md`
- Permissions model: `../07-backend/services.md`, `../11-security/security.md`
- Privacy classes: `../11-security/privacy.md`

*End of docs/01-product/personas.md*