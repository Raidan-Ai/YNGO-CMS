# docs/05-api/errors.md

> **Status:** Current (contract) — implementation `PLANNED` | **Owner:** API Owner
> **Last Updated:** 2026-10-02 | **Source:** `ARCHITECTURE.md` §8, `api.md`
> **Purpose:** one error envelope and one stable code catalogue for the whole API.

## 1. Envelope

Every non-2xx response uses exactly this shape — no exceptions, no stack traces,
no PII (R-03):

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "One or more fields are invalid",
    "details": [
      { "field": "start_date", "issue": "must be before end_date" }
    ],
    "request_id": "01J0ABCDEF"
  }
}
```

| Field | Rules |
|-------|-------|
| `code` | stable machine identifier from §3; never a raw exception class |
| `message` | short, human-readable, safe to display; localized by the client from `code` where possible |
| `details` | optional array of `{ field, issue }` or `{ key, issue }`; only for `VALIDATION_ERROR`, `CONFLICT`, and import/report row failures |
| `request_id` | always present; matches `X-Request-Id` and the server log/audit record |
| `meta` | optional, non-sensitive context (e.g. `retry_after_seconds`, `limit`, `window`) |

Multi-error imports and report jobs return per-row results in the job payload, not
as a second envelope shape.

## 2. HTTP Status Mapping

| Status | Meaning | Typical codes |
|--------|---------|---------------|
| 400 | Malformed request (unparseable body, bad cursor) | `BAD_REQUEST` |
| 401 | Not authenticated, expired, or invalid credentials | `AUTH_REQUIRED`, `AUTH_INVALID_CREDENTIALS`, `AUTH_TOKEN_EXPIRED`, `AUTH_TOKEN_REVOKED`, `AUTH_MFA_REQUIRED` |
| 403 | Authenticated but not allowed (or wrong tenant) | `PERMISSION_DENIED`, `FIELD_FORBIDDEN`, `TENANT_FORBIDDEN`, `TENANT_REQUIRED`*, `ACCOUNT_SUSPENDED` |
| 404 | Not found, or found in another tenant (deliberately indistinguishable) | `NOT_FOUND`, `<ENTITY>_NOT_FOUND` |
| 409 | State conflict / duplicate | `CONFLICT`, `<ENTITY>_STATE_CONFLICT`, `WORKFLOW_TRANSITION_INVALID`, `DUPLICATE_KEY` |
| 410 | Feature disabled by configuration | `FEATURE_DISABLED` |
| 413 | Upload too large | `UPLOAD_TOO_LARGE` |
| 415 | Rejected media type | `UPLOAD_TYPE_NOT_ALLOWED` |
| 422 | Well-formed but invalid input (validation) | `VALIDATION_ERROR` |
| 429 | Rate limited | `RATE_LIMITED` |
| 500 | Unexpected server error | `INTERNAL_ERROR` |
| 503 | Dependency unavailable / not ready | `DEPENDENCY_UNAVAILABLE`, `SERVICE_NOT_READY` |

\* `TENANT_REQUIRED` returns 401 when no tenant can be resolved from an
unauthenticated caller and 403 when a tenant is resolved but not permitted.

## 3. Code Catalogue

### 3.1 Authentication and session (`AUTH_*`)

| Code | Status | Meaning |
|------|--------|---------|
| `AUTH_REQUIRED` | 401 | No usable credential was supplied |
| `AUTH_INVALID_CREDENTIALS` | 401 | Email/password or API key did not match |
| `AUTH_TOKEN_EXPIRED` | 401 | Access token or session expired |
| `AUTH_TOKEN_REVOKED` | 401 | Session/refresh token was revoked or rotated away (BR-011) |
| `AUTH_MFA_REQUIRED` | 401 | Second factor required before the session is usable |
| `AUTH_INVITATION_INVALID` | 401 | Invitation token unknown, used, or expired (BR-012) |
| `AUTH_RESET_INVALID` | 401 | Password-reset token unknown, used, or expired |
| `ACCOUNT_SUSPENDED` | 403 | Account is suspended/deactivated, not merely unauthorized (BR-013) |

### 3.2 Tenancy (`TENANT_*`)

| Code | Status | Meaning |
|------|--------|---------|
| `TENANT_REQUIRED` | 401/403 | No tenant context could be resolved for a tenant-scoped route (BR-002) |
| `TENANT_FORBIDDEN` | 403 | Caller has no membership in the requested tenant |
| `TENANT_SUSPENDED` | 403 | Tenant is suspended; data paths are closed |
| `TENANT_DOMAIN_UNVERIFIED` | 403 | Hostname used for resolution is not verified |

Cross-tenant access to an existing foreign resource returns **404**
`<ENTITY>_NOT_FOUND` rather than 403, so resource existence is not leaked
(`ADR-0004` §Mandatory rules 5). `TENANT_FORBIDDEN` is reserved for routes where
the tenant itself is the subject (e.g. tenant settings endpoints).

### 3.3 Authorization (`PERM_*`)

| Code | Status | Meaning |
|------|--------|---------|
| `PERMISSION_DENIED` | 403 | Caller lacks the required `resource:action` permission (BR-003) |
| `SCOPE_DENIED` | 403 | Permission held, but the ABAC scope excludes this record (BR-004) |
| `FIELD_FORBIDDEN` | 403 | Field-level policy denies a specific attribute (BR-015/BR-066) |
| `REASON_REQUIRED` | 422 | Accessing/exporting a restricted record requires a captured reason (BR-061) |
| `FOUR_EYES_REQUIRED` | 409 | Approver must differ from the author for this flow |

### 3.4 Validation, conflict, workflow

| Code | Status | Meaning |
|------|--------|---------|
| `VALIDATION_ERROR` | 422 | One or more fields failed DTO validation; `details` lists them (ADR-015) |
| `UNKNOWN_FIELD` | 422 | Body/query contained a field outside the allow-list |
| `INVALID_FILTER` / `INVALID_SORT` | 422 | Filter or sort field is not whitelisted for the endpoint |
| `INVALID_CURSOR` | 400 | Cursor is malformed or from a different query shape |
| `CONFLICT` | 409 | Generic state conflict |
| `DUPLICATE_KEY` | 409 | Unique business key already exists in this tenant |
| `OPTIMISTIC_LOCK_CONFLICT` | 409 | Record was modified concurrently (`updated_at`/version mismatch) |
| `WORKFLOW_TRANSITION_INVALID` | 409 | Requested status change is not a defined transition (BR-021) |
| `RECORD_LOCKED` | 409 | Another user holds the content lock (BR-026) |
| `DEPENDENCY_CONFLICT` | 409 | A required related record is missing or incomplete (BR-033/BR-040/BR-050) |

### 3.5 Platform and infrastructure

| Code | Status | Meaning |
|------|--------|---------|
| `RATE_LIMITED` | 429 | Limit exceeded; `meta.retry_after_seconds` and `Retry-After` are set |
| `FEATURE_DISABLED` | 410 | Module/AI capability disabled by configuration |
| `DEPENDENCY_UNAVAILABLE` | 503 | Database, Redis, or object storage is unavailable (`ARCHITECTURE.md` §13) |
| `SERVICE_NOT_READY` | 503 | `/ready` is failing; retry after backoff |
| `UPLOAD_TOO_LARGE` | 413 | Upload exceeds the configured size limit (BR-022) |
| `UPLOAD_TYPE_NOT_ALLOWED` | 415 | MIME/magic-byte/extension check failed (BR-022) |
| `QUARANTINED` | 409 | Object is quarantined pending a scan result |
| `EXPORT_TOO_LARGE` | 422 | Result set exceeds synchronous export limits; use an async job (BR-073) |
| `INTERNAL_ERROR` | 500 | Unexpected failure; nothing sensitive is returned |

## 4. Per-Module `<ENTITY>_NOT_FOUND` Codes

Modules use the singular resource name in snake case:

```text
USER_NOT_FOUND · ROLE_NOT_FOUND · CONTENT_NOT_FOUND · MEDIA_ASSET_NOT_FOUND
DOCUMENT_NOT_FOUND · PROGRAM_NOT_FOUND · PROJECT_NOT_FOUND · INDICATOR_NOT_FOUND
GRANT_NOT_FOUND · DONOR_NOT_FOUND · PARTNER_NOT_FOUND · FORM_NOT_FOUND
BENEFICIARY_NOT_FOUND · CASE_NOT_FOUND · SAFEGUARDING_CASE_NOT_FOUND
COMPLAINT_NOT_FOUND · WORKFLOW_INSTANCE_NOT_FOUND · REPORT_RUN_NOT_FOUND
WEBHOOK_ENDPOINT_NOT_FOUND · API_KEY_NOT_FOUND
```

Rules:

1. A foreign-tenant id returns the same code as a non-existent id.
2. Restricted/sensitive resources never return a code that confirms existence to
   an unauthorized caller (`BENEFICIARY_NOT_FOUND` also covers denied access).
3. `NOT_FOUND` remains valid for non-resource routes (e.g. unknown report template).

## 5. Logging and Audit Ties

- Every error carries `request_id`; the server log record includes `code`,
  `route`, `tenant_id`, `user_id`, and `duration_ms` — never the request body of
  sensitive endpoints.
- `PERMISSION_DENIED`, `SCOPE_DENIED`, `FIELD_FORBIDDEN`, and failed logins write a
  **security event** (not a business audit row) with the same `request_id`.
- Error responses never include SQL, stack traces, internal hostnames, or
  third-party error text verbatim.

## 6. Client Handling Rules

```text
401 AUTH_TOKEN_EXPIRED          → refresh once, then retry; on failure sign out
401 AUTH_MFA_REQUIRED           → route to the MFA step
403 PERMISSION_DENIED / SCOPE_DENIED → render the unauthorized state; do not retry
403 FIELD_FORBIDDEN             → keep the record visible; hide/mask the field
409 WORKFLOW_TRANSITION_INVALID → refresh the record and re-present valid actions
409 OPTIMISTIC_LOCK_CONFLICT    → show a merge/refresh prompt; never overwrite blindly
422 VALIDATION_ERROR            → map details to form fields; server stays authoritative
429 RATE_LIMITED                → honour Retry-After with backoff and a visible message
503 DEPENDENCY_UNAVAILABLE      → show a retryable error state; queue nothing silently
```

## 7. Checklist for a New Endpoint

```text
[ ] declares the exact codes it can return
[ ] no hand-rolled response shape — the filter produces the envelope
[ ] denial paths return 403/404 per §2/§3 and are covered by an authz test
[ ] validation failures carry field-level details
[ ] the code list is reflected in the generated OpenAPI responses
```

## Related

- Conventions and headers: `api.md`
- Endpoint catalogue: `endpoints.md`
- OpenAPI generation policy: `openapi.md`
- Enforcement pipeline: `../07-backend/architecture.md`

*End of docs/05-api/errors.md*