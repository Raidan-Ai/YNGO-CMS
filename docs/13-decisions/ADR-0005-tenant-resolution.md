# ADR-0005 — Tenant Resolution

> **Status:** ACCEPTED | **Owner:** Lead Architect | **Date:** 2026-10-02
> **Canonical rationale:** `../../TECHNICAL_DECISIONS.md` (ADR-005)
> **Decides:** `../../OPEN_QUESTIONS.md` Q-03 | **Blocks:** TenantGuard, routing, auth, SDKs

## Context

Every request must be attributable to exactly one tenant before any data access.
The platform also serves a public website and external portals, and must remain
hostable on a single self-managed Linux server with simple TLS.

## Options

| Option | Pros | Cons |
|--------|------|------|
| A. Header `X-Tenant-Id` | Trivial for API clients, SDKs, integration tests, self-hosting | Requires client discipline; not human-friendly for browsers |
| B. Subdomain (`acme.yngo.ye`) | User-friendly, session-friendly, no client change | Wildcard TLS + DNS work per tenant; harder self-hosting; host-header pitfalls |
| C. Path prefix (`/t/acme`) | Simple hosting | Pollutes every route/contract; easy to forget; ugly in the public contract |

## Decision

**`X-Tenant-Id` header is the canonical v1 tenant resolution mechanism, resolved by
a global `TenantGuard` into an immutable `TenantContext`. Subdomain mapping may be
added later as an additional resolver behind the same context. Path-prefix tenant
routing is rejected.**

## Mandatory Rules

1. Requests with no resolvable tenant on a tenant-scoped route fail with
   `TENANT_REQUIRED` (no fallback to a default tenant).
2. The resolved tenant, when it comes from a credential (API key/session), must
   match the header; mismatch fails with `TENANT_FORBIDDEN` (`../05-api/errors.md`).
3. Session-bound users are pinned to their tenant; a header may not switch tenant
   within a session unless the user holds an explicit cross-tenant membership.
4. Public/anonymous routes are explicitly marked and carry a public tenant scope.
5. Adding a new resolver (subdomain, custom host) is additive: it must not change
   route contracts and must reuse `TenantContext`.

## Consequences

- Positive: one resolution rule for UI, SDKs, jobs, tests, and integrations.
- Negative: browser flows need the header injected by the web app's API client.
- Follow-up: document the resolver order in `../03-architecture/components.md`.

## Migration Impact

None — greenfield. Future subdomain support is additive.

## Related

- Isolation model: `ADR-0004-tenancy-isolation.md`
- Guards: `../07-backend/architecture.md` · Errors: `../05-api/errors.md`

*End of ADR-0005*
