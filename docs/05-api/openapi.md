# docs/05-api/openapi.md

> **Status:** Current (policy) — no spec generated yet | **Owner:** API Owner
> **Last Updated:** 2026-10-02 | **Source:** `AGENTS.md` §6, `ARCHITECTURE.md` §8, ADR-006
> **Rule:** OpenAPI is **generated from code**. A hand-written or hand-edited spec is
> forbidden and is a review blocker.

## 1. Why It Is Generated

| Reason | Consequence |
|--------|-------------|
| One source of truth | The decorators/DTOs are the contract; the document cannot drift (R-17) |
| Typed clients | The web app, SDKs, and tests consume the same generated types |
| Review signal | A diff in the generated spec means a contract change — reviewers see it |
| Honest docs | Cannot claim an endpoint that does not exist |

## 2. Generation Pipeline

```text
DTOs + decorators (apps/api)
   ↓  build
@nestjs/swagger document build (stable ordering, deterministic output)
   ↓
artifacts/openapi.json          (machine contract)
   ↓
├── /api/docs                   runtime UI (dev/staging; prod per policy)
├── packages/api-client         TypeScript client (generated)
├── packages/sdk                published SDK types (generated)
└── contract tests              response-shape assertions (generated expectations)
```

Build command (target, available from Phase 0):

```bash
pnpm --filter api openapi:generate     # writes artifacts/openapi.json
pnpm --filter api openapi:check        # fails if the committed artifact is stale
```

## 3. What Must Be Present for Every Endpoint

```text
[ ] summary + description (purpose, not a restatement of the path)
[ ] tags matching the module/bounded context
[ ] declared security scheme (or explicit public marker)
[ ] request DTO with all validation constraints reflected
[ ] response DTOs for 2xx success shapes
[ ] the error envelope for each documented failure (errors.md catalogue)
[ ] pagination/filter/sort parameters where the endpoint lists
[ ] tenant requirement implied by the module's guard policy
[ ] an example that contains no real personal data (ADR-013)
```

## 4. Versioning and Compatibility

- Generated spec describes `/api/v1` only. A future `/api/v2` is a separate
  document with an explicit migration note.
- Breaking change = removed/renamed field, changed type, changed enum, new required
  input, changed status code. These require a new major version.
- Additive changes (new optional field, new endpoint, new enum value consumed
  defensively) are allowed inside a major version.
- Removals inside a major version are announced with `Deprecation` and `Sunset`
  headers and documented before removal.

## 5. CI Gates

```text
[ ] generation succeeds with no schema errors
[ ] openapi:check passes — committed artifact matches code
[ ] linting of the spec (operationId uniqueness, tag coverage, no missing responses)
[ ] SDK/client regeneration produces no unexpected breaking diff
[ ] contract tests pass against the generated document
```

A PR that changes a handler without updating generated artifacts fails CI.

## 6. Client SDKs

- TypeScript client and SDK types are generated into `packages/api-client` and
  `packages/sdk`; hand-maintained parallel copies are forbidden.
- The Python SDK, when produced, is generated for integrators only — it never
  becomes part of `apps/api` (`ADR-001`).
- The web app imports the generated client; it does not hand-write fetch calls
  (`docs/06-frontend/architecture.md`).

## 7. Exclusions

The generated document **never** includes: internal worker/admin-only routes,
database or storage internals, secrets or credential fields, PII examples,
`/health` and `/ready` payload detail beyond status, or any route that exists only
for platform administration across tenants.

## Related

- Conventions (authoritative): `ARCHITECTURE.md` §8
- Errors documented in the spec: `errors.md`
- Catalogue of intent: `endpoints.md`
- Decision: `../13-decisions/ADR-0006-api-style.md`

*End of docs/05-api/openapi.md*