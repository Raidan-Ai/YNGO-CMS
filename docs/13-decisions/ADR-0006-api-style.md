# ADR-0006 — API Style and Contract

> **Status:** PROPOSED | **Owner:** API Owner | **Date:** 2026-10-02
> **Canonical rationale:** `../../TECHNICAL_DECISIONS.md` (ADR-006)
> **Blocks:** all endpoints, SDK generation, webhooks

## Context

The platform is API-first: the admin SPA, public website, portals, mobile/PWA
clients, and third-party integrations all consume the same surface. The contract
must be describable, versionable, and generatable into SDKs; it must also be
implementable by non-expert integrators.

## Options

| Option | Pros | Cons |
|--------|------|------|
| A. REST + OpenAPI `/api/v1` | Universal tooling, cacheable, easy SDK generation, matches NGO integrator expectations | Over/under-fetching for complex screens |
| B. GraphQL | Flexible reads | Harder authz-per-field story, caching complexity, heavier ops |
| C. RPC/tRPC | Fast for the first-party web app | Poor fit for third-party/self-hosted integrators |

## Decision

**REST under `/api/v1` with resource-oriented plural paths, JSON `snake_case`
bodies, a stable response envelope, and an OpenAPI document generated from the
implementation (never hand-written).**

## Mandatory Rules

1. Paths: `/api/v1/<resource>` plural, lowercase, hyphenated.
2. Resource ids are opaque UUIDs; no sequential ids in the contract.
3. Collections return `{ data, meta }` with cursor pagination; single resources
   return `{ data }`; errors return `{ error }` (`../05-api/errors.md`).
4. Filtering, sorting, and field selection use explicit query parameters; unknown
   parameters fail validation rather than being ignored.
5. State-critical POST/PATCH honour `Idempotency-Key`.
6. Breaking changes require a new version path or an additive-first strategy
   documented in an ADR; responses are additive within a version.
7. Every endpoint declares its permission(s) and is covered by an authz test.

## Consequences

- Positive: one contract for all clients; generated SDKs; simple debugging.
- Negative: dashboard-style screens may need dedicated aggregate endpoints
  (allowed, must be documented and permission-checked).
- Follow-up: keep `../05-api/api.md` and `../05-api/endpoints.md` in sync with code.

## Related

- Conventions: `../05-api/api.md` · Errors: `../05-api/errors.md`
- OpenAPI policy: `../05-api/openapi.md` · SDKs: `../08-integrations/integrations.md`

*End of ADR-0006*