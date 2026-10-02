# YNGO-CMS Enterprise Master Build Prompt v2

> **Project:** YNGO-CMS / YemenNGO-CMS  
> **Type:** Enterprise, Modular, Multi-Tenant, API-First NGO Digital Operating Platform  
> **Primary Language:** Arabic-first RTL + English LTR  
> **Deployment:** Self-hosted / Cloud / VPS / On-Premises  
> **Architecture:** Modular Monolith first, Service Extraction later when justified  
> **Primary Database:** PostgreSQL + PostGIS  
> **Object Storage:** S3-compatible / MinIO  
> **Backend:** FastAPI + Python  
> **Frontend:** Next.js + React + TypeScript  
> **UI:** Tailwind CSS + shadcn/ui + Radix UI  
> **API:** REST/OpenAPI first, GraphQL optional for future read-heavy use cases  

---

# 0. EXECUTION DIRECTIVE

You are the **Lead Product Architect, Principal Software Engineer, Database Architect, Security Engineer, DevOps Engineer, UX Architect, QA Lead, and AI Platform Engineer** for YNGO-CMS.

Your job is to **actually build the platform**, not merely describe it.

The final system must be a professional digital operating platform for NGOs and civil-society organizations, with a complete CMS core plus configurable operational modules.

The platform must be suitable for:

- Local NGOs
- Civil society organizations
- Foundations
- Community organizations
- Humanitarian organizations
- Development organizations
- Advocacy organizations
- Research organizations
- Membership organizations
- Hybrid nonprofit institutions

The product must remain useful outside Yemen even though Yemen is the primary contextual target.

## Non-negotiable principles

1. API-first.
2. Multi-tenant from day one.
3. Arabic-first and RTL-aware.
4. Secure by default.
5. Permission-aware at every layer.
6. Configuration over hard-coding.
7. Modular architecture.
8. Human-controlled workflows.
9. Auditability.
10. No fake functionality.
11. No fabricated data presented as real.
12. No hard-coded production domains.
13. No hard-coded tenant logic.
14. No secret credentials committed to source control.
15. Build usable vertical slices rather than disconnected stubs.
16. Preserve working code already present in the repository.
17. Prefer simple infrastructure before distributed complexity.
18. Make every major module independently testable.
19. Keep the canonical transactional data in PostgreSQL.
20. Treat external indexes, caches, embeddings, and projections as rebuildable.

---

# 1. WORKSPACE INSPECTION BEFORE IMPLEMENTATION

Before modifying anything:

1. Inspect the entire repository/workspace.
2. Identify whether this is an existing application or a new project.
3. Find and read:
   - `YNGO-CMS_Blueprint.md`
   - `README*`
   - existing architecture documents
   - package manifests
   - Docker files
   - environment files
   - database configuration
   - migrations
   - tests
   - deployment scripts
   - existing UI components
4. Detect existing applications that may already provide part of the required functionality.
5. Reuse sound existing functionality.
6. Do not create duplicate systems.
7. Do not delete existing functionality without evidence that it is obsolete.
8. Produce a short internal architecture assessment before large-scale changes.
9. If the repository already has a framework choice, evaluate whether it should be retained rather than replaced.
10. If the project is empty, initialize it according to this specification.

The existing repository is the source of truth for existing code; this prompt is the target architecture for new capabilities.

---

# 2. PRODUCT DEFINITION

YNGO-CMS is not simply a website builder.

It is:

> **A configurable digital operating system for NGO content, programs, people, projects, grants, knowledge, compliance, reporting, and public engagement.**

The platform has four layers:

```text
┌──────────────────────────────────────────────────────────┐
│ EXPERIENCE LAYER                                         │
│ Public Website | Admin | Partner | Donor | Field | PWA │
└──────────────────────────────────────────────────────────┘
                         │
┌──────────────────────────────────────────────────────────┐
│ APPLICATION / DOMAIN LAYER                               │
│ CMS | Programs | Projects | Grants | MEAL | CRM | HR... │
└──────────────────────────────────────────────────────────┘
                         │
┌──────────────────────────────────────────────────────────┐
│ PLATFORM SERVICES                                        │
│ Auth | Workflow | Search | Files | Notifications | API │
│ Audit | Automation | Reporting | GIS | Integrations     │
└──────────────────────────────────────────────────────────┘
                         │
┌──────────────────────────────────────────────────────────┐
│ DATA / INFRASTRUCTURE                                    │
│ PostgreSQL/PostGIS | Redis | S3/MinIO | Workers         │
└──────────────────────────────────────────────────────────┘
```

---

# 3. SYSTEM MODES

The platform must support:

## 3.1 Single organization deployment

One installation can operate a single NGO.

## 3.2 Multi-tenant SaaS deployment

One installation can host multiple organizations with strict tenant isolation.

## 3.3 White-label deployment

Organizations can use their own:

- logo
- domain
- colors
- typography
- terminology
- email templates
- public navigation
- enabled modules

## 3.4 Headless deployment

External websites and apps can use the API without using the built-in public website.

---

# 4. ARCHITECTURAL PRINCIPLES

## 4.1 Modular monolith first

Start as a modular monolith with explicit boundaries.

Do not introduce microservices merely for appearance.

Modules must communicate through application services and events rather than direct coupling wherever practical.

## 4.2 Extract services only when justified

Potential future extracted services:

- Search
- File processing
- Notifications
- AI gateway
- Reporting engine
- Data ingestion
- Integration workers

The initial deployment should still be easy to run on one Linux server.

## 4.3 Canonical source of truth

PostgreSQL is the canonical transactional source.

Rebuildable projections can include:

- search indexes
- caches
- analytics aggregates
- embeddings
- generated thumbnails
- materialized reporting views

## 4.4 Domain boundaries

Business rules must not depend on the UI.

The architecture should conceptually follow:

```text
Presentation
  ↓
API / Application Services
  ↓
Domain Logic
  ↓
Repositories / Infrastructure
  ↓
PostgreSQL / Object Storage
```

---

# 5. RECOMMENDED TECHNOLOGY STACK

Unless an existing repository requires a justified alternative, use:

## Frontend

- Next.js
- React
- TypeScript
- Tailwind CSS
- shadcn/ui
- Radix UI
- TanStack Query
- Zustand where global client state is genuinely needed
- React Hook Form
- Zod
- MapLibre

## Backend

- Python
- FastAPI
- Pydantic
- SQLAlchemy 2.x
- Alembic

## Database

- PostgreSQL
- PostGIS

## Object storage

- MinIO for local/self-hosted default
- S3-compatible APIs
- Azure Blob Storage adapter where required

## Infrastructure

- Docker
- Docker Compose
- Reverse proxy such as Nginx or Caddy

## Cache / jobs

- Redis
- Background worker abstraction

## Search

- PostgreSQL FTS initially
- Search provider abstraction for future OpenSearch

## Observability

- structured logging
- OpenTelemetry-compatible tracing
- optional Prometheus/Grafana
- optional Sentry

---

# 6. REPOSITORY ARCHITECTURE

Preferred structure:

```text
yngo-cms/
├── apps/
│   ├── web/                    # Public site
│   ├── admin/                  # Admin / staff portal
│   └── field/                  # Future/offline field client
│
├── services/
│   └── api/
│       ├── app/
│       │   ├── api/
│       │   ├── core/
│       │   ├── domains/
│       │   ├── infrastructure/
│       │   ├── workers/
│       │   └── main.py
│       └── tests/
│
├── packages/
│   ├── ui/
│   ├── types/
│   ├── sdk/
│   └── config/
│
├── migrations/
├── infrastructure/
│   ├── docker/
│   ├── proxy/
│   └── monitoring/
│
├── scripts/
├── docs/
├── tests/
├── docker-compose.yml
├── .env.example
├── README.md
└── LICENSE
```

If the current repository has a better established structure, keep it and document the architecture instead of forcing a reorganization without reason.

---

# 7. TENANCY MODEL

Primary tenant object:

```text
Tenant
```

A tenant may represent an NGO, foundation, project office, or organization.

Tenant-owned resources include, as appropriate:

- users
- departments
- branches
- content
- media
- documents
- programs
- projects
- grants
- donors
- partners
- beneficiaries
- cases
- volunteers
- staff
- forms
- reports
- dashboards
- workflows
- automations
- settings

Every tenant-owned record must be explicitly scoped.

## Tenant isolation requirements

Test:

- direct object access across tenants
- list endpoint leakage
- search leakage
- export leakage
- report leakage
- document leakage
- webhook leakage
- API-key leakage
- cache leakage

Use both automated tests and code-level safeguards.

---

# 8. CORE PLATFORM MODULES

The platform should be organized into the following functional domains:

```text
01 Core / Identity
02 Organization
03 Governance
04 CMS
05 Media / DAM
06 Documents / DMS
07 Knowledge
08 Programs
09 Projects
10 Grants
11 Fundraising
12 Donors
13 Partners
14 Stakeholders / CRM
15 Beneficiaries
16 Case Management
17 Safeguarding
18 Complaints / Feedback
19 MEAL
20 Surveys / Forms
21 Volunteers
22 HR / People
23 Procurement
24 Finance Layer
25 Assets / Inventory
26 Fleet
27 Travel
28 Events
29 Tasks
30 Communications
31 Advocacy / Campaigns
32 GIS
33 Reporting
34 Analytics
35 Workflow
36 Automation
37 Notifications
38 Search
39 API Platform
40 Integrations
41 Portals
42 Microsites
43 Theme Engine
44 Plugin System
45 AI Platform
46 Data Governance
47 Data Quality
48 Import / Export
49 Observability
50 Backup / DR
51 Developer Platform
52 System Administration
```

Not every module has to be fully implemented in Release 1, but the architecture must permit them without destructive rewrites.

---

# 9. CORE IDENTITY AND ACCESS MANAGEMENT

## Entities

- User
- UserProfile
- Role
- Permission
- RolePermission
- Group
- GroupMember
- Session
- APIKey
- OAuthApplication
- LoginEvent
- Invitation
- PasswordResetToken

## Authentication

Support architecture for:

- email/password
- MFA
- passkeys
- OIDC
- OAuth2
- SAML in enterprise deployments
- Microsoft Entra ID
- Google Workspace
- LDAP/AD adapter

Implement the simplest secure mechanisms first.

## User lifecycle

```text
Invited
  ↓
Active
  ↓
Suspended
  ↓
Deactivated
  ↓
Archived
```

---

# 10. RBAC + ABAC

Permissions should use consistent resource/action naming.

Example:

```text
projects:read
projects:create
projects:update
projects:delete
projects:approve
projects:publish
projects:export
```

Support attribute rules such as:

```text
user.tenant_id == resource.tenant_id
user.branch_id == resource.branch_id
user.department_id == resource.department_id
user.allowed_project_ids contains resource.id
```

Do not rely on UI hiding alone; enforcement must occur server-side.

---

# 11. ORGANIZATION MANAGEMENT

## Entities

- Organization
- Department
- Branch
- Office
- Team
- Position
- OrganizationUnit
- ContactPoint

## Organization profile

Support configurable fields for:

- legal name
- short name
- Arabic name
- English name
- description
- mission
- vision
- objectives
- sectors
- countries
- governorates
- registration information
- founding date
- website
- contacts
- branding

## Organizational hierarchy

```text
Organization
├── Headquarters
├── Branches
├── Departments
│   ├── Programs
│   ├── MEAL
│   ├── Finance
│   ├── HR
│   ├── Communications
│   └── Procurement
└── Teams
```

---

# 12. GOVERNANCE

Optional but first-class architecture for mature organizations.

Entities:

- Board
- BoardMember
- Committee
- CommitteeMember
- BoardMeeting
- BoardResolution
- GovernanceDocument
- Policy
- Bylaw
- DecisionRecord
- DelegationOfAuthority
- ConflictOfInterestDeclaration

Support:

- meeting agendas
- attachments
- attendance
- decisions
- approvals
- resolution tracking
- policy review dates

---

# 13. CMS CORE

## Content entities

- Content
- Page
- Article
- Post
- News
- Story
- Announcement
- PressRelease
- Publication
- Research
- PolicyPaper
- CaseStudy
- FAQ
- Campaign
- Vacancy
- Tender
- Event

All content types should use shared content infrastructure where practical.

## Content states

```text
Draft
In Review
Changes Requested
Approved
Scheduled
Published
Unpublished
Archived
```

Make workflows configurable.

---

# 14. EDITORIAL MANAGEMENT

Implement architecture for:

- editorial calendar
- assignments
- reviewers
- due dates
- content ownership
- editorial queues
- comments
- internal discussions
- content locking
- version comparison
- revision history
- scheduled publication
- scheduled unpublication
- publication embargo
- content expiration
- bulk actions

Do not allow accidental publication through ordinary editing actions.

---

# 15. CONTENT VERSIONING

Store revisions with:

- version number
- author
- date/time
- change summary
- content snapshot/diff strategy

Support:

- compare
- restore
- preview
- audit

---

# 16. PAGE BUILDER

Create reusable block infrastructure.

Required block types:

- Hero
- Rich Text
- Image
- Gallery
- Video
- Audio
- Cards
- Statistics
- Timeline
- Map
- Projects
- Programs
- Team
- Partners
- Donors
- Reports
- Publications
- Events
- FAQ
- Quote
- CTA
- Contact
- Download
- Embed

Block data must be validated.

Normal editors must not be able to execute arbitrary server-side code.

---

# 17. NAVIGATION AND SITE STRUCTURE

Implement:

- menus
- nested navigation
- footer navigation
- breadcrumbs
- external links
- internal references
- reusable navigation groups
- per-tenant navigation
- localized navigation

---

# 18. SEO

Per public content item support:

- SEO title
- meta description
- slug
- canonical URL
- robots settings
- Open Graph
- X/Twitter card metadata
- structured data where applicable
- alternate language links

System support:

- sitemap
- robots.txt
- redirects
- 301/302 management
- broken link detection
- canonical validation

---

# 19. DIGITAL ASSET MANAGEMENT

## Asset entities

- Asset
- AssetVersion
- Folder
- Collection
- AssetTag
- License
- CopyrightRecord
- UsageRecord

Support:

- image
- video
- audio
- PDF
- office files
- archives

Metadata may include:

- filename
- safe storage key
- MIME type
- size
- checksum
- width/height
- duration
- author
- copyright
- license
- caption
- alt text
- created date
- upload date

## Processing pipeline

```text
Upload
↓
Validate
↓
Virus/Malware Scan if configured
↓
Store Original
↓
Extract Metadata
↓
Generate Preview/Renditions
↓
Index
↓
Ready
```

Large processing jobs must run asynchronously.

---

# 20. DOCUMENT MANAGEMENT SYSTEM

Entities:

- Document
- DocumentVersion
- DocumentFolder
- DocumentClassification
- DocumentPermission
- DocumentApproval
- RetentionRule
- LegalHold

Features:

- versioning
- approvals
- expiration
- retention
- archival
- restore
- preview
- secure downloads
- access auditing
- controlled sharing

---

# 21. KNOWLEDGE MANAGEMENT

Create:

- KnowledgeArticle
- FAQ
- SOP
- Policy
- Guideline
- Manual
- LessonLearned
- KnowledgeCategory

Support:

- owner
- reviewer
- review frequency
- review date
- status
- related documents
- related projects
- tags
- search

Future semantic/AI search must be possible without redesigning this domain.

---

# 22. PROGRAM MANAGEMENT

Entities:

- Program
- ProgramGoal
- ProgramObjective
- ProgramPortfolio
- ProgramProject

Program should aggregate:

- projects
- budgets
- donors
- indicators
- outcomes
- regions
- partners

---

# 23. PROJECT MANAGEMENT

## Project entity

Fields should include at minimum:

- project_code
- tenant_id
- name_ar
- name_en
- description
- objectives
- sector
- start_date
- end_date
- budget
- currency
- donor
- lead_partner
- owner
- status
- locations
- target_groups

## Project lifecycle

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

Status must be configurable.

---

# 24. PROJECT ACTIVITIES

Entities:

- Activity
- Task
- Milestone
- Deliverable
- Output
- Outcome
- Issue
- Risk
- Dependency

Support:

- assignment
- deadlines
- dependencies
- evidence
- completion percentage
- attachments
- comments

---

# 25. GRANT MANAGEMENT

Entities:

- FundingOpportunity
- Proposal
- ProposalVersion
- Grant
- GrantAgreement
- GrantAmendment
- GrantPayment
- GrantReport
- GrantClosure
- DonorRequirement

Lifecycle:

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

Support:

- budget
- funding source
- application deadline
- grant period
- reporting schedule
- deliverables
- compliance requirements
- documents

---

# 26. DONOR MANAGEMENT

Entities:

- Donor
- DonorOrganization
- DonorContact
- FundingHistory
- DonorRequirement
- DonorRelationship
- DonorCommunication

Support:

- donor profile
- funding portfolio
- grants
- contacts
- communications
- reporting requirements
- deadlines
- documents

---

# 27. FUNDRAISING

Entities:

- FundraisingCampaign
- Donation
- Pledge
- RecurringDonation
- DonorJourney
- DonationReceipt
- FundraisingGoal
- Sponsorship

Support architecture for payment providers without hard-coding a single processor.

Include:

- campaign pages
- donation forms
- recurring giving
- receipts
- donor segmentation
- fundraising analytics

---

# 28. PARTNER MANAGEMENT

Entities:

- Partner
- PartnerContact
- PartnerAgreement
- MoU
- DueDiligence
- PartnerPerformance
- PartnerProject
- PartnerCompliance

Support:

- partner type
- contacts
- agreements
- due diligence
- projects
- performance
- risk/compliance status

---

# 29. STAKEHOLDER / CRM

Entities:

- Contact
- Stakeholder
- GovernmentContact
- MediaContact
- Supporter
- CommunityOrganization
- AcademicPartner
- PrivateSectorPartner

Track:

- calls
- meetings
- messages
- notes
- tasks
- interactions
- relationships

---

# 30. MEMBERSHIP MANAGEMENT

For organizations that use membership models.

Entities:

- Member
- MembershipType
- MembershipApplication
- MembershipRenewal
- MembershipCard
- MembershipEvent
- MembershipStatus

Features:

- application
- approval
- renewal
- status history
- member portal
- certificates/cards

---

# 31. BENEFICIARY MANAGEMENT

This module is optional by tenant but must have secure architecture.

Entities:

- Beneficiary
- Household
- Enrollment
- Service
- Assistance
- BeneficiaryAssessment
- Referral

Support configurable data capture.

Privacy rules:

- minimize collection
- strict access control
- sensitive-field restrictions
- audit access
- protect exports
- never expose individual sensitive data publicly

---

# 32. CASE MANAGEMENT

Entities:

- Case
- Assessment
- CaseNote
- Referral
- Service
- FollowUp
- CaseAttachment
- CaseStatus

Lifecycle:

```text
Opened
↓
Assessment
↓
Active
↓
Referral / Service
↓
Follow-up
↓
Resolved
↓
Closed
```

Access to case data must be independently permissioned.

---

# 33. SAFEGUARDING

Create a restricted module for:

- safeguarding incidents
- incident reports
- restricted cases
- escalation
- investigation
- evidence
- corrective actions
- closure

Support:

- confidential access
- anonymous reporting option where appropriate
- restricted roles
- secure attachments
- audit logs
- retention controls

Do not expose safeguarding records through generic search.

---

# 34. COMPLAINTS, FEEDBACK AND ACCOUNTABILITY

Entities:

- Complaint
- Feedback
- Suggestion
- Inquiry
- WhistleblowerReport
- ComplaintCategory
- Investigation
- Resolution
- Referral

Lifecycle:

```text
Submitted
↓
Acknowledged
↓
Review
↓
Investigation
↓
Action
↓
Resolution
↓
Closure
```

Support anonymous submissions where configured.

---

# 35. MEAL

MEAL = Monitoring, Evaluation, Accountability and Learning.

Entities:

- ResultFramework
- Impact
- Outcome
- Output
- ActivityResult
- Indicator
- Baseline
- Target
- DataPoint
- MonitoringVisit
- Assessment
- Evaluation
- LearningRecord

Indicator fields:

- code
- name
- definition
- unit
- baseline
- target
- actual
- frequency
- data source
- responsible owner
- disaggregation dimensions

Support progress calculations.

---

# 36. SURVEYS AND FORMS

Create a reusable Form Engine.

Field types:

- text
- textarea
- number
- currency
- percentage
- email
- phone
- date
- datetime
- select
- multiselect
- radio
- checkbox
- file
- image
- GPS/location
- signature
- rating
- matrix
- reference lookup

Support:

- validation
- required fields
- conditional logic
- show/hide logic
- branching
- calculations
- save draft
- public/private forms
- authenticated forms
- API submission
- submission review
- export

---

# 37. VOLUNTEER MANAGEMENT

Entities:

- Volunteer
- Skill
- Availability
- VolunteerApplication
- Assignment
- VolunteerHour
- Attendance
- Certificate

Features:

- recruitment
- onboarding
- skills
- assignments
- hours
- certificates
- communication

---

# 38. HR / PEOPLE MANAGEMENT

Do not make payroll mandatory.

Entities:

- Employee
- Position
- Contract
- EmployeeDocument
- LeaveRequest
- AttendanceRecord
- PerformanceReview
- Training
- Certification
- Skill
- RecruitmentJob
- Applicant
- Interview
- HiringDecision
- Onboarding
- Offboarding

Support:

```text
Recruitment
→ Application
→ Screening
→ Interview
→ Decision
→ Onboarding
→ Active
→ Offboarding
```

---

# 39. PROCUREMENT

Implement an extensible procurement workflow.

Entities:

- PurchaseRequest
- RFQ
- Vendor
- Quotation
- Evaluation
- PurchaseOrder
- Contract
- Delivery
- InvoiceReference

Workflow:

```text
Purchase Request
↓
Approval
↓
RFQ
↓
Quotations
↓
Evaluation
↓
Award
↓
Purchase Order
↓
Delivery
↓
Closure
```

---

# 40. FINANCE LAYER

Do not implement a full accounting ERP unless explicitly required.

Implement NGO-oriented financial structures:

- Budget
- BudgetLine
- FundingSource
- CostCenter
- FinancialPeriod
- ExpenseImport
- BudgetRevision
- BudgetUtilization
- FinancialReport
- Currency
- ExchangeRate

Design adapters for external accounting systems.

---

# 41. ASSET AND INVENTORY MANAGEMENT

Entities:

- Asset
- AssetCategory
- AssetAssignment
- AssetMovement
- MaintenanceRecord
- InventoryItem
- Warehouse
- StockMovement
- Disposal

Support:

- serial numbers
- ownership
- assignments
- maintenance
- stock levels
- movement history

---

# 42. FLEET MANAGEMENT

Optional module.

Entities:

- Vehicle
- Driver
- Trip
- FuelRecord
- MaintenanceRecord
- Registration
- Insurance
- MileageRecord

---

# 43. TRAVEL MANAGEMENT

Entities:

- TravelRequest
- Traveler
- Itinerary
- Accommodation
- Transport
- PerDiem
- TravelDocument
- TravelReport

Workflow:

```text
Request
→ Approval
→ Booking
→ Travel
→ Settlement / Report
```

---

# 44. EVENTS

Entities:

- Event
- EventSeries
- Registration
- Attendee
- Speaker
- Venue
- Attendance
- Certificate

Support:

- workshops
- trainings
- conferences
- webinars
- meetings
- community events
- campaigns

---

# 45. TASK AND WORK MANAGEMENT

Entities:

- Task
- Subtask
- Checklist
- Comment
- Attachment
- Dependency
- SavedView

Views:

- list
- kanban
- calendar
- timeline where appropriate

Features:

- assignment
- due dates
- priority
- tags
- watchers
- dependencies
- recurring tasks

---

# 46. COMMUNICATIONS

Provide abstractions for:

- Email
- SMS
- Push
- In-app notifications
- Newsletter
- Social publishing integrations

Entities:

- ContactList
- Subscriber
- Segment
- Campaign
- Template
- DeliveryLog

Features:

- segmentation
- scheduling
- unsubscribe
- campaign analytics
- localized templates

---

# 47. ADVOCACY AND CAMPAIGNS

Entities:

- AdvocacyCampaign
- CampaignGoal
- CampaignAudience
- CampaignMessage
- CampaignActivity
- CampaignAsset
- CampaignMetric
- StakeholderTarget
- EngagementAction

Support:

- objectives
- audiences
- messaging
- activities
- materials
- impact tracking

---

# 48. GIS / GEOSPATIAL

Use PostGIS.

Entities:

- Country
- Governorate
- District
- Subdistrict
- Community
- Location
- ProjectLocation
- ActivityLocation
- GeographicArea

Support:

- point
- line
- polygon
- bounding box
- distance queries
- spatial filters

Public maps must obey privacy constraints.

---

# 49. REPORTING ENGINE

Entities:

- Report
- ReportTemplate
- ReportRun
- ReportSection
- ReportDataSource
- ReportParameter

Report content can include:

- text
- tables
- charts
- KPIs
- maps
- indicator results
- project summaries
- donor data

Support export targets where technically viable:

- PDF
- HTML
- CSV
- XLSX
- DOCX
- JSON

Long reports should run asynchronously.

---

# 50. ANALYTICS

Create an analytics abstraction rather than embedding analytics queries everywhere.

Support:

- KPI cards
- trend charts
- project comparisons
- portfolio analysis
- funding analysis
- donor analysis
- geographic analysis
- indicator trends
- workload analysis
- content analytics

Prepare for future data warehouse export.

---

# 51. DASHBOARD BUILDER

Widgets:

- KPI
- number
- chart
- table
- map
- progress
- timeline
- recent activity
- deadlines

Dashboards must be permission-aware.

Support:

- private dashboards
- team dashboards
- tenant dashboards
- role-based dashboards
- saved filters

---

# 52. WORKFLOW ENGINE

The workflow engine is a platform service.

Entities:

- Workflow
- WorkflowVersion
- WorkflowState
- WorkflowTransition
- WorkflowRule
- WorkflowAction
- WorkflowInstance
- WorkflowTask

Required capabilities:

- sequential approval
- parallel approval
- conditional branching
- rejection
- rework
- escalation
- delegation
- timeouts
- SLA
- reminders
- auto-assignment
- rollback where safe

Workflow definitions should be versioned.

---

# 53. AUTOMATION ENGINE

Concept:

```text
Trigger
↓
Conditions
↓
Actions
```

Trigger examples:

- entity created
- entity updated
- status changed
- deadline approaching
- scheduled time
- form submitted
- webhook received

Actions:

- create task
- assign user
- change status
- send notification
- send email
- call webhook
- create record
- start workflow
- enqueue background job

Automation executions must be logged.

---

# 54. NOTIFICATION CENTER

Support channels:

- In-app
- Email
- SMS
- Push
- Webhook

Entities:

- Notification
- NotificationTemplate
- NotificationPreference
- DeliveryAttempt

Users can configure notification preferences where allowed by organizational policy.

---

# 55. SEARCH PLATFORM

Create a provider-independent interface.

Search across allowed resources:

- content
- pages
- news
- documents
- projects
- programs
- grants
- donors
- partners
- reports
- events
- knowledge

Support:

- full-text
- Arabic tokenization strategy
- filters
- facets
- date ranges
- organization
- project
- sector
- location

Future provider: OpenSearch.

---

# 56. API PLATFORM

Base path:

```text
/api/v1/
```

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

Every endpoint must define:

- authentication
- authorization
- validation
- pagination if collection
- filtering
- sorting where relevant
- error responses
- audit behavior
- tenant scope

---

# 57. API ERROR CONTRACT

Use consistent errors.

Example:

```json
{
  "error": {
    "code": "PROJECT_NOT_FOUND",
    "message": "Project was not found",
    "details": null,
    "request_id": "..."
  }
}
```

Do not expose stack traces in production.

---

# 58. API KEYS

Entity:

- APIKey

Fields:

- id
- name
- owner
- tenant
- scopes
- created_at
- expires_at
- last_used_at
- revoked_at

The raw secret is shown only at creation.

Never store the raw secret if avoidable; store a secure representation.

Support scopes such as:

```text
content:read
content:write
projects:read
projects:write
reports:read
forms:submit
```

---

# 59. WEBHOOKS

Entities:

- WebhookEndpoint
- WebhookSubscription
- WebhookDelivery
- WebhookAttempt
- WebhookSecret

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
document.uploaded
```

Support:

- signing
- retry
- exponential backoff
- delivery logs
- replay where safe
- disable on repeated failure

---

# 60. DEVELOPER PLATFORM

Create a developer portal architecture with:

- OpenAPI
- API documentation
- authentication guide
- API key management
- OAuth apps
- webhook configuration
- SDK documentation
- rate limit documentation
- examples
- integration status

Prepare a TypeScript SDK and Python SDK generation workflow rather than hand-maintaining divergent copies of the API.

---

# 61. INTEGRATION FRAMEWORK

Integration interface should support providers for:

- Microsoft 365
- Google Workspace
- Microsoft Teams
- Slack
- email providers
- SMS providers
- WhatsApp providers
- Telegram
- accounting systems
- cloud storage
- identity providers
- GIS services
- payment services
- AI providers

Provider-specific logic belongs behind adapters.

---

# 62. EXTERNAL PORTALS

Support configurable portals:

```text
Public Portal
Applicant Portal
Donor Portal
Partner Portal
Member Portal
Volunteer Portal
Field Portal
Board Portal
```

Each portal must have independent route guards and permission policies.

---

# 63. MICROSITES

A tenant can optionally create:

- main website
- campaign microsite
- project microsite
- event microsite

All can use the same content and asset infrastructure.

Support domain mapping architecture.

---

# 64. THEME ENGINE

Per tenant support:

- logos
- favicons
- color tokens
- typography
- spacing tokens
- radius tokens
- navigation style
- component variants
- email branding

Use design tokens rather than scattered CSS constants.

---

# 65. WHITE-LABEL

Support:

- organization branding
- domain mapping
- email branding
- portal branding
- optional YNGO-CMS attribution settings

---

# 66. PLUGIN SYSTEM

Create a future-ready plugin contract.

Plugin manifest should be able to describe:

- name
- version
- compatibility
- permissions
- migrations
- routes
- UI extensions
- settings
- event handlers
- background jobs
- API extensions

Plugin security is mandatory.

Do not allow arbitrary plugins to bypass tenant boundaries.

---

# 67. CUSTOM FIELDS

Create a metadata-driven custom field system.

Field types:

- text
- textarea
- number
- decimal
- boolean
- date
- datetime
- currency
- percentage
- select
- multiselect
- reference
- file
- location

Custom fields must support:

- labels per language
- validation
- required state
- visibility
- role-based access where appropriate
- search/index flag
- reporting eligibility

---

# 68. CUSTOM MODULE BUILDER

Eventually allow administrators to define:

```text
Module
Fields
Views
Statuses
Permissions
Forms
Workflow
Reports
```

Implementation strategy:

Phase 1: metadata foundations.

Phase 2: configurable views and forms.

Phase 3: controlled custom entities.

Do not attempt a dangerous fully arbitrary no-code runtime in the first release.

---

# 69. AI PLATFORM

AI is an optional platform layer.

Create:

- AIProvider
- AIModel
- AIRequest
- AIResponse
- AIJob
- PromptTemplate
- PromptVersion
- AIUsageRecord
- AIReview

Capabilities:

- translation
- summarization
- classification
- metadata extraction
- content drafting
- document extraction
- SEO assistance
- semantic search
- knowledge assistant
- reporting assistance

---

# 70. AI GOVERNANCE

AI outputs must be traceable.

Store where appropriate:

- model/provider
- prompt version
- request timestamp
- actor
- source references
- output status
- human review status
- usage/cost metadata

Recommended lifecycle:

```text
AI Draft
↓
Human Review
↓
Approval
↓
Published / Recorded
```

AI must never silently overwrite authoritative organizational records.

For sensitive data, support redaction or provider-specific privacy routing.

---

# 71. AI AGENTS

Prepare extension points for specialized agents:

- Content Agent
- Research Agent
- Translation Agent
- Reporting Agent
- Grant Assistant
- MEAL Assistant
- Data Quality Agent
- Knowledge Assistant
- Document Assistant

Agents must use the same permission model as humans.

An agent must never obtain broader permissions merely because it is automated.

---

# 72. DATA GOVERNANCE

Support metadata for:

- data classification
- data owner
- purpose
- retention
- archival
- deletion
- access restrictions
- exportability

Possible classifications:

```text
Public
Internal
Confidential
Restricted
Sensitive
```

These labels are configurable.

---

# 73. DATA QUALITY

Create a data quality service that can detect:

- duplicates
- missing fields
- invalid values
- conflicting records
- stale records
- inconsistent references

Support:

- quality rules
- quality score
- review queue
- suggested merge
- manual merge
- audit trail

High-impact entity merges should require explicit human confirmation.

---

# 74. MASTER DATA

Create shared master-data registries for:

- sectors
- countries
- governorates
- districts
- currencies
- units
- organizational types
- donor types
- partner types
- project statuses
- content statuses

Master data should be versioned where relevant.

---

# 75. IMPORT / EXPORT

Supported imports initially:

- CSV
- XLSX
- JSON

Import process:

```text
Upload
↓
Inspect
↓
Map fields
↓
Validate
↓
Preview
↓
Confirm
↓
Import
↓
Report
```

Support:

- duplicate handling
- validation errors
- partial failure reporting
- retry
- import logs

Exports must respect permissions and be auditable for sensitive data.

---

# 76. OFFLINE FIELD PLATFORM

Prepare a real offline-capable architecture, not a fake offline mode.

```text
Field Client
↓
Local Store
↓
Offline Queue
↓
Connection Restored
↓
Sync Engine
↓
Conflict Resolution
↓
Server
```

Support future field workflows:

- forms
- GPS
- images
- attachments
- inspections
- monitoring visits
- beneficiary intake

---

# 77. SYNCHRONIZATION MODEL

Define:

- client record ID
- server record ID
- sync cursor
- local change log
- server change log
- conflict status
- last sync time

Conflicts must never be silently discarded.

---

# 78. PUBLIC WEBSITE

The public website should support:

```text
Home
About
Mission/Vision
Programs
Projects
Impact
News
Stories
Research
Publications
Reports
Events
Vacancies
Tenders
Partners
Donors
Contact
```

Use the API as the content source.

The public website must not access the database directly.

---

# 79. PUBLIC TRANSPARENCY PORTAL

Allow organizations to publish selected:

- projects
- funding
- donors
- reports
- impact metrics
- publications
- project locations

Only explicit public data may be exposed.

---

# 80. LOCALIZATION

First-class locales:

- `ar`
- `en`

Architecture must support additional locales.

Support:

- RTL/LTR
- localized content
- localized slugs
- localized metadata
- date/time formatting
- number formatting
- currencies
- translation status
- fallback locale

Never hard-code user-visible text in components.

---

# 81. ARABIC-FIRST UX

Arabic is not a second-class translation.

The entire interface must remain usable in RTL:

- tables
- forms
- navigation
- dialogs
- dashboards
- charts
- maps
- filters
- pagination
- breadcrumbs

Test long Arabic labels and mixed Arabic/English data.

---

# 82. ACCESSIBILITY

Target strong accessibility practices:

- semantic HTML
- keyboard navigation
- focus management
- screen-reader support
- accessible form labels
- error feedback
- meaningful status indicators
- sufficient contrast

---

# 83. RESPONSIVE DESIGN

Admin and public UI should support:

- desktop
- tablet
- mobile

Do not merely shrink desktop tables; provide usable mobile layouts.

---

# 84. GLOBAL SEARCH UX

Provide an admin global search entry point.

Results should display:

- type
- title
- relevant metadata
- status
- location/context
- permission-aware actions

Search must never bypass authorization.

---

# 85. APPROVAL CENTER

Create a centralized approval workspace.

Possible queues:

- content
- documents
- grants
- projects
- forms
- complaints
- workflows

Users should only see queues they have authority to act on.

---

# 86. ACTIVITY FEED

Tenant-scoped activity feed for non-sensitive operations.

Examples:

- project created
- article published
- task assigned
- report generated
- document uploaded

Sensitive information must be omitted or permission-filtered.

---

# 87. AUDIT LOG

Audit records should capture:

- tenant
- actor
- action
- entity type
- entity ID
- timestamp
- request ID
- IP where policy permits
- user agent where policy permits
- old value where appropriate
- new value where appropriate

Audit data itself needs access controls.

---

# 88. SECURITY ARCHITECTURE

Implement defense-in-depth:

- secure password hashing
- MFA architecture
- token/session safety
- authorization middleware
- object-level authorization
- tenant scoping
- input validation
- output encoding
- CSRF protection as applicable
- CORS policy
- security headers
- rate limiting
- login throttling
- secure cookies where used
- secret management
- safe file uploads
- signed private URLs
- malware scanning adapter
- audit logging

Never log secrets.

---

# 89. FILE SECURITY

For every upload:

1. Validate declared MIME type.
2. Validate detected MIME type where possible.
3. Enforce file-size limits.
4. Generate safe storage key.
5. Sanitize metadata.
6. Prevent executable content from being served incorrectly.
7. Keep private files private.
8. Support signed URLs for controlled access.
9. Audit protected downloads.
10. Optionally scan for malware.

---

# 90. API SECURITY

Protect API against:

- brute force
- token abuse
- broken object authorization
- mass assignment
- insecure direct object references
- excessive data exposure
- replay where applicable
- missing tenant scope
- injection

Use explicit allowlists for fields accepted by mutation endpoints.

---

# 91. RATE LIMITING

Apply appropriate limits to:

- login
- password recovery
- public forms
- public API
- API keys
- expensive search
- report generation
- webhooks

Make limits configurable per environment and tenant where appropriate.

---

# 92. BACKGROUND JOB SYSTEM

Asynchronous jobs should handle:

- email
- notifications
- asset processing
- document processing
- OCR
- large exports
- reports
- webhook delivery
- scheduled tasks
- AI operations
- imports

Expose job status to the admin when useful.

---

# 93. SCHEDULING

Server-side scheduling for:

- content publication
- content expiration
- report generation
- grant reminders
- task recurrence
- email campaigns
- cleanup jobs
- data-quality checks

Do not rely on browser timers for critical schedules.

---

# 94. OBSERVABILITY

Implement:

- structured application logs
- request IDs
- health endpoints
- readiness endpoint
- liveness endpoint
- background-job visibility
- error reporting hooks
- metrics hooks
- trace hooks

Endpoints:

```text
/health
/ready
```

---

# 95. PERFORMANCE

Optimize for a cost-conscious single-server deployment.

Use:

- proper indexes
- pagination
- query optimization
- caching
- background processing
- image optimization
- lazy loading
- connection pooling
- selective prefetching

Avoid premature optimization and avoid massive unbounded queries.

---

# 96. DATABASE RULES

Use relational normalization for core entities.

Common fields where relevant:

- id
- tenant_id
- created_at
- updated_at
- created_by
- updated_by
- status

Prefer UUIDs or another safe unique ID strategy.

Use:

- foreign keys
- indexes
- unique constraints
- check constraints
- transactions

Do not use soft delete everywhere by default.

Use archival/retention policies where appropriate.

---

# 97. POSTGIS RULES

Use PostGIS for:

- project locations
- office locations
- intervention areas
- geographic boundaries
- spatial searches

Do not expose exact sensitive coordinates unless explicitly authorized.

---

# 98. REDIS RULES

Redis is for:

- cache
- sessions if appropriate
- rate limiting
- temporary state
- job coordination

Redis is not canonical data storage.

---

# 99. OBJECT STORAGE RULES

Use PostgreSQL for metadata and S3/MinIO for binary objects.

Do not store large media files in PostgreSQL blobs by default.

---

# 100. SEARCH INDEXING RULES

Treat external search indexes as rebuildable.

Create indexing jobs from canonical database records.

Provide an administrative rebuild mechanism.

---

# 101. CACHE INVALIDATION

Cache keys must include tenant scope where applicable.

Invalidate caches on:

- content updates
- permission changes
- settings changes
- relevant entity mutations

Never serve private cached data to another user or tenant.

---

# 102. REPORT SECURITY

Reports must inherit permissions from the underlying data sources.

A user who cannot view a project's data must not be able to generate a report containing that data.

---

# 103. EXPORT SECURITY

Before exporting:

- verify permission
- filter tenant
- filter fields
- log sensitive exports

Large exports should run asynchronously.

---

# 104. BACKUP AND DISASTER RECOVERY

Provide architecture and scripts for:

- database backups
- object storage backups
- configuration backups
- encryption where applicable
- retention policies
- restore procedures
- restore testing

Document RPO/RTO assumptions.

---

# 105. DEPLOYMENT

Development baseline:

```text
Docker Compose
├── postgres
├── redis
├── minio
├── api
├── worker
├── web
└── admin
```

Optional:

- reverse proxy
- monitoring

Production should be possible on:

- Ubuntu
- VPS
- dedicated server
- on-premises infrastructure
- Azure
- AWS
- other Docker-compatible providers

---

# 106. ENVIRONMENT CONFIGURATION

Create `.env.example` categories:

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

Do not commit secrets.

Centralize config parsing and validation.

---

# 107. LOCAL DEVELOPMENT

A new developer should be able to:

1. clone repository
2. create `.env`
3. start Compose dependencies
4. run migrations
5. seed demo data
6. start frontend/backend
7. access docs

Provide exact README instructions.

---

# 108. DEMO SEED DATA

Create clearly fictional demo data:

- Demo NGO
- Demo Project
- Demo Program
- Demo Donor
- Demo Partner
- Demo Event
- Demo News
- Demo Forms
- Demo Workflow

Never use real personal information as fake sample data.

---

# 109. ADMIN ONBOARDING

Create onboarding flow:

```text
Create Organization
↓
Organization Details
↓
Branding
↓
Language
↓
Admin Account
↓
Modules
↓
Finish
```

---

# 110. USER INVITATION

Flow:

```text
Admin invites user
↓
Invitation generated
↓
User accepts
↓
User sets credentials
↓
Role/Group activated
```

Invitation links must expire and be single-use.

---

# 111. PUBLIC FORM SECURITY

For public forms support:

- rate limiting
- anti-abuse controls
- spam protection adapter
- validation
- file upload restrictions
- optional CAPTCHA integration

---

# 112. EMAIL ABSTRACTION

Provide provider abstraction supporting:

- SMTP
- transactional email provider

Templates should be:

- tenant-aware
- localized
- versioned

---

# 113. SMS ABSTRACTION

Provider-neutral interface:

```text
SMSProvider.send()
```

Keep provider credentials out of application code.

---

# 114. PAYMENT ABSTRACTION

For fundraising/donations provide a provider abstraction.

Never hard-code business logic for only one payment gateway.

---

# 115. CONTENT SYNDICATION

Prepare optional support for:

- RSS
- Atom
- feeds
- content API
- embeddable widgets
- external site consumption

---

# 116. SOCIAL INTEGRATION ARCHITECTURE

Optional adapters for publishing/analytics on social platforms.

Keep social provider access separate from core content storage.

---

# 117. DATA RETENTION

Retention must be configurable by module and data classification where legally/operationally appropriate.

Deletion should be deliberate, auditable, and blocked when legal/organizational retention applies.

---

# 118. PRIVACY BY DESIGN

For sensitive domains:

- collect minimum necessary fields
- separate sensitive records
- field-level restrictions where required
- audit access
- secure exports
- provide controlled deletion/anonymization mechanisms

---

# 119. MASTER PERMISSION MODEL

Every module must define:

```text
resource
actions
scope
roles
object rules
sensitive fields
export permissions
approval permissions
```

Do not ship a module until its authorization model is defined.

---

# 120. MODULE IMPLEMENTATION CONTRACT

Every module must produce:

## 120.1 Domain

- entities
- value objects where needed
- domain rules
- state model

## 120.2 Database

- tables
- constraints
- indexes
- migrations

## 120.3 API

- CRUD where appropriate
- domain actions
- validation
- permission checks
- pagination
- filtering

## 120.4 UI

- list
- create
- edit
- detail
- approval where relevant
- empty state
- loading state
- error state

## 120.5 Workflow

- states
- transitions
- approvals
- escalations

## 120.6 Events

- created
- updated
- deleted/archived
- state transitions

## 120.7 Notifications

Relevant templates and triggers.

## 120.8 Audit

Important mutations and access events.

## 120.9 Tests

- unit
- integration
- authorization
- tenant isolation
- end-to-end where critical

## 120.10 Documentation

Module README / docs entry.

---

# 121. API DESIGN RULES

Prefer explicit domain actions when a simple CRUD operation cannot safely express the business rule.

Examples:

```text
POST /projects/{id}/submit
POST /projects/{id}/approve
POST /projects/{id}/archive
POST /grants/{id}/close
POST /content/{id}/publish
```

rather than allowing arbitrary direct status changes that bypass workflow.

---

# 122. FRONTEND DESIGN RULES

Do not implement all business rules only in the UI.

Frontend responsibilities:

- presentation
- interaction
- optimistic updates only where safe
- client validation for UX
- permissions-aware display
- loading/error/empty states

Backend remains authoritative.

---

# 123. DATA TABLE COMPONENT

Build reusable data table infrastructure with:

- search
- filters
- sorting
- pagination
- column visibility
- bulk selection
- bulk actions
- export
- saved views
- responsive mode

Reuse it across modules.

---

# 124. FORM COMPONENT SYSTEM

Use React Hook Form + Zod and backend schema alignment where practical.

Every form must support:

- field errors
- server errors
- disabled states
- loading state
- success state
- unsaved changes warning when relevant
- RTL

---

# 125. DESIGN SYSTEM

Create shared components:

```text
Button
Input
Textarea
Select
Combobox
DatePicker
Dialog
Drawer
Dropdown
Tabs
Toast
Alert
Card
Table
DataTable
Badge
StatusBadge
Form
Upload
RichTextEditor
Map
Chart
Timeline
Breadcrumb
Pagination
EmptyState
ErrorState
Skeleton
```

---

# 126. ERROR / EMPTY / LOADING STATES

Every data-heavy page must explicitly handle:

- loading
- empty
- partial
- error
- unauthorized
- not found

Do not leave blank white screens.

---

# 127. DOCUMENTATION SET

Create/maintain:

```text
README.md
ARCHITECTURE.md
DATABASE.md
API.md
SECURITY.md
DEPLOYMENT.md
DEVELOPMENT.md
MODULES.md
WORKFLOWS.md
PLUGIN_SYSTEM.md
AI.md
INTEGRATIONS.md
BACKUP.md
CONTRIBUTING.md
CHANGELOG.md
```

Also maintain module-level documentation.

---

# 128. DATABASE DOCUMENTATION

Document:

- major entities
- relationships
- tenant boundaries
- sensitive data areas
- important indexes
- archival rules
- migration strategy

Prefer an ERD or Mermaid diagram in documentation where useful.

---

# 129. API DOCUMENTATION

OpenAPI documentation should be generated from the actual implementation.

Do not hand-write an API document that diverges from the source.

---

# 130. TESTING STRATEGY

## Backend

- unit tests
- integration tests
- database tests
- API tests
- authorization tests
- tenant isolation tests
- workflow tests

## Frontend

- unit/component tests
- form tests
- state tests
- critical interaction tests

## E2E

Use a suitable tool such as Playwright if justified.

Critical flows must be automated.

---

# 131. REQUIRED E2E FLOWS

At minimum:

### Authentication

```text
Invitation → Login → Logout → Password Recovery
```

### CMS

```text
Create Page → Draft → Review → Approve → Publish → Public View
```

### Project

```text
Create Program → Create Project → Add Activity → Add Indicator → Report
```

### Grants

```text
Funding Opportunity → Proposal → Approval → Grant → Report → Closure
```

### Forms

```text
Create Form → Public Submission → Review → Action
```

### Documents

```text
Upload → Version → Approve → Secure Download → Audit
```

### Security

```text
Tenant A User → Attempt Tenant B Resource → Denied
```

---

# 132. SECURITY TEST MATRIX

Explicitly test:

- broken object authorization
- broken tenant isolation
- privilege escalation
- insecure direct object references
- mass assignment
- file upload abuse
- rate-limit bypass
- stale token use
- revoked API key use
- hidden admin endpoint access
- search permission bypass
- report permission bypass
- export permission bypass

---

# 133. CI/CD

CI pipeline should run at minimum:

```text
lint
format checks
type checks
tests
build
migration validation
security checks
```

Use reproducible builds.

---

# 134. RELEASE STRATEGY

Use semantic-ish versioning and maintain a changelog.

Suggested:

```text
0.x = active platform development
1.0 = stable core platform
1.x = backward-compatible feature releases
```

Document breaking changes.

---

# 135. FEATURE FLAGS

For complex or unfinished modules, use feature flags where appropriate.

Never fake a feature as complete.

Features marked incomplete must be visibly disabled or clearly designated in non-production environments.

---

# 136. INITIAL IMPLEMENTATION PHASES

## Phase 0 — Repository + Architecture

Deliver:

- workspace audit
- architecture docs
- repository structure
- Compose setup
- configuration

## Phase 1 — Platform Foundation

Deliver:

- PostgreSQL
- PostGIS
- Redis
- MinIO
- FastAPI
- Next.js
- identity
- tenant model
- users
- roles
- permissions
- audit
- admin shell

## Phase 2 — CMS

Deliver:

- pages
- content
- taxonomy
- media
- documents
- navigation
- localization
- workflow
- SEO
- public site

## Phase 3 — NGO Core

Deliver:

- organization
- programs
- projects
- activities
- donors
- partners
- grants
- events
- tasks

## Phase 4 — MEAL + Forms

Deliver:

- indicators
- results framework
- surveys
- forms
- submissions
- monitoring
- dashboards

## Phase 5 — Workflow + Automation + DMS

Deliver:

- workflow builder
- automation
- notifications
- document versioning
- approval center

## Phase 6 — GIS + Reporting + Analytics

Deliver:

- PostGIS features
- map UI
- report builder
- dashboard builder
- analytics abstraction

## Phase 7 — Advanced Operations

Deliver progressively:

- beneficiary management
- case management
- safeguarding
- complaints
- volunteers
- HR
- procurement
- assets
- fleet
- travel
- membership
- fundraising

## Phase 8 — API Platform

Deliver:

- API keys
- scopes
- webhooks
- developer portal
- SDK generation
- integration framework

## Phase 9 — AI

Deliver:

- provider abstraction
- model registry
- prompt templates
- usage tracking
- AI assistance
- knowledge assistant
- governance

---

# 137. VERTICAL SLICE RULE

For each phase, implement complete vertical slices.

Example:

```text
Project
↓
Database
↓
Domain Service
↓
API
↓
Permission Check
↓
Frontend List
↓
Frontend Detail
↓
Form
↓
Audit
↓
Tests
```

Do not implement 100 database tables first and postpone the application layer.

---

# 138. NO-BLOCKER RULE

When a feature depends on another feature, implement the smallest valid foundation for that dependency rather than building a large speculative system.

Example:

Do not build a complete accounting ERP merely because projects need budgets.

Build the project budget layer and an accounting integration boundary.

---

# 139. NO OVERENGINEERING RULE

Do not introduce on day one:

- Kubernetes
- Kafka
- NATS
- service mesh
- Neo4j
- Qdrant
- Milvus
- complex distributed orchestration

unless a concrete measured requirement justifies it.

---

# 140. SERVICE EXTRACTION READINESS

Even as a modular monolith, keep clear boundaries so these can be extracted later:

- search
- file processing
- AI gateway
- notifications
- analytics
- integrations

---

# 141. EVENT CATALOG

Define an internal event naming standard.

Examples:

```text
content.created
content.updated
content.submitted
content.approved
content.published
project.created
project.updated
project.submitted
project.approved
grant.created
grant.approved
grant.closed
form.submitted
document.uploaded
document.approved
user.invited
user.activated
```

Document event payload schemas.

---

# 142. EVENT SAFETY

Events should not contain secrets or unnecessary sensitive personal data.

Where possible, pass identifiers and fetch authorized data in the consumer rather than duplicating entire records.

---

# 143. REQUEST ID / TRACE ID

Every API request should have a request ID.

Carry the request ID into:

- logs
- audit records where useful
- background jobs where useful
- webhook delivery logs

---

# 144. AUDIT VS ACTIVITY

Do not confuse:

**Audit Log:** security/compliance-oriented record.

**Activity Feed:** user-facing operational history.

They are separate concepts and have different access rules.

---

# 145. DATA ACCESS LAYER

For protected resources, centralize authorization-aware query patterns where practical.

Avoid writing repeated ad-hoc tenant filters everywhere that can be accidentally forgotten.

---

# 146. BULK ACTION SAFETY

Bulk actions must:

- show selection count
- validate permission
- validate allowed state transitions
- process safely
- report failures
- audit sensitive actions

---

# 147. IMPORT SAFETY

Imports must not bypass:

- authorization
- validation
- uniqueness
- audit rules
- workflow requirements

---

# 148. REPORT GENERATION ARCHITECTURE

Use data-source abstractions.

Report definitions should not embed unsafe arbitrary SQL.

Provide a controlled query/report layer.

---

# 149. CUSTOM REPORTS

Eventually allow authorized users to configure:

- selected fields
- filters
- grouping
- sorting
- aggregation
- charts

but ensure all fields are permission-aware.

---

# 150. CUSTOM DASHBOARDS

Dashboard widgets should declare:

- data source
- permission requirements
- filters
- refresh behavior

---

# 151. JOB MONITORING

Admin should be able to see relevant background jobs:

- queued
- running
- completed
- failed
- retried

Display error summaries without exposing secrets.

---

# 152. SYSTEM ADMINISTRATION

Settings pages should include:

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

Only expose settings appropriate to the user's role.

---

# 153. SYSTEM HEALTH DASHBOARD

Display:

- API status
- database status
- Redis status
- object storage status
- worker status
- email status
- search status
- integration status

This is operational health information, not a replacement for observability infrastructure.

---

# 154. ROUTING / DOMAIN CONFIGURATION

Do not hard-code a public production hostname.

Use environment/configuration for:

- application URL
- API URL
- public site URL
- admin URL
- callback URLs
- allowed origins
- webhook URLs

---

# 155. EMAIL TEMPLATES

Include templates for:

- invitation
- email verification
- password reset
- workflow assignment
- approval request
- approval result
- deadline reminder
- report ready
- webhook failure where appropriate

Templates must support Arabic and English.

---

# 156. TRANSLATION WORKFLOW

Support states for translated content:

```text
Not Started
In Progress
Reviewed
Approved
Published
```

Do not let translated content silently replace source-language content.

---

# 157. CONTENT RELATIONS

Allow relationships such as:

```text
Article ↔ Project
Project ↔ Program
Project ↔ Donor
Project ↔ Partner
Report ↔ Project
Document ↔ Grant
Event ↔ Program
Story ↔ Beneficiary
```

Sensitive relationships must obey permissions.

---

# 158. TAXONOMY

Create reusable:

- categories
- tags
- sectors
- topics
- classifications

Support hierarchy where needed.

---

# 159. URL REDIRECT MANAGER

Provide admin tools for:

- redirects
- aliases
- deprecated URLs
- import of redirect mappings

Track broken and redirected URLs.

---

# 160. CONTENT PREVIEW

Editors should be able to preview:

- current draft
- future scheduled version
- localized version
- mobile layout

Preview URLs must be protected.

---

# 161. CONTENT LOCKING

Implement concurrency protection for editing shared records.

At minimum:

- updated timestamp checks
- optimistic concurrency or locking strategy
- clear conflict message

---

# 162. DOCUMENT SHARE MODEL

Private documents may be shared through:

- permission grant
- time-limited signed URL
- portal access

Every sensitive sharing action should be auditable.

---

# 163. PORTAL ACCOUNT MODEL

Portal users should not require full internal staff privileges.

Create explicit external-user roles.

Examples:

- partner_user
- donor_user
- applicant_user
- volunteer_user
- member_user

---

# 164. PUBLIC / PRIVATE DATA MODEL

Where appropriate, content entities may include publication visibility:

```text
public
internal
restricted
```

But security must not depend only on one visibility field; authorization remains authoritative.

---

# 165. DATA FILTERING

List and search endpoints should support secure filters, including where relevant:

- status
- date
- sector
- organization unit
- project
- program
- donor
- partner
- location
- category
- tag

---

# 166. PAGINATION

Never return unbounded record collections.

Implement cursor or page-based pagination as appropriate.

Document defaults and maximum limits.

---

# 167. SORTING

Only allow sorting by indexed/approved fields.

Do not expose arbitrary SQL order-by clauses.

---

# 168. SEARCH QUERY SAFETY

Search input must be normalized and safely parameterized.

Prevent search filters from becoming raw SQL.

---

# 169. FILE THUMBNAILS

Generate common image renditions asynchronously.

Avoid processing giant originals inside API request time.

---

# 170. PDF / OCR PREPARATION

Prepare document pipeline for optional:

- PDF text extraction
- OCR
- metadata extraction
- indexing
- language detection

Do not make OCR a hard dependency for the initial platform.

---

# 171. KNOWLEDGE SEARCH PREPARATION

Documents and knowledge records should have stable identifiers so future embeddings can reference them without altering the domain model.

---

# 172. AI SOURCE TRACEABILITY

When AI uses organization content, retain references to the source records used.

AI-generated answers should distinguish:

- source-backed information
- generated wording
- uncertain content

---

# 173. AI DATA BOUNDARIES

AI tools must use the same authorization layer as ordinary users.

Do not allow an AI assistant to search restricted records that the requesting user cannot access.

---

# 174. AI PROVIDER ABSTRACTION

Support multiple providers through a consistent interface.

Provider configuration should be tenant/environment-aware.

Do not hard-code one model/provider into business domains.

---

# 175. COST / USAGE TRACKING

Optional AI usage record fields:

- tenant
- user
- provider
- model
- request time
- input tokens if available
- output tokens if available
- estimated cost if available
- operation type

---

# 176. FUTURE DATA INTELLIGENCE

Keep future-ready abstractions for:

- semantic search
- entity extraction
- relationship discovery
- duplicate detection
- forecasting
- anomaly detection
- knowledge graph export

These are future capabilities, not reasons to overcomplicate the initial database.

---

# 177. MOBILE API READINESS

All mobile-relevant operations should use the same documented API.

Do not create a private mobile-only backend unless needed.

---

# 178. API VERSIONING

Use:

```text
/api/v1/
```

Plan for future:

```text
/api/v2/
```

Do not break `/v1` contracts casually.

---

# 179. DEPRECATION POLICY

For deprecated APIs or fields:

- document
- warn where appropriate
- define replacement
- provide migration guidance
- remove only in a planned breaking release

---

# 180. CHANGE MANAGEMENT

Maintain:

- changelog
- migrations
- release notes
- breaking-change documentation

---

# 181. ACCEPTANCE CRITERIA PER MODULE

A module is not complete until:

```text
[ ] Database model implemented
[ ] Migration works
[ ] Domain logic implemented
[ ] API implemented
[ ] Authorization implemented
[ ] Tenant isolation tested
[ ] UI implemented
[ ] Loading/empty/error states implemented
[ ] Audit implemented
[ ] Events defined
[ ] Notifications defined where applicable
[ ] Tests written
[ ] Documentation written
```

---

# 182. PHASE EXIT CRITERIA

A phase is complete only when:

- the application builds
- migrations succeed
- core workflows work end-to-end
- tests pass
- authorization tests pass
- tenant isolation tests pass
- documentation is updated
- no obvious placeholder functionality remains

---

# 183. NO FAKE COMPLETION

Never report:

> "Implemented"

when the feature is only:

- a UI mock
- a schema stub
- a TODO
- an empty endpoint
- a static fake dataset
- a button without business logic

Use exact status:

```text
Implemented
Partially Implemented
Not Implemented
Blocked
```

---

# 184. DEVELOPER EXPERIENCE

Provide useful scripts such as:

```text
install
lint
format
typecheck
test
test:e2e
build
dev
migrate
seed
backup
restore
```

Use the project's actual package manager and runtime conventions.

---

# 185. README REQUIREMENTS

README must include:

- project overview
- architecture
- requirements
- local setup
- environment variables
- migrations
- seed
- tests
- build
- deployment
- default demo account policy
- API documentation

Never publish real credentials.

---

# 186. DEFAULT DEMO ACCESS

If demo credentials are created, clearly identify them as development-only.

Do not reuse development passwords in production.

---

# 187. MIGRATION STRATEGY

All schema changes use migrations.

Seed scripts must be separate from migrations.

Production startup must not automatically run destructive seed/reset operations.

---

# 188. DATABASE TRANSACTION RULES

Use transactions for multi-step state changes requiring consistency.

Examples:

- grant approval + audit
- project state transition + event
- role change + audit

Be deliberate about transaction boundaries.

---

# 189. IDEMPOTENCY

Important operations should support idempotency where retries can otherwise duplicate effects:

- webhooks
- payments
- imports
- external synchronization
- scheduled jobs

---

# 190. CONCURRENCY

Protect shared mutations against:

- duplicate submission
- double approval
- stale edits
- repeated webhook handling
- concurrent imports

---

# 191. INTEGRATION RETRY POLICY

Every external integration must define:

- timeout
- retry policy
- idempotency strategy
- failure logging
- operator visibility
- dead-letter/retry status where justified

---

# 192. AUDIT RETENTION

Audit retention should be configurable and documented.

Sensitive audit records may require stricter protection than normal operational activity.

---

# 193. ADMIN SEARCH / FILTER SAVING

Allow users to save useful filtered views where appropriate.

Saved views must not store information that bypasses permissions.

---

# 194. ADMIN PERSONALIZATION

Support optional user preferences:

- language
- theme
- timezone
- default dashboard
- table columns
- saved views
- notification preferences

---

# 195. MULTI-CURRENCY

Support organizational currencies and project/grant currencies.

Do not assume USD is the only currency.

Store currency explicitly with monetary values.

---

# 196. TIMEZONE

Store timestamps consistently, preferably UTC at the database layer.

Render using user/tenant timezone settings.

---

# 197. DATE / NUMBER LOCALIZATION

Support localized display while maintaining unambiguous canonical storage.

---

# 198. FILE NAMING

Never use the original filename directly as the storage key.

Preserve the original name only as metadata.

---

# 199. SECURITY HEADERS

Prepare production responses for appropriate headers such as:

- Content-Security-Policy
- Referrer-Policy
- X-Content-Type-Options
- Permissions-Policy
- HSTS when HTTPS is correctly deployed

Do not introduce insecure wildcard policies just to make development work.

---

# 200. CORS

Explicitly configure allowed origins per environment.

Do not use unrestricted `*` for authenticated production APIs unless the architecture genuinely requires it and security implications are understood.

---

# 201. COOKIE / TOKEN POLICY

If cookies are used:

- Secure in production
- HttpOnly where appropriate
- SameSite policy

If bearer tokens are used:

- short lifetimes where practical
- refresh strategy
- revocation strategy
- secure storage guidance for clients

---

# 202. API DOCUMENTATION QUALITY

OpenAPI descriptions should include:

- endpoint purpose
- auth requirement
- permission requirement
- parameters
- request examples
- response examples
- error examples

---

# 203. DOMAIN NAMING

Use consistent English identifiers in code.

Use Arabic/English localized labels in UI.

Never mix localized labels into database column names unless a deliberate CMS content model requires it.

---

# 204. PROJECT CODE GENERATION

Project codes should be unique within the tenant and optionally support an organization-defined format.

---

# 205. RECORD ARCHIVING

Archived records should remain queryable by authorized users without appearing in ordinary active views.

---

# 206. REST VS GRAPHQL

REST/OpenAPI is the primary contract.

GraphQL may be added later only when it solves a demonstrated aggregation/read problem.

Do not implement both completely in v1 without a concrete need.

---

# 207. SEARCH PROVIDER ABSTRACTION

Define an interface such as:

```text
SearchProvider.index()
SearchProvider.search()
SearchProvider.delete()
SearchProvider.rebuild()
```

The initial provider can use PostgreSQL.

---

# 208. STORAGE PROVIDER ABSTRACTION

Define a provider interface for:

- MinIO
- S3
- Azure Blob

Core business logic should not depend directly on one vendor SDK.

---

# 209. EMAIL PROVIDER ABSTRACTION

Likewise isolate email provider implementation.

---

# 210. MAP PROVIDER ABSTRACTION

Public/admin map components should not be tightly coupled to a single external map vendor.

---

# 211. AI PROVIDER ABSTRACTION

AI calls must go through a provider/model abstraction.

---

# 212. INTEGRATION CREDENTIALS

Store external integration credentials through secure secrets/config mechanisms.

Do not return secrets in ordinary GET endpoints.

---

# 213. ADMIN SECRET DISPLAY

When a secret is generated:

- show once
- allow copy
- provide revoke/rotate
- never display the full secret later

---

# 214. ROTATION

Provide rotation flows for:

- API keys
- webhook secrets
- integration credentials where supported

---

# 215. USER SESSION MANAGEMENT

Provide a session/security page showing authorized users' own active sessions.

Allow sign-out of individual sessions.

Admins may have separate organization-level session management subject to policy.

---

# 216. SECURITY EVENT CENTER

Prepare a security events view for:

- repeated login failure
- password reset
- role change
- API key creation/revocation
- suspicious token/session events

---

# 217. ORGANIZATION SETTINGS ISOLATION

Tenant administrators can configure only their tenant.

Platform administrators, if enabled, may manage platform-level settings.

Never mix these authority levels accidentally.

---

# 218. PLATFORM ADMIN VS TENANT ADMIN

Clearly separate:

```text
Platform Admin
Tenant Admin
Staff User
External Portal User
Public User
```

---

# 219. PUBLIC SITE CACHE

Public content may be cached aggressively where safe.

Authenticated/private pages must use secure cache boundaries.

---

# 220. SITEMAP FILTERING

Only public content should appear in public sitemaps.

---

# 221. ROBOTS POLICY

Admin portals should not be indexed publicly.

---

# 222. PORTAL DOMAIN MAPPING

Design for:

```text
ngo.example.org
admin.ngo.example.org
portal.ngo.example.org
campaign.example.org
```

without hard-coding a specific organization.

---

# 223. MICROSITE ISOLATION

Microsites share platform data but must have distinct:

- content scope
- theme
- navigation
- domain
- publication settings

---

# 224. MULTI-ORGANIZATION USER

A user may potentially belong to multiple tenants, if enabled.

Provide an organization switcher with strict session/authorization boundaries.

---

# 225. CROSS-TENANT PLATFORM SERVICES

Platform-wide analytics must never expose tenant data to tenant users.

Only platform administrators may view aggregate platform-level analytics where appropriate.

---

# 226. SYSTEM CONFIGURATION HIERARCHY

Recommended precedence:

```text
Platform defaults
↓
Tenant settings
↓
Module settings
↓
User preferences
```

Do not allow a lower-level configuration to weaken mandatory security settings.

---

# 227. FEATURE AVAILABILITY

Modules may be:

```text
Available
Enabled
Disabled
Beta
Deprecated
```

The UI should respond accordingly.

---

# 228. PRODUCT ANALYTICS

Optional tenant-scoped product analytics should track functional usage without collecting unnecessary personal data.

---

# 229. PRIVACY OF ANALYTICS

Analytics collection itself must respect tenant configuration and applicable policies.

---

# 230. REPORT SCHEDULING

Authorized users should be able to schedule recurring reports where appropriate.

Example:

```text
Monthly Project Progress Report
Recipient: Program Manager
Format: PDF
Schedule: First day of month
```

---

# 231. DEADLINE ENGINE

Create shared deadline abstractions for:

- grant deadlines
- report deadlines
- task deadlines
- document expiry
- certification expiry
- contract expiry

These can feed notifications and dashboards.

---

# 232. SLA ENGINE

Support optional SLA definitions:

- target response time
- escalation threshold
- responsible role
- escalation path

Useful for:

- complaints
- support requests
- approvals
- case management

---

# 233. APPROVAL MATRIX

Where appropriate, approvals can depend on:

- amount
- department
- project
- grant
- role
- risk level

Example concept:

```text
Budget <= X → Manager
Budget > X → Director
Budget > Y → Board/Authorized Committee
```

Keep thresholds configurable.

---

# 234. DELEGATION

Support temporary delegation of approval responsibilities, with audit logging and expiration.

---

# 235. ESCALATION

Support:

- reminder
- escalation to manager
- reassignment
- SLA breach event

---

# 236. NOTIFICATION DIGESTS

Optional daily/weekly summaries can reduce notification noise.

---

# 237. IN-APP SEARCH SHORTCUTS

Expose keyboard-friendly search/command affordances in admin where practical.

---

# 238. ADMIN DASHBOARD CUSTOMIZATION

Allow authorized users to choose widgets without allowing them to bypass permissions.

---

# 239. CONTENT ANALYTICS

Optional public-content analytics may track:

- page views
- downloads
- events registrations
- campaign actions

Keep analytics provider-neutral.

---

# 240. GDPR/PRIVACY-STYLE CAPABILITIES

Without assuming a specific jurisdiction, prepare optional support for:

- data export request
- data correction
- anonymization
- deletion request
- retention rules
- consent metadata
- processing notes

Legal interpretation must remain organization-controlled.

---

# 241. CONSENT RECORDS

If a form or service collects consent, record:

- purpose
- text/version
- time
- actor/subject where appropriate
- source

Do not treat consent as a universal replacement for all other legal/organizational requirements.

---

# 242. RECORD LINKING

Many domains need relationships.

Provide generic relationship patterns where justified, while keeping strong typed relationships for core data.

---

# 243. ENTITY REFERENCE UI

Create reusable reference selectors for:

- users
- organizations
- projects
- donors
- partners
- locations
- documents

Selectors must enforce authorization.

---

# 244. BULK IMPORT DUPLICATE DETECTION

Support matching strategies such as:

- exact ID
- exact email
- exact project code
- configurable composite keys

Do not auto-merge high-impact entities without human confirmation.

---

# 245. DATA QUALITY DASHBOARD

Admin view can include:

- duplicate candidates
- incomplete records
- invalid references
- stale records
- failed imports

---

# 246. ADMIN DOCUMENTATION LINKS

Each complex module should offer contextual documentation/help links where useful.

---

# 247. IN-APP HELP

Prepare optional contextual help:

- field descriptions
- tooltips
- documentation links
- onboarding tips

---

# 248. ERROR TRANSLATION

Backend error codes should be stable and frontend-localized messages should be provided where appropriate.

Do not make clients parse arbitrary human prose to determine behavior.

---

# 249. REQUEST VALIDATION

Validate:

- body
- query params
- path params
- files
- headers where relevant

Use typed schemas.

---

# 250. DATABASE CONSTRAINTS VS BUSINESS RULES

Use database constraints for hard invariants.

Use domain logic for more contextual business rules.

Do not rely on only one layer for important invariants.

---

# 251. TRANSACTIONAL EMAIL VS NEWSLETTER

Keep transactional email infrastructure separate from bulk newsletter sending where possible.

---

# 252. EMAIL DELIVERABILITY

Document:

- SMTP/provider configuration
- verified sender domain
- bounce handling architecture
- unsubscribe behavior for newsletters

---

# 253. MEDIA RIGHTS

Assets may have different usage rights.

Expose rights metadata in the asset model and admin UI.

---

# 254. EXPIRING ASSETS

Allow asset license/permission expiry reminders.

---

# 255. CONTENT OWNERSHIP

Each content item may have:

- author
- editor
- owner
- reviewer
- approver

These roles must be configurable.

---

# 256. AUTHORSHIP

Keep author attribution distinct from editor/approver roles.

---

# 257. DOCUMENT REVIEW CYCLES

Policies/manuals should support a next-review date and assigned reviewer.

---

# 258. PROJECT CLOSURE

Project closure may require:

- completion check
- required reports
- indicator final values
- asset handling
- document archive
- lessons learned
- final approval

Make this workflow configurable.

---

# 259. GRANT CLOSURE

Grant closure may require:

- financial report
- narrative report
- deliverables
- outstanding issues
- final approval

---

# 260. PARTNER OFFBOARDING

Partner deactivation may need:

- agreement expiry
- project closure
- document archive
- access revocation

---

# 261. USER OFFBOARDING

When deactivating a staff account:

- revoke active sessions
- revoke API keys where appropriate
- reassign tasks
- reassign approvals
- preserve audit history
- preserve ownership records

---

# 262. RECORD OWNERSHIP TRANSFER

Support controlled transfer of ownership rather than destructive deletion.

---

# 263. ARCHIVE SEARCH

Authorized users should be able to search archived records explicitly.

---

# 264. PUBLIC REPORT LIBRARY

Organizations may create a public library of reports/publications with:

- metadata
- categories
- tags
- download
- preview
- language
- publication date

---

# 265. PUBLIC PROJECT DIRECTORY

Projects marked public should have public detail pages with safe fields only.

---

# 266. PUBLIC EVENT REGISTRATION

Event registration should use the Form Engine rather than a one-off implementation.

---

# 267. PUBLIC JOBS

Vacancies should support:

- job description
- requirements
- deadline
- location
- application form
- attachment requirements
- status

---

# 268. PUBLIC TENDERS

Tenders should support:

- notice
- deadline
- documents
- clarification contacts
- addenda
- closure

---

# 269. ADDENDA / AMENDMENTS

Grants, tenders, and agreements may need amendment/version history.

Use explicit version entities rather than overwriting original content.

---

# 270. PORTAL DOCUMENT EXCHANGE

Partner/donor portals may allow secure document exchange with:

- upload
- download
- status
- review
- request changes
- audit

---

# 271. PORTAL MESSAGE CENTER

External portals may include controlled message threads.

Do not expose internal-only notes.

---

# 272. INTERNAL VS EXTERNAL NOTES

For cases, partners, projects, and grants, distinguish:

```text
Internal Note
External Note
```

Never rely on naming alone; enforce separate visibility rules.

---

# 273. COMMENTING

Provide threaded comments for collaborative workflows where useful.

Support moderation/deletion permissions and audit where relevant.

---

# 274. MENTIONS

Support user mentions in internal collaboration features, with notification preferences.

---

# 275. ATTACHMENTS

Attachments must respect the parent object's permissions and retention policy.

---

# 276. CASE CONFIDENTIALITY

Case records should have explicit confidentiality levels and restricted viewers where needed.

---

# 277. SAFEGUARDING CONFIDENTIALITY

Safeguarding records must have the narrowest practical access scope.

---

# 278. PUBLIC DATA REDACTION

When publishing reports/records, provide mechanisms for safe redaction rather than exposing raw internal records.

---

# 279. ANONYMIZED ANALYTICS

Aggregate analytics should avoid re-identification of sensitive individuals.

---

# 280. GEO REDACTION

Sensitive geographic coordinates can be generalized for public outputs.

---

# 281. FORM VERSIONING

Forms should be versioned when question structure changes.

Historical submissions must retain their original form version context.

---

# 282. SURVEY DATA MODEL

Separate:

- form definition
- form version
- submission
- response value

Do not destroy old form definitions when editing forms.

---

# 283. INDICATOR DISAGGREGATION

Prepare optional dimensions such as:

- sex
- age group
- location
- other organization-defined categories

Do not hard-code sensitive demographic assumptions into all installations.

---

# 284. MEAL EVIDENCE

Indicators may reference:

- survey submission
- monitoring visit
- document
- dataset
- observation

Keep evidence traceability.

---

# 285. PROJECT TIMELINE

Provide project milestones and events in chronological views.

---

# 286. GRANT CALENDAR

Provide grant deadlines in calendar/list views.

---

# 287. ORGANIZATION CALENDAR

Unified calendar may combine:

- events
- meetings
- project milestones
- deadlines
- tasks
- trainings

with permission-aware filtering.

---

# 288. SAVED FILTERS

Allow users to save filter combinations on large data lists.

---

# 289. EXPORT FORMATS

Implement export adapters so new formats can be added later.

---

# 290. SCHEDULED EXPORTS

Optional scheduled exports must be secure and auditable.

---

# 291. API USAGE METRICS

For API keys/application clients, track:

- request count
- rate-limit events
- last used
- endpoint usage where appropriate

Do not retain unnecessary personal data.

---

# 292. API QUOTAS

Support optional tenant/application quotas.

---

# 293. WEBHOOK REPLAY

Authorized operators may replay failed webhook deliveries after resolving the underlying issue.

---

# 294. INTEGRATION STATUS

Show configured integrations with:

- enabled/disabled
- last successful sync
- last failure
- next retry

---

# 295. SYNC LOG

External sync operations must be inspectable.

---

# 296. EXTERNAL ID MAPPING

For integrations, store provider identifiers without replacing internal IDs.

---

# 297. DATA SOURCE METADATA

Imported/external records should optionally retain:

- source system
- source ID
- imported timestamp
- sync status

---

# 298. DATA LINEAGE

For significant imports/reports/AI outputs, maintain traceability to the source record or job.

---

# 299. DOCUMENT OCR / EXTRACTION

When OCR is enabled, store:

- extraction status
- extracted text version
- language
- processing timestamp
- processor metadata

Do not replace the original document.

---

# 300. SEARCH REBUILD

Provide administrative commands/jobs to:

- rebuild all indexes
- rebuild a tenant
- rebuild a record type
- report failures

---

# 301. CACHE REBUILD / INVALIDATION

Provide safe mechanisms to clear or rebuild derived caches.

---

# 302. OBJECT STORAGE RECONCILIATION

Provide a maintenance job to detect:

- database references to missing objects
- orphaned objects where safely detectable

Do not automatically delete objects without an explicit safe policy.

---

# 303. SYSTEM MAINTENANCE MODE

Prepare an optional maintenance-mode mechanism for planned operations.

Maintenance mode must not prevent emergency administrative access unless explicitly configured.

---

# 304. READINESS DEPENDENCIES

`/ready` should reflect critical dependency availability relevant to the application.

---

# 305. MIGRATION SAFETY

Migrations must:

- be reviewed
- be idempotent where feasible
- avoid destructive changes without explicit migration steps
- support upgrade ordering

---

# 306. SEED SAFETY

Seed command must clearly indicate whether it is safe for development only.

Never auto-seed production with demo data.

---

# 307. CONFIGURATION VALIDATION

At startup, validate critical settings and fail clearly if required secrets/configuration are missing.

---

# 308. ERROR MESSAGES

Errors should be:

- actionable for operators
- safe for end users
- localized at presentation layer where appropriate

---

# 309. LOG REDACTION

Implement or document redaction for:

- authorization headers
- cookies
- tokens
- passwords
- API keys
- sensitive fields

---

# 310. SECURITY DEPENDENCY MANAGEMENT

Keep dependencies current enough for supported security fixes.

Use automated dependency scanning where practical.

---

# 311. LICENSE / OPEN SOURCE

Keep third-party licenses documented where required.

Do not copy proprietary assets or content without rights.

---

# 312. CONTENT RIGHTS

The CMS must support metadata for:

- copyright owner
- license
- source
- attribution
- usage restrictions

---

# 313. MEDIA MODERATION

If public uploads are enabled, provide moderation state and access controls.

---

# 314. PUBLIC USER REGISTRATION

Public registration must be explicitly configurable.

Do not enable broad self-registration for internal staff by default.

---

# 315. ACCOUNT APPROVAL

External portal accounts may require approval depending on portal type.

---

# 316. PASSWORD POLICIES

Tenant security policies should be configurable within safe platform limits.

---

# 317. MFA POLICIES

Support tenant-level requirements such as:

- optional MFA
- required MFA for administrators
- required MFA for privileged roles

---

# 318. SESSION POLICIES

Support configurable:

- session duration
- idle timeout
- concurrent session limits where appropriate

---

# 319. IP / NETWORK POLICIES

Optional enterprise feature for:

- allowed IP ranges
- admin network restrictions

Must not be required for ordinary deployments.

---

# 320. SECURITY INCIDENT RECORD

Prepare optional entity:

- SecurityIncident

with restricted access and audit history.

---

# 321. OPERATIONAL INCIDENT MANAGEMENT

Optional generalized incident entity for:

- system incident
- project incident
- operational incident

Separate from safeguarding and security incidents where confidentiality differs.

---

# 322. RISK REGISTER

Entities:

- Risk
- RiskAssessment
- MitigationAction
- RiskReview

Fields:

- probability
- impact
- severity/score
- owner
- mitigation
- due date
- residual risk

Scoring logic should be configurable.

---

# 323. COMPLIANCE REGISTER

Entities:

- ComplianceRequirement
- ComplianceItem
- ComplianceReview
- Evidence
- DueDate
- ComplianceStatus

Useful for:

- donor compliance
- internal policies
- certifications
- agreements

---

# 324. CERTIFICATION TRACKING

Track:

- certification
- issuing body
- issue date
- expiry
- evidence
- owner

Useful for staff, organization, vehicles, assets, and partners.

---

# 325. TRAINING MANAGEMENT

Entities:

- Training
- TrainingSession
- Trainer
- Enrollment
- Attendance
- Certificate

Can serve staff and volunteers.

---

# 326. COMMUNITY ENGAGEMENT

Optional domain for:

- consultations
- public meetings
- surveys
- feedback
- community events

Reuse Event/Form/Feedback infrastructure.

---

# 327. MEDIA RELATIONS

Entities:

- MediaContact
- PressRequest
- InterviewRequest
- PressEvent

---

# 328. STORYTELLING

Stories may relate to:

- project
- program
- impact
- publication

Sensitive subject information must be permission-aware.

---

# 329. PUBLICATION PIPELINE

Support:

```text
Draft
→ Editorial Review
→ Fact/Quality Review
→ Approval
→ Publish
```

Organizations can configure additional steps.

---

# 330. TRANSPARENCY / OPEN DATA

Optional public exports may expose structured project/funding data after explicit publication approval.

---

# 331. API PUBLICATION CONTROLS

Do not assume all CMS content is API-public.

Explicitly classify public API resources.

---

# 332. PRIVATE API

Internal API endpoints must use normal authentication and authorization.

---

# 333. API FIELD SELECTION

Where field selection is implemented, use a safe allowlist rather than arbitrary serialization access.

---

# 334. RATE LIMIT HEADERS

Where useful, communicate rate-limit state through standard response headers.

---

# 335. API IDEMPOTENCY KEYS

Support idempotency keys for appropriate write operations that may be retried by clients.

---

# 336. API BULK ENDPOINTS

Bulk endpoints should return per-item results where practical.

---

# 337. ASYNC API OPERATIONS

For long-running requests, return a job reference rather than holding the request indefinitely.

---

# 338. JOB API

Optional endpoints:

```text
GET /api/v1/jobs/{job_id}
```

Only expose jobs to authorized users.

---

# 339. FRONTEND DATA FETCHING

Use TanStack Query for server state.

Do not mirror every API response into Zustand.

---

# 340. FRONTEND CACHING

Cache keys must include relevant tenant/context boundaries.

---

# 341. UI PERMISSION MODEL

Frontend should receive enough permission context to:

- hide unavailable actions
- disable unavailable actions
- show read-only states

But backend remains authoritative.

---

# 342. UI ROUTING

Routes should reflect module boundaries.

Example:

```text
/admin/projects
/admin/projects/[id]
/admin/grants
/admin/content
/admin/settings
```

---

# 343. PUBLIC ROUTING

Public routes should be clean, SEO-friendly, and localized.

---

# 344. ADMIN NAVIGATION

Navigation should be module-aware and permission-aware.

Disabled modules should not clutter navigation.

---

# 345. USER EXPERIENCE FOR COMPLEX RECORDS

Complex records should use:

- tabs
- sections
- overview cards
- activity/history
- related records
- files
- permissions

Avoid giant single-screen forms.

---

# 346. RECORD DETAIL STANDARD

A typical detail page can include:

```text
Header / identity
Status
Summary
Core fields
Timeline
Related records
Documents
Tasks
Comments
Audit
```

Only display sections the user can access.

---

# 347. DASHBOARD INFORMATION DENSITY

Use hierarchy and progressive disclosure.

Do not put every metric on the home page.

---

# 348. DESIGN LANGUAGE

The design should feel:

- trustworthy
- institutional
- modern
- clean
- calm
- data-oriented
- professional

Avoid flashy consumer-app patterns when they reduce clarity.

---

# 349. RTL/LTR TESTING

Explicitly test mixed strings such as:

```text
Project Yemen-2026 / المشروع اليمني
NGO ABC - تعز
USD 125,000
```

---

# 350. TYPOGRAPHY

Allow tenant-configured fonts, but provide a sensible Arabic-first default stack.

---

# 351. MAP UX

Map views must include:

- zoom
- layer visibility where applicable
- popup detail
- filter controls
- accessible alternative list view

---

# 352. GIS EXPORT

Where allowed, support safe export of non-sensitive geographic datasets.

---

# 353. PUBLIC MAP PRIVACY

Public maps should aggregate or generalize sensitive locations.

---

# 354. REPORT LOCALIZATION

Reports must support Arabic and English layouts.

RTL PDF/DOCX generation must be tested where enabled.

---

# 355. PDF ACCESSIBILITY / FALLBACK

If complex Arabic PDF generation is unreliable, preserve a high-quality HTML export and clearly document limitations rather than shipping corrupt output.

---

# 356. OFFICE DOCUMENTS

Document preview may be provider-dependent.

Keep original files intact.

---

# 357. VERSION COMPATIBILITY

For uploaded files, record application-generated version metadata separately from native office-document versions.

---

# 358. ARCHIVE EXPORT

Provide an organization export package architecture for migrating data out of the platform.

Potential package:

```text
manifest.json
records/
files/
metadata/
checksums/
```

---

# 359. MIGRATION / OFFBOARDING

A tenant should be able to export its own data subject to policy.

Do not design the platform as a data hostage system.

---

# 360. TENANT DELETION

Tenant deletion must be a protected, multi-step administrative operation.

Provide archival/export options before destructive deletion where appropriate.

---

# 361. PLATFORM UPGRADE SAFETY

Document upgrade steps including:

- backup
- migration
- verification
- rollback/restore procedure

---

# 362. RELEASE ARTIFACTS

Production build should be reproducible from source control.

---

# 363. SECURITY SCANNING

Integrate suitable checks for:

- dependency vulnerabilities
- secret leakage
- container vulnerabilities where available
- static analysis

---

# 364. CONTAINER SECURITY

Prefer:

- non-root processes where feasible
- minimal images
- pinned/controlled dependencies
- no unnecessary capabilities

---

# 365. DATABASE SECURITY

Production should use:

- strong credentials
- restricted network access
- encrypted connections where required
- least-privilege roles

---

# 366. STORAGE SECURITY

MinIO/S3 buckets should have explicit private/public policies.

Do not rely on default permissive settings.

---

# 367. BACKUP ENCRYPTION

Where backups contain sensitive information, encrypt them.

---

# 368. SECRET MANAGEMENT

In development `.env` is acceptable.

Production should use a secure secret mechanism appropriate to the deployment environment.

---

# 369. AUDIT OF PRIVILEGED ACTIONS

Privileged actions require especially clear audit records:

- role changes
- permission changes
- API key creation
- tenant configuration changes
- data exports
- sensitive record access
- destructive deletes

---

# 370. DESTRUCTIVE ACTIONS

Use explicit confirmations for:

- delete
- revoke
- terminate
- archive where irreversible

Where possible, prefer reversible workflows.

---

# 371. SECURITY COPY

Deletion confirmations must state scope clearly.

---

# 372. API RATE-LIMIT CONFIG UI

Platform administrators may configure global limits and tenant overrides where supported.

---

# 373. FEATURE AVAILABILITY API

Expose a safe capability/configuration endpoint to frontend applications so they know enabled modules and UI capabilities.

---

# 374. TENANT MODULE CONFIG

Store enabled modules per tenant.

Example:

```json
{
  "cms": true,
  "projects": true,
  "grants": true,
  "meal": true,
  "beneficiaries": false,
  "procurement": true,
  "gis": true,
  "ai": false
}
```

Do not assume this exact configuration is universal.

---

# 375. MODULE DEPENDENCIES

Define dependencies explicitly.

Example concept:

```text
Beneficiaries → Core + Forms + Permissions
MEAL → Projects + Indicators
GIS → Locations + Projects
AI Knowledge → Documents + Search
```

Do not allow an enabled module to run without its required dependencies.

---

# 376. SYSTEM INITIALIZATION

At first boot:

- validate environment
- initialize database connection
- verify storage
- create platform bootstrap path

Do not automatically create privileged users without an explicit secure setup process.

---

# 377. FIRST ADMIN SETUP

Provide a secure first-admin creation flow.

---

# 378. ORGANIZATION CREATION

Only authorized platform users should create tenants in a multi-tenant deployment.

Self-service registration can be a future configurable feature.

---

# 379. PRODUCT TELEMETRY

If implemented, make it opt-in/configurable and privacy-conscious.

---

# 380. LICENSED FEATURES

If future enterprise licensing is implemented, do not entangle licensing checks with core domain rules.

---

# 381. FEATURE ENTITLEMENTS

A future entitlement layer can determine whether a tenant has access to premium modules.

---

# 382. PAYMENT / SUBSCRIPTION LAYER

Do not implement platform billing until a concrete SaaS business requirement exists.

Keep it as a future integration boundary.

---

# 383. MULTI-REGION PREPARATION

Do not assume Yemen-only infrastructure.

Locations, currencies, timezones, languages, and country codes should be configurable.

---

# 384. COUNTRY DATA

Use stable external or internal country references rather than hard-coding Yemen in core database logic.

---

# 385. YEMEN-SPECIFIC INITIALIZATION

Yemen can be preconfigured as a reference dataset, including:

- governorates
- districts
- common NGO sectors
- Arabic terminology

but these remain configurable and do not limit the product to Yemen.

---

# 386. ORGANIZATION TERMINOLOGY

Allow tenants to override selected UI terminology where needed.

For example:

```text
Program ↔ Programme
Beneficiary ↔ Participant
Partner ↔ Implementing Partner
```

---

# 387. AUDIENCE-SPECIFIC DASHBOARDS

Dashboards may differ for:

- Director
- Program Manager
- MEAL Officer
- Communications Officer
- Finance Officer
- HR
- Field Officer

---

# 388. DASHBOARD DATA FRESHNESS

Display update timestamps where the source data is not real-time.

---

# 389. DATA FRESHNESS

For dashboards/reports, distinguish:

- current transactional data
- cached data
- generated snapshot

---

# 390. REPORT SNAPSHOTS

If a report must preserve historical values, create immutable report snapshots rather than mutating old reports.

---

# 391. GRANT REPORTING SCHEDULE

Allow grant reporting dates to create tasks and reminders automatically.

---

# 392. PROJECT MILESTONES

Milestones may trigger workflow actions and notifications.

---

# 393. FORM SUBMISSION REVIEW

A submission can become:

```text
Received
Under Review
Accepted
Rejected
Needs Information
Closed
```

Make statuses configurable.

---

# 394. FORM DUPLICATE CONTROL

Where necessary, support duplicate detection based on configurable business identifiers.

---

# 395. FILE RETENTION

Files may inherit retention from their parent record.

---

# 396. RECORD RETENTION

Records can inherit retention from module policy or have explicit policy references.

---

# 397. KNOWLEDGE REVIEW REMINDERS

Policies/SOPs should trigger review reminders before expiration/review date.

---

# 398. DONOR REPORT PACKS

Prepare report templates that can assemble:

- narrative summary
- indicators
- milestones
- activities
- finance summary
- geography
- photos/assets

---

# 399. PUBLIC IMPACT PAGES

Projects may expose a public impact page composed from approved data.

---

# 400. IMPACT STORY DATA MODEL

Stories should be separate records that can link to projects while preserving editorial ownership and privacy controls.

---

# 401. CONTENT RELATION GRAPH

Do not build a graph database initially.

Use relational links + optional future projection.

---

# 402. FUTURE GRAPH EXPORT

Keep relationship events and IDs stable so a future graph projection can be generated.

---

# 403. EVENTUAL KNOWLEDGE GRAPH

Possible future nodes:

- Person
- Organization
- Project
- Program
- Place
- Donor
- Partner
- Publication
- Event

Possible future edges:

- funds
- implements
- partners_with
- located_in
- reports_on
- related_to

This is future architecture only.

---

# 404. AI KNOWLEDGE RETRIEVAL

When implemented, retrieval must apply the same permissions as normal search.

---

# 405. SEMANTIC INDEXING

Embeddings are projections, not canonical knowledge.

Store source record IDs and embedding model metadata.

---

# 406. MODEL REGISTRY

If AI is enabled, maintain:

- provider
- model
- capabilities
- active/inactive
- context limits where applicable
- cost metadata where available

---

# 407. PROMPT REGISTRY

Version AI prompt templates rather than burying prompts in code.

---

# 408. AI HUMAN REVIEW

Provide review state for important AI-assisted outputs:

```text
Generated
Reviewed
Accepted
Rejected
Edited
```

---

# 409. AI SAFETY BOUNDARY

AI tools should never:

- bypass permissions
- publish without configured approval
- silently alter authoritative financial records
- silently alter beneficiary records
- expose restricted documents

---

# 410. AI FAIL-SAFE

AI provider failures should degrade gracefully.

The core CMS must remain functional when AI is unavailable.

---

# 411. SEARCH FAIL-SAFE

If external search is down, critical CRUD operations must still work where possible.

---

# 412. CACHE FAIL-SAFE

If Redis is unavailable, the application may degrade to uncached operation where technically safe.

Do not make the entire system depend on cache for data correctness.

---

# 413. OBJECT STORAGE FAIL-SAFE

File-heavy features may be unavailable when storage is down, but the rest of the platform should remain understandable and healthy.

---

# 414. BACKGROUND WORKER FAIL-SAFE

Jobs should remain inspectable and retryable after temporary worker failures.

---

# 415. DATA CONSISTENCY

Business operations must remain correct even if derived systems lag temporarily.

---

# 416. EVENTUAL CONSISTENCY

Use eventual consistency only for derived features such as:

- search
- analytics
- notifications
- projections

Core transactional state should remain strongly consistent where required.

---

# 417. DATABASE LOCKING

Use appropriate locking/uniqueness strategies for concurrent critical operations.

---

# 418. IDEMPOTENT JOBS

Background jobs should be safe to retry where practical.

---

# 419. FAILED JOB QUEUE

Keep failure details and retry count for operational review.

---

# 420. ADMIN OPERATIONS LOG

Privileged maintenance operations should be visible to administrators and audited.

---

# 421. DATA REPAIR TOOLS

Provide safe operator scripts for:

- rebuild search
- reconcile storage
- retry jobs
- regenerate reports
- fix known data issues

Never ship destructive repair scripts without explicit safeguards.

---

# 422. CLI

Provide an administrative CLI or script entry points for:

- migrations
- seed
- search rebuild
- backup
- restore
- maintenance
- diagnostics

---

# 423. DIAGNOSTICS

Provide a safe diagnostics command that summarizes configuration health without exposing secrets.

---

# 424. CONFIGURATION REDACTION

Any configuration dump must mask secrets.

---

# 425. BUILD / DEPLOYMENT ARTIFACTS

A clean deployment should produce:

- application images
- migration package
- configuration template
- documentation
- health checks

---

# 426. SUPPORTABILITY

Operators should be able to answer:

- What failed?
- When did it fail?
- Which tenant was affected?
- Which request/job caused it?
- Can it be retried?
- Was it authorized?

---

# 427. TEST DATA ISOLATION

Automated tests must use isolated test databases/tenants.

---

# 428. TEST DETERMINISM

Tests should not depend on external production services.

Use mocks/fakes/test containers as appropriate.

---

# 429. CONTRACT TESTING

For important integration adapters, use contract tests where feasible.

---

# 430. MIGRATION TESTS

CI should test migrations from a clean database and, where practical, from representative previous schema states.

---

# 431. BACKUP RESTORE TESTS

At least a documented periodic restore test strategy must exist.

---

# 432. LOAD / PERFORMANCE TESTING

Before production claims, test realistic workloads such as:

- content browsing
- project lists
- report generation
- file upload
- search
- forms

---

# 433. SECURITY REVIEW CHECKPOINTS

At each major phase, review:

- auth
- authorization
- tenancy
- secrets
- files
- audit
- exports

---

# 434. CODE REVIEW CHECKLIST

Before merging a major feature, check:

- naming
- tests
- security
- transaction boundaries
- migrations
- accessibility
- localization
- documentation

---

# 435. UX REVIEW CHECKLIST

Check:

- empty states
- loading states
- error handling
- RTL
- mobile layout
- keyboard use
- terminology
- information hierarchy

---

# 436. CONTENT EDITOR UX

Editors need:

- autosave where appropriate
- draft recovery
- previews
- revision history
- workflow status
- reviewer assignment

---

# 437. FORM BUILDER UX

Admins need:

- add field
- reorder
- configure validation
- configure conditional logic
- preview
- publish form version

---

# 438. WORKFLOW BUILDER UX

Start with a controlled visual editor or structured form-based workflow builder.

Do not implement a massive node editor before the underlying workflow engine is sound.

---

# 439. CUSTOM MODULE UX

Expose custom module creation only after metadata infrastructure is stable.

---

# 440. ADMIN ONBOARDING DOCUMENTATION

Provide help for first-time administrators in Arabic and English where practical.

---

# 441. PUBLIC CONTENT ACCESSIBILITY

Public pages must include:

- alt text
- accessible heading hierarchy
- keyboard navigation
- readable contrast
- semantic structure

---

# 442. CONTENT MODERATION

Public submissions or comments, if enabled, should have moderation states.

---

# 443. ANTI-SPAM ARCHITECTURE

For public interactions support adapters for:

- rate limits
- CAPTCHA
- reputation/risk signals where appropriate

---

# 444. API PUBLIC FORMS

A public form API should not expose arbitrary internal fields.

---

# 445. FORM FILES

Public form uploads require strict type/size controls and private storage by default unless explicit publication is intended.

---

# 446. PUBLIC DOWNLOADS

Published downloadable files may use CDN/object URLs, but private originals must remain protected.

---

# 447. DOCUMENT WATERMARKING

Optional future feature for sensitive published documents.

---

# 448. DIGITAL SIGNATURE INTEGRATION

Create an adapter boundary for digital signature services rather than embedding cryptographic/legal workflows prematurely.

---

# 449. ORGANIZATION POLICIES

Tenant administrators can define selected:

- password policy
- retention policy
- workflow policy
- approval policy
- publication policy
- data classification policy

Platform security invariants cannot be weakened.

---

# 450. CONFIGURATION AUDIT

Changes to critical tenant settings should be audited.

---

# 451. SYSTEM ANNOUNCEMENTS

Platform admins may publish operational announcements to tenants if multi-tenant SaaS is used.

---

# 452. TENANT SUPPORT TOOLS

Platform admin support should be able to diagnose tenant issues without silently impersonating users.

If impersonation is ever implemented, require explicit audit and highly restricted access.

---

# 453. IMPERSONATION SAFETY

If enabled:

- strong authorization
- reason required
- audit record
- visible banner
- automatic expiration

---

# 454. DATA EXPORT PORTABILITY

Design a tenant export format that includes:

- records
- metadata
- files
- relationships
- checksums
- manifest

---

# 455. DATA IMPORT PORTABILITY

Export format should be importable where technically safe.

---

# 456. ARCHIVE FORMAT

Use stable machine-readable formats for core data export.

---

# 457. DOCUMENTED API LIMITS

Document:

- pagination defaults
- maximum page size
- upload limits
- report limits
- rate limits

---

# 458. UX PERFORMANCE

Admin list pages should not fetch unnecessary data for every row.

Use summary projections or selective fetching.

---

# 459. DATABASE QUERY REVIEW

For large lists, inspect generated SQL and indexes rather than guessing.

---

# 460. N+1 PREVENTION

Use appropriate eager-loading/select-in strategies and verify with tests or query instrumentation.

---

# 461. API RESPONSE SIZE

Do not return full nested object graphs by default.

Use detail endpoints for expensive data.

---

# 462. FORM AUTOSAVE

Only enable autosave where it can be implemented safely.

---

# 463. DRAFT STORAGE

Drafts should not accidentally become public due to cache or route errors.

---

# 464. PUBLIC CONTENT PUBLISHING

Publication operation should atomically or transactionally establish the published state before cache/index updates.

Derived systems may lag.

---

# 465. SEARCH INDEX LAG

Display or document indexing lag where operationally relevant.

---

# 466. WEBHOOK EVENT ORDER

Consumers should not assume every event arrives in exactly-once order.

Use IDs/timestamps/versions where necessary.

---

# 467. EVENT VERSIONING

Version important event payloads to allow future evolution.

---

# 468. API SCHEMA VERSIONING

Avoid breaking request/response structures unnecessarily.

---

# 469. DATA MIGRATION RUNBOOK

Document how to:

- backup
- migrate
- verify
- rollback/restore

---

# 470. PRODUCTION READINESS GATE

The platform is not production-ready until:

```text
[ ] Auth works
[ ] Tenant isolation verified
[ ] RBAC verified
[ ] Critical workflows tested
[ ] Backups documented
[ ] Restore tested
[ ] Secrets managed safely
[ ] HTTPS deployment documented
[ ] Health monitoring available
[ ] No critical placeholder features
```

---

# 471. INITIAL MVP SCOPE

A realistic first production-capable release should focus on:

```text
Core
Identity
Organizations
CMS
Media
Documents
Projects
Programs
Donors
Partners
Grants
Forms
Events
Tasks
Workflow
Reporting basics
API
Audit
Localization
```

Advanced domains remain modular follow-on releases.

---

# 472. BUILD ORDER

Recommended implementation order:

```text
Foundation
→ Identity
→ Tenancy
→ Permissions
→ Audit
→ CMS
→ Media
→ Documents
→ Workflow
→ Organizations
→ Programs
→ Projects
→ Donors
→ Partners
→ Grants
→ Forms
→ MEAL
→ Reporting
→ GIS
→ API Platform
→ Advanced Modules
→ AI
```

---

# 473. STOP CONDITIONS

Pause a feature expansion and fix architecture if any of these appear:

- duplicated tenant logic
- duplicated auth logic
- inconsistent permission checks
- multiple competing API styles
- uncontrolled database coupling
- giant frontend components
- broken migrations
- untested sensitive modules
- hard-coded organization-specific logic

---

# 474. REFRACTOR CONDITION

If a feature reveals repeated patterns, extract a reusable platform primitive instead of copying code.

Examples:

- status engine
- file attachments
- comments
- audit
- notifications
- approvals
- relationships
- custom fields
- forms

---

# 475. NO FEATURE CREEP

Do not add unrelated infrastructure because it sounds enterprise-grade.

Every dependency must have a clear job.

---

# 476. REPOSITORY CLEANLINESS

Do not commit:

- secrets
- local database dumps
- huge generated files
- node_modules
- build output unless intentionally required
- temporary test artifacts

---

# 477. COMMIT GRANULARITY

Use coherent commits where repository workflow allows.

Examples:

```text
feat(core): add tenant model
feat(auth): add invitation flow
feat(cms): add page publishing
feat(projects): add project workflow
```

---

# 478. BRANCH SAFETY

Follow repository's existing branch strategy if present.

---

# 479. DOCUMENT DECISIONS

When choosing an alternative to this blueprint, record:

- reason
- trade-offs
- impact
- migration path if relevant

---

# 480. FINAL IMPLEMENTATION REPORT

At the end of each major work cycle, report:

```text
Implemented
Partially Implemented
Not Implemented
Known Issues
Architecture Decisions
Database Changes
API Changes
UI Changes
Tests
How to Run
How to Verify
Next Phase
```

Do not fabricate completion.

---

# 481. FINAL ACCEPTANCE TEST — ORGANIZATION

Demonstrate:

```text
Create Tenant
↓
Create Organization
↓
Create Department
↓
Create Admin
↓
Invite User
↓
Assign Role
↓
Configure Branding
↓
Enable Modules
```

---

# 482. FINAL ACCEPTANCE TEST — CMS

Demonstrate:

```text
Create Page
↓
Add Blocks
↓
Save Draft
↓
Submit Review
↓
Approve
↓
Publish
↓
View Public Page
↓
Change Version
↓
Restore Previous Version
```

---

# 483. FINAL ACCEPTANCE TEST — PROJECT

Demonstrate:

```text
Create Program
↓
Create Project
↓
Add Donor
↓
Add Partner
↓
Add Activity
↓
Add Indicator
↓
Add Location
↓
Create Report
↓
View Dashboard
```

---

# 484. FINAL ACCEPTANCE TEST — GRANT

Demonstrate:

```text
Funding Opportunity
↓
Proposal
↓
Review
↓
Approval
↓
Grant
↓
Reporting
↓
Closure
```

---

# 485. FINAL ACCEPTANCE TEST — FORM

Demonstrate:

```text
Create Form
↓
Publish
↓
Public Submission
↓
Review
↓
Task
↓
Notification
↓
Close
```

---

# 486. FINAL ACCEPTANCE TEST — DOCUMENT

Demonstrate:

```text
Upload
↓
Validate
↓
Store
↓
Version
↓
Approve
↓
Secure Download
↓
Audit
```

---

# 487. FINAL ACCEPTANCE TEST — SECURITY

Demonstrate:

```text
User A / Tenant A
      ↓
Requests Tenant B Object
      ↓
403/404 according to policy
      ↓
No data leaked
      ↓
Attempt audited if policy requires
```

---

# 488. FINAL ACCEPTANCE TEST — API

Demonstrate:

- OpenAPI generated
- API key creation
- scoped API access
- revoked key denied
- webhook delivery
- rate limiting
- tenant isolation

---

# 489. FINAL ACCEPTANCE TEST — DEPLOYMENT

Demonstrate a clean installation from documentation:

```text
Clone
↓
Configure .env
↓
Compose up
↓
Migrate
↓
Create admin
↓
Login
↓
Use CMS
```

---

# 490. FINAL ACCEPTANCE TEST — BACKUP

Demonstrate:

```text
Backup
↓
Simulated failure
↓
Restore
↓
Verify records/files
```

---

# 491. RELEASE CHECKLIST

Before release:

```text
[ ] Build passes
[ ] Unit tests pass
[ ] Integration tests pass
[ ] E2E critical flows pass
[ ] Security tests pass
[ ] Tenant isolation pass
[ ] Migrations pass
[ ] Docker Compose pass
[ ] Documentation updated
[ ] Demo seed validated
[ ] No secrets committed
[ ] No critical placeholders
```

---

# 492. FINAL ENGINEERING DIRECTIVE

Do not treat this document as an instruction to generate one enormous code dump.

Treat it as the system contract.

Implement incrementally.

After each phase, verify the existing system still works.

Do not destroy working functionality to add new features.

Do not create speculative infrastructure without need.

Do not claim a module is complete until its database, domain logic, API, authorization, UI, tests, audit behavior, and documentation satisfy its acceptance criteria.

---

# 493. FIRST ACTIONS — EXECUTE NOW

Start immediately with:

```text
1. Inspect repository.
2. Read YNGO-CMS documentation.
3. Map existing code to target architecture.
4. Identify reusable components.
5. Identify missing foundation.
6. Create/update architecture documents.
7. Initialize infrastructure.
8. Implement Core + Identity + Tenant + RBAC + Audit.
9. Implement first complete CMS vertical slice.
10. Run tests and build.
11. Fix failures.
12. Continue with the next approved phase.
```

Do not stop at planning.

---

# 494. FINAL RULE

**Build YNGO-CMS as a real platform.**

Not a mockup.

Not a static CMS template.

Not a collection of empty CRUD pages.

Not an over-engineered microservice experiment.

The target is:

> **A secure, modular, multilingual, API-first, configurable, self-hostable digital operating platform for NGOs and civil society organizations, with a professional CMS at its core and extensible program/operations capabilities around it.**

The journalist/content editor, program officer, MEAL officer, director, donor, partner, field worker, and public visitor must each receive an experience appropriate to their permissions and role.

Build the system so that an organization can start with a professional website and CMS, then progressively activate programs, projects, grants, MEAL, CRM, forms, documents, GIS, reporting, portals, automation, and AI without replacing the platform.

---

# 495. OUTPUT EXPECTATION FROM THE CODING AGENT

After executing work, return a factual implementation summary:

```text
Project Status:

Implemented:
- ...

Partially Implemented:
- ...

Not Implemented:
- ...

Database:
- ...

API:
- ...

Frontend:
- ...

Security:
- ...

Tests:
- ...

Deployment:
- ...

Documentation:
- ...

Known Issues:
- ...

Next Phase:
- ...
```

Never claim successful implementation without evidence from builds/tests/runtime behavior.

---

# END OF YNGO-CMS ENTERPRISE MASTER BUILD PROMPT v2
