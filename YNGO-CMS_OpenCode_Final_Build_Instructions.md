# YNGO-CMS / YemenNGO-CMS
## Final OpenCode Build Instructions
### Documentation-Driven • No Mock Data • Skills • GitHub • Local-First • Production-Oriented

You are the principal architect and implementation agent for **YNGO-CMS / YemenNGO-CMS**.

Your task is to inspect the existing workspace and YNGO-CMS documentation, discover and install the engineering skills/tools required by the documented architecture, initialize/reconcile the local Git repository, create or connect the GitHub repository, build the real application locally, verify it, and only then commit and push the verified implementation to GitHub.

This is a real implementation task. Do not deliver a prototype-only implementation, static mockup, fake dashboard, or fake API.

---

## 1. SOURCE OF TRUTH

First recursively inspect the workspace. Locate and read all YNGO-CMS documentation, especially:

```text
YNGO-CMS_Engineering_Documentation_Package_v2.md
YNGO-CMS_Enterprise_Master_Build_Prompt_v2.md
YNGO-CMS_Blueprint.md
YNGO-CMS_TECHNICAL_ARCHITECTURE_INSTRUCTIONS.md
```

Also locate any PRD, Domain Model, Database Blueprint, API Contract, Permission Matrix, Module Specifications, UI/UX, Security, Data Governance, Deployment, QA, Acceptance Test, ADR, or other YNGO-CMS documents.

Do not assume paths or filenames; search recursively.

Priority when resolving conflicts:

1. Explicit user requirements
2. Engineering Documentation Package
3. Enterprise Master Build Prompt
4. Technical Architecture Instructions
5. Domain Model
6. Database Blueprint
7. API Contract
8. Permission Matrix
9. Module Specifications
10. Security/Data Governance
11. UI/UX
12. Deployment/Operations
13. Existing implementation

Never silently invent a resolution to a documented conflict. Record material decisions in an ADR.

---

## 2. INSPECT BEFORE CHANGING

Before creating files or replacing code, inspect:

- current directory
- Git status/history/remotes/branches
- package manager and lockfiles
- Node.js version
- npm/pnpm/yarn availability
- Docker/Docker Compose
- existing frontend/backend
- database config and migrations
- tests
- CI/CD
- infrastructure
- environment files
- scripts
- documentation

If a working project already exists, preserve useful work and integrate/refactor rather than blindly replacing it.

---

## 3. SKILLS AND DEVELOPMENT CAPABILITIES

Discover the available OpenCode/project skills and identify what is required by the documentation.

At minimum evaluate skills/tools for:

- TypeScript
- React
- Next.js
- NestJS
- Fastify
- PostgreSQL
- PostGIS
- SQL/ORM
- REST/OpenAPI
- Authentication
- RBAC/ABAC
- Security
- Testing
- E2E/Playwright
- Docker/Docker Compose
- Git/GitHub
- CI/CD
- Documentation
- Accessibility
- RTL/i18n
- MapLibre/GIS
- Object storage
- Background jobs
- APIs/Webhooks
- Observability
- AI integration when documented

Install only missing skills/tools that are justified by the project.
Do not install random, redundant, or unrelated tooling.

Create/update:

```text
docs/ENGINEERING_SKILLS.md
```

Record what was already available, what was installed, why it is needed, where it is used, and relevant versions.

Never expose credentials, tokens, connection secrets, or private authentication material.

---

## 4. FINAL TECHNICAL ARCHITECTURE

Use the documented architecture unless the existing repository has a justified compatible alternative.

### Frontend

```text
Next.js
React
TypeScript
Tailwind CSS
shadcn/ui
Radix UI
TanStack Query
Zustand where justified
React Hook Form
Zod
MapLibre
```

### Backend

```text
Node.js
NestJS
TypeScript
Fastify adapter
```

### Data

```text
PostgreSQL
PostGIS
Redis
S3-compatible Object Storage
MinIO for local development
```

### API

```text
REST
OpenAPI
/api/v1/*
```

### Architecture

```text
Modular Monolith first
API-first
Domain-oriented modules
Multi-tenant
RBAC + ABAC
```

Python may be introduced only as a clearly separated service for documented AI/data/NLP workloads. Do not move the core backend to Python unless explicitly required by higher-priority documentation.

Do not introduce Kubernetes, Kafka, NATS, Neo4j, Qdrant, Milvus, service mesh, or other heavy infrastructure unless a documented requirement justifies it.

---

## 5. REPOSITORY STRUCTURE

Use the existing repository structure when sound. Otherwise use a clean structure such as:

```text
apps/
  web/
  admin/
  api/
  field/                # only when documented
  partner-portal/       # only when documented
  donor-portal/         # only when documented
  developer-portal/     # only when documented

packages/
  ui/
  types/
  api-client/
  config/
  i18n/
  validation/

infrastructure/
docs/
tests/
scripts/
.github/
```

Do not create parallel competing implementations of the same feature.

---

## 6. GITHUB REPOSITORY

Inspect existing remotes first.

If the correct YNGO-CMS GitHub repository already exists, use it and preserve history.
If it does not exist, create the official repository using the authenticated GitHub identity available in the environment.

Preferred repository name:

```text
YNGO-CMS
```

Product name:

```text
YemenNGO-CMS
```

Do not invent a GitHub owner or organization.

Prefer GitHub CLI when available. Verify authentication with `gh auth status` without exposing secrets.

If repository visibility is not explicitly specified, default to private and document the choice.

If repository creation is impossible because authentication/permissions are missing, leave the local repo ready and report the precise blocker; do not fabricate success.

---

## 7. GIT STRATEGY

Use:

```text
main = stable/default branch
feature/* = active implementation branches
```

Do not force-push or rewrite history unnecessarily.

Keep commits coherent and reviewable.

Examples:

```text
feat(core): implement tenant isolation
feat(cms): implement publishing workflow
feat(projects): implement project lifecycle
fix(auth): prevent cross-tenant access
```

---

# 8. LOCAL-FIRST WORKFLOW

The required lifecycle is:

```text
Documentation
  ↓
Local repository
  ↓
Implementation
  ↓
Local build
  ↓
Local tests
  ↓
Security/quality checks
  ↓
Local runtime verification
  ↓
Commit
  ↓
Push to GitHub
  ↓
GitHub CI
  ↓
Verify remote state
```

Do not push obviously broken code merely to show progress.

---

# 9. ABSOLUTE NO-MOCK-DATA POLICY

The runtime application must contain **ZERO fabricated business data**.

Do not create:

- fake NGOs
- fake organizations
- fake projects
- fake programs
- fake donors
- fake grants
- fake partners
- fake beneficiaries
- fake employees
- fake volunteers
- fake news
- fake articles
- fake events
- fake reports
- fake statistics
- fake KPIs
- fake funding
- fake audit history
- fake notifications
- fake dashboard values
- fake search results

Do not create a Demo Organization or Demo Mode.
Do not create `admin/admin`, `demo/demo`, or other shared credentials.
Do not populate an empty database merely to make the UI look populated.

The production database may contain only:

- schema/migrations
- necessary system configuration
- necessary system permission/locale definitions
- actual organizational data entered by the real user/organization

---

# 10. TEST FIXTURES ARE ISOLATED

Synthetic data is allowed only in isolated automated tests.

Allowed:

```text
tests/fixtures/
tests/factories/
test database
```

Never import test fixtures into production runtime.
Never create runtime files such as:

```text
src/data/demo-projects.ts
src/data/fake-users.ts
src/mocks/production-data.ts
```

---

# 11. EMPTY STATES

A fresh installation must be a professional empty application.

Each collection must support:

```text
Loading
Empty
Error
Unauthorized
Not Found
Populated
```

Example:

```text
No projects yet.
Create a project or import existing project data.
```

Use real actions. Never fake records to fill the page.

Charts with no data must show a no-data state, not fabricated values.

---

# 12. SECURE FIRST-RUN BOOTSTRAP

Because there is no seed administrator, implement a real first-run bootstrap mechanism.

Preferred options:

- secure CLI bootstrap
- first-run setup wizard
- one-time bootstrap token
- environment-controlled bootstrap

The bootstrap must create the real organization's:

```text
Tenant/Organization
First Administrator
```

No hard-coded credentials.
No default shared password.

Create:

```text
docs/BOOTSTRAP.md
```

---

# 13. DOCUMENTATION-DRIVEN MODULE IMPLEMENTATION

For every module, read the specification and implement its complete contract:

```text
Purpose
Actors
Entities
Fields
Relationships
Permissions
Workflow
API
Events
Notifications
Audit
Search
Security
UI
Tests
Acceptance Criteria
```

A module is not complete if only its database or UI exists.

Every completed module must connect:

```text
Database
 → Domain/Application service
 → API
 → Authorization
 → Typed client
 → Frontend
 → Tests
```

---

# 14. IMPLEMENTATION ORDER

Use dependency-aware phases unless the documentation requires another order.

### Phase 0 — Preparation

- inspect repository
- read documentation
- discover/install skills
- identify conflicts
- establish architecture
- create implementation tracker
- create ADR for material decisions

### Phase 1 — Foundation

- configuration
- logging
- request IDs
- error handling
- PostgreSQL/PostGIS
- Redis
- object storage
- Docker Compose
- migrations
- tenancy
- organization
- identity
- users
- roles
- permissions
- authentication
- audit

### Phase 2 — CMS

- pages
- content
- posts/articles/news
- taxonomy
- media
- documents
- localization
- SEO
- versioning
- publishing workflow

### Phase 3 — NGO Operations

- programs
- projects
- activities
- milestones
- indicators
- grants
- donors
- partners
- stakeholders
- events
- tasks

### Phase 4 — MEAL/Form Systems

- indicators/results framework
- surveys
- assessments
- forms
- submissions
- monitoring
- evaluation
- learning

### Phase 5 — People/Accountability

- staff
- volunteers
- beneficiaries
- households
- cases
- referrals
- complaints
- feedback
- safeguarding

### Phase 6 — Operations

- procurement
- assets
- inventory
- fleet
- travel
- finance layer/integrations

### Phase 7 — Platform Intelligence

- reporting
- dashboards
- GIS
- search
- notifications
- automation
- workflows

### Phase 8 — Developer Platform

- API keys
- scopes
- webhooks
- developer documentation
- integrations

### Phase 9 — Portals/Field

- public portal
- partner portal
- donor portal
- applicant portal
- volunteer/member portals where documented
- field/PWA/offline only when documented

### Phase 10 — AI

- provider abstraction
- authorized knowledge retrieval
- translation/summarization/classification
- document extraction
- content assistance
- AI governance/provenance

---

# 15. DATABASE

Implement the Domain Model and Database Blueprint exactly enough to preserve their intended semantics.

Use:

```text
PostgreSQL
PostGIS
Alembic or the documented migration system
Single ORM strategy
```

Use:

- foreign keys
- indexes
- unique constraints
- check constraints
- transactions
- tenant_id
- audit fields
- appropriate JSONB

Do not use JSONB as a substitute for all relational design.

Every schema change must have a migration.

---

# 16. MULTI-TENANCY

Every tenant-owned query and mutation must enforce tenant boundaries.

Mandatory tests:

```text
Tenant A cannot read Tenant B.
Tenant A cannot modify Tenant B.
Tenant A cannot delete Tenant B.
Tenant A cannot export Tenant B.
Tenant A cannot access Tenant B private files.
Tenant A cannot use Tenant B API credentials.
```

Do not rely only on frontend filtering.

---

# 17. AUTHORIZATION

Implement the documented:

```text
RBAC
ABAC
Object-level authorization
Field-level restrictions where required
```

Example:

```text
user.tenant_id == resource.tenant_id
user.branch_id == resource.branch_id
user.project_ids contains resource.project_id
```

Backend enforcement is mandatory.

---

# 18. SECURITY

Implement secure defaults:

- password hashing
- secure sessions/tokens
- validation
- rate limiting
- brute-force protection
- security headers
- CSRF protection where applicable
- XSS protection
- safe file uploads
- secret management
- tenant isolation
- audit logging

Never log:

```text
passwords
tokens
API secrets
private keys
sensitive beneficiary data
```

---

# 19. API

Implement the documented REST API under:

```text
/api/v1/
```

Use:

- DTOs
- validation
- OpenAPI
- pagination
- filtering
- sorting
- consistent errors
- request IDs
- authorization
- rate limiting

Do not create fake frontend endpoints.

If the documentation requires an endpoint, implement it properly.

---

# 20. API CONTRACT

Keep backend and frontend contracts synchronized through OpenAPI/generated types where practical.

Changes to an API contract must update:

```text
API implementation
OpenAPI
Frontend client/types
Tests
Documentation
```

---

# 21. API KEYS + WEBHOOKS

Implement real API keys with:

```text
name
owner
scopes
created_at
expires_at
last_used_at
revoked_at
```

Never display the full secret after initial generation.

Implement real webhook delivery with:

```text
pending
sending
success
failed
retrying
```

Use signatures/secrets as documented.

---

# 22. FRONTEND

Build the actual Next.js/React application described in the UI/UX documentation.

Use the real API for data.

Implement:

- Arabic RTL
- English LTR
- responsive UI
- accessibility
- loading/error/empty states
- reusable components
- real forms
- real tables
- real pagination/filtering

Do not create a static HTML representation of the application.

---

# 23. CMS

Implement the documented CMS, including where specified:

- pages
- content types
- posts/articles/news
- taxonomy
- media
- documents
- navigation
- localization
- SEO
- versioning
- scheduling
- approvals
- publishing
- redirects
- reusable blocks/page builder

Content must be stored in the database and files in object storage.

---

# 24. NGO OPERATIONS

Implement actual persistent modules for:

- organizations
- programs
- projects
- activities
- indicators
- grants
- donors
- partners
- beneficiaries
- cases
- volunteers
- staff
- events
- tasks
- MEAL
- forms
- reports
- workflows
- complaints/feedback/safeguarding where documented

Follow each module specification; do not interpret module names as sufficient requirements.

---

# 25. SENSITIVE DATA

Beneficiary, case, safeguarding, complaint, and other sensitive modules must use stricter permissions and audit controls.

Do not expose sensitive records in:

- public APIs
- public websites
- public maps
- unrestricted search
- dashboards visible to unauthorized roles

---

# 26. FILES / DOCUMENTS

Use MinIO locally and S3-compatible storage in production.

Implement:

- MIME validation
- extension validation
- size limits
- safe object keys
- private/public separation
- signed URLs where appropriate
- document versions
- audit downloads where required

Do not generate fake PDFs, images, reports, or organizational documents.

---

# 27. SEARCH

Start with the documented search architecture; use PostgreSQL search initially if no stronger requirement is specified.

Search only real records and only authorized records.

No fake search results.

Arabic search must be considered explicitly.

---

# 28. WORKFLOWS + AUTOMATION

Implement real persisted workflow state.

Where documented support:

- sequential approvals
- parallel approvals
- conditions
- rejection/rework
- delegation
- escalation
- reminders
- timers
- SLA
- automated assignments
- notifications
- webhook actions

Long-running/scheduled operations must not be simulated by browser timers.

---

# 29. REPORTING + DASHBOARDS

Reports and dashboards must be derived from actual database records.

Example:

```text
Active Projects = COUNT(actual projects with active status)
```

Never:

```text
Active Projects = 17
```

If there is no data, show a real no-data state.

Exports must respect permissions and be auditable when required.

---

# 30. GIS

Use:

```text
PostGIS
MapLibre
```

Maps must use actual stored locations.

Do not generate fake markers.

Never expose sensitive individual beneficiary locations publicly.

---

# 31. IMPORT/EXPORT

Provide real population mechanisms instead of seed data:

```text
CSV
Excel
JSON
API
integrations
manual entry
```

Imports should support:

```text
preview
mapping
validation
deduplication
error report
confirmation
import
```

No blind imports.

---

# 32. AI

AI is optional and must remain a platform service, not a substitute for the application database.

If implemented:

- respect tenant isolation
- respect permissions
- use real authorized data
- preserve provenance where documented
- distinguish AI drafts from approved data
- never silently overwrite canonical records

If no organizational data exists, answer truthfully with a no-data state rather than hallucinating.

---

# 33. BACKGROUND JOBS

Use background processing for long-running work such as:

- email
- imports/exports
- reports
- media processing
- document processing
- AI jobs
- webhook retries
- scheduled actions

Do not fake asynchronous work.

---

# 34. OBSERVABILITY

Implement:

```text
structured logs
request IDs
health checks
readiness checks
error handling
metrics hooks
```

At minimum:

```text
/health
/ready
```

Do not expose secrets in logs.

---

# 35. DOCKER / LOCAL DEVELOPMENT

The documented development environment must be reproducible.

Provide:

- Dockerfile(s)
- docker-compose.yml
- PostgreSQL/PostGIS
- Redis
- MinIO/object storage
- API
- Web/admin services
- persistent volumes
- health checks

Create or update:

```text
.env.example
```

Never commit `.env` or secrets.

---

# 36. TESTING

Required layers:

```text
Unit
Integration
API
Authorization
Tenant Isolation
Database
Frontend
E2E
```

Critical E2E workflows should include:

```text
first-run bootstrap
login
create user
assign role
create/publish content
create project
create grant
create form
submit form
upload document
generate report
create API key
call API
webhook delivery
cross-tenant denial
```

Use synthetic records only inside isolated test environments.

---

# 37. QUALITY GATES

Before a phase is marked VERIFIED:

- migrations work
- application starts
- API works
- UI works
- permissions work
- tenant isolation passes
- tests pass
- build passes
- no critical security regression exists
- documentation is synchronized

---

# 38. IMPLEMENTATION TRACKING

Create/update:

```text
docs/IMPLEMENTATION_STATUS.md
docs/TRACEABILITY.md
```

Track:

```text
Module
Specification
Database
Backend
API
Permissions
Frontend
Tests
Acceptance
Documentation
Status
Known Issues
```

Use statuses:

```text
PLANNED
IN_PROGRESS
IMPLEMENTED
VERIFIED
BLOCKED
```

Do not mark VERIFIED without actual verification.

---

# 39. ARCHITECTURE DECISIONS

For material decisions not explicitly specified, create an ADR under:

```text
docs/adr/
```

Record:

```text
Context
Decision
Alternatives
Consequences
Migration/compatibility impact
```

---

# 40. CODE REVIEW / NO FAKE COMPLETION

Search the repository before finalizing for:

```text
mockData
demoData
sampleData
fakeData
dummyData
exampleProjects
hardcodedStats
hardcoded dashboard values
admin/admin
demo/demo
Coming Soon
Not Implemented
TODO
FIXME
HACK
```

Review every occurrence.

Test-only fixtures are acceptable only when isolated.

Do not claim a feature is complete because the UI looks finished.

---

# 41. GITHUB CI

Create a CI workflow that runs at minimum:

- dependency install
- lint
- typecheck
- backend build
- frontend build
- unit tests
- integration tests where practical
- migration validation
- security/dependency checks

Do not automatically deploy to production unless explicitly documented.

---

# 42. LOCAL-FIRST GITHUB PUSH

Do not push first and test later.

Before the first significant push:

```text
git status
git diff
lint
typecheck
tests
build
migration checks
Docker startup
health checks
API smoke tests
frontend smoke tests
secret scan
mock-data scan
```

Then:

```text
commit
push
verify GitHub CI
```

If CI fails:

```text
fix locally
verify locally
push again
```

---

# 43. PROJECT DOCUMENTATION FILES

Maintain at minimum:

```text
README.md
CONTRIBUTING.md
SECURITY.md
.env.example

docs/BOOTSTRAP.md
docs/ENGINEERING_SKILLS.md
docs/IMPLEMENTATION_STATUS.md
docs/TRACEABILITY.md
docs/adr/
```

Also preserve and update all supplied YNGO-CMS documentation.

---

# 44. README REQUIREMENTS

README must include:

- product description
- architecture
- repository structure
- prerequisites
- local setup
- environment variables
- database migration
- first-run bootstrap
- local development commands
- testing commands
- production build
- API documentation
- GitHub workflow
- security notes

Do not publish fake screenshots/data merely to make README look complete.

---

# 45. DEFINITION OF DONE

A feature is DONE only when all applicable items below are true:

```text
[ ] specification read
[ ] domain model implemented
[ ] database migration implemented
[ ] backend logic implemented
[ ] API implemented
[ ] authorization implemented
[ ] tenant isolation tested
[ ] frontend implemented
[ ] validation implemented
[ ] error handling implemented
[ ] audit implemented where required
[ ] events implemented where required
[ ] notifications implemented where required
[ ] search implemented where required
[ ] tests written
[ ] tests passing
[ ] acceptance criteria passing
[ ] documentation updated
```

---

# 46. FINAL EXECUTION PROCEDURE

START NOW.

### Step 1
Inspect the workspace and repository.

### Step 2
Read all YNGO-CMS documentation and establish the implementation map.

### Step 3
Discover the required skills/tools and install only justified missing capabilities.

### Step 4
Reconcile the repository architecture with the documented architecture.

### Step 5
Create/connect the GitHub repository using the real authenticated GitHub account.

### Step 6
Implement the foundation locally.

### Step 7
Implement the modules in dependency order.

### Step 8
Run tests continuously.

### Step 9
Keep documentation and implementation synchronized.

### Step 10
Perform security, tenant-isolation, mock-data, secret, and dependency checks.

### Step 11
Run the full local verification suite.

### Step 12
Commit coherent verified changes.

### Step 13
Push to GitHub.

### Step 14
Verify GitHub CI and remote repository state.

### Step 15
Fix any failure locally and push the verified fix.

### Step 16
Produce a truthful final report.

---

# 47. FINAL REPORT FORMAT

At the end report:

```text
Repository:
GitHub URL:
Local Path:
Branch:
Latest Commit:
Build Status:
Test Status:
CI Status:

Skills Used/Installed:

Implemented Modules:

Partially Implemented Modules:

Blocked Modules:

Known Issues:

Security Findings:

Bootstrap Instructions:

Local Run Commands:

API Documentation:

Documentation Updated:
```

Do not claim GitHub creation, build success, tests, CI, or deployment success unless actually verified.

---

# 48. NON-NEGOTIABLE FINAL RULES

Never:

```text
invent business data
create demo business records
seed fake NGOs
seed fake users
fake API responses
fake dashboard values
fake reports
fake search results
fake integrations
fake audit history
hard-code credentials
commit secrets
bypass tenant isolation
bypass backend authorization
replace missing functionality with placeholders
claim incomplete work is complete
```

Always:

```text
read the documentation
inspect the existing repository
preserve useful work
use real persistence
use real APIs
use real authorization
use real tests
use real imports for data population
use professional empty states
keep code and documentation synchronized
build locally first
verify before pushing
push verified changes to GitHub
report reality honestly
```

# TARGET

Build a real, secure, multilingual, modular, API-first:

**YNGO-CMS / YemenNGO-CMS**

suitable for deployment to an actual civil society organization.

The system must begin empty rather than fabricated, and organizations must populate it through real onboarding, manual entry, imports, APIs, and configured integrations.
