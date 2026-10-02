# docs/07-backend/services.md

> **Status:** Current (design) — implementation `PLANNED` | **Owner:** Backend Lead
> **Last Updated:** 2026-10-02 | **Source:** `ARCHITECTURE.md` §2/§9, ADR-004/007/009/010/014
> **Purpose:** the cross-cutting services and ports every module depends on.

## 1. Core Services (`apps/api/src/core/`)

| Service | Responsibility | Must never |
|---------|----------------|-----------|
| `config` | Typed, validated configuration loaded once at boot; fail-fast on any missing/invalid variable | contain defaults for secrets or tenant values |
| `database` | DataSource, transaction helper, tenant-scoped repository base, RLS session binding (`app.tenant_id`) | be bypassed for "quick" queries |
| `logging` | Structured JSON logger carrying `request_id`, `tenant_id`, `user_id`, `module`, `action`, `duration_ms` | log secrets/PII or accept `console.log` |
| `errors` | Domain error classes, error-code registry, global exception filter producing the envelope | leak stack traces or internal identifiers |
| `health` | `/health` liveness and `/ready` (DB + Redis + storage probes) | require tenant context |
| `events` | In-process domain event bus for module decoupling (outbox-style ordering with the transaction) | be treated as the queue for durable work |
| `security` | Argon2id hashing, token signing/rotation, API-key hashing, CSRF helpers, constant-time comparison | expose raw secrets after creation |
| `pagination` | Cursor encode/decode, filter/sort allow-list helpers, response `meta` shaping | accept arbitrary filter fields |
| `validation` | DTO factories + Zod-aligned contracts re-exported for the web app (ADR-015) | become the only validation (DB constraints also apply) |
| `idempotency` | `Idempotency-Key` storage/lookup for state-critical POSTs | be optional on job-creating endpoints |

## 2. Access Service Contract (used by every module)

```text
AccessService
  ├── can(actor, permission, resource?) → boolean
  ├── assert(actor, permission, resource?)                  → throws PERMISSION_DENIED
  ├── scopeFilter(actor, resourceType)  → predicate/objects for ABAC
  ├── fieldPolicy(actor, resourceType)  → allowed|masked|denied fields
  └── effectivePermissions(actor)       → union of roles minus explicit denials (BR-014)
```

Rules:

1. Authorization is evaluated **server-side only**; UI checks are UX (`R-18`).
2. Field-level policy is applied in the response mapper so serializers cannot
   forget it (`FieldPolicyInterceptor`).
3. `assert` failures emit a security event with the same `request_id`.
4. ABAC narrows, never widens: it is applied *after* RBAC and can only reduce.

## 3. Ports and Adapters

```text
Port            Default implementation        Swap targets
──────────────  ────────────────────────────  ────────────────────────────────
StoragePort     MinIO (S3 API)                AWS S3, Azure Blob, GCS
SearchPort      PostgreSQL FTS                OpenSearch / Elasticsearch
NotifierPort    in-app + SMTP                 SMS, WhatsApp, Telegram, push
IdpPort         local credentials             OIDC, SAML, Entra ID, Google, LDAP
AiPort          disabled no-op                provider adapters (ADR-014)
AntivirusPort   no-op (logged)                ClamAV / commercial AV
MapPort         public OSM tiles              self-hosted tiles (Yemen/offline, Q-07)
PaymentPort     disabled                      gateways for donations
```

Every port is defined as an interface in `core/ports` with a token for DI.
Selection is by configuration — never by an `if (tenantId === ...)` branch
(`ARCHITECTURE.md` P10). Adding a new backing provider is a new adapter plus an
ADR when it changes an architectural assumption.

## 4. Port Contracts (shape, not implementation)

### `StoragePort`

```text
put(key, stream, { contentType, checksum, metadata }) → StoredObject
get(key) → stream
head(key) → StoredObject metadata
delete(key) → void
signedUrl(key, { expiresIn, disposition }) → string      // time-limited only (BR-023)
```

Rules: keys are generated server-side and normalised (never user-supplied paths);
`contentType` is the detected magic-byte type, not the client's claim; signed URLs
are short-lived and audited for sensitive objects.

### `SearchPort`

```text
index(doc: SearchDocument) → void
remove(entityType, id) → void
search({ q, filters, tenantId, permissionScope, pagination }) → SearchResult
reindex(entityType) → jobId
```

Rules: `tenantId` **and** `permissionScope` are mandatory arguments — a call
without them does not compile (Q-11); results are filtered by the owner module's
visibility rules before indexing; restricted/sensitive entities are excluded by
policy (BR-062).

### `NotifierPort`

```text
send({ channel, templateKey, recipient, variables, tenantId, requestId }) → DeliveryRef
status(deliveryRef) → delivery state
```

Rules: templates are per-tenant and localized; delivery attempts are recorded;
failures retry with backoff then dead-letter (BR-083); no provider SDK types leak
into the domain.

### `IdpPort`

```text
authenticate(credentials) → IdentityResult
provision(profile, tenantId) → UserRef
```

Rules: local credentials are the default; external IdPs are adapters added when
configured; provisioning never auto-grants elevated roles.

### `AiPort`

```text
request({ capability, input, tenantId, actorId, promptVersion }) → DraftRef
```

Rules: disabled by default (no-op); output is always a **draft** requiring human
review (BR-090); inherits the caller's permissions and never broadens them
(BR-091); sensitive data is redacted or blocked per the privacy matrix (BR-094);
requests/responses store provider, model, prompt version, actor, review status.

## 5. Worker Services (`apps/worker`)

| Queue | Purpose | Idempotency key |
|-------|---------|-----------------|
| `exports` | Async exports → signed artifact | `export_job.id` |
| `renditions` | Image/media derivatives | `media_asset.id + preset` |
| `notifications` | Outbound delivery attempts | `notification.id + channel + attempt` |
| `webhooks` | Signed delivery + retries | `webhook_delivery.id + attempt` |
| `reports` | Report generation with progress | `report_run.id` |
| `automation` | Rule evaluation and actions | `automation_execution.id` |
| `imports` | Validate-then-commit batch processing | `import_job.id + batch` |
| `search-index` | Projection rebuild/refresh | `entity_type + entity_id + version` |
| `maintenance` | Retention, cleanup, integrity checks | date bucket |

Rules: every job carries `tenant_id` + `request_id`; jobs are idempotent
(re-running must not duplicate effects); exponential backoff with a max attempt
count; dead-lettered jobs are visible in the admin console (`ARCHITECTURE.md` §13).

## 6. Reusable Behaviours Modules Must Not Re-invent

```text
[ ] tenant scoping and RLS binding                     → database service
[ ] permission + ABAC evaluation                       → access service
[ ] field-level masking in responses                   → field policy interceptor
[ ] audit writing inside the mutation transaction       → audit interceptor/service
[ ] cursor pagination + filter/sort allow-lists        → pagination helpers
[ ] idempotency for state-critical POSTs               → idempotency service
[ ] file validation/quarantine/signed URLs             → files module + StoragePort
[ ] notification delivery + retries                    → notifications module
[ ] error envelope and code registry                   → errors filter
```

Duplicating any of these inside a module is a review blocker.

## Related

- Pipeline and guards: `architecture.md` · Modules: `modules.md`
- Data access and tenancy: `../04-data/database.md`, `ADR-0004`
- Errors: `../05-api/errors.md` · Security: `../11-security/security.md`

*End of docs/07-backend/services.md*