# docs/11-security/privacy.md

> **Status:** Current (model) — controls `PLANNED` | **Owner:** Security Lead
> **Last Updated:** 2026-10-02 | **Source:** `ARCHITECTURE.md` §9; ADR-014; `AGENTS.md` §7
> **Scope:** sensitive-data classification, handling, retention, and the AI privacy
> matrix. Enforcement points live in `../03-architecture/security.md`; the threat
> model and audit model live in `security.md`.

## 1. Data Classification

Every entity and field carries exactly one class. The class drives access, masking,
search indexing, export, logging, and retention.

| Class | Definition | Examples |
|-------|-----------|----------|
| `public` | Safe for anonymous publication | Published pages, published project summaries |
| `internal` | Tenant staff only, low harm if exposed within tenant | Programs, projects, non-sensitive settings |
| `confidential` | Restricted to specific roles within the tenant | Grants, donors, staff records, budgets, audit |
| `restricted` | Sensitive personal data with field-level rules | Beneficiary, household, case, referral |
| `sensitive` | Highest scrutiny; separate handling and audit | Safeguarding incidents, complaints (identity) |

The per-entity class is catalogued in `../02-domain/entities.md`. A field may carry a
**stricter** class than its entity (field-level policy, BR-015/BR-066).

## 2. Handling Rules by Class

| Rule | public | internal | confidential | restricted | sensitive |
|------|:------:|:--------:|:------------:|:----------:|:---------:|
| Requires tenant context | — | ✅ | ✅ | ✅ | ✅ |
| Requires permission | — | ✅ | ✅ | ✅ + ABAC | ✅ + ABAC |
| Field-level masking | — | — | sometimes | ✅ | ✅ |
| Indexed in generic search | ✅ | ✅ | tenant-scoped | ❌ | ❌ |
| Included in generic export | ✅ | ✅ | role-scoped | ❌ | ❌ |
| Reason capture on access/export | — | — | export only | ✅ | ✅ |
| Separate audit stream | — | — | — | — | ✅ |
| Redacted in audit before/after | — | — | ✅ | ✅ | ✅ |
| Eligible for AI calls | ✅ | ✅ | redact/block | block | block |

## 3. Field-Level Policy

- Field policy is **independent of record readability**: a record may be readable
  while specific attributes are denied or masked (BR-015).
- Masking applies in lists, aggregates, and reports unless the actor holds the field
  grant (BR-066). Direct identifiers are masked by default.
- Denied field reads/writes emit a **security event** (not a business audit row).
- Elevated access to restricted/sensitive fields requires the elevated permission
  **and** a captured reason (BR-061).

## 4. Retention & Erasure

| Data | Default retention posture | Notes |
|------|---------------------------|-------|
| Audit records | Long-lived; never shortened without a decision | Append-only; immutability required |
| Sensitive case/safeguarding | Per policy; controlled deletion workflow | Deletion is audited; legal-hold overrides |
| Beneficiary data | Per consent + programme need | Consent recorded before processing (BR-065) |
| Public content | Indefinite until unpublished/archived | Version history retained |
| Uploads/documents | Per classification + retention policy | Signed URLs; no public object paths |

- Deletion and erasure are **controlled workflows**, never ad-hoc SQL (`../04-data/migrations.md`).
- Retention enforcement is automated by the `data-governance` module (Phase 5, BR-074).
- Every export and deletion is permission-checked, rate-limited, and audited (BR-064).

## 5. AI Privacy Matrix (ADR-014)

| Class | AI eligibility | Handling |
|-------|----------------|----------|
| `public` | Allowed | May be sent to a configured provider |
| `internal` | Allowed with logging | Provider, model, actor recorded |
| `confidential` | Redact or block | Identifiers redacted before the call; block if not safely redactable |
| `restricted` | Blocked | Never sent to an AI provider |
| `sensitive` | Blocked | Never sent; separate audit stream respected |

Additional AI rules: AI inherits the caller's permissions and never broadens them
(BR-091); AI output is a draft requiring human approval (BR-090); AI never silently
overwrites a record (BR-092); the sidecar has no canonical-store ownership.

## 6. GIS Privacy

- Precise coordinates of sensitive locations are never published in public map
  layers (BR-034); public layers use reduced precision or aggregated areas.
- Layer policy decides what is visible to which audience (R-08).
- Geofence and boundary data are tenant-scoped and permission-filtered.

## 7. Logging & Error Hygiene

- No PII, secrets, tokens, or full sensitive field values in logs, traces, or error
  payloads (`AGENTS.md` §7; `../05-api/errors.md`).
- `request_id` correlates a request across UI → API → worker without carrying PII.
- Audit before/after values are redacted per classification before storage.

## 8. Privacy Review Checklist (per task touching personal data)

```text
[ ] Entity/field class declared and consistent with entities.md
[ ] Access requires permission (+ ABAC where applicable)
[ ] Field-level masking applied in lists/aggregates/reports
[ ] Search + export exclusion where required
[ ] Reason capture for restricted/sensitive access & export
[ ] Separate audit stream for safeguarding writes
[ ] Retention/erasure path defined and audited
[ ] AI privacy matrix respected (redact/block)
[ ] No PII in logs, traces, or error payloads
[ ] Negative tests for read/list/search/export/report surfaces
```

## Related

- Threat model + audit model: `security.md`
- Enforcement points: `../03-architecture/security.md` · Rules: `../02-domain/business-rules.md` §7, §10
- Entity classes: `../02-domain/entities.md` · Acceptance: `../09-testing/acceptance-criteria.md` §6
- Decisions: ADR-013, ADR-014

*End of docs/11-security/privacy.md*
