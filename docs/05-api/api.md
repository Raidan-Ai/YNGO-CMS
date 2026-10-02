# docs/05-api/api.md

> **Status:** Current | **Owner:** API Owner | **Last Updated:** 2026-10-02
> **Source:** Technical_Architecture_Instructions §45–§58; Eng_Package §6.43–§6.45
> **Single source of truth for API conventions:** `ARCHITECTURE.md` §8
> **Machine contract:** `docs/05-api/openapi.yaml` — **generated** from code, never hand-edited.

## Purpose

This document explains API usage and conventions for consumers. The authoritative
machine-readable contract is the generated OpenAPI document.

## Base Path and Versioning

```text
https://<host>/api/v1/...
```

- Breaking changes require a new major version (`/api/v2`) plus `Deprecation` and
  `Sunset` response headers on the old version.
- Non-breaking additions are allowed within a major version.

## Authentication

| Caller | Mechanism |
|--------|-----------|
| Admin SPA | Server session cookie (httpOnly, SameSite) |
| Scripts / integrations | Bearer access token + rotating refresh token |
| Machine-to-machine | API key (`Authorization: Bearer <key>` or `X-API-Key`) |
| Public site/portals | Anonymous where the resource is public; otherwise the same mechanisms |

All requests to tenant-scoped routes must resolve a tenant (see ADR-005). The
tenant is never accepted from the request body.

## Standard Headers

```text
X-Request-Id        accepted or generated; echoed in the response and logs
X-Tenant-Id         tenant hint; validated against the caller's membership
Idempotency-Key     required on state-critical POSTs
Accept-Language     ar | en (affects localized content where applicable)
```

## Pagination, Filtering, Sorting

```text
GET /api/v1/projects?limit=25&cursor=<opaque>&sort=-created_at&status=active
```

- Cursor pagination for large/streamed collections (default 25, max 100).
- `page`/`per_page` permitted for admin tables where total counts are needed.
- Only whitelisted filter and sort fields are accepted; unknown fields → 422.
- Responses include `meta` with `next_cursor`, `has_more`, and (where cheap) `total`.

## Error Contract

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

Stable code families:

```text
AUTH_*        unauthenticated / expired / invalid credentials
TENANT_*      TENANT_REQUIRED, TENANT_FORBIDDEN
PERM_*        PERMISSION_DENIED, FIELD_FORBIDDEN
VALIDATION_ERROR
NOT_FOUND / <ENTITY>_NOT_FOUND
CONFLICT / <ENTITY>_STATE_CONFLICT
RATE_LIMITED
DEPENDENCY_UNAVAILABLE
INTERNAL_ERROR
```

HTTP mapping: 400 malformed · 401 unauthenticated · 403 forbidden/tenant ·
404 not found · 409 conflict · 422 validation · 429 rate limited ·
503 dependency unavailable · 500 internal. Stack traces are never returned.

## Rate Limiting

Applied per API key, per user, and per tenant with `429` + `Retry-After`.
Limits are documented per endpoint group in the developer portal.

## Audit Behaviour

Mutating endpoints emit an audit record (actor, tenant, entity, before/after,
request id). Sensitive exports and reads of restricted records are audited too.

## Webhooks

Events: `content.created|updated|published`, `project.created|updated`,
`grant.created|approved`, `form.submitted`, `user.created`, `document.uploaded`.

Delivery: HMAC-signed payload, exponential backoff retry, delivery log with
attempt count and response status, automatic disable after repeated failure.

## Endpoint Groups

`auth` · `tenants` · `users` · `organizations` · `content` · `media` ·
`documents` · `programs` · `projects` · `indicators` · `grants` · `donors` ·
`partners` · `stakeholders` · `beneficiaries` · `cases` · `safeguarding` ·
`volunteers` · `staff` · `assets` · `fleet` · `travel` · `procurement` ·
`finance` · `events` · `tasks` · `communications` · `advocacy` · `forms` ·
`workflows` · `automations` · `notifications` · `reports` · `dashboards` ·
`analytics` · `search` · `gis` · `integrations` · `webhooks` · `api-keys` ·
`import-export` · `audit`.

Full per-endpoint detail (method, path, purpose, auth, authz, request, response,
validation, errors, pagination, filtering, sorting, rate limits, audit) is defined
in the generated OpenAPI document.

## SDKs

TypeScript and Python SDKs are **generated** from OpenAPI in CI. Hand-maintained
divergent copies are forbidden.

## Related

- Conventions (authoritative): `ARCHITECTURE.md` §8
- OpenAPI: `docs/05-api/openapi.yaml`
- Authz model: `docs/11-security/security.md`
- Backend layering: `docs/07-backend/architecture.md`