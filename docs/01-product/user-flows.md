# docs/01-product/user-flows.md

> **Status:** Current | **Owner:** Product Owner | **Last Updated:** 2026-10-02
> **Source:** Blueprint.md, Engineering_Package_v2 | **Depends on:** `personas.md`
> **Rule:** each flow lists its security-relevant steps explicitly.

## 1. Onboarding Flow (first install)

```text
1  Operator deploys the stack (docker compose up)
2  Operator runs `pnpm bootstrap:org`  (ADR-012)
     → supplies org name, admin email, admin password (typed, never defaulted)
3  System creates Tenant + Organization + Admin user + default role bundle
     → writes audit record; bootstrap route self-disables
4  Admin signs in → sees a genuinely empty system with guided next actions
5  Admin invites staff (P-01) → invitations are single-use and expiring
```

**Security notes:** no default credentials; bootstrap token required for the web
wizard variant; re-running bootstrap without a token fails.

## 2. Invite → Login → CMS Publish (critical E2E)

```text
1  Admin invites an Editor (P-13)          → Invitation created (expiring token)
2  Editor accepts invitation → sets password → Active
3  Editor creates a Page draft
4  Editor requests review                  → status Draft → InReview
5  Content Approver (P-14) reviews
     ├─ Changes requested → ChangesRequested (back to Editor)
     └─ Approve           → Approved
6  Approver publishes (or schedules)       → Published (public site revalidates)
7  All steps emit audit records + workflow transitions
```

**Invariants:** content cannot reach Published without an Approved revision;
status only changes through guarded transitions (FR-009, FR-020).

## 3. Program → Project → Indicator → Report (critical E2E)

```text
1  Programme Manager creates a Program                       (P-02)
2  Project Officer creates a Project under the Program        (P-03)
3  Locations added → rendered on map (privacy-filtered)      (FR-014)
4  Milestones and workplan defined
5  MEAL Officer defines Indicator + reporting period targets (P-04)
6  Indicator values recorded for the period (auditable)
7  Report generated asynchronously with progress             (FR-015)
8  Export (PDF/CSV/XLSX) is permission-checked, rate-limited, audited
```

**Invariants:** indicator values are period-scoped and auditable; no fabricated
measurements — an empty project shows an honest empty state (ADR-013).

## 4. Grant Lifecycle Flow

```text
Identified → Applied → Awarded → Active → Reporting → Closed
  1  Opportunity recorded / proposal created (P-05)
  2  Award recorded → Grant created, linked to Donor
  3  Milestones tracked; GrantReport blocked until milestones complete (BR)
  4  Deadline automation triggers notifications (real records only)
```

## 5. Sensitive Case Flow (restricted)

```text
1  Case Worker opens a Case for a Beneficiary/Household      (P-09)
2  Assessment recorded; consent captured where required
3  Referral / follow-up recorded
4  Case resolved → closed
   Access control:
   - record readable only with `cases:read` AND ABAC scope
   - restricted fields require an elevated permission (field-level policy)
   - every access/export writes an audit record with actor + reason
   - cases never appear in unscoped search or generic exports
```

## 6. Safeguarding Flow (highest restriction)

```text
Reported → Triage → Investigating → Actioned → Closed
  - visible only to Safeguarding Officers (P-10)
  - separate audit stream; excluded from generic search/exports
  - exports are controlled and require elevated permission + justification
  - investigator assignment is audited
```

## 7. Complaints / Feedback Flow (public intake)

```text
1  External complainant submits via public form (optionally anonymous)  (X-07)
   → rate-limited; spam controls; no account required
2  Officer triages → assigns → investigates → resolves
3  Complainant optionally notified of the outcome
```

## 8. External API Integration Flow

```text
1  Developer creates an API key (secret shown once)             (P-18)
2  Client calls /api/v1/... with bearer key + X-Tenant-Id
3  Rate limits + permission checks + tenant scoping applied
4  Client subscribes to webhooks (HMAC-signed, retried, logged)
5  Failures retry with exponential backoff → dead-letter, visible in admin
```

## 9. Field Offline Flow (Phase 10, opt-in)

```text
1  Collector downloads assigned forms to the PWA                (P-16)
2  Fills submissions offline (local encrypted store) + captures GPS
3  Queue syncs on reconnect; retries; conflicts resolved deterministically
4  Submissions enter the normal review workflow
   Rule: offline sync must be real; simulated offline mode is forbidden.
```

## 10. Flow-Level Non-Functional Expectations

| Flow | Expectation |
|------|-------------|
| Any data view | loading / empty / error / unauthorized / populated states |
| Any mutation | audit record + `request_id` traceable UI→API→worker |
| Any export | permission check + rate limit + audit |
| Any AI-assisted step | produces a draft requiring human approval (ADR-014) |
| Arabic UI | RTL layout, logical CSS properties, correct numerals/dates |

## Related

- Personas: `personas.md` · Requirements: `requirements.md`
- E2E tests: `../09-testing/acceptance-criteria.md`

*End of docs/01-product/user-flows.md*