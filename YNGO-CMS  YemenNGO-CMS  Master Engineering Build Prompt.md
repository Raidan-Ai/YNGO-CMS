# YNGO-CMS / YemenNGO-CMS

## Master Engineering Build Prompt

You are the lead software architect, senior full-stack engineer, DevOps engineer, security engineer, database architect, UX engineer, QA engineer, and AI systems engineer responsible for building:

# YNGO-CMS

## YemenNGO-CMS

A professional, modular, multilingual, API-first digital platform for civil society organizations and NGOs.

The system must be designed as a **Digital Operating Platform for NGOs**, not merely as a website CMS.

Core principle:

> Content + Programs + Projects + People + Partners + Grants + MEAL + Documents + Workflows + Reporting + API + Integrations

The system must be production-oriented, maintainable, secure, extensible, self-hostable, and suitable for organizations operating in Yemen and other countries.

---

# 1. FIRST: INSPECT THE WORKSPACE

Before writing or changing code:

1. Inspect the entire current workspace.
2. Identify whether an existing project exists.
3. Identify existing source code, configuration, databases, Docker files, documentation, environment files, assets, and deployment configuration.
4. Locate and read:

```text
YNGO-CMS_Blueprint.md
```

and any other YNGO-CMS documentation available in the workspace.

5. Do not blindly replace an existing project.
6. Preserve useful existing work.
7. Reuse existing components when they are technically sound.
8. Refactor instead of duplicating functionality.
9. Do not create competing implementations of the same feature.
10. Establish a clear architecture before implementing large features.

If the workspace is empty, initialize the project using the architecture defined in this prompt.

---

# 2. PRODUCT DEFINITION

YNGO-CMS must support two major layers.

## Layer A — CMS

Professional content management:

* Pages
* Posts
* News
* Articles
* Announcements
* Press Releases
* Stories
* Publications
* Reports
* Research
* Events
* Campaigns
* Vacancies
* Tenders
* FAQs
* Media
* Documents
* Navigation
* SEO
* Forms
* Page Builder
* Themes
* Localization

## Layer B — NGO Operations

Operational management:

* Organizations
* Departments
* Branches
* Programs
* Projects
* Activities
* Indicators
* MEAL
* Grants
* Donors
* Partners
* Stakeholders
* Beneficiaries
* Case Management
* Volunteers
* Staff
* Tasks
* Events
* Workflows
* Documents
* Reports
* Dashboards
* GIS
* Notifications
* Automations
* Integrations
* API

The CMS must never become tightly coupled to the public website.

The same backend must support:

```text
Admin Dashboard
Public Website
Partner Portal
Beneficiary Portal
Field Application / PWA
Mobile Applications
External Integrations
Third-party Applications
AI Services
```

through a shared API platform.

---

# 3. ARCHITECTURAL PRINCIPLES

Follow these principles throughout implementation.

## 3.1 API-first

Every major operation must be available through the API.

Do not implement business logic only inside the frontend.

Business logic belongs to backend/domain services.

## 3.2 Modular architecture

Modules must be isolated and independently maintainable.

Recommended logical architecture:

```text
Core
├── Identity
├── Organizations
├── Permissions
├── Audit
├── Configuration
└── Tenancy

CMS
├── Content
├── Pages
├── Media
├── Documents
├── Navigation
├── SEO
└── Forms

NGO Operations
├── Programs
├── Projects
├── Activities
├── Indicators
├── Grants
├── Donors
├── Partners
├── Beneficiaries
├── Cases
├── Volunteers
├── Staff
├── Tasks
└── Events

Platform
├── Workflows
├── Notifications
├── Search
├── GIS
├── Reports
├── Dashboards
├── Automation
├── Integrations
├── API Management
└── Developer Platform

Optional
├── AI
├── Finance
├── Procurement
└── Advanced Analytics
```

## 3.3 Configuration over hard-coding

Do not hard-code:

* Organization names
* Branding
* URLs
* Domains
* API endpoints
* Languages
* Project statuses
* Workflow states
* NGO-specific fields
* sectors
* categories
* roles
* donor requirements

Use configuration and database-driven metadata.

## 3.4 Extensibility

The system must support:

* Custom fields
* Custom statuses
* Custom workflows
* Custom forms
* Custom modules
* Plugins
* Integrations
* Themes

without modifying core business logic whenever possible.

---

# 4. REQUIRED TECHNOLOGY STACK

Use the following stack unless an existing repository requires a justified alternative.

## Frontend

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

Use modern App Router architecture.

Support:

```text
SSR
SSG where appropriate
CSR where appropriate
Server Components where appropriate
```

## Backend

Use:

```text
Python
FastAPI
Pydantic
SQLAlchemy 2.x
Alembic
```

Use domain-oriented service architecture.

## Database

```text
PostgreSQL
PostGIS
```

Use PostgreSQL as the canonical transactional database.

Use JSONB where appropriate for:

* metadata
* custom fields
* flexible configuration

Do not abuse JSONB when normalized relational structures are more appropriate.

## Object Storage

Use:

```text
S3-compatible object storage
```

Development default:

```text
MinIO
```

Production-compatible:

```text
AWS S3
Azure Blob Storage
MinIO
```

## Cache / background processing

Use:

```text
Redis
```

for:

* caching
* rate limiting
* temporary state
* background-job coordination where required

Use a proper background task architecture.

Do not use Redis as the canonical database.

## Search

Start with:

```text
PostgreSQL Full Text Search
```

Create an abstraction so OpenSearch can be introduced later without changing application-level search APIs.

## Containers

Use:

```text
Docker
Docker Compose
```

The project must run locally using a simple Compose workflow.

---

# 5. REPOSITORY STRUCTURE

Create a clean monorepo or clearly separated application structure.

Preferred:

```text
yngo-cms/
├── apps/
│   ├── web/
│   ├── admin/
│   └── field/
│
├── services/
│   └── api/
│
├── packages/
│   ├── ui/
│   ├── types/
│   ├── config/
│   └── sdk/
│
├── infrastructure/
│   ├── docker/
│   ├── nginx/
│   └── monitoring/
│
├── scripts/
├── tests/
├── docs/
├── migrations/
├── storage/
├── .env.example
├── docker-compose.yml
├── README.md
└── LICENSE
```

If a simpler repository layout is better for the existing project, keep it clean and document the decision.

---

# 6. MULTI-TENANCY

YNGO-CMS must support multiple organizations.

Core entity:

```text
Tenant
```

Each Tenant represents an NGO or organization.

Tenant-aware resources include:

```text
Users
Departments
Branches
Content
Projects
Programs
Grants
Partners
Donors
Beneficiaries
Documents
Reports
Forms
Workflows
Settings
```

All tenant-owned data must have explicit tenant isolation.

Never allow cross-tenant data access accidentally.

Implement automated tests specifically for tenant isolation.

---

# 7. ORGANIZATION MANAGEMENT

Create:

```text
Organization
Department
Branch
Office
Team
Position
```

Organization fields should include:

```text
id
name_ar
name_en
short_name
description
logo
website
email
phone
address
country
governorates
sectors
registration_data
founded_at
status
settings
```

Support organizational hierarchy.

---

# 8. IDENTITY AND ACCESS MANAGEMENT

Implement:

```text
Users
Roles
Permissions
Groups
Sessions
API Keys
OAuth/OIDC integration
```

Roles should support examples such as:

```text
Super Admin
Organization Admin
Director
Program Manager
Project Manager
MEAL Officer
Finance Officer
HR Officer
Communications Officer
Editor
Author
Reviewer
Field Officer
Volunteer
Partner User
Public User
```

Do not assume these are immutable.

Allow organizations to create roles.

---

# 9. RBAC

Implement permission granularity:

```text
create
read
update
delete
publish
approve
export
share
manage
```

Example:

```text
projects:read
projects:create
projects:update
projects:delete
projects:approve
projects:export
```

Implement object-level authorization where required.

---

# 10. ABAC

Support attribute-based restrictions.

Examples:

```text
user.branch_id == project.branch_id

user.department_id == document.department_id

user.allowed_project_ids contains project.id
```

Create a policy abstraction so advanced authorization can be added later.

---

# 11. SECURITY

Treat security as a core feature.

Implement:

```text
HTTPS-ready deployment
Password hashing
Secure sessions
JWT where appropriate
Refresh token rotation
MFA architecture
CSRF protection
XSS protection
SQL injection protection
Input validation
Output encoding
File upload validation
File size limits
MIME validation
Rate limiting
Brute-force protection
Security headers
Audit logs
Secret management
```

Never log:

```text
Passwords
Access tokens
API secrets
Private credentials
Sensitive beneficiary data
```

---

# 12. AUDIT SYSTEM

Create immutable-style audit records for important actions.

Record:

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

Examples:

```text
content.created
content.updated
content.published
project.created
project.updated
grant.approved
document.downloaded
user.role_changed
beneficiary.updated
```

Implement audit filters and export.

---

# 13. CMS CONTENT MODEL

Create a flexible content model.

Required entities:

```text
Content
Page
Post
Article
News
Story
Publication
Announcement
PressRelease
Research
FAQ
Event
Campaign
Vacancy
Tender
```

Use reusable taxonomy:

```text
Category
Tag
Topic
Sector
```

Content must support:

```text
draft
review
approved
published
scheduled
archived
```

---

# 14. CONTENT WORKFLOW

Implement:

```text
Draft
 ↓
Review
 ↓
Approval
 ↓
Publish
 ↓
Archive
```

The workflow engine must make this configurable.

Do not hard-code one workflow for all organizations.

---

# 15. PAGE BUILDER

Create a block-based page builder.

Required blocks:

```text
Hero
Rich Text
Image
Gallery
Video
Audio
Cards
Statistics
Timeline
Map
Projects
Programs
Team
Partners
Donors
Documents
Events
FAQ
Quote
CTA
Contact
Custom HTML/Embed
```

Store blocks safely.

Do not permit arbitrary code execution by normal content editors.

Provide:

```text
drag/drop
preview
responsive layout
reusable blocks
global blocks
draft
publish
versioning
```

---

# 16. MEDIA MANAGEMENT

Implement Digital Asset Management.

Support:

```text
Images
Video
Audio
PDF
DOCX
XLSX
PPTX
ZIP
```

Store files in object storage.

Store metadata in PostgreSQL.

Implement:

```text
folder
collection
tags
caption
alt text
copyright
license
author
uploaded_by
created_at
```

Generate thumbnails/previews where technically appropriate.

---

# 17. DOCUMENT MANAGEMENT

Implement:

```text
Document
DocumentVersion
DocumentCategory
DocumentPermission
DocumentApproval
```

Features:

```text
Versioning
Restore
Permissions
Approval
Expiry
Metadata
Search
Download
Preview
Audit
```

---

# 18. MULTILINGUAL SUPPORT

Arabic must be a first-class language.

Support:

```text
ar
en
```

with architecture allowing additional languages.

Implement:

```text
RTL
LTR
localized content
localized SEO
localized slugs
translation status
missing translation detection
```

The admin UI must correctly support Arabic RTL.

The public website must support switching language without losing context.

---

# 19. PROGRAM MANAGEMENT

Implement:

```text
Program
Project
Activity
Milestone
Output
Outcome
Impact
```

Project fields:

```text
project_code
name
description
objectives
sector
start_date
end_date
budget
currency
donor
implementing_partner
locations
target_groups
status
```

Project lifecycle:

```text
Idea
Proposal
Submitted
Approved
Funded
Planning
Implementation
Monitoring
Reporting
Completed
Archived
```

Make status configuration-driven.

---

# 20. PROJECT ACTIVITIES

Each project must support:

```text
Activities
Tasks
Milestones
Outputs
Outcomes
Indicators
Documents
Risks
Issues
Partners
Locations
Budgets
Reports
```

---

# 21. MEAL

Implement:

```text
Monitoring
Evaluation
Accountability
Learning
```

Entities:

```text
Indicator
Baseline
Target
Actual
DataPoint
Outcome
Output
Survey
Assessment
Evaluation
LearningRecord
```

Support:

```text
indicator code
name
definition
unit
baseline
target
actual
frequency
data_source
responsible_person
```

Implement progress calculations and dashboards.

---

# 22. GRANTS

Implement:

```text
FundingOpportunity
Proposal
Grant
GrantAgreement
GrantAmendment
GrantReport
GrantClosure
```

Grant lifecycle:

```text
Opportunity
 ↓
Proposal
 ↓
Review
 ↓
Approval
 ↓
Award
 ↓
Implementation
 ↓
Reporting
 ↓
Closure
```

---

# 23. DONORS

Implement:

```text
Donor
DonorContact
FundingHistory
DonorRequirement
GrantRelationship
CommunicationLog
```

---

# 24. PARTNERS

Implement:

```text
Partner
PartnerContact
PartnerAgreement
MoU
DueDiligence
PartnerPerformance
PartnerProject
```

Support partner types.

---

# 25. BENEFICIARIES

Implement an optional beneficiary module with strong access controls.

Entities:

```text
Beneficiary
Household
Enrollment
Service
Assistance
Referral
Case
```

Use configurable fields.

Avoid storing sensitive information unless the organization explicitly enables it.

Build privacy controls from the beginning.

---

# 26. CASE MANAGEMENT

Implement:

```text
Case
Assessment
CaseNote
Referral
Service
FollowUp
CaseAttachment
CaseStatus
```

Lifecycle:

```text
Opened
Assessment
Active
Referred
Follow-up
Resolved
Closed
```

---

# 27. STAFF AND VOLUNTEERS

Implement:

```text
Employee
Volunteer
Position
Contract
Certification
Skill
Availability
Assignment
Attendance
VolunteerHours
Certificate
```

Do not turn this into a complete payroll system unless explicitly required.

---

# 28. EVENTS

Implement:

```text
Event
Registration
Attendee
Speaker
Venue
Attendance
Certificate
```

Support:

```text
workshop
training
conference
meeting
webinar
campaign
community event
```

---

# 29. FORMS ENGINE

Build a reusable form engine.

Field types:

```text
text
textarea
number
email
phone
date
datetime
select
multiselect
radio
checkbox
file
image
location
gps
signature
rating
matrix
```

Support:

```text
validation
required fields
conditional fields
dynamic logic
submission workflow
notifications
API submissions
CSV export
```

Forms must be reusable across:

```text
contact
applications
surveys
beneficiary intake
volunteers
events
complaints
feedback
assessments
```

---

# 30. WORKFLOW ENGINE

Build a general workflow engine.

Concept:

```text
Trigger
 ↓
Condition
 ↓
Action
 ↓
Transition
```

Actions may include:

```text
assign user
assign role
change status
send notification
send email
create task
create document
call webhook
invoke integration
schedule action
```

Build it in a modular way.

---

# 31. TASK MANAGEMENT

Entities:

```text
Task
Subtask
Checklist
Assignment
Comment
Attachment
Dependency
```

Features:

```text
Kanban
List
Calendar
Deadline
Priority
Assignee
Team
Status
```

---

# 32. NOTIFICATIONS

Implement notification abstraction:

```text
InApp
Email
SMS
Push
Webhook
```

Notifications must be template-driven.

Support notification preferences per user.

---

# 33. AUTOMATION

Build:

```text
Trigger
Condition
Action
```

Examples:

```text
New Project
→ create default tasks

Grant deadline in 7 days
→ notify program manager

New form submission
→ create review task

Content approved
→ notify publisher
```

---

# 34. GIS

Use:

```text
PostGIS
MapLibre
```

Entities:

```text
Location
Governorate
District
ProjectLocation
ActivityLocation
```

Support:

```text
point
line
polygon
bounding areas
```

Do not expose individual sensitive beneficiary locations publicly.

---

# 35. REPORTING

Create a report abstraction:

```text
Report
ReportTemplate
ReportRun
ReportSection
ReportDataSource
```

Support:

```text
PDF
HTML
CSV
Excel
DOCX
```

where practical.

Allow reports to combine:

```text
text
tables
charts
statistics
maps
project data
indicators
```

---

# 36. DASHBOARDS

Build configurable dashboards.

Widgets:

```text
KPI
Number
Chart
Table
Map
Progress
Timeline
Activity Feed
```

Dashboards must support role-based data access.

---

# 37. PUBLIC WEBSITE

Build the public website as a separate presentation layer using the same API.

Required pages:

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
Jobs
Tenders
Partners
Donors
Contact
```

The page structure must be customizable.

---

# 38. PUBLIC TRANSPARENCY

Provide optional public transparency modules:

```text
Projects
Funding
Donors
Reports
Impact
Locations
Publications
```

Only data explicitly marked public may appear.

---

# 39. CRM

Implement lightweight NGO CRM:

```text
Contact
Stakeholder
Donor
Partner
Supporter
MediaContact
GovernmentContact
```

Track:

```text
meetings
calls
emails
notes
tasks
interactions
```

---

# 40. API ARCHITECTURE

Use:

```text
/api/v1/
```

Required areas:

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
/api/v1/volunteers
/api/v1/staff
/api/v1/events
/api/v1/forms
/api/v1/tasks
/api/v1/workflows
/api/v1/notifications
/api/v1/reports
/api/v1/dashboards
/api/v1/search
/api/v1/integrations
/api/v1/webhooks
/api/v1/api-keys
```

Use OpenAPI documentation automatically.

---

# 41. API FEATURES

Implement:

```text
pagination
filtering
sorting
field selection where useful
search
bulk operations
validation errors
standard error schema
request IDs
rate limiting
authentication
authorization
versioning
```

Use consistent API response conventions.

---

# 42. API KEYS

Support:

```text
API Key
Name
Owner
Scopes
Created At
Expiration
Last Used
Revoked
```

Example scopes:

```text
content:read
content:write
projects:read
projects:write
reports:read
forms:submit
```

Never display the full secret after initial generation.

---

# 43. WEBHOOKS

Implement:

```text
WebhookEndpoint
WebhookEvent
WebhookDelivery
WebhookSecret
```

Events:

```text
content.created
content.updated
content.published
project.created
project.updated
grant.created
grant.approved
form.submitted
user.created
```

Implement retry mechanism.

---

# 44. SDK / DEVELOPER PLATFORM

Generate OpenAPI documentation.

Prepare architecture for:

```text
Python SDK
TypeScript SDK
```

Do not manually duplicate every API definition.

Generate from schema when practical.

---

# 45. INTEGRATION FRAMEWORK

Create abstraction for integrations.

Potential integrations:

```text
Microsoft 365
Google Workspace
Microsoft Teams
Slack
Email providers
SMS providers
WhatsApp providers
Telegram
Accounting systems
Cloud storage
SSO
GIS services
AI providers
```

Do not hard-code provider-specific logic into core domain services.

---

# 46. AI LAYER

AI must be optional.

Create an abstraction:

```text
AIProvider
AIModel
AIRequest
AIResponse
AIJob
```

Capabilities:

```text
translation
summarization
classification
metadata extraction
SEO assistance
content drafting
document extraction
semantic search
knowledge assistant
```

AI-generated content must remain distinguishable from verified organizational data.

Recommended workflow:

```text
AI Draft
 ↓
Human Review
 ↓
Approval
 ↓
Published / Canonical Data
```

Never allow AI output to silently overwrite authoritative organizational records.

---

# 47. SEARCH

Create a unified search abstraction.

Search across:

```text
pages
posts
news
documents
projects
programs
events
partners
reports
knowledge base
```

Support:

```text
keyword search
filters
categories
tags
date ranges
organization
project
location
```

Arabic search must work properly.

---

# 48. CUSTOM FIELDS

Provide a configurable custom-field engine.

Examples:

```text
text
number
boolean
date
select
multiselect
currency
percentage
location
reference
file
```

Allow administrators to attach custom fields to supported entities.

---

# 49. CUSTOM MODULES

Design a metadata-driven module builder.

An administrator should eventually be able to define:

```text
Module
Fields
Views
Statuses
Permissions
Workflow
Forms
Reports
```

without modifying source code.

Do not attempt an uncontrolled no-code platform in the first release.

Create extensible foundations first.

---

# 50. THEME ENGINE

Support per-tenant:

```text
logo
favicon
primary color
secondary color
font
typography
navigation
footer
email templates
domain
```

Support Arabic-first typography.

Do not hard-code one organization's branding.

---

# 51. WHITE LABEL

The platform must be usable under an organization's own branding.

YNGO-CMS branding can be hidden in tenant-facing/public contexts where configuration allows.

---

# 52. ADMIN UX

Create a professional administrative dashboard.

Layout:

```text
Sidebar
Topbar
Workspace
Notifications
User menu
Search
Breadcrumbs
```

Provide:

```text
light mode
dark mode
Arabic RTL
English LTR
responsive UI
accessible components
keyboard navigation
```

Use shadcn/ui components consistently.

---

# 53. DASHBOARD HOME

Create useful cards:

```text
Projects
Programs
Active Grants
Upcoming Deadlines
Tasks
Events
Pending Approvals
Recent Content
```

Display data based on permissions.

Do not expose modules a user cannot access.

---

# 54. GLOBAL COMMAND CENTER

Implement a command/search interface where possible.

Examples:

```text
Search project
Create task
Create article
Open grant
Find donor
Open document
```

---

# 55. PWA / FIELD SUPPORT

Prepare a field application architecture.

Support future:

```text
offline forms
local queue
GPS
camera
photos
sync
conflict handling
```

Do not fake offline support.

Build a real synchronization layer before claiming offline capability.

---

# 56. DATABASE DESIGN

Use relational modeling for core entities.

Every major table should include appropriate:

```text
id
tenant_id
created_at
updated_at
created_by
updated_by
status
```

where relevant.

Use UUIDs or another safe distributed identifier strategy.

Create proper:

```text
foreign keys
indexes
unique constraints
check constraints
```

Use soft deletion only where business requirements justify it.

Never use soft deletion as a substitute for proper archival.

---

# 57. DATA GOVERNANCE

Build support for:

```text
data classification
retention
archival
export
deletion
consent metadata where appropriate
access logs
```

Sensitive beneficiary information must have stricter permission handling.

---

# 58. FILE STORAGE SECURITY

For uploaded files:

1. Validate extension.
2. Validate MIME type.
3. Enforce size limits.
4. Generate safe storage keys.
5. Never trust original filenames.
6. Do not execute uploaded files.
7. Keep private files private.
8. Use signed URLs for controlled access where appropriate.
9. Log sensitive downloads.

---

# 59. TESTING

Testing is mandatory.

Implement:

## Backend

```text
unit tests
integration tests
API tests
authorization tests
tenant isolation tests
workflow tests
database tests
```

## Frontend

```text
component tests
form tests
state tests
critical user-flow tests
```

## End-to-end

Test critical workflows:

```text
login
create page
publish page
create project
create grant
create user
change permissions
submit form
approve content
upload document
create API key
webhook delivery
```

---

# 60. SECURITY TESTS

Explicitly test:

```text
unauthorized access
cross-tenant access
broken object-level authorization
invalid file uploads
rate limits
expired tokens
revoked API keys
permission escalation
```

---

# 61. DATABASE MIGRATIONS

Use Alembic.

Every schema modification must be represented by a migration.

Do not modify production schema manually.

Provide:

```text
migration generation
upgrade
downgrade where safe
seed
```

---

# 62. SEED DATA

Create development seed data.

At minimum:

```text
demo organization
admin user
sample departments
sample projects
sample program
sample donors
sample partners
sample content
sample events
sample forms
sample workflows
```

Clearly mark all seed data as demo data.

Never ship fake donor or beneficiary information disguised as real data.

---

# 63. LOCAL DEVELOPMENT

The developer must be able to run the platform with a documented process such as:

```bash
docker compose up -d
```

Then:

```bash
npm install
npm run dev
```

or the equivalent architecture chosen.

Provide setup instructions in README.

---

# 64. ENVIRONMENT VARIABLES

Create:

```text
.env.example
```

Organize variables:

```text
APP
DATABASE
REDIS
OBJECT_STORAGE
AUTH
EMAIL
SMS
SEARCH
AI
MAPS
OBSERVABILITY
```

Never commit secrets.

Clearly distinguish:

```text
development
test
production
```

---

# 65. CONFIGURATION

Centralize backend configuration.

Do not scatter environment-variable access throughout the codebase.

Create typed configuration settings.

Frontend URLs must be configurable.

Do not hard-code production domains.

---

# 66. OBSERVABILITY

Prepare:

```text
structured logging
request ID
error tracking
health endpoints
readiness endpoint
liveness endpoint
metrics
```

Health endpoints:

```text
/health
/ready
```

Check dependencies where appropriate.

---

# 67. DOCUMENTATION

Create:

```text
README.md
ARCHITECTURE.md
API.md
DATABASE.md
SECURITY.md
DEPLOYMENT.md
DEVELOPMENT.md
CONTRIBUTING.md
MODULES.md
PLUGIN_SYSTEM.md
```

Document major architectural decisions.

---

# 68. DESIGN SYSTEM

Create shared UI components:

```text
Button
Input
Select
Combobox
Modal
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
RichTextEditor
Upload
Map
Chart
Timeline
StatusBadge
```

Avoid creating one-off UI components when reusable components can be created.

---

# 69. DATA TABLE SYSTEM

Build a reusable advanced data table supporting:

```text
search
filter
sort
pagination
column visibility
bulk selection
bulk actions
export
row actions
saved views
```

Use it across:

```text
Projects
Users
Donors
Partners
Documents
Content
Grants
Events
Tasks
```

---

# 70. FORM SYSTEM

Build reusable validated forms using:

```text
React Hook Form
Zod
```

Forms should support:

```text
validation
server errors
draft state
autosave where appropriate
conditional fields
file uploads
Arabic RTL
```

---

# 71. CONTENT VERSIONING

Content should support:

```text
draft versions
published version
revision history
restore
change summary
```

Publishing must be explicit.

---

# 72. SCHEDULING

Support:

```text
scheduled publication
scheduled notification
project deadlines
grant deadlines
recurring tasks
events
```

Do not implement background schedules using browser timers.

Use server-side job scheduling.

---

# 73. EMAIL SYSTEM

Create a provider abstraction.

Support:

```text
SMTP
transactional email provider
```

Email templates should be tenant-aware.

Provide:

```text
verification
password reset
notification
invitation
approval
deadline
newsletter
```

---

# 74. INTERNATIONALIZATION

Do not hard-code UI strings.

Use proper translation files.

Arabic must support:

```text
RTL
Arabic numerals configuration where appropriate
localized date formatting
localized validation messages
```

English must be fully supported.

---

# 75. ACCESSIBILITY

Target modern accessibility practices.

Support:

```text
semantic HTML
keyboard navigation
focus management
screen readers
sufficient contrast
form labels
error messages
```

---

# 76. SEO

Public CMS content must support:

```text
title
description
canonical
slug
Open Graph
Twitter/X cards
robots
sitemap
structured metadata
localized SEO
```

Generate:

```text
sitemap.xml
robots.txt
```

dynamically or at build time as appropriate.

---

# 77. URL SYSTEM

Support clean URLs.

Examples:

```text
/about
/programs
/projects/project-name
/news/article-slug
/publications/report-name
/events/event-name
```

Localized URL strategy must be configurable.

---

# 78. PUBLIC CONTENT SECURITY

Public pages must never directly expose:

```text
internal IDs where avoidable
private documents
beneficiary data
internal notes
audit logs
administrative metadata
```

Only explicitly public fields are rendered publicly.

---

# 79. PERFORMANCE

Optimize for reasonable low-cost servers.

Use:

```text
pagination
indexes
caching
lazy loading
image optimization
background processing
efficient queries
connection pooling
```

Avoid premature microservices.

Start as a modular monolith unless there is a demonstrated need to split services.

---

# 80. DO NOT OVERENGINEER

Do NOT introduce Kubernetes initially.

Do NOT introduce:

```text
Kafka
NATS
Neo4j
Qdrant
Milvus
complex service mesh
```

unless a concrete requirement justifies them.

The initial deployment should be manageable on a single Linux server using Docker Compose.

---

# 81. DEPLOYMENT

Provide production deployment for:

```text
Ubuntu Linux
Docker Compose
Reverse Proxy
TLS
PostgreSQL
Redis
MinIO
API
Web
```

Prepare for future deployment to:

```text
Azure
AWS
Hetzner
VPS
on-premises
```

---

# 82. BACKUPS

Document and implement:

```text
database backup
object storage backup
configuration backup
restore procedure
backup verification
```

Provide scripts where appropriate.

---

# 83. MIGRATION / IMPORT

Prepare import architecture for:

```text
CSV
Excel
JSON
API
```

Allow mapping source fields to system fields.

Do not implement dangerous automatic data imports without validation.

---

# 84. EXPORT

Support exporting appropriate datasets:

```text
CSV
Excel
JSON
PDF
```

Respect permissions when exporting.

Every sensitive export should be auditable.

---

# 85. ERROR HANDLING

Use consistent errors.

Example:

```json
{
  "error": {
    "code": "PROJECT_NOT_FOUND",
    "message": "Project was not found",
    "request_id": "..."
  }
}
```

Never expose stack traces in production responses.

---

# 86. DOMAIN SERVICES

Business rules must live in domain/application services.

Avoid:

```text
huge route handlers
huge React components
database access scattered everywhere
```

Use clear separation:

```text
API
 ↓
Application Service
 ↓
Domain Logic
 ↓
Repository
 ↓
Database
```

---

# 87. REPOSITORY PATTERN

Use repository abstractions where helpful.

Do not create pointless abstraction layers.

Follow practical clean architecture, not architecture for its own sake.

---

# 88. FRONTEND ARCHITECTURE

Organize frontend by feature/module.

Example:

```text
features/
├── auth/
├── content/
├── pages/
├── projects/
├── grants/
├── donors/
├── partners/
├── beneficiaries/
├── documents/
├── reports/
├── dashboards/
├── users/
└── settings/
```

---

# 89. API SDK

Generate or maintain typed API clients.

Frontend should not duplicate backend schemas manually.

Use OpenAPI-derived types where practical.

---

# 90. INITIAL RELEASE STRATEGY

Do NOT attempt to build every feature at maximum complexity before anything works.

Implement in controlled phases.

## Phase 0 — Foundation

Build:

```text
repository
Docker Compose
PostgreSQL
Redis
MinIO
FastAPI
Next.js
authentication
tenant model
users
roles
permissions
audit
configuration
```

Goal:

> Platform boots cleanly and secure authentication works.

---

## Phase 1 — CMS Core

Build:

```text
Pages
Posts
News
Media
Documents
Categories
Tags
Navigation
SEO
Localization
Publishing workflow
```

Goal:

> A complete professional CMS works.

---

## Phase 2 — NGO Core

Build:

```text
Organizations
Departments
Branches
Programs
Projects
Activities
Partners
Donors
Grants
Events
Tasks
```

Goal:

> Organization operations become manageable inside the platform.

---

## Phase 3 — MEAL + Forms

Build:

```text
Indicators
Results framework
Surveys
Forms
Submissions
Monitoring
Evaluation
Dashboards
```

---

## Phase 4 — Documents + Workflows + Automation

Build:

```text
DMS
Versioning
Workflow builder
Notifications
Automation engine
```

---

## Phase 5 — GIS + Reporting

Build:

```text
PostGIS
Maps
Geographic projects
Reports
Report builder
Dashboard builder
```

---

## Phase 6 — API Platform

Build:

```text
API keys
Scopes
Webhooks
Developer documentation
OpenAPI
SDK foundation
```

---

## Phase 7 — Advanced NGO Modules

Build:

```text
Beneficiary Management
Case Management
Volunteers
Staff
Procurement
Finance integration
Stakeholder management
```

---

## Phase 8 — AI

Build:

```text
AI provider abstraction
translation
summarization
classification
document extraction
content assistant
knowledge assistant
```

AI is an optional platform service, not a prerequisite for the CMS.

---

# 91. ADMIN SETTINGS

Create a comprehensive settings area:

```text
Organization
Branding
Languages
Users
Roles
Permissions
Security
Authentication
Email
SMS
Storage
Search
Maps
AI
Integrations
API
Webhooks
Notifications
Workflows
Custom Fields
Custom Modules
Audit
Backup
System
```

---

# 92. MODULE ENABLE/DISABLE

Organizations must eventually be able to enable modules.

Example:

```text
CMS             ON
Projects        ON
Grants          ON
MEAL            ON
Beneficiaries   OFF
Finance         OFF
Procurement     ON
GIS             ON
AI              OFF
```

Disabled modules should not pollute the UI.

Design module registration from the beginning.

---

# 93. PLUGIN SYSTEM

Create a future-compatible plugin interface.

A plugin should be able to define:

```text
metadata
routes
permissions
database migrations
UI pages
settings
events
webhooks
background jobs
```

Do not make plugins able to arbitrarily compromise tenant isolation.

---

# 94. EVENTS / INTERNAL EVENT BUS

Create an application event abstraction.

Examples:

```text
ContentPublished
ProjectCreated
GrantApproved
FormSubmitted
DocumentUploaded
UserCreated
```

This makes automation/integrations extensible without tight coupling.

---

# 95. DOMAIN EVENT RULE

Events must not be treated as an excuse to create unnecessary distributed infrastructure.

Use an internal application event mechanism initially.

---

# 96. SEED DEMO ORGANIZATION

Create:

```text
YNGO Demo Organization
```

with fictional data.

Use clearly fake sample data such as:

```text
Demo NGO
Demo Project
Demo Donor
Demo Partner
```

Never use personal information copied from the internet.

---

# 97. UX LANGUAGE

The interface must be suitable for NGO staff, not developers.

Avoid unnecessary technical terminology.

Arabic terminology should be professional and understandable.

Example:

```text
Programs = البرامج
Projects = المشاريع
Grants = المنح
Donors = الجهات المانحة
Partners = الشركاء
Indicators = المؤشرات
Reports = التقارير
Documents = الوثائق
Beneficiaries = المستفيدون
Workflows = سير العمل
```

Keep internal code naming in English.

---

# 98. SEARCH UX

The admin should include global search.

Search should eventually search:

```text
Projects
Content
Documents
People
Organizations
Events
Grants
Reports
```

Display result type and permissions.

---

# 99. NOTIFICATION CENTER

Create:

```text
Unread count
Notifications list
Mark read
Mark all read
Notification preferences
```

---

# 100. APPROVAL CENTER

Create a centralized approval inbox:

```text
Pending Content
Pending Grants
Pending Documents
Pending Forms
Pending Workflow Actions
```

Users see only what they are authorized to approve.

---

# 101. ACTIVITY FEED

Create tenant-scoped activity feed for authorized users:

```text
Project created
Article published
Task assigned
Document uploaded
Grant approved
```

Do not reveal confidential information.

---

# 102. REPORTING DATA MODEL

Reports should reference live data through queries or snapshots where appropriate.

Avoid duplicating the entire database for every report.

Long-running reports should execute asynchronously.

---

# 103. BACKGROUND JOBS

Use background jobs for:

```text
email sending
document processing
thumbnail generation
report generation
large exports
webhook retries
scheduled jobs
AI requests
```

Do not block HTTP requests for long-running tasks.

---

# 104. RATE LIMITING

Implement rate limiting for:

```text
login
password reset
public forms
public API
API keys
webhooks
expensive search
```

---

# 105. PUBLIC API

Provide an optional public read-only API for content.

Example:

```text
GET /api/v1/public/pages
GET /api/v1/public/news
GET /api/v1/public/projects
GET /api/v1/public/events
```

Public API must never expose private tenant data.

---

# 106. CONTENT DELIVERY

Public content should be cacheable.

Use suitable caching headers.

Prepare architecture for CDN usage.

---

# 107. IMAGE PROCESSING

Support:

```text
thumbnail
small
medium
large
original
```

Use an image-processing pipeline.

Do not process huge images synchronously during normal web requests.

---

# 108. FILE VERSIONING

Where appropriate:

```text
Document v1
Document v2
Document v3
```

Keep version history.

---

# 109. DATA VALIDATION

Validation occurs at:

```text
frontend
API schema
business logic
database constraints
```

Do not trust client-side validation.

---

# 110. DATABASE INDEX STRATEGY

Create indexes for common:

```text
tenant_id
status
created_at
updated_at
project_id
program_id
donor_id
partner_id
location
slug
published_at
```

Use query analysis for real performance problems.

---

# 111. LOGGING

Use structured JSON logs where appropriate.

Include:

```text
timestamp
level
service
request_id
tenant_id
user_id
event
```

Never expose secrets.

---

# 112. CI/CD

Create CI pipeline that runs:

```text
lint
typecheck
tests
build
migration validation
security checks
```

Do not permit obviously broken builds to pass.

---

# 113. QUALITY GATES

Before considering a phase complete:

```text
Application starts
Database migrations work
API documentation works
Authentication works
Authorization works
Tests pass
Frontend builds
Docker Compose starts
No obvious security vulnerabilities
No hard-coded production secrets
No placeholder lorem ipsum content
```

---

# 114. NO FAKE FEATURES

Do not implement visual placeholders and call them complete.

If a feature is not fully implemented:

```text
mark it TODO
document it
hide unsupported production actions
```

Never create fake buttons that appear to work but do nothing.

---

# 115. NO PLACEHOLDER WEBSITE

Do not create a generic template with dummy lorem ipsum content.

The public website must demonstrate the actual NGO information architecture.

---

# 116. NO HARDCODED TENANT LOGIC

Avoid code such as:

```python
if organization == "XYZ":
```

Use configuration and modules.

---

# 117. NO MONOLITHIC COMPONENTS

Avoid components containing thousands of lines.

Split large features into:

```text
components
hooks
services
schemas
types
utils
```

---

# 118. API ERROR CONSISTENCY

Every API endpoint must use the same error conventions.

Document errors in OpenAPI.

---

# 119. PAGINATION

Never return unlimited records.

All large collections must be paginated.

---

# 120. BULK OPERATIONS

Support bulk actions where safe:

```text
bulk archive
bulk publish
bulk assign
bulk export
bulk tag
```

Require authorization.

---

# 121. IMPORT VALIDATION

Imports should use:

```text
preview
validation
error report
confirmation
execution
```

not immediately insert unvalidated records.

---

# 122. ADMIN ONBOARDING

Build onboarding wizard:

```text
Create Organization
 ↓
Organization details
 ↓
Branding
 ↓
Language
 ↓
Admin
 ↓
Modules
 ↓
Finish
```

---

# 123. USER INVITATION

Implement invitation flow:

```text
Admin invites user
 ↓
Email invitation
 ↓
User sets password
 ↓
Assigned role
 ↓
Access granted
```

---

# 124. PASSWORD RECOVERY

Implement secure password reset.

Tokens must:

```text
expire
be single-use
be hashed where appropriate
not appear in logs
```

---

# 125. API DOCUMENTATION UI

Expose:

```text
OpenAPI
Swagger/ReDoc
```

behind suitable authentication if exposing sensitive endpoints.

---

# 126. SYSTEM HEALTH

Admin should be able to see:

```text
API
Database
Redis
Object Storage
Background Jobs
Email
Search
```

with clear status.

---

# 127. ERROR REPORTING

Prepare Sentry-compatible error integration but make it optional.

---

# 128. FUTURE DATA WAREHOUSE

Keep data model compatible with future analytical exports.

Do not design the transactional database as a fake data warehouse.

---

# 129. FUTURE MOBILE

Do not couple public/admin web UI tightly to business logic.

Mobile clients should be able to use the same API.

---

# 130. FUTURE AI KNOWLEDGE PLATFORM

Documents and content should have metadata allowing future:

```text
embeddings
semantic search
document extraction
knowledge graph
```

without requiring them on day one.

---

# 131. DEVELOPMENT METHOD

Work incrementally.

For every feature:

1. Analyze requirements.
2. Define data model.
3. Create migration.
4. Create backend/domain services.
5. Create API.
6. Add authorization.
7. Add tests.
8. Create frontend.
9. Add loading/error/empty states.
10. Document it.
11. Run tests.
12. Review implementation.
13. Fix issues before moving on.

---

# 132. SELF-REVIEW LOOP

After each significant phase:

```text
Inspect
Test
Review
Refactor
Document
```

Look specifically for:

```text
security holes
duplicate code
tenant isolation problems
bad database queries
broken Arabic RTL
inconsistent API contracts
missing permissions
unfinished UI
```

---

# 133. FINAL ACCEPTANCE TEST

The final system must demonstrate the following complete workflow:

```text
Create Organization
        ↓
Create Admin
        ↓
Create Department
        ↓
Create User
        ↓
Assign Role
        ↓
Create Program
        ↓
Create Project
        ↓
Assign Donor
        ↓
Assign Partner
        ↓
Create Activities
        ↓
Create Indicators
        ↓
Publish Project information
        ↓
Create Report
        ↓
Display Dashboard
```

Also demonstrate:

```text
Create News Article
 ↓
Draft
 ↓
Review
 ↓
Approve
 ↓
Publish
```

And:

```text
Create Form
 ↓
Public Submission
 ↓
Review
 ↓
Task
 ↓
Notification
```

---

# 134. PRODUCTION READINESS CHECKLIST

Before final completion verify:

### Architecture

```text
[ ] Modular
[ ] API-first
[ ] Multi-tenant
[ ] Extensible
[ ] Documented
```

### Security

```text
[ ] Authentication
[ ] Authorization
[ ] RBAC
[ ] Tenant isolation
[ ] Audit logs
[ ] Rate limiting
[ ] Secure file handling
```

### CMS

```text
[ ] Pages
[ ] Content
[ ] Media
[ ] Documents
[ ] Workflow
[ ] Publishing
[ ] SEO
[ ] RTL
[ ] Localization
```

### NGO

```text
[ ] Organizations
[ ] Programs
[ ] Projects
[ ] Grants
[ ] Donors
[ ] Partners
[ ] MEAL
[ ] Events
[ ] Tasks
```

### Platform

```text
[ ] API
[ ] API Keys
[ ] Webhooks
[ ] Search
[ ] Notifications
[ ] Automation
[ ] Reports
[ ] Dashboards
[ ] GIS
```

### Engineering

```text
[ ] Tests
[ ] Docker
[ ] Migrations
[ ] Backup
[ ] Logging
[ ] Health checks
[ ] Documentation
```

---

# 135. IMPORTANT ENGINEERING RULES

Never:

```text
invent requirements silently
delete useful existing code
commit secrets
hard-code production URLs
hard-code tenant names
mix domain logic into UI
bypass authorization
store files unnecessarily inside PostgreSQL
use Redis as primary database
claim incomplete features are complete
ignore migrations
ignore tests
```

Always:

```text
preserve working functionality
write maintainable code
use typed interfaces
validate input
test authorization
test tenant boundaries
document architectural decisions
keep modules replaceable
prefer simple architecture
```

---

# 136. START NOW

Begin with the following sequence:

```text
STEP 1
Inspect workspace and Blueprint.

STEP 2
Produce a concise architecture assessment internally from the repository state.

STEP 3
Create or improve project structure.

STEP 4
Initialize infrastructure:
PostgreSQL
PostGIS
Redis
MinIO

STEP 5
Initialize FastAPI backend.

STEP 6
Initialize Next.js frontend.

STEP 7
Implement:
Tenant
Organization
User
Role
Permission
Authentication
Audit

STEP 8
Implement CMS Core.

STEP 9
Implement NGO Core.

STEP 10
Implement API.

STEP 11
Implement tests.

STEP 12
Implement Docker Compose.

STEP 13
Implement documentation.

STEP 14
Run the complete test/build cycle.

STEP 15
Fix discovered problems.

STEP 16
Continue to the next phase only when the current phase is coherent and working.
```

Do not stop after generating an architectural proposal.

**Actually implement the project.**

Do not merely create mockups.

Do not merely create database schemas.

Do not merely create API stubs.

Build functional vertical slices and connect:

```text
Database
→ Backend
→ API
→ Frontend
→ Authentication
→ Permissions
→ UI
→ Tests
```

for each implemented module.

At the end, report:

```text
Implemented
Partially implemented
Not yet implemented
Known issues
How to run
Default development credentials
API documentation location
Next recommended implementation phase
```

Do not fabricate completion.

The final result must be a real, maintainable **YNGO-CMS / YemenNGO-CMS** foundation capable of becoming a production NGO digital operating platform.
