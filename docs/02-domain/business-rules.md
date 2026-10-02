# docs/02-domain/business-rules.md

> **Status:** Current (authoritative rule list; enforcement `PLANNED`)
> **Owner:** Lead Architect | **Last Updated:** 2026-10-02
> **Source:** Engineering_Documentation_Package_v2.md, Blueprint.md, ADRs
> **Usage:** rules carry stable IDs (`BR-NNN`) referenced by tasks and tests.

## 1. Platform Rules

| ID | Rule | Enforced in | Test |
|----|------|-------------|------|
| BR-001 | Every tenant-owned row has `tenant_id` matching the request context | Repository base + DB constraint | Tenant negative test |
| BR-002 | Requests without valid tenant context are rejected on data paths | `TenantGuard` | API test |
| BR-003 | Authorization is deny-by-default; every handler declares permissions | Guard + decorator | Authz matrix test |
| BR-004 | ABAC restricts records to the actor's org unit / project assignment | Policy service | Negative test |
| BR-005 | Every mutation writes an append-only audit record | Audit interceptor + service | Audit assertion |
| BR-006 | API keys are hashed at rest; raw secret returned once | Access module | Key lifecycle test |
| BR-007 | Soft-deleted rows are excluded from default queries and search | Repository base + index | Query test |
| BR-008 | No fabricated business data in runtime code | ADR-013 + CI scan | CI scan |
| BR-009 | `request_id` propagates UI → API → worker → provider | Middleware + context | Trace test |

## 2. Identity & Access Rules

| ID | Rule |
|----|------|
| BR-010 | First tenant/org/admin is created only by bootstrap; password never defaulted (ADR-012) |
| BR-011 | Refresh-token rotation invalidates the previous token; replay is rejected |
| BR-012 | Invitations and password-reset tokens are single-use and expire |
| BR-013 | Suspended/deactivated users cannot authenticate or refresh sessions |
| BR-014 | Effective permissions = union of assigned roles, minus explicit denials |
| BR-015 | Field-level policy can deny an attribute even when the record is readable |

## 3. Content & Media Rules

| ID | Rule |
|----|------|
| BR-020 | Content cannot reach `Published` without an `Approved` revision |
| BR-021 | Status changes occur only through defined workflow transitions |
| BR-022 | Uploads pass MIME allow-list + magic-byte check + size limit + quarantine |
| BR-023 | Downloads use time-limited signed URLs; no public object paths |
| BR-024 | Restricted documents are excluded from public search and public layers |
| BR-025 | Published content keeps version history; edits create a new revision |
| BR-026 | A content lock prevents conflicting concurrent edits by different users |

## 4. Programs, Projects & MEAL Rules

| ID | Rule |
|----|------|
| BR-030 | A Project belongs to exactly one Program and one Organization (same tenant) |
| BR-031 | `IndicatorValue` references an `Indicator` and `ReportingPeriod` in the same project |
| BR-032 | Indicator values are auditable; edits preserve the previous value in audit |
| BR-033 | A project cannot be `Completed` with open mandatory milestones |
| BR-034 | Sensitive coordinates are never published in public map layers |
| BR-035 | Reports state explicitly when they contain no data — no fabricated figures |

## 5. Funding & Relationship Rules

| ID | Rule |
|----|------|
| BR-040 | `GrantReport` cannot be submitted while required milestones are incomplete |
| BR-041 | A Grant must reference a Donor or FundingSource |
| BR-042 | Grant amendments are versioned; awarded-amount history is preserved |
| BR-043 | CRM interactions are append-only with actor + timestamp |
| BR-044 | Deadline automations produce notifications only for real, dated records |

## 6. Procurement & Finance Rules

| ID | Rule |
|----|------|
| BR-050 | A `PurchaseOrder` cannot exist without an awarded `Quotation` |
| BR-051 | Procurement evaluations require the configured minimum quotations |
| BR-052 | Budget utilisation derives from committed/actual costs; overrides are audited |
| BR-053 | Finance is a budgeting/utilisation layer; no double-entry ledger (`../00-overview/scope.md`) |

## 7. Sensitive-Domain Rules (highest scrutiny)

| ID | Rule |
|----|------|
| BR-060 | Beneficiary/Case/Safeguarding records are never returned by unscoped queries |
| BR-061 | Access to restricted records requires elevated permission **and** a captured reason |
| BR-062 | Safeguarding rows never appear in generic search, generic exports, or activity feeds |
| BR-063 | Safeguarding writes to a separate audit stream readable only by designated roles |
| BR-064 | Exports of sensitive data are permission-checked, rate-limited, watermarked, and audited |
| BR-065 | Consent, where required, is recorded before the related processing occurs |
| BR-066 | Field-level masking applies in lists/aggregates unless the actor holds the field grant |

## 8. Form, Import & Report Rules

| ID | Rule |
|----|------|
| BR-070 | Form submissions keep the form version they were captured with |
| BR-071 | Import is validate-then-commit; failures are reported per row |
| BR-072 | A partial import failure never leaves committed rows corrupted |
| BR-073 | Report/export jobs are permission-checked at creation **and** at download |
| BR-074 | Data-quality issues (duplicates, missing values) are surfaced, not silently corrected |

## 9. Workflow, Automation & Notification Rules

| ID | Rule |
|----|------|
| BR-080 | Workflow definitions are versioned; in-flight instances keep their version |
| BR-081 | Automatic transitions are permission-aware and rate-limited |
| BR-082 | Automations cannot exceed the permissions of their configuring actor |
| BR-083 | Notification delivery attempts are recorded; failures retry then dead-letter |
| BR-084 | Webhooks are HMAC-signed with retry, backoff, and a delivery log |

## 10. AI Rules (optional, ADR-014)

| ID | Rule |
|----|------|
| BR-090 | AI output is always a draft requiring explicit human approval |
| BR-091 | AI inherits the caller's permissions and never broadens them |
| BR-092 | AI never silently overwrites an existing record |
| BR-093 | AI requests/responses store provider, model, prompt version, actor, and review status |
| BR-094 | Sensitive data is redacted or blocked from AI calls per the privacy matrix |

## 11. Rule Lifecycle

```text
Proposed (this file, with source citation)
   ↓
Implemented (code + guard/policy + tests)
   ↓
Verified (negative test per boundary, per AGENTS.md §12)
   ↓
Deprecated (superseded by a new BR-NNN, with an ADR if architectural)
```

A rule may not be marked `Verified` in this file until its test exists and passes
in CI. Until then it stays `PLANNED`.

## Related

- Invariants (structural): `domain-model.md` §Invariants
- Enforcement points: `../03-architecture/security.md`, `../11-security/security.md`
- Acceptance tests: `../09-testing/acceptance-criteria.md`

*End of docs/02-domain/business-rules.md*