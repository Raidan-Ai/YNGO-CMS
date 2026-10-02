# docs/02-domain/domain-model.md

> **Status:** Current | **Owner:** Lead Architect | **Last Updated:** 2026-10-02
> **Source:** Engineering_Documentation_Package_v2.md §6, §9–§14; Blueprint.md
> **Purpose:** canonical entity list, ownership, and lifecycle. Single source of truth.

## Scope

~180 entities are declared across the frozen specs. This document defines the
canonical set per bounded context, with the standard column contract and the
lifecycle/state rules that implementations must follow. Additional detail lives in
`entities.md` and `business-rules.md`.

## Universal Entity Contract

Every tenant-owned table/entity:

```text
id          uuid          PK
tenant_id   uuid          NOT NULL, FK -> tenant(id), indexed
created_at  timestamptz   NOT NULL
updated_at  timestamptz   NOT NULL
created_by  uuid          FK -> user(id), nullable for system actions
updated_by  uuid          FK -> user(id), nullable
status      text|enum     only where a lifecycle exists
deleted_at  timestamptz   ONLY where soft delete is a genuine business rule
```

Platform-global tables (e.g. `tenant`, `permission`, master-data registries,
`ai_provider`) omit `tenant_id` deliberately — that omission must be marked in
`docs/04-data/database.md` as intentional.

## Entities by Bounded Context

### Tenancy (root — platform-global)
`Tenant`, `TenantSettings`, `TenantDomain`.

### Identity
`User`, `UserProfile`, `Session`, `Invitation`, `PasswordResetToken`, `MfaFactor`,
`LoginEvent`, `UserTenantMembership`.

### Access
`Role`, `Permission`, `RolePermission`, `Group`, `GroupMember`, `UserRole`,
`ApiKey`, `OAuthApplication`.

### Organization
`Organization`, `Department`, `Branch`, `Office`, `Team`, `Position`,
`OrganizationUnit`, `ContactPoint`.

### Governance
`Board`, `BoardMember`, `Committee`, `CommitteeMember`, `BoardMeeting`,
`BoardResolution`, `GovernanceDocument`, `Policy`, `Bylaw`, `DecisionRecord`,
`DelegationOfAuthority`, `ConflictOfInterestDeclaration`.

### CMS
`Content` (base), `Page`, `Article`, `Post`, `News`, `Story`, `Announcement`,
`PressRelease`, `Publication`, `Research`, `PolicyPaper`, `CaseStudy`, `FAQ`,
`Campaign`, `Vacancy`, `Tender`, `Event`, `Category`, `Tag`, `ContentVersion`,
`ContentRevision`, `ContentLock`, `Redirect`, `NavigationMenu`, `MenuItem`.

### Media / DAM
`MediaAsset`, `Rendition`, `Folder`, `Collection`, `MediaTag`, `AssetUsage`.

### Documents / DMS
`Document`, `DocumentVersion`, `DocumentCategory`, `DocumentAccess`,
`RetentionPolicy`, `ClassificationLabel`.

### Programs / Projects
`Program`, `ProgramSector`, `Project`, `ProjectLocation`, `ProjectTeam`,
`ProjectPartner`, `Milestone`, `Activity`, `ActivityLocation`, `Workplan`.

### MEAL
`Indicator`, `IndicatorValue`, `LogFrame`, `LogFrameRow`, `Baseline`,
`Assessment`, `Evaluation`, `EvaluationFinding`, `ResultChain`,
`ReportingPeriod`, `Target`.

### Grants / Funding
`Grant`, `GrantMilestone`, `GrantReport`, `GrantAmendment`, `Opportunity`,
`Proposal`, `ProposalVersion`, `FundingSource`, `Award`.

### Donors / Partners / CRM
`Donor`, `DonorContact`, `Partner`, `PartnerAgreement`, `Stakeholder`, `Contact`,
`Interaction`, `Meeting`, `Call`, `CRMTask`, `CommunicationLog`.

### People (privacy-critical)
`Staff`, `StaffContract`, `LeaveRequest`, `TrainingRecord`, `Volunteer`, `Skill`,
`Availability`, `VolunteerAssignment`, `Attendance`, `VolunteerHours`,
`Certificate`, `Beneficiary`, `Household`, `HouseholdMember`, `Enrollment`,
`Service`, `Assistance`, `Referral`, `Case`, `CaseAssessment`, `CaseNote`,
`CaseFollowUp`, `SafeguardingCase`, `Incident`, `Investigator`,
`RestrictedAttachment`, `Resolution`, `Complaint`, `Feedback`, `Investigation`.

### Operations
`Asset`, `AssetAssignment`, `AssetMaintenance`, `InventoryItem`,
`InventoryTransaction`, `Vehicle`, `Driver`, `Trip`, `FuelLog`,
`VehicleMaintenance`, `TravelRequest`, `Itinerary`, `PerDiem`, `TravelReport`,
`ProcurementRequest`, `RFQ`, `Vendor`, `Quotation`, `ProcurementEvaluation`,
`PurchaseOrder`, `Delivery`, `Budget`, `CostCenter`, `ExpenseImport`,
`ExchangeRate`.

### Forms / Survey
`Form`, `FormVersion`, `FormField`, `FormSubmission`, `FormReview`,
`FormSubmissionFile`.

### Workflow / Automation
`WorkflowDefinition`, `WorkflowVersion`, `WorkflowStep`, `WorkflowTransition`,
`WorkflowCondition`, `WorkflowInstance`, `WorkflowTask`, `Approval`,
`Escalation`, `AutomationRule`, `AutomationExecution`, `AutomationLog`.

### Notifications
`Notification`, `NotificationTemplate`, `NotificationPreference`,
`DeliveryAttempt`, `NotificationChannelConfig`.

### Platform services
`SearchDocument` (projection), `ImportJob`, `ImportRow`, `ExportJob`,
`ReportTemplate`, `ReportRun`, `ReportSnapshot`, `Dashboard`, `Widget`,
`SavedView`, `WebhookEndpoint`, `WebhookSubscription`, `WebhookDelivery`,
`WebhookAttempt`, `WebhookSecret`, `IntegrationProvider`, `IntegrationConfig`,
`AuditLog`, `Setting`, `MasterDataEntry`, `DataQualityRule`, `DataQualityIssue`.

### GIS
`Location`, `Boundary`, `MapLayer`, `ProjectMap`, `Geofence`.

### AI (optional)
`AiProvider`, `AiModel`, `AiRequest`, `AiResponse`, `AiJob`, `AiPromptVersion`,
`AiReview`.

## Lifecycles (canonical states)

| Entity group | Lifecycle |
|--------------|-----------|
| User | Invited → Active → Suspended → Deactivated → Archived |
| Content | Draft → InReview → ChangesRequested → Approved → Scheduled → Published → Unpublished → Archived |
| Form | Draft → Published → Closed → Archived |
| Case | Opened → Assessment → Active → Referral/FollowUp → Resolved → Closed |
| Safeguarding | Reported → Triage → Investigating → Actioned → Closed |
| Grant | Identified → Applied → Awarded → Active → Reporting → Closed |
| Project | Draft → Planned → Active → Suspended → Completed → Closed |
| Workflow instance | Created → InProgress → AwaitingApproval → Completed / Rejected / Cancelled |
| Approval | Pending → Approved / Rejected / Escalated / Delegated / TimedOut |

## Invariants (enforced in domain code + DB constraints)

1. Every tenant-owned row has a non-null `tenant_id` matching the request context.
2. A `Project` belongs to exactly one `Program` and one `Organization` (same tenant).
3. `IndicatorValue` references a `ReportingPeriod` and an `Indicator` in the same project.
4. `GrantReport` cannot be submitted while required milestones are incomplete.
5. `Case` and `SafeguardingCase` records are never returned by an unscoped query.
6. `Content` cannot reach `Published` without an `Approved` revision.
7. `PurchaseOrder` cannot exist without an awarded `Quotation`.
8. `WorkflowInstance` steps advance only through defined transitions.
9. Soft-deleted rows are excluded from all default queries and searches.
10. Audit rows are immutable (no UPDATE/DELETE grants).

## Entity Detail Template

Every entity documented in `entities.md` states:

```text
Name
Purpose
Owner (bounded context)
Fields (name, type, nullable, default, constraints)
Relationships (FKs, cardinality)
Lifecycle / states
Permissions (resource:action)
Validation rules
Business rules
Events emitted
Sensitivity class (public|internal|confidential|restricted|sensitive)
```

## Related

- Entities detail: `docs/02-domain/entities.md`
- Rules: `docs/02-domain/business-rules.md`
- Workflows: `docs/02-domain/workflows.md`
- Schema: `docs/04-data/schema.md`
- Boundary definitions: `ARCHITECTURE.md` §6