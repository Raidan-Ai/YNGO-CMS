# AGENTS.md — YNGO-CMS / YemenNGO-CMS Execution Contract

> **Audience:** AI coding agents and human contributors working in this repository.
> **Status:** BINDING once adopted. Deviations require an ADR.
> **Read this file first, every session, before touching code.**

## 0. Quick Orientation

| Item | Value |
|------|-------|
| Project | YNGO-CMS (YemenNGO-CMS) — modular, multi-tenant, API-first NGO platform |
| Root | repository root (never assume a hard-coded absolute path) |
| Current state | **Greenfield** — specification complete, no application code yet |
| Canonical architecture | `ARCHITECTURE.md` |
| Decisions | `TECHNICAL_DECISIONS.md` (ADRs) |
| Work plan | `IMPLEMENTATION_PLAN.md` |
| Open decisions | `OPEN_QUESTIONS.md` |
| Risks | `RISK_REGISTER.md` |
| Audit baseline | `PROJECT_AUDIT.md` |
| Living docs | `docs/**` (single source of truth) |
| Frozen inputs | the original `YNGO-CMS_*.md` specification files (read-only) |

## 1. Absolute Rules

**NEVER**
```text
run destructive git commands (push --force, reset --hard, branch -D, clean -fdx)
delete files or code without searching for references first
commit secrets, tokens, passwords, or .env files
hard-code tenant ids, production domains, credentials, or limits
create mock/demo/sample/fake business data in runtime code
present AI output as an authoritative record without human approval
bypass tenant scoping, authorization, or audit on any data path
weaken security/audit/isolation to simplify delivery
claim work is complete when tests or validation have not run
change architecture without an ADR
```

**ALWAYS**
```text
read AGENTS.md + the relevant ARCHITECTURE/domain/API docs before coding
inspect the current code before editing it
keep changes minimal and scoped to the task
write tests with the change (unit + integration + authz/tenant where relevant)
update docs and regenerate OpenAPI when behaviour changes
run lint, typecheck, and tests before declaring done
review `git diff` before committing, and keep commits coherent
report reality honestly (pass/fail, implemented vs partial vs blocked)
```

## 2. Repository Layout (target)

```text
apps/      web (Next.js) · api (NestJS) · worker (BullMQ) · field (later P2) · docs
packages/  ui · types · sdk · config · i18n · auth · api-client
infra/     docker · compose · proxy · monitoring
migrations/
scripts/
tests/     e2e
docs/      structured documentation (source of truth)
```

## 3. Tech Stack (binding)

| Layer | Choice |
|-------|--------|
| Frontend | Next.js + React + TypeScript, Tailwind, shadcn/ui + Radix, TanStack Query, Zustand (client state only), React Hook Form + Zod, MapLibre |
| Backend | Node.js + TypeScript + NestJS + Fastify |
| Database | PostgreSQL + PostGIS (canonical) |
| Cache/Queue | Redis + BullMQ (`worker` entrypoint) |
| Object storage | S3-compatible (MinIO reference) behind `StoragePort` |
| API | REST `/api/v1`, OpenAPI generated |
| Optional AI | FastAPI + Python sidecar behind `AiPort` — bounded, human-reviewed |
| Package manager | pnpm workspaces |
| Containerization | Docker + Docker Compose |

Python MUST NOT be used in `apps/api`. Postgres is the only canonical store.

## 4. Coding Rules

**Backend (NestJS)**
- One module per bounded context under `apps/api/src/modules/<context>/`.
- Layering: Controller → DTO → Service → Domain → Repository. Controllers stay thin.
- Modules expose **only** their public service; import another module's repository is forbidden.
- Deny-by-default authorization: every handler declares required permissions.
- Tenant context is mandatory on every data path; use the tenant-scoped repository base.
- Validation at the edge with DTOs + global validation pipe.
- Structured JSON logging; no `console.log` in application code.
- No `any` in domain code; explicit types everywhere in the domain layer.
- No raw SQL in modules except PostGIS/performance cases with a comment explaining why.

**Frontend (Next.js)**
- App Router with locale segments (`/[locale]/...`); server components by default.
- Server state via TanStack Query; Zustand only for genuine client state.
- Forms: React Hook Form + Zod; server remains authoritative.
- RTL-safe styling only: logical properties (`ms/me/ps/pe/text-start/end`), never `left/right`.
- Every data view implements loading, empty, error, unauthorized, populated states.
- No business logic in components; no direct DB access.
- Typed API client generated from OpenAPI where practical.

**Naming**
- DB columns and JSON payloads: `snake_case`.
- TypeScript identifiers: `camelCase` / `PascalCase` (types/classes).
- Files: kebab-case for modules/routes, PascalCase for components.

## 5. Testing Rules

```text
Unit          every service/domain rule
Integration   repository + DB behaviour against a real test database
API           request/response contracts + validation + error codes
Authorization deny-by-default proven for every protected route
Tenant        cross-tenant negative tests for read/list/search/export/report/file/webhook
E2E           critical flows (invite→login→CMS publish; program→project→indicator→report)
```

- Fixtures live in `tests/**` only and are never imported by runtime code.
- A task is not complete until its tests pass locally.
- Do not weaken or delete a failing test to make a build pass.

## 6. Documentation Rules

- `docs/**` is the single source of truth; the six original spec files are frozen inputs.
- One fact lives in one place; other documents link to it rather than copy it.
- OpenAPI is generated from code — never hand-edited.
- Every task that changes behaviour updates the relevant doc in the same change.
- Each doc carries: `Status`, `Owner`, `Last Updated`, and `Source` where relevant.
- Architecture changes require an ADR before implementation.

## 7. Security Rules

```text
[ ] No secrets in source or logs; .env is git-ignored; .env.example documents keys
[ ] Passwords hashed (Argon2id); no default credentials; no admin/admin
[ ] RBAC (+ABAC) enforced server-side; frontend permissions are UX only
[ ] Tenant isolation proven by negative tests on every data surface
[ ] Field-level policy for Beneficiary/Case/Safeguarding (privacy-critical)
[ ] Uploads: MIME + magic-byte + extension allow-list + size + quarantine + signed URLs
[ ] Sensitive downloads and exports are audited
[ ] API keys hashed at rest; raw secret shown once
[ ] Webhooks HMAC-signed with retry/backoff and delivery logs
[ ] Audit log append-only; no UPDATE/DELETE grants
[ ] Dependencies scanned; no known critical vulnerabilities shipped
[ ] No PII in error payloads or tracing
```

## 8. Git Rules

- Work on a branch per task; never commit directly to the protected default branch.
- Commit messages: `type(scope): summary` (e.g. `feat(projects): add milestone CRUD`).
- One coherent change per commit; review `git diff` before committing.
- No secrets, no build artifacts, no `node_modules`, no editor files.
- Never force-push shared branches; never rewrite published history.
- Tag releases semantically; maintain `CHANGELOG.md`.

## 9. Migration Rules

- Append-only migrations; never edit a migration that has been applied.
- Every schema change ships with a tested rollback path.
- Back up before destructive/irreversible changes; record an ADR for breaking changes.
- Migrations run as a separate one-shot step before application rollout.
- New tenant-owned tables MUST include `tenant_id NOT NULL`, indexes leading with
  `tenant_id`, and the standard audit columns where meaningful.

## 10. Environment Rules

- `.env.example` lists every required variable with a safe placeholder and comment.
- Required variables are validated at boot; the app fails fast if any is missing.
- Secrets never appear in source, tests, logs, or CI output.
- `dev` / `test` / `staging` / `prod` use identical images with different configuration.
- No production URLs hard-coded anywhere in the codebase.

## 11. Task Execution Loop

For each task, in this exact order:

```text
1  READ      AGENTS.md + relevant ARCHITECTURE / domain / API / data docs
2  INSPECT   current code and tests for the affected area
3  PLAN      minimal change set; confirm dependencies and blast radius
4  IMPLEMENT smallest coherent change; no unrelated refactoring
5  TEST      write/run unit + integration + authz/tenant tests
6  VALIDATE  lint, typecheck, build, tests (exact commands from the task)
7  DOCUMENT  update docs + regenerate OpenAPI if behaviour changed
8  REVIEW    inspect git diff; ensure scope is correct
9  COMMIT    coherent message; one task per commit
```

Do not batch many tasks into one change. Do not leave a task partially done
without marking it blocked and recording the reason.

## 12. Definition of Done

A task is Done only when all of the following hold:

```text
[ ] Implementation matches the documented requirement/ADR
[ ] Unit + integration tests pass
[ ] Authorization + tenant-isolation tests pass where data is involved
[ ] Lint + typecheck + build pass
[ ] Error handling + logging + audit implemented
[ ] Docs updated (and OpenAPI regenerated if the API changed)
[ ] Migration tested (forward + rollback) if the schema changed
[ ] No mock data, no secrets, no hard-coded tenant/domain values
[ ] git diff reviewed and scoped to the task
```

## 13. Forbidden Actions (explicit)

```text
X  writing application code before Phase 0 gate approval
X  adding a 53rd module or a new bounded context without an ADR
X  using Python inside apps/api
X  using Redis or object storage as a database
X  returning fabricated statistics or seeding demo business records
X  hiding UI elements as a substitute for backend authorization
X  editing applied migrations
X  committing .env or any credential
X  force-pushing or rewriting shared history
X  declaring success without having run the validation commands
```

## 14. Reporting Format

Every agent session ends with a truthful report:

```text
Task:            <TASK-NNN or analysed scope>
Status:          DONE | PARTIAL | BLOCKED
Files changed:   <paths>
Tests run:       <command → result>
Validation:      lint/typecheck/build/test → results
Docs updated:    <paths or "none required">
Blockers:        <what is needed, or none>
Next action:     <next task or decision required>
```

Never claim GitHub pushes, deployments, CI runs, or test passes that were not
actually performed and observed.

## 15. Documentation Map (where to look)

| Need | File |
|------|------|
| System design, boundaries, security flow | `ARCHITECTURE.md` |
| Why a technology was chosen | `TECHNICAL_DECISIONS.md`, `docs/13-decisions/` |
| What to build and in what order | `IMPLEMENTATION_PLAN.md` |
| Unresolved decisions | `OPEN_QUESTIONS.md` |
| Risk context | `RISK_REGISTER.md` |
| Audit findings and gaps | `PROJECT_AUDIT.md` |
| Product/domain/entity detail | `docs/01-product/`, `docs/02-domain/` |
| Data model and rules | `docs/04-data/` |
| API contract | `docs/05-api/` |
| Frontend conventions | `docs/06-frontend/` |
| Backend conventions | `docs/07-backend/` |
| Testing strategy | `docs/09-testing/` |
| Local dev, deploy, ops | `docs/10-devops/`, `docs/12-operations/` |
| Security | `docs/11-security/` |

*End of AGENTS.md*