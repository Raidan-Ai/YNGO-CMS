# docs/06-frontend/routes.md

> **Status:** Current (catalogue) — implementation `PLANNED` | **Owner:** Frontend Lead
> **Last Updated:** 2026-10-02 | **Source:** `ARCHITECTURE.md` §4; `../01-product/user-flows.md`
> **Purpose:** every route, its surface, its access requirement, and its states.

## 1. Route Conventions

```text
Locale prefix        every route lives under /[locale] with ar (default) and en
Route groups         (site) public · (admin) authenticated · (portal) external
                     authenticated · (auth) unauthenticated auth flows
Logical CSS only     ms/me/ps/pe/start/end — physical left/right is forbidden
States               every data route implements loading · empty · error ·
                     unauthorized · populated · partial
Permissions in UI    hide/disable only — never the security boundary
```

## 2. Public Site — `/[locale]/(site)/...`

| Route | Purpose | Access | Rendering |
|-------|---------|--------|-----------|
| `/` | Home: hero, featured content, programs, impact, news | Public | ISR |
| `/about`, `/about/governance`, `/about/policies` | Organization profile, governance, published policies | Public | ISR |
| `/programs`, `/programs/[slug]` | Program list and detail (published only) | Public | ISR |
| `/projects`, `/projects/[slug]` | Project list and detail with privacy-filtered map | Public | ISR |
| `/impact` | Aggregated results from published data only | Public | ISR |
| `/news`, `/articles`, `/stories`, `/announcements` | Content type archives | Public | ISR |
| `/press`, `/press/[slug]` | Press releases and media room | Public | ISR |
| `/publications`, `/research`, `/policy-papers`, `/case-studies` | Publication archives | Public | ISR |
| `/faqs` | FAQ by category | Public | ISR |
| `/campaigns`, `/campaigns/[slug]` | Campaigns | Public | ISR |
| `/vacancies`, `/vacancies/[slug]` | Vacancies + application entry | Public (+ applicant auth for submit) | ISR |
| `/tenders`, `/tenders/[slug]` | Tenders | Public | ISR |
| `/events`, `/events/[slug]` | Events + registration | Public | ISR |
| `/contact` | Contact points and office locations | Public | Static/ISR |
| `/feedback`, `/complaints` | Public intake form (rate-limited, optional anonymity) | Public | Client |
| `/forms/[code]` | Public form where enabled by configuration | Public | Client |
| `/search` | Search over published content only | Public | Client |
| `/legal/privacy`, `/legal/terms`, `/legal/accessibility` | Legal pages | Public | Static |

Public routes cache with publish-time revalidation; only `Published` content is
reachable and unpublished slugs return `notFound()`.

## 3. Authentication — `/[locale]/(auth)/...`

| Route | Purpose | Access |
|-------|---------|--------|
| `/login` | Credential login (MFA step when enabled) | Guest |
| `/login/mfa` | Second factor challenge | Partially authenticated |
| `/forgot-password` | Request reset | Guest |
| `/reset-password` | Complete reset with token | Guest (token) |
| `/accept-invitation` | Accept an invitation and set a password | Guest (token) |
| `/logout` | Session termination (action route) | Authenticated |
| `/session-expired` | Explains expiry and offers re-login | Anyone |

Auth pages never reveal whether an account exists; errors use the API's stable
codes (`../05-api/errors.md`).

## 4. Admin — `/[locale]/(admin)/...`

| Route | Purpose | Required permission |
|-------|---------|---------------------|
| `/dashboard` | Role-aware landing: tasks, approvals, deadlines, system status | authenticated |
| `/content`, `/content/new`, `/content/[id]`, `/content/[id]/versions` | Content workspace with editor, review, versions | `content:read` / `content:create` / `content:update` |
| `/content/calendar` | Editorial calendar and assignments | `content:read` |
| `/media`, `/media/folders`, `/media/collections` | DAM browser and uploader | `media:read` / `media:upload` |
| `/documents`, `/documents/[id]` | DMS with versions and classification | `documents:read` |
| `/programs`, `/programs/[id]` | Programs | `programs:read` |
| `/projects`, `/projects/[id]`, `/projects/[id]/milestones`, `/projects/[id]/workplan` | Projects, milestones, workplan, map | `projects:read` |
| `/indicators`, `/indicators/[id]/values` | MEAL: indicators and period values | `indicators:read` |
| `/logframes`, `/assessments`, `/evaluations` | MEAL structures | `indicators:read` / `meal:read` |
| `/grants`, `/grants/[id]`, `/grants/[id]/reports` | Grants, milestones, donor reports | `grants:read` |
| `/opportunities`, `/proposals` | Funding pipeline | `grants:read` |
| `/donors`, `/partners`, `/stakeholders` | Relationships | `donors:read` / `partners:read` / `crm:read` |
| `/crm/interactions`, `/crm/tasks` | Relationship activity | `crm:read` |
| `/beneficiaries`, `/beneficiaries/[id]` | Pseudonymous list; detail needs field grant | `beneficiaries:read` |
| `/households`, `/households/[id]` | Household units | `beneficiaries:read` |
| `/cases`, `/cases/[id]` | Case workspace (reason required on access) | `cases:read` |
| `/safeguarding`, `/safeguarding/[id]` | Safeguarding register (separate audit) | `safeguarding:read` |
| `/complaints`, `/complaints/[id]` | Complaints queue | `complaints:read` |
| `/forms`, `/forms/[id]`, `/forms/[id]/builder`, `/forms/[id]/submissions` | Form builder and review | `forms:read` / `forms:update` / `forms:review` |
| `/volunteers`, `/staff`, `/memberships` | People modules | `volunteers:read` / `staff:read` / `memberships:read` |
| `/governance/boards`, `/governance/meetings`, `/governance/policies` | Governance | `governance:read` |
| `/procurement/*`, `/finance/*`, `/assets/*`, `/fleet/*`, `/travel/*` | Operations modules | respective `*:read` |
| `/tasks` | Task list and board | authenticated |
| `/events` | Event management | `events:read` |
| `/communications` | Outbound campaigns and send logs | `communications:read` |
| `/reports`, `/reports/templates`, `/reports/runs` | Reporting with progress and downloads | `reports:read` |
| `/dashboards`, `/analytics` | KPI dashboards (data-driven widgets only) | `dashboards:read` / `analytics:read` |
| `/imports`, `/imports/[id]` | Import wizard with per-row errors | `imports:run` |
| `/gis/layers`, `/gis/map` | Layer management and map workspace | `gis:read` / `gis:manage` |
| `/workflows`, `/workflows/[id]`, `/approvals` | Workflow definitions and my approvals | `workflows:read` / `approvals:decide` |
| `/automations` | Automation rules and execution log | `automations:read` |
| `/integrations` | Adapter configuration and tests | `integrations:read` |
| `/webhooks`, `/api-keys` | Developer surfaces | `webhooks:read` / `api_keys:read` |
| `/users`, `/users/invitations`, `/roles`, `/groups` | Access administration | `users:read` / `roles:read` |
| `/organization`, `/organization/units`, `/organization/positions` | Organization structure | `organization:read` |
| `/settings/*` | Tenant settings, branding, locales, notifications | `settings:read` |
| `/audit` | Audit search and export | `audit:read` |
| `/data-quality` | Quality issues and controlled merges | `data_quality:read` |
| `/ai/*` | AI registry, drafts, review queue (when enabled) | `ai:read` / `ai:review` |
| `/system/*` | Platform admin surfaces (cross-tenant, audited) | platform admin only |

Admin navigation is rendered from the caller's effective permissions; a route the
user cannot access renders the **unauthorized** state, never a blank page.

## 5. Portals — `/[locale]/(portal)/...`

| Route | Portal | Scope |
|-------|--------|-------|
| `/portal/donor/*` | Donor | Own agreements, reports, impact summaries |
| `/portal/partner/*` | Partner | Own collaboration records, documents, reports |
| `/portal/applicant/*` | Applicant / beneficiary self-service | Own submissions and status only |
| `/portal/volunteer/*` | Volunteer | Own assignments, hours, certificates |
| `/portal/member/*` | Member | Own membership status and dues |
| `/portal/board/*` | Board / governance | Read-heavy board scope |

Portals call the same `/api/v1` with narrower tokens; there is no portal-specific
backend logic (`../01-product/personas.md` §3). Every portal list is scoped to the
signed-in party's records.

## 6. Route Access Rules

```text
[ ] every (admin) route maps to at least one permission
[ ] unauthorized render is a real state with a clear next action
[ ] no route fetches data the API would deny — denial surfaces as unauthorized
[ ] public routes never render unpublished/restricted data
[ ] every portal route scopes to the signed-in party (server-enforced)
[ ] locale is preserved across navigation and hreflang alternates are emitted
```

## Related

- Frontend architecture: `architecture.md` · Components: `components.md`
- Design system and states: `design-system.md`
- Personas: `../01-product/personas.md` · Flows: `../01-product/user-flows.md`

*End of docs/06-frontend/routes.md*