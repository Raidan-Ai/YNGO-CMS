# docs/02-domain/entities.md

> **Status:** Current (catalogue) — implementation `PLANNED` | **Owner:** Lead Architect
> **Last Updated:** 2026-10-02 | **Source:** Engineering_Documentation_Package_v2.md
> **Depends on:** `domain-model.md` (contract, lifecycles, invariants)

## How to Read

The full field-level specification for every entity follows the template in
`domain-model.md` §"Entity Detail Template". This file provides the **catalogue**:
owner context, key relationships, and sensitivity class — what tasks and tests
need. Field-by-field detail is added per entity as its phase is executed (inside
that module's docs), so this file never becomes stale.

**Sensitivity classes:** `public` · `internal` · `confidential` · `restricted` · `sensitive`

## Catalogue (by bounded context)

| Context | Entities | Key relationships | Sensitivity |
|---------|----------|-------------------|-------------|
| Tenancy | `Tenant`, `TenantSettings`, `TenantDomain` | Tenant → everything | internal |
| Identity | `User`, `UserProfile`, `Session`, `Invitation`, `PasswordResetToken`, `MfaFactor`, `LoginEvent`, `UserTenantMembership` | User ↔ Tenant (membership); User → Roles | confidential |
| Access | `Role`, `Permission`, `RolePermission`, `Group`, `GroupMember`, `UserRole`, `ApiKey`, `OAuthApplication` | Role → Permission; User → Role | confidential |
| Organization | `Organization`, `Department`, `Branch`, `Office`, `Team`, `Position`, `OrganizationUnit`, `ContactPoint` | Org → units → users (ABAC scope) | internal |
| Governance | `Board`, `BoardMember`, `Committee`, `CommitteeMember`, `BoardMeeting`, `BoardResolution`, `GovernanceDocument`, `Policy`, `Bylaw`, `DecisionRecord`, `DelegationOfAuthority`, `ConflictOfInterestDeclaration` | Board → members / meetings / resolutions | confidential |
| CMS | `Content` (base), `Page`, `Article`, `Post`, `News`, `Story`, `Announcement`, `PressRelease`, `Publication`, `Research`, `PolicyPaper`, `CaseStudy`, `FAQ`, `Campaign`, `Vacancy`, `Tender`, `Event`, `Category`, `Tag`, `ContentVersion`, `ContentRevision`, `ContentLock`, `Redirect`, `NavigationMenu`, `MenuItem` | Content → versions/revisions; Content → categories/tags | public (published) / internal (draft) |
| Media/DAM | `MediaAsset`, `Rendition`, `Folder`, `Collection`, `MediaTag`, `AssetUsage` | Asset → renditions; Asset → folder/collections | internal |
| Documents/DMS | `Document`, `DocumentVersion`, `DocumentCategory`, `DocumentAccess`, `RetentionPolicy`, `ClassificationLabel` | Document → versions; Document → classification | confidential / restricted |
| Programs/Projects | `Program`, `ProgramSector`, `Project`, `ProjectLocation`, `ProjectTeam`, `ProjectPartner`, `Milestone`, `Activity`, `ActivityLocation`, `Workplan` | Program 1→N Project; Project → locations/team/milestones | internal |
| MEAL | `Indicator`, `IndicatorValue`, `LogFrame`, `LogFrameRow`, `Baseline`, `Assessment`, `Evaluation`, `EvaluationFinding`, `ResultChain`, `ReportingPeriod`, `Target` | Indicator → values per period; LogFrame → rows | internal |
| Grants/Funding | `Grant`, `GrantMilestone`, `GrantReport`, `GrantAmendment`, `Opportunity`, `Proposal`, `ProposalVersion`, `FundingSource`, `Award` | Grant → milestones/reports; Grant → Donor | confidential |
| Donors/Partners/CRM | `Donor`, `DonorContact`, `Partner`, `PartnerAgreement`, `Stakeholder`, `Contact`, `Interaction`, `Meeting`, `Call`, `CRMTask`, `CommunicationLog` | Donor → contacts/grants; Stakeholder → interactions | confidential |
| People (privacy-critical) | `Staff`, `StaffContract`, `LeaveRequest`, `TrainingRecord`, `Volunteer`, `Skill`, `Availability`, `VolunteerAssignment`, `Attendance`, `VolunteerHours`, `Certificate`, `Beneficiary`, `Household`, `HouseholdMember`, `Enrollment`, `Assistance`, `Referral`, `Case`, `CaseNote`, `CaseAssessment`, `FollowUp`, `SafeguardingCase`, `SafeguardingIncident`, `SafeguardingAction`, `Investigator`, `Complaint`, `ComplaintMessage`, `ComplaintResolution`, `Membership`, `MembershipDue` | Beneficiary → household/enrollment/assistance/referral; Case → notes/assessments/follow-ups; Safeguarding → actions/investigators | **restricted / sensitive** |
| Assets/Procurement/Finance | `Asset`, `AssetAssignment`, `MaintenanceRecord`, `InventoryItem`, `StockMovement`, `Vehicle`, `Trip`, `Itinerary`, `PerDiem`, `TravelReport`, `ProcurementRequest`, `RFQ`, `Vendor`, `Quotation`, `ProcurementEvaluation`, `PurchaseOrder`, `Delivery`, `Budget`, `CostCenter`, `ExpenseImport`, `ExchangeRate` | PO → awarded Quotation; Budget → cost centre/project | confidential |
| Forms/Survey | `Form`, `FormVersion`, `FormField`, `FormSubmission`, `FormReview`, `FormSubmissionFile` | Form → versions → submissions | varies (may be sensitive) |
| Workflow/Automation | `WorkflowDefinition`, `WorkflowVersion`, `WorkflowStep`, `WorkflowTransition`, `WorkflowCondition`, `WorkflowInstance`, `WorkflowTask`, `Approval`, `Escalation`, `AutomationRule`, `AutomationExecution`, `AutomationLog` | Definition → versions; Instance → tasks/approvals | internal |
| Notifications | `Notification`, `NotificationTemplate`, `NotificationPreference`, `DeliveryAttempt`, `NotificationChannelConfig` | Notification → attempts | internal |
| Platform services | `SearchDocument` (projection), `ImportJob`, `ImportRow`, `ExportJob`, `ReportTemplate`, `ReportRun`, `ReportSnapshot`, `Dashboard`, `Widget`, `SavedView`, `WebhookEndpoint`, `WebhookSubscription`, `WebhookDelivery`, `WebhookAttempt`, `WebhookSecret`, `IntegrationProvider`, `IntegrationConfig`, `AuditLog`, `Setting`, `MasterDataEntry`, `DataQualityRule`, `DataQualityIssue` | Projections derive from owned entities | internal (audit: confidential) |
| GIS | `Location`, `Boundary`, `MapLayer`, `ProjectMap`, `Geofence` | Location ↔ project/activity; Layer → privacy policy | restricted (precision) |
| AI (optional) | `AiProvider`, `AiModel`, `AiRequest`, `AiResponse`, `AiJob`, `AiPromptVersion`, `AiReview` | Request → response/review | confidential |

## Entity Ownership Rules

1. **One owner context per entity.** No entity is written by two modules.
2. **Cross-context reads** go through the owner's exported service or a domain
   event projection — never by importing another module's repository (`AGENTS.md` §4).
3. **Projection tables** (`SearchDocument`, report/dashboard snapshots) are
   rebuildable and never authoritative.
4. **Sensitive entities** are implemented in Phase 6 with field-level policy
   (`business-rules.md`, `../11-security/privacy.md`).
5. **Every entity** carries the universal column contract from `domain-model.md`.

## Related

- Contract + invariants: `domain-model.md` · Rules: `business-rules.md`
- Schema: `../04-data/schema.md` · Modules: `../07-backend/modules.md`

*End of docs/02-domain/entities.md*