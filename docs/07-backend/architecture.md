# docs/07-backend/architecture.md

> **Status:** Current | **Owner:** Backend Lead | **Last Updated:** 2026-10-02
> **Source:** Technical_Architecture_Instructions §4–§6; **Authoritative:** `ARCHITECTURE.md` §5
> **Purpose:** backend implementation detail under the approved architecture.

## Runtime

```text
Node.js + TypeScript + NestJS + Fastify adapter
```

One deployable image, two entrypoints: `api` (HTTP) and `worker` (jobs).

## Module Anatomy (mandatory)

```text
apps/api/src/modules/<context>/
├── <context>.module.ts        # wiring; exports ONLY the public service
├── <context>.controller.ts    # thin HTTP boundary
├── dto/                       # request/response DTOs + validation
├── <context>.service.ts       # application service / use-cases
├── domain/                    # entities, value objects, invariants, policies
├── repositories/              # tenant-scoped data access
├── events/                    # domain events emitted/consumed
├── policies/                  # authorization + field-level rules
└── <context>.spec.ts          # co-located tests
```

## Request Pipeline

```text
Request
 → RequestIdInterceptor (attach/propagate X-Request-Id)
 → LoggingInterceptor (structured JSON)
 → RateLimitGuard
 → AuthenticationGuard (session | bearer | api key)
 → TenantGuard (resolve + enforce tenant)
 → PermissionsGuard (RBAC resource:action)
 → PolicyGuard (ABAC attribute rules)
 → FieldPolicyInterceptor (strip/deny restricted attributes)
 → ValidationPipe (DTO, whitelist, forbidNonWhitelisted)
 → Controller → Service → Domain → Repository
 → AuditInterceptor (append-only audit on mutations)
 → ResponseMapper (snake_case, envelope)
 → ExceptionFilter (stable error codes)
```

Guards and interceptors are registered globally in `app.module.ts`; per-route
overrides use decorators. **Deny by default:** a route without an explicit
permission declaration is treated as denied.

## Core Services (`core/`)

| Service | Responsibility |
|---------|----------------|
| `config` | Typed, validated configuration loaded once at boot; fail-fast on missing vars |
| `database` | DataSource, tenant-scoped repository base, transaction helper, RLS session binding |
| `logging` | Structured JSON logger with request/tenant/user context |
| `errors` | Domain error classes + global exception filter + error code registry |
| `health` | `/health` (liveness) and `/ready` (DB + Redis + storage checks) |
| `events` | In-process domain event bus (module decoupling) |
| `security` | Password hashing, token signing/rotation, CSRF helpers, API-key hashing |

## Ports and Adapters (`core/ports` + implementations)

```text
StoragePort    MinIO (S3 API)          | Azure Blob, AWS S3
SearchPort     PostgreSQL FTS          | OpenSearch
NotifierPort   in-app + SMTP           | SMS, WhatsApp, Telegram, push
IdpPort        local credentials       | OIDC, SAML, Entra, Google, LDAP
AiPort         disabled (no-op)        | provider adapters
AntivirusPort  no-op (logged)          | ClamAV / vendor
```

Adapters are selected by configuration; the domain depends only on the interface.

## Worker

BullMQ queues (Redis). Jobs are idempotent, carry `tenant_id` + `request_id`, use
exponential backoff, and dead-letter after max attempts.

```text
queues: exports · renditions · notifications · webhooks · reports · automation · imports
```

## Conventions

- No business logic in controllers; no repository access from controllers.
- Cross-module communication only through exported services or domain events.
- Transactions wrap multi-write use-cases; audit is written in the same transaction.
- All list endpoints implement filtering/sorting/pagination helpers from `common/`.
- No `any` in domain code; DTOs are the only place untrusted input is shaped.
- Python is forbidden inside `apps/api` (ADR-001).

## Testing (co-located)

```text
*.spec.ts        unit — services, domain rules, policies
*.int.spec.ts    integration — repositories against a real test database
*.api.spec.ts    API — contracts, validation, error codes
*.authz.spec.ts  authorization + tenant isolation (mandatory per protected route)
```

See `docs/09-testing/strategy.md` for the full pyramid and commands.

## Related

- Architecture (authoritative): `ARCHITECTURE.md` §5, §9
- Modules catalog: `docs/07-backend/modules.md`
- API conventions: `docs/05-api/api.md`
- Data access: `docs/04-data/database.md`