# YNGO-CMS Technical Architecture Instructions
## YemenNGO-CMS — Engineering Instructions

> **Purpose:** This document defines the mandatory technical architecture and implementation rules for YNGO-CMS / YemenNGO-CMS.
>
> **Primary stack:** Next.js + React + TypeScript on the frontend; Node.js + NestJS + Fastify on the backend; PostgreSQL + PostGIS as the canonical database; Redis for cache/background coordination; S3-compatible object storage for files.

---

# 1. Architecture Decision

Build YNGO-CMS as a **modular monolith first**, with clear internal module boundaries and an API-first architecture.

Do **not** begin with a distributed microservices architecture.

The application must be designed so that selected capabilities can later be extracted into independent services without requiring a rewrite of the core domain model or API contracts.

The initial architecture is:

```text
                                    YNGO-CMS
                                        │
                    ┌───────────────────┴───────────────────┐
                    │                                       │
                Frontend                                  Backend
                    │                                       │
          Next.js + React + TS                    Node.js + NestJS
          Tailwind + shadcn/ui                       Fastify adapter
                    │                                       │
                    └───────────────────┬───────────────────┘
                                        │
                                   REST API
                                   /api/v1/*
                                        │
                    ┌───────────────────┼───────────────────┐
                    │                   │                   │
                PostgreSQL             Redis          Object Storage
                 + PostGIS                              S3 / MinIO /
                                                        Azure Blob
                    │
                    └───────────────────────────────────────┐
                                                            │
                                                      Optional Services
                                                            │
                                          ┌─────────────────┼─────────────────┐
                                          │                 │                 │
                                         AI              Search           Analytics
                                      Services          OpenSearch          /
                                      /Python             later          Data jobs
```

---

# 2. Stack Requirements

## 2.1 Frontend

Mandatory default stack:

```text
Next.js
React
TypeScript
Tailwind CSS
shadcn/ui
Radix UI
TanStack Query
Zustand
React Hook Form
Zod
MapLibre
```

Use modern Next.js App Router patterns.

Use server components, client components, SSR, SSG, and CSR only where they provide a clear benefit.

Do not introduce unnecessary client-side state.

Prefer TanStack Query for server state and Zustand only for genuine client/application state.

---

# 3. Backend Requirements

## 3.1 Runtime

Use:

```text
Node.js
TypeScript
NestJS
Fastify
```

NestJS is the application framework and Fastify is the HTTP adapter.

Do not create a second parallel backend framework inside the same application.

---

# 4. Why NestJS Is the Default Backend

NestJS is the preferred backend framework because YNGO-CMS requires a large number of independently understandable domains:

```text
Identity
Tenancy
Organizations
CMS
Media
Documents
Programs
Projects
Grants
Donors
Partners
Beneficiaries
Cases
MEAL
Procurement
HR
Events
Tasks
Workflows
Notifications
GIS
Reports
Analytics
Integrations
Automation
AI
API Management
```

The implementation must use NestJS modules to enforce logical boundaries between these domains.

Do not turn the codebase into one giant `app.module` with business logic spread across controllers.

---

# 5. Backend Layering

Use the following logical flow:

```text
HTTP Controller
      ↓
DTO / Validation
      ↓
Application Service / Use Case
      ↓
Domain Logic
      ↓
Repository / Data Access
      ↓
PostgreSQL / External Provider
```

Controllers must remain thin.

Do not put complex business rules inside controllers.

Do not put business logic into React components.

---

# 6. NestJS Module Structure

Organize the backend by business capability.

Recommended:

```text
apps/api/src/
├── app.module.ts
├── main.ts
│
├── core/
│   ├── config/
│   ├── database/
│   ├── logging/
│   ├── errors/
│   ├── health/
│   ├── events/
│   └── security/
│
├── modules/
│   ├── identity/
│   ├── tenancy/
│   ├── organizations/
│   ├── users/
│   ├── roles/
│   ├── permissions/
│   ├── audit/
│   ├── cms/
│   ├── media/
│   ├── documents/
│   ├── programs/
│   ├── projects/
│   ├── activities/
│   ├── indicators/
│   ├── grants/
│   ├── donors/
│   ├── partners/
│   ├── beneficiaries/
│   ├── cases/
│   ├── safeguarding/
│   ├── feedback/
│   ├── volunteers/
│   ├── hr/
│   ├── procurement/
│   ├── finance/
│   ├── assets/
│   ├── inventory/
│   ├── fleet/
│   ├── travel/
│   ├── events/
│   ├── tasks/
│   ├── workflows/
│   ├── notifications/
│   ├── forms/
│   ├── gis/
│   ├── reports/
│   ├── dashboards/
│   ├── search/
│   ├── integrations/
│   ├── automation/
│   ├── api-keys/
│   ├── webhooks/
│   └── ai/
│
└── common/
    ├── decorators/
    ├── guards/
    ├── interceptors/
    ├── pipes/
    ├── filters/
    └── utils/
```

Adapt the exact tree to the repository when useful, but preserve the same domain separation.

---

# 7. Database Requirements

Use:

```text
PostgreSQL
PostGIS
```

PostgreSQL is the **canonical transactional source of truth**.

PostGIS is mandatory for geographic features required by YNGO-CMS.

Do not introduce a graph database as the canonical store.

Do not use Redis as a database.

Do not use object storage as a database.

---

# 8. Database Design Rules

Use relational tables for core business entities.

Use JSONB only when flexible metadata or genuinely schema-variable information is required.

Do not put the entire domain model into one or two JSON columns.

Use:

```text
Primary Keys
Foreign Keys
Unique Constraints
Check Constraints
Indexes
Transactions
```

where appropriate.

Every tenant-owned entity must have an explicit tenant boundary.

Typical fields:

```text
id
tenant_id
created_at
updated_at
created_by
updated_by
```

only where appropriate to the domain.

---

# 9. ORM / Data Access

Use a mature TypeScript-compatible PostgreSQL data-access layer.

Preferred options:

```text
Prisma
```

or

```text
Drizzle ORM
```

or another documented TypeScript PostgreSQL ORM/query layer approved by the architecture decision.

The selected ORM must support the needs of PostgreSQL/PostGIS and must not obscure important SQL capabilities.

If PostGIS functionality requires raw SQL, encapsulate it cleanly in repositories/services rather than scattering raw SQL throughout controllers.

Document the ORM decision in an ADR.

---

# 10. Migrations

All database schema changes must use versioned migrations.

Never rely on automatic production schema synchronization.

Required flow:

```text
Schema Change
   ↓
Migration
   ↓
Migration Validation
   ↓
Automated Tests
   ↓
Deployment
```

---

# 11. Object Storage

Use an S3-compatible abstraction.

Development:

```text
MinIO
```

Production options:

```text
S3
Azure Blob Storage
MinIO
```

Store:

```text
Images
Video
Audio
PDF
DOCX
XLSX
PPTX
Other approved files
```

inside object storage, not directly in PostgreSQL.

PostgreSQL stores metadata, ownership, permissions, checksums, versions, and references.

---

# 12. Redis

Redis is allowed for:

```text
Caching
Rate Limiting
Temporary State
Job Coordination
Short-lived queues where appropriate
```

Redis must never become the canonical source of business data.

---

# 13. Background Jobs

Long-running or asynchronous operations must not block normal HTTP requests.

Examples:

```text
Email sending
Large imports
Large exports
Report generation
Image processing
Video processing
Document processing
AI processing
Webhook retries
Scheduled notifications
Scheduled publication
```

Use a proper job abstraction and document the selected worker/queue technology.

---

# 14. Frontend Architecture

Use feature-oriented structure.

Recommended:

```text
apps/web/
├── app/
├── components/
├── features/
│   ├── auth/
│   ├── content/
│   ├── projects/
│   ├── grants/
│   ├── donors/
│   ├── partners/
│   ├── beneficiaries/
│   ├── documents/
│   ├── reports/
│   ├── dashboards/
│   ├── users/
│   └── settings/
├── lib/
├── hooks/
├── providers/
└── styles/
```

Do not create giant page components.

Split complex screens into reusable feature components.

---

# 15. UI Design System

Use:

```text
shadcn/ui
Radix UI
Tailwind CSS
```

Create a shared component system.

Core components include:

```text
Button
Input
Select
Combobox
Dialog
Drawer
Table
DataTable
Tabs
Dropdown
Toast
Alert
Card
Form
DatePicker
Upload
RichTextEditor
Map
Chart
Timeline
StatusBadge
```

Avoid duplicating equivalent components across modules.

---

# 16. API Architecture

YNGO-CMS is API-first.

Primary API base path:

```text
/api/v1/
```

Required API domains should align with the Domain Model and API Contract.

Examples:

```text
/api/v1/auth
/api/v1/users
/api/v1/organizations
/api/v1/tenants
/api/v1/content
/api/v1/pages
/api/v1/media
/api/v1/documents
/api/v1/programs
/api/v1/projects
/api/v1/activities
/api/v1/indicators
/api/v1/grants
/api/v1/donors
/api/v1/partners
/api/v1/beneficiaries
/api/v1/cases
/api/v1/events
/api/v1/forms
/api/v1/tasks
/api/v1/workflows
/api/v1/notifications
/api/v1/reports
/api/v1/dashboards
/api/v1/search
/api/v1/api-keys
/api/v1/webhooks
```

Do not expose undocumented endpoints merely for convenience.

---

# 17. API Contract

Use OpenAPI as the canonical machine-readable API contract.

The contract must specify:

```text
Request schemas
Response schemas
Authentication
Authorization
Errors
Pagination
Filtering
Sorting
Search
Rate limits
```

Keep frontend and backend types synchronized using generated or contract-derived types where practical.

---

# 18. API Response Standards

Use consistent response and error structures.

Example error:

```json
{
  "error": {
    "code": "PROJECT_NOT_FOUND",
    "message": "Project was not found",
    "request_id": "..."
  }
}
```

Never expose stack traces or internal secrets in production API responses.

---

# 19. Authentication

Implement real authentication.

Support the methods defined by the project specification, potentially including:

```text
Email / Password
MFA
OIDC
OAuth
SAML where required
Microsoft Entra ID where required
Google Workspace where required
```

Never hard-code development credentials into production.

---

# 20. Authorization

Implement:

```text
RBAC
ABAC
Object-level access control
Field-level restrictions where required
```

Authorization must be enforced at the backend.

Frontend permissions are for UX only and must not be treated as the security boundary.

---

# 21. Multi-Tenancy

YNGO-CMS must be multi-tenant by design.

Typical structure:

```text
Platform
 ├── Tenant A
 ├── Tenant B
 └── Tenant C
```

Every tenant-owned request must resolve and enforce tenant context.

Required invariant:

```text
Tenant A cannot access Tenant B data.
```

This includes:

```text
Database records
API responses
Documents
Media
Search results
Exports
Reports
Webhooks
Background jobs
AI retrieval
```

---

# 22. Audit

Important business and security operations must produce real audit records.

Typical audit fields:

```text
actor
tenant
timestamp
action
entity_type
entity_id
old_value
new_value
ip_address
user_agent
request_id
```

Never fabricate audit history.

---

# 23. Search

Start with PostgreSQL search capabilities where practical.

Create a search abstraction so OpenSearch can be introduced later.

Search must operate on actual records.

Do not hard-code search results.

Arabic search must be treated as a first-class requirement.

---

# 24. GIS

Use:

```text
PostGIS
MapLibre
```

Geographic data remains canonical in PostgreSQL/PostGIS.

Do not store authoritative coordinates only in browser state or external maps.

Sensitive beneficiary locations must never be exposed publicly without explicit authorization and documented policy.

---

# 25. AI Architecture

AI is an optional platform capability, not the source of truth.

Create an abstraction for:

```text
AI Provider
AI Model
AI Request
AI Response
AI Job
```

AI may later be connected to:

```text
translation
summarization
classification
document extraction
content assistance
knowledge search
reporting assistance
```

AI must respect:

```text
Tenant Isolation
Permissions
Privacy
Data Classification
Audit
Human Review
```

Do not let AI silently overwrite canonical organizational records.

---

# 26. AI/Data Services with Python

Python is allowed for specialized services where it provides a clear technical benefit, particularly:

```text
AI inference orchestration
Data science
Advanced NLP
OCR pipelines
Document processing
Scientific/analytical workloads
```

Python services must remain optional and bounded.

Do not move the entire YNGO-CMS backend into Python merely because AI exists.

Recommended separation:

```text
Next.js / React
      ↓
NestJS API
      ↓
Domain + Data
      ↓
Optional Python AI/Data Services
```

---

# 27. Reporting

Reports and dashboards must query actual application data.

Do not create fake statistics.

If no data exists:

```text
Display an empty state.
```

Do not insert fabricated records to make charts look populated.

---

# 28. Empty-State Standard

A clean installation is expected to be empty.

Each collection view must support:

```text
Loading
Empty
Error
Populated
Unauthorized
Partial / degraded
```

Example:

```text
No projects have been created yet.
Create a project or import existing project data to begin.
```

---

# 29. NO MOCK DATA RULE

This is mandatory.

Do not create runtime mock business data.

Forbidden examples:

```text
fake NGOs
fake projects
fake donors
fake beneficiaries
fake reports
fake news
fake dashboard numbers
fake activity feed
fake notifications
fake GIS markers
fake CRM records
```

No `admin/admin`.

No automatic demo organization.

No fake sample records inserted at startup.

The application must function correctly with a genuinely empty production database.

---

# 30. Test Fixtures

Synthetic fixtures are allowed only in:

```text
tests/
test fixtures
integration tests
component tests
end-to-end tests
```

They must never be imported into the normal application runtime.

Example allowed:

```text
tests/fixtures/projects.fixture.ts
```

Example forbidden:

```text
src/data/demo-projects.ts
```

when used by the runtime.

---

# 31. Bootstrap

Because no demo data is allowed, provide secure first-run onboarding.

Possible mechanisms:

```text
CLI bootstrap
First-run setup wizard
Environment-controlled bootstrap token
```

The first real deployment should create:

```text
Organization
First Administrator
```

using actual operator-provided data.

Never create shared default passwords.

---

# 32. Public Website

The public website is a consumer of the YNGO API.

Do not duplicate business data in frontend source files.

The public site must read:

```text
Pages
News
Projects
Events
Publications
Reports
```

from the API/database.

---

# 33. Portals

Where enabled, external portals must use the same backend API and authorization model.

Potential portals:

```text
Public Portal
Partner Portal
Donor Portal
Applicant Portal
Volunteer Portal
Field Portal
Member Portal
Developer Portal
```

Do not create separate backend logic for each portal when the domain capability already exists.

---

# 34. Modular Monolith Rule

Keep these inside one deployable backend initially:

```text
CMS
Projects
Programs
Grants
MEAL
CRM
Documents
Workflow
Notifications
GIS
Reporting
```

Extract services only when there is a demonstrated operational or scaling requirement.

Potential future extractions:

```text
AI
Large report generation
Search indexing
Media processing
Data/analytics workloads
```

---

# 35. Avoid Premature Infrastructure

Do not introduce on day one:

```text
Kubernetes
Kafka
NATS
Service Mesh
Neo4j
Qdrant
Milvus
Complex distributed orchestration
```

unless the project documentation explicitly requires them and an ADR justifies the decision.

---

# 36. Monorepo Guidance

A monorepo is preferred when multiple first-party apps and shared packages exist.

Recommended conceptual structure:

```text
yngo-cms/
├── apps/
│   ├── web/
│   ├── admin/
│   ├── partner-portal/
│   ├── donor-portal/
│   ├── field/
│   └── api/
│
├── packages/
│   ├── ui/
│   ├── types/
│   ├── api-client/
│   ├── validation/
│   ├── config/
│   └── i18n/
│
├── infrastructure/
├── docs/
├── tests/
└── package.json
```

If the repository benefits from a simpler structure, adapt without breaking the architectural rules.

---

# 37. Shared Types

Avoid manually duplicating the same business schemas across frontend and backend.

Prefer:

```text
OpenAPI
Generated TypeScript Types
Shared Validation Contracts where appropriate
```

Do not tightly couple frontend internals to database entities.

The API contract is the boundary.

---

# 38. Validation

Use Zod on the frontend and DTO/schema validation in NestJS.

Validation must exist on the backend even when frontend validation exists.

Never trust client-provided IDs, tenant IDs, roles, permissions, or workflow states.

---

# 39. Configuration

Centralize configuration.

Environment variables should cover categories such as:

```text
APP
DATABASE
REDIS
OBJECT_STORAGE
AUTH
EMAIL
SMS
SEARCH
MAPS
AI
OBSERVABILITY
```

Do not hard-code production domains or provider secrets.

---

# 40. Observability

Implement:

```text
Structured logging
Request IDs
Health endpoints
Readiness endpoint
Liveness endpoint
Error tracking integration point
Metrics integration point
```

Required endpoints:

```text
/health
/ready
```

---

# 41. Testing Architecture

Required coverage includes:

```text
Unit Tests
Integration Tests
API Tests
Authorization Tests
Tenant Isolation Tests
Database Tests
Frontend Component Tests
End-to-End Tests
```

Critical security tests must explicitly attempt cross-tenant access.

---

# 42. CI/CD

The standard pipeline should validate:

```text
Lint
Typecheck
Unit Tests
Integration Tests
Build
Migration Checks
Security Checks
```

A broken API or frontend build must fail CI.

---

# 43. Deployment

Initial deployment target:

```text
Ubuntu Linux
Docker
Docker Compose
Reverse Proxy
TLS
```

Typical services:

```text
web
api
postgres
redis
minio
worker
reverse-proxy
```

Add monitoring services only where justified.

---

# 44. Production Scalability

The architecture must allow future horizontal scaling.

The API should remain stateless where practical.

Shared state belongs in:

```text
PostgreSQL
Redis
Object Storage
```

Do not store critical state only inside one API process.

---

# 45. API Versioning

Public API changes must be versioned.

Primary version:

```text
/api/v1
```

Do not make breaking changes silently.

Document migrations for future API versions.

---

# 46. Webhooks

Webhook delivery must use actual outbound HTTP requests when configured.

Track:

```text
event
endpoint
attempt
status
response
retry_count
timestamps
```

Support retry policies.

Never fake delivery success.

---

# 47. Event-Driven Internal Architecture

Use internal domain/application events to decouple modules.

Examples:

```text
ProjectCreated
ProjectUpdated
GrantApproved
ContentPublished
FormSubmitted
DocumentUploaded
UserCreated
```

Initially keep the event mechanism within the modular monolith.

Do not deploy a distributed message broker merely to implement internal events.

---

# 48. Security Boundaries

Sensitive boundaries include:

```text
Authentication
Authorization
Tenant Resolution
Files
Beneficiaries
Cases
Safeguarding
Exports
API Keys
Webhooks
AI Retrieval
```

Every one must have explicit tests.

---

# 49. Data Flow Rule

The preferred data flow is:

```text
UI
 ↓
API Client
 ↓
NestJS Controller
 ↓
DTO Validation
 ↓
Application Service
 ↓
Domain Logic
 ↓
Repository
 ↓
PostgreSQL/PostGIS
```

External services follow the same application boundary:

```text
Application Service
 ↓
Integration Adapter
 ↓
External Provider
```

Do not make domain services depend directly on provider-specific SDKs when an adapter abstraction is appropriate.

---

# 50. Integration Architecture

Third-party providers must be encapsulated.

Example:

```text
Notifications Domain
        ↓
Email Provider Interface
        ↓
SMTP / Provider A / Provider B
```

The same approach applies to:

```text
Storage
Maps
AI
SMS
Email
Accounting
Identity
Messaging
```

---

# 51. Future Python Service Boundary

If a Python service is required, expose a stable service/API contract.

Example:

```text
NestJS
  ↓
AI/Data Service API
  ↓
Python
```

Do not allow arbitrary direct database writes by the Python service unless explicitly designed and secured.

Canonical writes must still respect domain ownership and authorization.

---

# 52. No Direct Frontend Database Access

The browser must never directly connect to PostgreSQL.

The browser communicates through the API.

This is mandatory for:

```text
Web
Admin
Partner Portal
Donor Portal
Field App
```

---

# 53. No Business Logic in the Database Alone

Use database constraints for integrity, but keep complex business processes in application/domain services.

Do not attempt to encode the entire workflow engine as database triggers.

---

# 54. No Business Logic in the UI

Do not implement critical authorization, approval rules, project transitions, or financial checks only in React.

The backend remains authoritative.

---

# 55. Performance Rules

Use:

```text
Pagination
Indexes
Efficient SQL
Caching
Connection Pooling
Background Jobs
Lazy Loading
Image Optimization
Streaming where appropriate
```

Do not use fake/abbreviated data to improve perceived performance.

---

# 56. Internationalization

Arabic is first-class.

The architecture must support:

```text
Arabic RTL
English LTR
Additional locales later
Localized dates
Localized numbers
Localized currency
Localized content
```

Frontend strings must not be hard-coded in component logic.

---

# 57. Accessibility

The shared UI system must support:

```text
Semantic HTML
Keyboard Navigation
Focus Management
Screen Readers
Accessible Forms
Contrast
Error Messaging
```

---

# 58. Documentation Rules

Maintain:

```text
ARCHITECTURE.md
API.md
DATABASE.md
SECURITY.md
DEPLOYMENT.md
DEVELOPMENT.md
IMPLEMENTATION_STATUS.md
TRACEABILITY.md
```

When the actual implementation changes a documented architectural decision, create/update an ADR.

---

# 59. ADR Requirements

Use ADRs for significant decisions such as:

```text
ORM selection
Queue selection
Authentication strategy
Multi-tenancy strategy
Storage provider strategy
Search architecture
AI service boundary
Monorepo structure
Microservice extraction
```

---

# 60. Implementation Priority

Build in this order unless the master project specification explicitly changes it:

```text
1. Repository / Documentation Inspection
2. Infrastructure
3. Core Configuration
4. Database
5. Tenancy
6. Identity / Auth
7. Authorization
8. Audit
9. CMS Core
10. Media / Documents
11. Programs / Projects
12. Grants / Donors / Partners
13. MEAL / Forms
14. People / Cases / Safeguarding
15. Operations Modules
16. Workflows / Automation
17. Reporting / Dashboards
18. GIS
19. API Management
20. Portals
21. Integrations
22. AI
23. Performance / Hardening
24. Production Deployment
```

---

# 61. Definition of a Correct Implementation

The system is considered architecturally correct only when:

```text
Frontend uses Next.js/React/TypeScript
Backend uses NestJS/Node.js/TypeScript
API is the application boundary
PostgreSQL/PostGIS is canonical
Redis is non-canonical infrastructure
Files use object storage
Modules have clear boundaries
Tenant isolation is enforced
Authorization is backend-enforced
AI is optional and bounded
Python is optional and specialized
No production mock data exists
No hard-coded business records exist
Tests cover critical boundaries
```

---

# 62. Mandatory Final Review

Before releasing any major version, inspect the codebase for:

```text
mockData
mock-data
demoData
demo-data
sampleData
sample-data
fakeData
fake-data
dummyData
dummy-data
hardcoded statistics
hardcoded dashboard numbers
hardcoded organizations
admin/admin
placeholder production records
```

Review every occurrence.

Test fixtures are allowed only when isolated from runtime code.

---

# 63. Final Engineering Instruction

Treat this document as a mandatory architecture constraint.

The YNGO-CMS Engineering Documentation Package remains the product and domain source of truth.

This document defines the technology and implementation architecture beneath those product requirements.

When there is a conflict:

```text
User Requirements
        ↓
YNGO-CMS Engineering Documentation
        ↓
Domain / API / Database / Permission Specifications
        ↓
This Technical Architecture Instructions document
        ↓
Implementation details
```

Do not override higher-level product requirements merely because a technical shortcut is easier.

The target architecture is:

# Next.js + React + TypeScript
# NestJS + Node.js + Fastify
# PostgreSQL + PostGIS
# Redis
# S3 / MinIO / Azure Blob
# REST/OpenAPI
# Modular Monolith
# Optional Python AI/Data Services

Build the system so that it is:

```text
Secure
Modular
API-first
Multi-tenant
Arabic-first
Extensible
Self-hostable
Cloud-compatible
Testable
Observable
Maintainable
```

and suitable for a real NGO deployment without relying on fabricated application data.
