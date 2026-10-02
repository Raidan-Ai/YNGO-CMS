# YNGO-CMS Blueprint

## YemenNGO-CMS

**Open, Modular, API-First Digital Platform for Civil Society Organizations**

> **Content · Programs · People · Impact**

---

## 1. Executive Summary

YNGO-CMS (YemenNGO-CMS) is a modular, multi-tenant, Arabic-first, API-first digital platform designed for civil society organizations, NGOs, non-profits, community organizations, development organizations, humanitarian actors, and social-impact institutions.

The platform combines a professional Content Management System with organization and program operations, grants, projects, MEAL, stakeholders, beneficiaries, documents, workflows, reporting, GIS, integrations, automation, and optional AI capabilities.

YNGO-CMS is intentionally designed **not** to become a monolithic ERP. The platform is built around a small, stable Core and optional modules that can be enabled, disabled, upgraded, or replaced independently.

### Primary product principles

1. **API-first** — every important capability is available through a versioned API.
2. **Modular** — functionality is delivered as independent modules/plugins.
3. **Multi-tenant** — one platform can host multiple organizations with strong tenant isolation.
4. **Arabic-first** — Arabic RTL is a first-class experience; English and additional languages are supported.
5. **Headless-capable** — public websites, portals, mobile apps, and external systems can consume the same APIs.
6. **Configurable** — custom fields, workflows, forms, permissions, themes, modules, dashboards, and content blocks can be configured without code where practical.
7. **Secure by design** — RBAC, ABAC, MFA, audit logging, data protection, backups, and least privilege are foundational capabilities.
8. **Human-controlled AI** — AI assists users but does not silently convert AI output into authoritative organizational records.
9. **Self-hostable** — the reference deployment supports Linux servers and Docker Compose, with a path to larger infrastructure later.
10. **Open integration model** — external services can be connected through REST, webhooks, OAuth/OIDC, and provider adapters.

---

# 2. Product Vision

## Vision

Build the digital operating layer for civil society organizations, with a content platform at its center and extensible operational modules around it.

## Mission

Make it possible for an NGO to manage its public presence, internal information, projects, people, partners, programs, evidence, documents, workflows, and reporting from one coherent platform without sacrificing openness or interoperability.

## Positioning

YNGO-CMS should be understood as:

> **A Digital Operating Platform for NGOs, powered by a modular CMS and an API-first architecture.**

It is not intended to be merely:

- a WordPress clone;
- a website builder only;
- a donor CRM only;
- an ERP replacement on day one;
- a data-collection app only.

---

# 3. Target Organizations

YNGO-CMS should support:

- Local NGOs
- International NGOs
- Non-profit organizations
- Civil society organizations
- Community-based organizations
- Associations and foundations
- Social enterprises with non-profit programs
- Research and advocacy organizations
- Humanitarian and development organizations
- Media/civil society initiatives
- Networks and coalitions
- Multi-branch organizations

The platform should remain useful for both small organizations and larger multi-program organizations.

---

# 4. Core Product Model

```text
                         YNGO-CMS
                            |
              +-------------+-------------+
              |                           |
          CMS / Content              NGO Operations
              |                           |
      Pages / News / Media       Programs / Projects / Grants
      Publications / Events      MEAL / People / Partners
              |                  Documents / Reporting / GIS
              +-------------+-------------+
                            |
                    Platform Services
                            |
        Auth / RBAC / ABAC / Workflow / Files / Search
        Notifications / Automation / Audit / API / Webhooks
                            |
                         AI Layer
                            |
                    External Integrations
```

---

# 5. Architecture Principles

## 5.1 Core vs Modules

The Core must remain small and stable. It owns cross-cutting concerns such as:

- Tenancy
- Identity
- Authorization
- Configuration
- Audit
- Files
- API conventions
- Events
- Workflow primitives
- Notifications
- Search interfaces

Operational and business-domain capabilities belong in modules.

## 5.2 Source of truth

PostgreSQL is the primary transactional source of truth.

Object storage is the source of truth for large binary assets.

Search indexes, caches, analytics projections, and derived materializations must be rebuildable from canonical data wherever practical.

## 5.3 Configuration over forks

Organizations should customize behavior through configuration rather than maintaining separate code forks.

## 5.4 Explicit boundaries

Each module should have:

- Domain entities
- Service layer
- API layer
- Events
- Permission scopes
- Database migrations
- Tests
- Documentation

## 5.5 Upgradeability

Module versioning and database migration discipline must allow upgrades without rewriting tenant data manually.

---

# 6. Reference Technology Stack

## Frontend

Recommended reference stack:

- Next.js
- React
- TypeScript
- Tailwind CSS
- shadcn/ui
- Radix UI
- TanStack Query
- Zustand
- React Hook Form
- Zod
- MapLibre GL / compatible mapping layer

### Frontend requirements

- Arabic-first RTL
- English LTR
- Responsive desktop/tablet/mobile
- Accessible UI
- Keyboard navigation
- Theme variables
- Dark/light modes
- Design token system
- Tenant-level branding
- Public site + Admin console + optional portals

## Backend

Recommended reference stack:

- Python
- FastAPI
- Pydantic
- SQLAlchemy
- Alembic

An equivalent TypeScript backend may be supported, but the reference implementation should use one primary stack to avoid duplicate architectural patterns.

## Database

- PostgreSQL
- PostGIS
- JSONB
- PostgreSQL full-text capabilities initially

## Object Storage

S3-compatible storage such as:

- MinIO
- Azure Blob Storage via adapter
- AWS S3 via adapter

## Cache / background jobs

- Redis for caching, short-lived state, rate limiting, and queue support where appropriate.
- Background jobs through a dedicated job framework.
- Temporal or another durable workflow engine may be introduced when workflow requirements justify it.

## Reverse proxy / TLS

- Caddy, Traefik, or Nginx

## Observability

- OpenTelemetry
- Prometheus-compatible metrics
- Grafana
- Sentry or equivalent error monitoring

## Deployment

### Initial reference deployment

```text
Ubuntu Linux
  |
Docker Compose
  |
+-------------------------------+
| Reverse Proxy                 |
| Web Frontend                  |
| API Backend                   |
| Worker                        |
| PostgreSQL + PostGIS          |
| Redis                         |
| Object Storage                |
| Optional Search               |
+-------------------------------+
```

Kubernetes is explicitly not required for the initial release.

---

# 7. Logical Architecture

```text
+--------------------------------------------------------------+
|                       Presentation Layer                     |
|--------------------------------------------------------------|
| Public Website | Admin | Partner Portal | Applicant Portal  |
| Field PWA      | Mobile App | Developer Portal              |
+--------------------------------------------------------------+
                         |
                         v
+--------------------------------------------------------------+
|                         API Layer                            |
|--------------------------------------------------------------|
| REST v1 | OpenAPI | Auth | Rate Limit | API Keys | Webhooks |
+--------------------------------------------------------------+
                         |
                         v
+--------------------------------------------------------------+
|                       Application Layer                      |
|--------------------------------------------------------------|
| Core Services | Domain Modules | Workflow | Notifications   |
| Automation    | Search        | Reporting | Integrations   |
+--------------------------------------------------------------+
                         |
                         v
+--------------------------------------------------------------+
|                         Data Layer                           |
|--------------------------------------------------------------|
| PostgreSQL/PostGIS | Object Storage | Redis | Search Index |
+--------------------------------------------------------------+
```

---

# 8. Multi-Tenancy Model

YNGO-CMS is multi-tenant by design.

```text
Platform
|
+-- Tenant / Organization A
|    +-- Users
|    +-- Content
|    +-- Projects
|    +-- Programs
|    +-- Files
|    +-- Data
|
+-- Tenant / Organization B
|
+-- Tenant / Organization C
```

## Tenant isolation requirements

- Every tenant-owned entity carries a tenant boundary.
- Authorization is evaluated within tenant context.
- Cross-tenant access is disabled by default.
- System administrators require explicit privileged access for support operations.
- Tenant data export must be possible.
- Tenant deletion/retention policies must be defined.
- Database-level protections may be introduced, including PostgreSQL Row Level Security, where operationally justified.

---

# 9. Core Modules

## 9.1 Identity & Access Management

### Features

- Users
- Organizations
- Departments
- Teams
- Roles
- Permissions
- User groups
- MFA
- Session management
- Password policies
- OAuth/OIDC
- SSO
- API identities
- Service accounts
- Login history
- Device/session revocation

### Built-in roles

- Super Admin
- Organization Admin
- Director
- Program Manager
- Project Manager
- MEAL Officer
- Finance Officer
- HR Officer
- Communications Officer
- Editor
- Author
- Reviewer
- Field Officer
- Volunteer
- External Partner
- Public User

Roles are templates, not hard-coded permission ceilings.

---

# 10. Authorization Model

## RBAC

Permissions should be expressed as:

```text
resource:action
```

Examples:

```text
content:read
content:create
content:update
content:publish
projects:read
projects:update
grants:approve
reports:export
beneficiaries:read
```

## ABAC

Policies may additionally depend on attributes such as:

- Tenant
- Organization branch
- Department
- Project
- Program
- Location
- Data classification
- User role
- Ownership

Example rule:

> A project officer can update records only for projects assigned to their department.

---

# 11. CMS Module

## Content types

Core content types:

- Pages
- Posts
- News
- Announcements
- Press Releases
- Stories
- Success Stories
- Reports
- Publications
- Research
- Policy Papers
- Case Studies
- Events
- Campaigns
- Jobs
- Tenders
- Vacancies
- FAQs
- Landing Pages

## Content lifecycle

```text
Draft
  |
  v
Review
  |
  v
Approved
  |
  v
Scheduled
  |
  v
Published
  |
  v
Archived
```

## Content capabilities

- Drafting
- Versioning
- Preview
- Scheduled publishing
- Expiration
- Revisions
- Content ownership
- Approvals
- Categories
- Tags
- Authors
- SEO metadata
- Social metadata
- Canonical URLs
- Structured metadata
- Redirect management

---

# 12. Headless Content Model

All public content must be consumable over API.

Example:

```text
GET /api/v1/content/pages/home
GET /api/v1/posts?status=published
GET /api/v1/projects?status=active
```

The public website must not require direct database access.

---

# 13. Block/Page Builder

Reusable components:

- Hero
- Rich text
- Image
- Gallery
- Video
- Audio
- Cards
- Statistics
- Timeline
- Map
- Project list
- Team
- Partners
- Donors
- Documents
- FAQ
- Testimonials
- Newsletter form
- Contact block
- CTA
- Custom embed

Blocks should support responsive settings and reusable instances.

---

# 14. Media & Digital Asset Management

Supported assets:

- Images
- Video
- Audio
- PDF
- DOCX
- XLSX
- PPTX
- CSV
- ZIP and permitted archives

### Asset metadata

- File name
- MIME type
- Size
- Checksum
- Owner
- Created date
- Tags
- Collections
- Copyright
- License
- Usage restrictions
- Captions
- Alternative text
- EXIF metadata where available

### Requirements

- Resumable upload for large files
- Virus/malware scanning adapter
- Image derivatives
- Video thumbnail extraction
- Access control
- Versioning
- Object retention policies

---

# 15. Document Management System

Features:

- Folders and collections
- Document classification
- Version history
- Approval workflow
- Document expiration
- Ownership
- Permissions
- Sharing
- Download history
- Audit trail
- Retention policy
- Search metadata

Document entity model:

```text
Document
 |
 +-- Metadata
 +-- Versions
 +-- Permissions
 +-- Workflow
 +-- Related Project
 +-- Related Program
 +-- Related Organization
```

---

# 16. Programs Module

A Program groups related projects and interventions.

### Program attributes

- Program ID
- Name
- Description
- Sector
- Objectives
- Geographic scope
- Target population
- Funding sources
- Partners
- Projects
- KPIs
- Dates
- Status

---

# 17. Projects Module

### Project fields

- Project ID
- Title
- Description
- Objective
- Start date
- End date
- Status
- Budget
- Currency
- Donor
- Partner
- Manager
- Program
- Locations
- Target groups
- Activities
- Outputs
- Outcomes
- Indicators
- Risks
- Documents
- Reports

### Project lifecycle

```text
Idea
 -> Proposal
 -> Submitted
 -> Approved
 -> Funded
 -> Planning
 -> Implementation
 -> Monitoring
 -> Reporting
 -> Completed
 -> Archived
```

---

# 18. Activities Module

Each project can contain:

- Activities
- Tasks
- Milestones
- Outputs
- Deliverables
- Responsible teams
- Dates
- Locations
- Attachments
- Activity status

---

# 19. Grants Module

### Grant pipeline

```text
Opportunity
 -> Application
 -> Proposal
 -> Review
 -> Approval
 -> Award
 -> Agreement
 -> Implementation
 -> Reporting
 -> Closure
```

### Grant entities

- Donor
- Funding Opportunity
- Application
- Proposal
- Grant Award
- Agreement
- Amendment
- Grant Report
- Grant Closure

---

# 20. Donor Management

Each donor can have:

- Organization profile
- Contacts
- Funding history
- Grants
- Requirements
- Reporting schedule
- Agreements
- Communications
- Related projects

---

# 21. Partner Management

Partner categories:

- NGOs
- INGOs
- Government entities
- UN agencies
- Community organizations
- Academic institutions
- Private sector
- Networks
- Coalitions

Capabilities:

- Partner profile
- Contacts
- MoUs
- Agreements
- Due diligence
- Projects
- Performance
- Documents
- Communications

---

# 22. Beneficiary Management

This module is optional and must be protected by strict privacy controls.

Capabilities:

- Individual beneficiaries
- Households
- Groups
- Service enrollment
- Program participation
- Assistance history
- Location
- Referral status
- Case linkage
- Beneficiary status

### Privacy requirements

- Data minimization
- Field-level access where necessary
- Sensitive-data classification
- Audit logging
- Export controls
- Masking/redaction where required
- Configurable retention

Public dashboards must never expose personally identifying beneficiary information by default.

---

# 23. Case Management

Case workflow:

```text
Opened
 -> Assessment
 -> Referral
 -> Service
 -> Follow-up
 -> Resolved / Closed
```

Case components:

- Case profile
- Case notes
- Tasks
- Referrals
- Appointments
- Attachments
- Activity log
- Assigned case workers
- Status history

---

# 24. MEAL Module

**Monitoring, Evaluation, Accountability and Learning**

### Results framework

```text
Impact
  |
Outcome
  |
Output
  |
Activity
```

### Indicator structure

- Indicator ID
- Name
- Definition
- Unit
- Baseline
- Target
- Actual
- Frequency
- Collection method
- Data source
- Responsible owner
- Verification method

### Features

- Indicator tracking
- Baselines
- Targets
- Actuals
- Variance analysis
- Surveys
- Monitoring visits
- Evaluations
- Lessons learned
- Accountability mechanisms

---

# 25. Forms Builder

Supported field types:

- Text
- Long text
- Number
- Email
- Phone
- Date
- Date/time
- Select
- Multi-select
- Radio
- Checkbox
- File upload
- Signature
- Rating
- Matrix
- GPS/location

Advanced:

- Conditional logic
- Validation rules
- Required fields
- Calculated fields
- Dynamic lists
- Multi-page forms
- Save and continue
- Draft submissions
- Offline support for field forms
- Submission workflows
- API submission
- CSV/Excel export

---

# 26. Volunteer Module

Manage:

- Volunteer profiles
- Skills
- Availability
- Applications
- Assignments
- Attendance
- Hours
- Certificates
- Activities
- Training

---

# 27. HR Lite Module

YNGO-CMS should provide practical organization records without attempting to replace full payroll/ERP systems.

Features:

- Employee profiles
- Positions
- Departments
- Contracts
- Documents
- Certifications
- Leave records
- Attendance integration
- Performance notes
- Organization structure

Payroll should generally be provided through integration rather than embedded in Core.

---

# 28. Events Module

Event types:

- Workshop
- Training
- Conference
- Webinar
- Meeting
- Campaign event
- Community event

Features:

- Registration
- Attendance
- Speakers
- Agenda
- Venue
- Calendar
- Certificates
- Event media
- Event reports

---

# 29. Communications Module

Internal:

- Notifications
- Announcements
- Mentions
- Comments
- Activity feed

External integrations may include:

- Email
- SMS
- WhatsApp provider
- Telegram
- Push notifications

Provider adapters must be replaceable.

---

# 30. Newsletter Module

Features:

- Subscribers
- Lists
- Segments
- Templates
- Campaigns
- Scheduling
- Delivery provider integration
- Open/click analytics where supported
- Unsubscribe management

---

# 31. CRM / Stakeholder Module

Entity types:

- Donors
- Partners
- Supporters
- Members
- Volunteers
- Government contacts
- Media contacts
- Stakeholders

Track:

- Interactions
- Meetings
- Calls
- Emails
- Notes
- Tasks
- Relationship history

---

# 32. Applicant Portal

Reusable application pipeline for:

- Jobs
- Grants
- Volunteer opportunities
- Training
- Fellowships
- Tenders where appropriate

Pipeline:

```text
Account
 -> Application
 -> Documents
 -> Review
 -> Interview / Evaluation
 -> Decision
```

---

# 33. Public Transparency Portal

Organizations may expose selected public data:

- Active projects
- Program areas
- Funding information
- Donors
- Public reports
- Locations
- Impact indicators
- Publications
- Events

Every transparency dataset requires an explicit publication setting.

---

# 34. GIS Module

Use PostGIS as the authoritative spatial layer.

Supported objects:

- Country
- Governorate
- District
- Sub-district
- Community
- Project location
- Intervention area
- Event location

Map layers may show:

- Projects
- Activities
- Service locations
- Indicators
- Program coverage

Sensitive personal locations must be protected or aggregated.

---

# 35. Reporting Module

Report types:

- Project report
- Program report
- Donor report
- Annual report
- Impact report
- MEAL report
- Communications report
- Financial summary

Exports:

- PDF
- DOCX
- XLSX
- CSV
- HTML

The reporting engine should support reusable templates and data bindings.

---

# 36. Dashboard Builder

Dashboard widgets:

- KPI cards
- Charts
- Tables
- Maps
- Progress indicators
- Timelines
- Activity feeds
- Alerts

Dashboards can be scoped to:

- Organization
- Branch
- Department
- Program
- Project
- Donor

---

# 37. Workflow Engine

Workflow model:

```text
Trigger
  |
Condition
  |
Action
  |
Next State
```

Supported actions should include:

- Assign user
- Change status
- Create task
- Send notification
- Send email
- Request approval
- Generate document
- Call webhook
- Invoke integration

Example:

```text
New Project
 -> Create default tasks
 -> Notify project manager
 -> Create reporting schedule
 -> Request approval
```

---

# 38. Automation Engine

Automation is a separate layer above workflows.

Example:

```text
IF grant.deadline <= 7 days
AND grant.status = active
THEN notify grant manager
```

The automation engine must include:

- Triggers
- Conditions
- Actions
- Retry policy
- Error handling
- Execution log
- Disable/enable controls

---

# 39. Task Management

Features:

- Tasks
- Subtasks
- Assignees
- Teams
- Deadlines
- Priorities
- Status
- Labels
- Dependencies
- Comments
- Attachments
- Activity history

---

# 40. Calendar

Unified calendar surfaces:

- Events
- Meetings
- Tasks
- Milestones
- Grant deadlines
- Report deadlines
- Training sessions
- Campaign activities

Support iCal/CalDAV/Google/Microsoft integrations via adapters where applicable.

---

# 41. Search Platform

### Initial search

Use PostgreSQL search capabilities for the first release.

### Scaled search

Add OpenSearch or another search engine when:

- Dataset volume requires it;
- advanced faceting is needed;
- cross-document indexing becomes important;
- semantic search needs dedicated infrastructure.

Global search should cover:

- Content
- Documents
- Projects
- Programs
- Donors
- Partners
- Events
- Reports
- Knowledge base

Filters should include:

- Tenant
- Type
- Status
- Date
- Location
- Program
- Project
- Author/owner
- Tags

---

# 42. Knowledge Base

The Knowledge Base stores institutional knowledge:

- Policies
- Procedures
- Manuals
- Guidelines
- FAQs
- Lessons learned
- Research
- Internal knowledge articles

Potential future AI functions:

- Semantic retrieval
- Summarization
- Question answering
- Similarity search
- Document classification

Access must respect the same organization and authorization rules as the main system.

---

# 43. AI Layer

AI is an optional platform capability, not a prerequisite for running YNGO-CMS.

## Content Assistant

- Drafting
- Rewriting
- Summarization
- Translation
- SEO assistance
- Metadata extraction
- Tag suggestions
- Classification

## Knowledge Assistant

Users may ask questions against authorized organizational data.

Example:

> Which active projects operate in Taiz and what are their reporting deadlines?

The assistant should retrieve authorized records and cite the underlying records when possible.

## AI governance

```text
AI Suggestion
     |
     v
Human Review
     |
     v
Approval
     |
     v
Canonical Record
```

AI output should not silently overwrite authoritative organizational records.

---

# 44. API-First Contract

Base path:

```text
/api/v1/
```

Core API groups:

```text
/api/v1/auth
/api/v1/users
/api/v1/organizations
/api/v1/departments
/api/v1/roles
/api/v1/permissions
/api/v1/content
/api/v1/pages
/api/v1/posts
/api/v1/media
/api/v1/documents
/api/v1/programs
/api/v1/projects
/api/v1/activities
/api/v1/grants
/api/v1/donors
/api/v1/partners
/api/v1/stakeholders
/api/v1/beneficiaries
/api/v1/cases
/api/v1/events
/api/v1/forms
/api/v1/submissions
/api/v1/tasks
/api/v1/workflows
/api/v1/notifications
/api/v1/reports
/api/v1/dashboards
/api/v1/search
/api/v1/integrations
/api/v1/webhooks
/api/v1/audit
```

---

# 45. API Requirements

The API must provide:

- OpenAPI specification
- JSON request/response conventions
- Pagination
- Filtering
- Sorting
- Field selection where appropriate
- Validation errors
- Stable error codes
- Idempotency for sensitive operations
- Rate limiting
- Authentication and scopes
- API versioning
- Audit context

### Example error format

```json
{
  "error": {
    "code": "PROJECT_NOT_FOUND",
    "message": "Project was not found",
    "request_id": "req_123"
  }
}
```

---

# 46. Authentication for APIs

Support:

- OAuth2
- OpenID Connect
- JWT
- API keys
- Service accounts

API keys must have scopes and optional restrictions.

Example scopes:

```text
content:read
content:write
projects:read
projects:write
reports:read
reports:export
```

---

# 47. Webhooks

Example domain events:

```text
organization.created
user.created
content.published
content.updated
project.created
project.updated
project.completed
grant.approved
grant.deadline_approaching
form.submitted
report.submitted
workflow.completed
```

Webhook requirements:

- Signing secret
- Retry
- Backoff
- Delivery history
- Disable/enable
- Manual replay
- Failure visibility

---

# 48. Integrations Framework

Adapters should be supported for:

- Microsoft 365
- Google Workspace
- Email providers
- SMS providers
- WhatsApp providers
- Telegram
- Slack
- Microsoft Teams
- Accounting systems
- Identity providers
- Cloud storage
- GIS providers
- AI model providers

The integration layer should isolate vendor-specific code from domain logic.

---

# 49. Custom Fields

Any configurable entity may support custom fields.

Examples:

```text
Project
  + Donor Reference
  + Funding Code
  + Intervention Type

Partner
  + Partner Classification
  + Registration Number
```

Supported field primitives should include:

- Text
- Number
- Boolean
- Date
- DateTime
- Select
- Multi-select
- User reference
- Organization reference
- Entity reference
- File
- Location
- JSON where explicitly allowed

---

# 50. Custom Modules

Organizations may create lightweight custom modules without altering Core source code.

Example:

```text
Module: Community Complaints

Fields:
- Complaint ID
- Category
- Location
- Description
- Priority
- Status
- Attachment

Workflow:
Submitted -> Review -> Investigation -> Resolved
```

Custom modules should generate:

- Schema metadata
- Forms
- Permissions
- CRUD APIs
- Audit events
- Basic views
- Workflow hooks

Strict constraints should prevent custom modules from bypassing security boundaries.

---

# 51. Theming & White Label

Every tenant can configure:

- Logo
- Favicon
- Brand colors
- Fonts
- Button style
- Layout options
- Header
- Footer
- Navigation
- Email templates
- Public-domain configuration

White-label support should allow the public interface to hide YNGO-CMS branding.

---

# 52. Localization

Localization engine must support:

- Arabic
- English
- Additional languages later

Features:

- RTL/LTR
- Localized content
- Translation workflow
- Locale-aware dates and numbers
- Translation completeness reports
- Localized URLs
- Language fallback

The platform should not hard-code Arabic-only UI labels even though Arabic is the primary launch language.

---

# 53. Notifications Architecture

Channels:

```text
In-App
Email
SMS
Push
Webhook
```

Notification model:

```text
Event
 -> Rule
 -> Recipient resolution
 -> Template
 -> Provider
 -> Delivery log
```

---

# 54. Audit Logging

Every important mutation should produce an audit record.

Suggested fields:

```text
id
tenant_id
actor_user_id
action
resource_type
resource_id
old_value
new_value
ip_address
user_agent
request_id
created_at
```

Audit logs must be append-oriented and protected against ordinary users modifying historical records.

---

# 55. Data Model Overview

Core entities:

```text
Organization
User
Role
Permission
Department
Team
Branch

Content
Page
Post
MediaAsset
Document

Program
Project
Activity
Indicator
Milestone

Donor
Grant
FundingOpportunity
Proposal
Agreement

Partner
Stakeholder

Beneficiary
Household
Case
Service
Referral

Event
Form
FormSubmission

Task
Workflow
WorkflowRun
Notification

Report
Dashboard

Integration
ApiKey
Webhook

AuditLog
CustomField
CustomModule
```

---

# 56. Entity Relationship Overview

```text
Organization
 |
 +-- Users
 +-- Departments
 +-- Branches
 +-- Programs
 |     +-- Projects
 |           +-- Activities
 |           +-- Indicators
 |           +-- Reports
 |
 +-- Donors
 |     +-- Grants
 |           +-- Projects
 |
 +-- Partners
 |     +-- Projects
 |
 +-- Beneficiaries
 |     +-- Cases
 |
 +-- Content
 +-- Documents
 +-- Events
 +-- Forms
 +-- Tasks
```

---

# 57. Data Governance

The platform must distinguish between:

- user-generated content;
- operational records;
- published public content;
- confidential information;
- sensitive personal information;
- system metadata;
- audit records;
- derived analytics.

Recommended metadata on important entities:

```text
created_at
created_by
updated_at
updated_by
status
version
tenant_id
```

---

# 58. Security Architecture

Mandatory baseline:

- TLS everywhere in deployed environments
- Secure password hashing
- MFA support
- Secure session handling
- CSRF protection
- XSS mitigation
- SQL injection protection
- Input validation
- Output encoding
- Secure file validation
- Upload size limits
- Rate limiting
- Secrets management
- Least privilege
- Audit logging
- Security headers
- Backup encryption where possible

Optional/advanced:

- WAF
- SSO
- IP restrictions
- Device policies
- Field-level encryption
- Database encryption at rest

---

# 59. Privacy & Data Protection

Privacy requirements should be configurable to the organization and operating environment.

Capabilities:

- Data classification
- Consent records where required
- Retention policies
- Redaction
- Export requests
- Deletion workflows where legally/operationally appropriate
- Access reviews
- Sensitive field permissions

Privacy-sensitive modules such as beneficiaries and cases must have stricter defaults than ordinary content.

---

# 60. File Security

File uploads must support:

- MIME validation
- Extension validation
- Filename normalization
- Maximum size limits
- Virus/malware scanning adapter
- Quarantine state
- Safe preview rules
- Signed URLs for protected objects
- Access logs for sensitive documents

---

# 61. Backup & Disaster Recovery

Reference policy:

```text
Daily database backup
Weekly full backup
Object versioning
Off-site copy
Restore verification
Periodic disaster-recovery test
```

The system should expose backup health and last-success indicators to administrators.

---

# 62. Observability

The application should expose:

- Health endpoint
- Readiness endpoint
- Liveness endpoint
- Application logs
- Structured JSON logs
- Metrics
- Error tracking
- Background job metrics
- API latency
- Database health
- Storage health

Every request should have a trace/request identifier.

---

# 63. Administration Console

Suggested navigation:

```text
DASHBOARD

CONTENT
  Pages
  Posts
  News
  Media
  Documents

PROGRAMS
  Programs
  Projects
  Activities
  Indicators

FUNDING
  Donors
  Opportunities
  Grants
  Proposals

PEOPLE
  Staff
  Volunteers
  Beneficiaries
  Cases

PARTNERS
  Partners
  Stakeholders

OPERATIONS
  Tasks
  Calendar
  Events
  Workflows
  Automations

REPORTING
  Reports
  Dashboards
  Analytics

SYSTEM
  Users
  Roles
  Settings
  Integrations
  API
  Webhooks
  Audit Logs
```

Navigation should change according to enabled modules and user permissions.

---

# 64. Public Website Architecture

A tenant public website can expose:

```text
Home
About
Programs
Projects
Impact
News
Stories
Publications
Reports
Events
Vacancies
Tenders
Partners
Donors
Contact
```

It must use the same content API as other clients.

---

# 65. Field / Offline Architecture

A future field application should support offline operation:

```text
Field Device
   |
Local storage
   |
Offline forms / tasks / observations
   |
Connectivity restored
   |
Sync engine
   |
Conflict handling
   |
YNGO API
```

Requirements:

- Offline forms
- Queued uploads
- Retry
- Local encryption where needed
- Sync status
- Conflict resolution
- GPS capture

---

# 66. API Consumers

Potential clients:

- YNGO web application
- Public NGO websites
- Mobile applications
- Field applications
- BI systems
- Donor reporting systems
- GIS dashboards
- AI assistants
- External partner systems
- Government/open-data portals where appropriate

---

# 67. Plugin Architecture

Plugin structure:

```text
Plugin
 |
 +-- Manifest
 +-- Version
 +-- Permissions
 +-- Database migrations
 +-- Backend module
 +-- API routes
 +-- Frontend pages
 +-- UI components
 +-- Events
 +-- Documentation
```

Plugins should declare dependencies and compatibility ranges.

---

# 68. Suggested Plugin Modules

Potential first-party plugins:

- MEAL
- GIS
- Grants
- CRM
- Volunteer
- Beneficiary
- Case Management
- Procurement
- HR Lite
- AI Assistant
- Newsletter
- Social Publishing
- Advanced Search
- Advanced Reporting

---

# 69. Developer Platform

YNGO-CMS should provide a developer portal containing:

- OpenAPI
- Authentication guide
- API reference
- Webhook documentation
- SDK examples
- Integration examples
- Sandbox instructions
- Rate limits
- Error codes
- Changelog

SDK targets can include:

- TypeScript
- Python

Additional SDKs may be generated later from the OpenAPI contract.

---

# 70. Coding Standards

Backend:

- Type hints
- Pydantic schemas
- Service/domain boundaries
- Repository patterns only where they add value
- Explicit transactions
- Structured logging
- Automated migrations
- Unit/integration tests

Frontend:

- TypeScript strict mode
- Typed API client
- Reusable UI primitives
- Form validation with Zod
- Server/client boundary discipline
- Accessible components
- RTL testing

---

# 71. Repository Structure

Recommended monorepo structure:

```text
yngo-cms/
|
+-- apps/
|   +-- web/
|   +-- api/
|   +-- worker/
|   +-- field/
|   +-- docs/
|   +-- developer-portal/
|
+-- packages/
|   +-- ui/
|   +-- api-client/
|   +-- auth/
|   +-- config/
|   +-- i18n/
|   +-- types/
|   +-- workflow/
|
+-- modules/
|   +-- cms/
|   +-- programs/
|   +-- projects/
|   +-- grants/
|   +-- meal/
|   +-- partners/
|   +-- beneficiaries/
|   +-- gis/
|   +-- reporting/
|   +-- crm/
|   +-- ai/
|
+-- infra/
|   +-- docker/
|   +-- compose/
|   +-- migrations/
|   +-- monitoring/
|
+-- docs/
+-- scripts/
+-- tests/
```

An alternative modular monolith structure may be used initially if it keeps boundaries explicit.

---

# 72. Recommended Initial Architecture: Modular Monolith

Do **not** begin with microservices.

Use one deployable backend with explicit domain modules.

```text
FastAPI App
|
+-- core
+-- auth
+-- cms
+-- media
+-- organizations
+-- programs
+-- projects
+-- grants
+-- forms
+-- reporting
+-- workflows
+-- integrations
```

Why:

- easier deployment;
- easier local development;
- lower operational cost;
- easier transactions;
- simpler debugging;
- fewer distributed-system failure modes.

Modules can later be extracted into services if scale demands it.

---

# 73. API Design Rules

1. URLs use plural resource names.
2. API versions are explicit.
3. Mutations are idempotent where appropriate.
4. Authorization occurs server-side.
5. Tenant context is never trusted from a client-supplied arbitrary identifier alone.
6. Validation occurs at boundaries.
7. Public and private APIs are separated through permissions, not duplicate business logic.
8. Deprecated endpoints have a documented migration period.

---

# 74. Content API Example

```http
GET /api/v1/pages/home
```

Response concept:

```json
{
  "id": "page_home",
  "slug": "home",
  "locale": "ar",
  "status": "published",
  "title": "الرئيسية",
  "blocks": [
    {
      "type": "hero",
      "data": {
        "title": "...",
        "image": "..."
      }
    }
  ]
}
```

---

# 75. Project API Example

```http
GET /api/v1/projects?status=active&location=taiz
```

Potential response fields:

```json
{
  "items": [
    {
      "id": "project_001",
      "name": "...",
      "status": "active",
      "program_id": "program_001",
      "donor_id": "donor_001",
      "start_date": "2026-01-01",
      "end_date": "2026-12-31"
    }
  ],
  "page": 1,
  "page_size": 20,
  "total": 1
}
```

---

# 76. Search API Example

```http
GET /api/v1/search?q=education&type=project,document
```

Response should provide:

- Result type
- Entity ID
- Title
- Highlighted match
- URL/deep link
- Metadata
- Access-aware filtering

---

# 77. Tenant Settings

Tenant administrators should be able to configure:

- Organization identity
- Branding
- Locales
- Time zone
- Fiscal year
- Default currency
- Content settings
- Privacy settings
- Notification settings
- Email provider
- Storage provider
- Domain settings
- API settings
- Enabled modules
- Workflow defaults

---

# 78. Custom Taxonomies

Organizations should be able to create controlled vocabularies for:

- Project sectors
- Program categories
- Content topics
- Regions
- Stakeholder types
- Document classifications
- Priority levels

Taxonomies should support localization.

---

# 79. Data Import / Export

Import:

- CSV
- XLSX
- JSON
- API

Export:

- CSV
- XLSX
- JSON
- PDF/DOCX for reports

Import flows should support:

- Field mapping
- Preview
- Validation
- Error report
- Dry run
- Duplicate detection
- Rollback where feasible

---

# 80. Bulk Operations

Administrators should be able to:

- Bulk publish
- Bulk archive
- Bulk tag
- Bulk assign
- Bulk export
- Bulk update status
- Bulk move between categories

Sensitive entities should have stricter bulk-operation permissions.

---

# 81. Approval System

Generic approval primitives should work across modules.

```text
Draft
 -> Submitted
 -> Under Review
 -> Approved / Rejected
```

Approvals should support:

- One approver
- Multiple approvers
- Sequential approval
- Parallel approval
- Delegation
- Rejection reason
- Approval history

---

# 82. Data Quality

Quality controls should include:

- Required fields
- Validation rules
- Duplicate detection
- Referential integrity
- Status constraints
- Numeric validation
- Date consistency
- Indicator validation
- Import validation

Future advanced controls may include anomaly detection.

---

# 83. Versioning Strategy

For content and configuration:

- Draft versions
- Published version
- Revision history
- Restore
- Compare changes

For APIs:

```text
/api/v1/
/api/v2/
```

Breaking changes require a new major API version.

---

# 84. Testing Strategy

Required test categories:

### Unit

Domain rules and utility functions.

### Integration

Database, storage, authentication, API, workflow.

### Contract tests

API request/response compatibility.

### End-to-end

Critical user journeys.

### Security tests

- Authorization bypass
- Tenant isolation
- File upload attacks
- Input validation
- Rate limits

### Accessibility

Automated and manual RTL/LTR accessibility checks.

---

# 85. Performance Targets

Initial goals should be defined as measurable SLOs rather than absolute assumptions.

Track:

- API latency p50/p95
- Page load time
- Time to interactive
- Search latency
- Upload throughput
- Job execution time
- Database query latency

Performance work should be driven by actual measurements.

---

# 86. Scalability Path

### Stage 1

Single Ubuntu server + Docker Compose.

### Stage 2

Separate database/storage and add managed/object infrastructure.

### Stage 3

Read replicas, dedicated search, dedicated workers.

### Stage 4

Extract high-load modules/services where justified.

### Stage 5

Optional Kubernetes or managed container platform.

The architecture should not require Stage 5 to operate correctly.

---

# 87. Deployment Profiles

## Development

```text
Local Docker Compose
SQLite may be used for limited tooling only
PostgreSQL preferred for real application development
```

## Staging

```text
Linux
Docker Compose
PostgreSQL
Object Storage
Redis
TLS
Monitoring
```

## Production

```text
Reverse Proxy
Web
API
Worker
PostgreSQL/PostGIS
Redis
Object Storage
Monitoring
Backups
Optional Search
```

---

# 88. Environment Variables

Suggested names:

```env
APP_ENV=production
APP_URL=
API_URL=
DATABASE_URL=
REDIS_URL=
STORAGE_ENDPOINT=
STORAGE_BUCKET=
STORAGE_ACCESS_KEY=
STORAGE_SECRET_KEY=
JWT_SECRET=
OIDC_ISSUER=
SMTP_HOST=
SMTP_PORT=
SMTP_USER=
SMTP_PASSWORD=
SENTRY_DSN=
```

Secrets must be supplied through a secrets mechanism in production and must never be committed to Git.

---

# 89. Administration of API Keys

Admin UI should make credential management explicit.

```text
Developer / Integrations
        |
        +-- Create API Key
        +-- Scope
        +-- Expiry
        +-- Restrictions
        +-- Last Used
        +-- Revoke
```

Full secret values should be shown only at creation time where practical.

---

# 90. Auditability of APIs

API actions should capture:

- Actor
- Client/application
- API key or OAuth client identifier where appropriate
- Request ID
- Resource
- Action
- Timestamp
- Result

Secrets and sensitive payloads must not be written into logs.

---

# 91. AI Provider Abstraction

AI integrations should not be hard-wired to a single provider.

```text
YNGO AI Gateway
      |
 +----+-----------------------+
 |                            |
Provider A                Provider B
 |                            |
Model 1                   Model 2
```

Capabilities should be abstracted by task when possible:

- text generation
- summarization
- translation
- extraction
- embedding
- OCR
- classification

---

# 92. AI Permissions

AI assistants must inherit the current user's authorization scope.

The AI layer must never retrieve:

- a document;
- beneficiary record;
- private case;
- internal report;

unless the calling user is authorized to access that information.

---

# 93. Reporting / Analytics Architecture

Initial analytics can run from PostgreSQL read models or reporting queries.

As the platform grows:

```text
Transactional DB
      |
      +--> Reporting Views
      |
      +--> Analytics Export
      |
      +--> Data Warehouse (future)
```

Do not introduce a full data warehouse before actual reporting requirements justify it.

---

# 94. Public SEO

Public content must support:

- Meta title
- Meta description
- Canonical URL
- Open Graph
- Twitter/X cards
- Sitemap
- Robots configuration
- Structured data where applicable
- Redirects
- Localized URLs
- 404/410 handling

---

# 95. Accessibility

Target WCAG-compatible implementation.

Requirements:

- Keyboard navigation
- Visible focus
- Semantic HTML
- Screen-reader labels
- Sufficient contrast
- Form error messaging
- RTL support
- Accessible tables/charts
- Reduced-motion consideration

---

# 96. Content Governance

Content governance should distinguish roles:

```text
Author
  |
Editor
  |
Reviewer
  |
Publisher
```

Optional:

```text
Legal / Compliance Review
```

Published content should retain a clear publication history.

---

# 97. Compliance / Governance Features

The platform should provide primitives for organizational policies, not claim legal compliance automatically.

Examples:

- Policy documents
- Approval records
- Conflict-of-interest declarations
- Due diligence
- Data retention
- Audit trails
- Access review records
- Incident records

Specific regulatory mappings should be configured based on the organization and jurisdiction.

---

# 98. Incident Management

Optional module for:

- Operational incidents
- Security incidents
- Data incidents
- Program incidents
- Response actions
- Escalation
- Root-cause notes
- Closure

This module must have strong permissions.

---

# 99. Feedback & Complaints

Public-facing configurable module:

- Feedback
- Complaints
- Suggestions
- Case references
- Categories
- Severity
- Assignment
- Resolution

Can integrate with the Workflow Engine.

---

# 100. Notification Rules Examples

```text
Project ending in 30 days
 -> notify project manager

Grant report overdue
 -> notify grant manager

New form submission
 -> create review task

Document expires in 14 days
 -> notify owner

Content submitted for review
 -> notify editor
```

---

# 101. Recommended MVP

The MVP should not implement every module above.

## MVP Core

1. Multi-tenancy
2. Organizations
3. Users/Roles/Permissions
4. Authentication
5. Audit log
6. Settings
7. Media/File storage
8. REST API
9. OpenAPI documentation
10. Notifications

## MVP CMS

11. Pages
12. Posts/News
13. Media library
14. Page builder
15. Categories/Tags
16. SEO
17. Localization
18. Publishing workflow

## MVP NGO Operations

19. Programs
20. Projects
21. Donors
22. Partners
23. Documents
24. Events
25. Forms
26. Tasks
27. Basic reporting

---

# 102. Phase 2

- Grants
- MEAL
- Beneficiaries
- Case management
- GIS
- Dashboard builder
- CRM
- Workflow builder
- Automation engine
- Advanced search
- Developer portal

---

# 103. Phase 3

- Field/offline PWA
- AI assistant
- Knowledge base intelligence
- Advanced analytics
- Procurement
- HR Lite
- Advanced transparency portal
- Additional integrations
- Plugin marketplace

---

# 104. Phase 4

- Enterprise SSO
- Advanced tenancy
- Dedicated search clusters
- Data warehouse integrations
- High-scale workers
- Optional service extraction
- Marketplace ecosystem

---

# 105. Non-Goals for V1

The first version should avoid:

- Full accounting ERP
- Full payroll
- Complex procurement ERP
- Kubernetes requirement
- Microservices everywhere
- Multiple databases for every module
- Mandatory AI
- Mandatory external search cluster
- Mandatory mobile app

These can be integrations or later modules.

---

# 106. Product Success Criteria

YNGO-CMS V1 is successful when an organization can:

1. Create its organization workspace.
2. Configure branding and languages.
3. Publish a professional public website.
4. Manage pages, news, media, and documents.
5. Create programs and projects.
6. Associate donors and partners.
7. Collect information through configurable forms.
8. Manage tasks and workflows.
9. Generate operational reports.
10. Use APIs to integrate an external application.
11. Audit important changes.
12. Operate securely in a self-hosted environment.

---

# 107. Suggested Initial Screens

## Public

- Home
- About
- Programs
- Projects
- Impact
- News
- Publications
- Reports
- Events
- Careers
- Contact

## Admin

- Dashboard
- Content
- Media
- Programs
- Projects
- Donors
- Partners
- Forms
- Tasks
- Reports
- Documents
- Settings
- Users
- Roles
- API
- Integrations
- Audit

---

# 108. UX Principles

1. Arabic-first and RTL by default.
2. Avoid dense enterprise UI where a simpler flow works.
3. Surface status and next actions clearly.
4. Keep permissions understandable.
5. Make drafts and approvals visible.
6. Design mobile-friendly screens for field workflows.
7. Never hide important system warnings.
8. Use consistent entities and IDs across modules.

---

# 109. Naming Conventions

Use consistent technical names.

Examples:

```text
organization_id
tenant_id
project_id
program_id
donor_id
partner_id
created_at
updated_at
published_at
```

Public display names may be localized independently of internal identifiers.

---

# 110. Event-Driven Internal Architecture

Modules should communicate using domain events when useful.

Example:

```text
project.created
      |
      +--> notification
      +--> task provisioning
      +--> audit log
      +--> reporting projection
```

Events should be explicit, versioned, and observable.

---

# 111. Idempotency

Operations that can create duplicate business effects should support idempotency.

Examples:

- webhook processing
- payment/funding imports
- repeated form submissions
- synchronization
- email-triggered actions

---

# 112. Data Synchronization

External integrations should use:

- External IDs
- Sync timestamps
- Provider metadata
- Conflict status
- Retry state
- Last successful sync

Do not overwrite authoritative local data silently.

---

# 113. Import Safety

Every large import should support:

```text
Upload
 -> Map fields
 -> Validate
 -> Dry run
 -> Review errors
 -> Commit
```

---

# 114. Localization of Data

Localized entities may use either:

- dedicated translation records; or
- a normalized JSON/document translation layer.

The chosen approach must remain queryable and enforce unique localized slugs.

---

# 115. Public API vs Internal API

The same domain services should support both public and private API routes.

Do not duplicate business logic into separate public and admin codebases.

Public endpoints expose only explicitly published data.

---

# 116. Security Boundary for Public Content

A public request must never be able to infer or retrieve private draft content merely by knowing an object ID.

Publication state is a security boundary.

---

# 117. Rate Limiting

Apply rate limits to:

- Login
- Password reset
- Public forms
- Public search
- API keys
- Webhooks
- Upload endpoints

Limits should be configurable at:

- IP level
- User level
- Tenant level
- API-key level

---

# 118. Search Security

Search indexes must preserve authorization boundaries.

The system should filter unauthorized records **before** returning results.

Never rely on UI filtering as a security mechanism.

---

# 119. Storage Security

Private files should be delivered using short-lived signed URLs or equivalent controlled access.

Public media may use CDN-backed public URLs.

---

# 120. Documentation Requirements

Repository documentation should include:

```text
README.md
ARCHITECTURE.md
API.md
SECURITY.md
DEPLOYMENT.md
CONTRIBUTING.md
MODULES.md
DATA_MODEL.md
WORKFLOWS.md
INTEGRATIONS.md
CHANGELOG.md
```

Every module should provide its own README and domain notes.

---

# 121. Engineering Workflow

Recommended workflow:

```text
Requirement
 -> ADR / Design note
 -> Domain model
 -> API contract
 -> Migration
 -> Backend
 -> Frontend
 -> Tests
 -> Documentation
 -> Review
 -> Release
```

---

# 122. Architecture Decision Records

Maintain ADRs for major decisions such as:

- Modular monolith
- PostgreSQL/PostGIS
- Object storage
- Search architecture
- Authentication provider
- Workflow engine
- Plugin model
- API versioning
- AI provider abstraction

---

# 123. Migration Strategy

All schema changes must use versioned migrations.

Production migrations must be:

- repeatable;
- reviewed;
- reversible where feasible;
- tested against realistic datasets.

---

# 124. Seed Data

Development/staging should include optional seed data:

- Demo organization
- Roles
- Permissions
- Sample project
- Sample donor
- Sample partner
- Example pages
- Example workflows
- Example forms

Production must never use development credentials or sample secrets.

---

# 125. Feature Flags

Feature flags may be used for:

- experimental modules;
- AI features;
- beta integrations;
- tenant-specific rollouts.

Feature flags should be observable and removable after rollout.

---

# 126. Module Dependency Rules

Example:

```text
Core
 |
 +-- CMS
 +-- Organizations
 +-- Projects
 |
 +-- Grants -> Projects
 +-- MEAL -> Programs/Projects
 +-- Beneficiary -> Programs/Projects
 +-- CRM -> Organizations/Users/Partners
 +-- GIS -> Projects/Activities
```

Core must not depend on advanced business modules.

---

# 127. Recommended Database Domains

Logical schemas may be separated for maintainability:

```text
core
identity
cms
programs
grants
people
partners
meal
operations
reporting
integrations
audit
```

A single PostgreSQL cluster can host these schemas initially.

---

# 128. Future Data Intelligence

Long-term extensions can support:

- semantic search;
- knowledge graphs;
- anomaly detection;
- forecasting;
- trend analysis;
- document intelligence;
- automated reporting drafts.

These capabilities should consume controlled canonical data rather than bypassing the platform data model.

---

# 129. Open Standards

Where practical, use established standards:

- OpenAPI
- OAuth 2.0
- OpenID Connect
- JSON Schema
- iCalendar
- S3-compatible object APIs
- GeoJSON
- Webhooks with signed payloads

---

# 130. Vendor Independence

The system should avoid hard dependency on one cloud provider.

Every major external capability should use an adapter where practical:

```text
EmailProviderInterface
StorageProviderInterface
AIProviderInterface
SMSProviderInterface
IdentityProviderInterface
SearchProviderInterface
```

---

# 131. Recommended API Client Strategy

Generate a typed API client from OpenAPI where practical.

Frontend:

```text
OpenAPI
  -> Typed client
  -> TanStack Query hooks
  -> UI
```

External developers can use the same API contract.

---

# 132. CI/CD

Pipeline should execute:

```text
Lint
 -> Type check
 -> Unit tests
 -> Integration tests
 -> Build
 -> Security checks
 -> Migration validation
 -> Container build
 -> Deployment
```

Production deployments should include health validation and rollback capability.

---

# 133. Containerization

Services:

```text
nginx/caddy
web
api
worker
scheduler
postgres
redis
minio
opensearch (optional)
monitoring (optional)
```

Do not deploy components that are not enabled by the selected profile.

---

# 134. Initial Docker Compose

Example service graph:

```text
reverse-proxy
   |
   +-- web
   +-- api
          |
          +-- postgres
          +-- redis
          +-- object-storage
          +-- worker
```

---

# 135. Production Readiness Checklist

### Core

- [ ] Tenant isolation tested
- [ ] RBAC tested
- [ ] Audit enabled
- [ ] MFA configured
- [ ] Backups configured
- [ ] Restore tested

### CMS

- [ ] Publishing workflow
- [ ] Media processing
- [ ] SEO
- [ ] Localization

### API

- [ ] OpenAPI
- [ ] Rate limits
- [ ] Authentication
- [ ] API keys
- [ ] Webhooks

### Operations

- [ ] Projects
- [ ] Programs
- [ ] Donors
- [ ] Partners
- [ ] Forms
- [ ] Reports

### Operations/Infrastructure

- [ ] Health checks
- [ ] Metrics
- [ ] Error tracking
- [ ] TLS
- [ ] Secret management

---

# 136. Product Governance

A release should include:

- Product version
- Database migration version
- API version
- Module versions
- Breaking changes
- Security changes
- Migration notes

---

# 137. Suggested Versioning

```text
YNGO-CMS 0.x  -> active development
YNGO-CMS 1.0  -> first stable platform
YNGO-CMS 1.1  -> compatible feature release
YNGO-CMS 2.0  -> breaking platform/API changes
```

Module versions should follow a similar discipline.

---

# 138. Suggested Initial Roadmap

## Release 0.1

- Repository
- Docker Compose
- PostgreSQL
- Auth
- Tenant management
- Users/Roles
- Core audit
- API foundation

## Release 0.2

- CMS
- Pages
- Posts
- Media
- Localization
- Public frontend

## Release 0.3

- Organizations
- Programs
- Projects
- Partners
- Donors
- Tasks
- Forms

## Release 0.4

- Workflows
- Reporting
- Dashboards
- Documents
- Events

## Release 0.5

- Grants
- MEAL
- GIS
- CRM

## Release 1.0

- Security hardening
- API stabilization
- Documentation
- Backup/restore validation
- Production deployment profile
- Plugin framework baseline

---

# 139. Phase 2 Product Roadmap

- Beneficiary management
- Case management
- Field/offline application
- Advanced reporting
- Advanced search
- Automation
- AI gateway
- Knowledge base
- Developer portal

---

# 140. Phase 3 Ecosystem Roadmap

- Plugin marketplace
- Theme marketplace
- Integration marketplace
- Public transparency datasets
- Mobile applications
- Cross-organization networks
- Advanced analytics

---

# 141. Reference User Journeys

## Website publishing

```text
Author
 -> Draft Article
 -> Editor Review
 -> Approval
 -> Schedule
 -> Publish
```

## Project lifecycle

```text
Program Manager
 -> Create Project
 -> Add Donor
 -> Add Partner
 -> Add Activities
 -> Add Indicators
 -> Assign Team
 -> Track Progress
 -> Report
 -> Close
```

## Grant lifecycle

```text
Opportunity
 -> Application
 -> Proposal
 -> Review
 -> Approval
 -> Grant
 -> Project
 -> Reports
 -> Closure
```

---

# 142. Example Permission Matrix

| Module | Author | Editor | Project Manager | Finance | Admin |
|---|---|---|---|---|---|
| Pages | write | publish | read | read | full |
| News | write | publish | read | read | full |
| Projects | read | read | write | read | full |
| Grants | read | read | manage | manage | full |
| Beneficiaries | none | none | limited | none | full/audited |
| Reports | read | read | write | write | full |
| Users | none | none | team scope | none | full |
| API | none | none | limited | limited | full |

This table is illustrative. Actual authorization should be policy-driven rather than role-column hard-coded.

---

# 143. Example Tenant Configuration

```yaml
tenant:
  name: Example NGO
  locale: ar-YE
  secondary_locales:
    - en-US
  timezone: Asia/Aden
  currency: USD
  enabled_modules:
    - cms
    - programs
    - projects
    - donors
    - partners
    - forms
    - reports
    - events
    - documents
```

---

# 144. Example Module Manifest

```yaml
name: meal
version: 0.1.0
requires:
  - core>=0.1.0
  - programs>=0.1.0
  - projects>=0.1.0
permissions:
  - indicators:read
  - indicators:write
  - evaluations:read
  - evaluations:write
routes:
  - /api/v1/indicators
  - /api/v1/evaluations
```

---

# 145. Example Workflow Definition

```yaml
name: project-publication
trigger:
  event: project.ready_for_publication
steps:
  - action: approval.request
    role: communications_editor
  - action: notification.send
    template: project-review
  - action: publish
    when: approval.status == approved
```

---

# 146. Example Automation Definition

```yaml
name: grant-deadline-reminder
trigger:
  schedule: daily
condition:
  all:
    - grant.status == active
    - grant.days_until_deadline <= 7
action:
  type: notification
  template: grant-deadline-warning
```

---

# 147. Public/Private Data Model

Each publishable entity should support a distinction such as:

```text
Private Draft
Internal
Partner-visible
Public
Archived
```

The visibility model must map to authorization policies and API exposure.

---

# 148. Data Export for Tenant Exit

A tenant export should support:

- Structured data
- Media files
- Documents
- Configuration
- Selected audit logs
- Content relationships

A documented export manifest should accompany the archive.

---

# 149. Disaster Recovery Objectives

Production should define organization-specific:

- RPO
- RTO
- Backup retention
- Failover procedures

Values should be configured according to deployment tier rather than hard-coded globally.

---

# 150. Recommended First Repository Deliverables

Before implementing advanced modules, create:

```text
README.md
BLUEPRINT.md
ARCHITECTURE.md
DATA_MODEL.md
API_CONTRACT.md
SECURITY.md
DEPLOYMENT.md
ADR/
  0001-modular-monolith.md
  0002-postgresql-postgis.md
  0003-object-storage.md
  0004-api-first.md
  0005-multitenancy.md
```

---

# 151. Definition of Done for Each Module

A module is not considered complete until it includes:

- Domain model
- Database migration
- API
- Authorization
- Audit events
- Admin UI
- Validation
- Unit tests
- Integration tests
- Documentation
- Seed/demo data where useful
- Import/export behavior where applicable
- Localization support

---

# 152. Recommended Build Order

```text
1. Core
2. Identity & Tenancy
3. API foundation
4. Files / Media
5. CMS
6. Localization
7. Organizations
8. Programs
9. Projects
10. Tasks / Events / Forms
11. Donors / Partners
12. Documents
13. Reporting
14. Workflows
15. Grants
16. MEAL
17. GIS
18. CRM
19. Beneficiary / Cases
20. Automation
21. AI
22. Field / Offline
23. Marketplace
```

---

# 153. Core Engineering Rule

Do not allow optional modules to contaminate the Core.

The Core should remain responsible for reusable platform capabilities, while domain modules own their own business concepts.

---

# 154. Final Architecture Position

YNGO-CMS should be implemented as:

```text
                 YNGO-CMS
                    |
        +-----------+-----------+
        |                       |
      Core                  Modules
        |                       |
 Auth / Tenant           CMS / Projects
 RBAC / ABAC             Programs / Grants
 Files / Audit           MEAL / GIS
 API / Events            CRM / People
 Workflow primitives     Reporting / AI
        |                       |
        +-----------+-----------+
                    |
                 API-First
                    |
       +------------+------------+
       |            |             |
     Web          Mobile       External
                               Systems
```

The platform should begin as a **secure modular monolith**, use **PostgreSQL/PostGIS + object storage** as its primary data foundation, expose functionality through a **versioned API**, support **multi-tenant configuration**, and add specialized operational modules incrementally.

---

# 155. Final Product Statement

> **YNGO-CMS is a modular, Arabic-first, API-first digital platform for civil society organizations, combining professional content management with programs, projects, grants, people, partners, documents, MEAL, workflows, reporting, GIS, integrations, automation, and optional AI — while preserving tenant isolation, security, configurability, and interoperability.**

---

# 156. Implementation Note

This Blueprint is the product and architecture baseline. The next implementation artifacts should derive from it rather than bypass it:

1. Product Requirements Document (PRD)
2. Architecture Decision Records (ADR)
3. ERD / database schema
4. OpenAPI contract
5. Permission catalog
6. Module manifests
7. UX/navigation specification
8. Docker Compose development stack
9. CI/CD pipeline
10. V1 implementation backlog

The V1 implementation should prioritize a reliable Core + CMS + API + Projects/Programs foundation before adding high-complexity modules.
