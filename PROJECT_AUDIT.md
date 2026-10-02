# PROJECT AUDIT — YNGO-CMS / YemenNGO-CMS

> **Date:** 2026-10-02
> **Auditor:** Principal Architect (Automated Discovery)
> **Project Root:** `I:\YNGO`
> **Status:** GREENFIELD — No implementation exists
> **Sources:** 6 specification markdown files (~550KB total)

---

## 1. Executive Summary

YNGO-CMS is a modular, multi-tenant, Arabic-first, API-first digital operating
platform for civil-society / NGOs. It combines a headless-capable CMS with NGO
operations (programs, projects, grants, MEAL, beneficiaries, cases, etc.) plus
platform services (auth/RBAC/ABAC, tenancy, files, search, notifications,
audit, workflows, GIS, reporting, AI).

**Current state: Empty workspace.** No source code, no package manifests, no
Docker, no migrations, no tests, no CI/CD, no .env files. Only specification
markdown files exist. No `.git` directory was found.

**Top problems:**
1. **Critical stack contradiction** — FastAPI/Python vs NestJS/Node/Fastify.
2. No codebase to audit; all analysis is doc-vs-doc.
3. Scope is 52 modules — impossible in one release.
4. Missing operational prerequisites (env contract, migration tool, tenancy
   mechanism).

**Recommendation:** Record ADR-001 (stack decision) and approve
IMPLEMENTATION_PLAN before writing code. Initialize as modular monolith.

---

## 2. Current State

| Dimension | Finding |
|-----------|---------|
| Source code | None |
| Documentation | 5 large spec docs (49k-171k bytes each) + 1 duplicate |
| Configuration | None (no package.json, no yaml, no env) |
| Database | None (no migrations, no ORM) |
| Infrastructure | None (no Dockerfile, no compose) |
| Tests | None |
| CI/CD | None |
| Git | Not initialized |

Workspace is **GREENFIELD**.

---

## 3. Existing Architecture

No existing architecture. Documents describe a *target*:

- Frontend: Next.js + React + TypeScript + Tailwind + shadcn/ui
- Backend (CONTRADICTED): FastAPI/Python vs NestJS/Node/Fastify
- DB (unanimous): PostgreSQL + PostGIS (canonical)
- Cache/Queue: Redis
- Storage: S3-compatible (MinIO / Blob)
- API: REST /api/v1 + OpenAPI
- Deployment: Docker Compose on Ubuntu, reverse proxy, TLS
- Pattern: Modular monolith first

---

## 4. Technology Stack

### Contradiction

| Document | Backend |
|----------|---------|
| Eng. Package v2 | FastAPI / Python |
| Tech Arch Instructions | **NestJS / Fastify / Node / TS (mandatory)** |
| Blueprint | Not pinned |
| OpenCode Instructions | NestJS / Fastify listed |

Unanimous elsewhere: PG+PostGIS, Redis, S3, REST /api/v1, modular monolith.

**Proposed resolution (ADR-001):** NestJS/Node as primary API + optional
Python sidecar for AI/OCR. See TECHNICAL_DECISIONS.md.

---

## 5. Features — Declared Scope (all PLANNED, none implemented)

52 modules: Identity, Organization, Governance, CMS, Media, Documents,
Knowledge, Programs, Projects, Grants, Fundraising, Donors, Partners, CRM,
Beneficiaries, Cases, Safeguarding, Complaints, MEAL, Forms, Volunteers, HR,
Procurement, Finance, Assets, Fleet, Travel, Events, Tasks, Communications,
Advocacy, GIS, Reporting, Analytics, Workflow, Automation, Notifications,
Search, API Platform, Integrations, Portals, Microsites, Theme, Plugins, AI,
Data Governance, Data Quality, Import/Export, Observability, Backup, Developer
Platform, System Admin.

---

## 6. Implemented / Missing

- Implemented: **None.**
- Missing: **All 52 modules** (no code).

Documentation gaps within specs: no ERD file, no openapi.yaml, no permission
matrix file, no wireframes, no ADRs, no .env.example, no SLOs, no threat model.

---

## 7. Documentation Gaps

| Gap | Severity |
|-----|----------|
| No .env.example inventory | High |
| No ERD / schema file | High |
| No OpenAPI file | High |
| No permission matrix artifact | High |
| No ADRs | Medium |
| No glossary | Medium |
| Duplicate specs (6 files, ~80% overlap) | Medium |
| No runbook | High |
| No testing plan artifact | Medium |

---

## 8. Architecture Problems

| # | Problem | Severity |
|---|---------|----------|
| A-01 | Backend stack contradiction | Critical |
| A-02 | 52 modules with no phasing gate | Critical |
| A-03 | Tenancy isolation mechanism undecided | High |
| A-04 | Migration tool undecided | High |
| A-05 | File storage abstraction missing | Medium |
| A-06 | Search abstraction missing | Medium |
| A-07 | API versioning beyond /api/v1 prefix | Low |

---

## 9. Security Findings (spec review, no code to scan)

| # | Finding | Severity |
|---|---------|----------|
| S-01 | Tenant isolation — untested, foundational | Critical |
| S-02 | Beneficiary/safeguarding field-level restrictions | Critical |
| S-03 | File handling (MIME spoof, path traversal) | High |
| S-04 | No secrets management design | High |
| S-05 | No rate limiting design | High |
| S-06 | No audit log schema | High |
| S-07 | AI must not silently overwrite canonical records | High |

---

## 10. Performance (anticipatory)

CMS+GIS+Reporting queries heaviest — need pagination, GIST indexes, async
exports, async media renditions, permission-aware caching.

---

## 11. Technical Debt

Zero code debt. Documentation debt: 6 overlapping spec files, no docs/
hierarchy (this audit creates it), no ADRs, no CHANGELOG.

---

## 12. Contradictions

| # | Between | What | Severity |
|---|---------|------|----------|
| C-01 | Eng Package vs Tech Arch | FastAPI vs NestJS | Critical |
| C-02 | Docs vs missing | "API-first" but no OpenAPI file | High |
| C-03 | Docs vs docs | Redis role overload (cache/queue/coord) | Medium |
| C-04 | Blueprint vs Eng Package | repo layout phrasing differs | Low |

---

## 13. Recommended Changes

1. Record ADR-001 (stack) and approve.
2. `git init`, README, LICENSE, .gitignore, .env.example, docker-compose.yml.
3. Deduplicate: promote docs/ hierarchy to single source of truth.
4. Phase scope: V1 = Core + CMS + Programs/Projects + Media + Auth + Tenancy.
5. Gate: tenant isolation + audit tests must pass before any module merges.

---

## 14. Migration Strategy

Greenfield incremental (no legacy data):

```
Foundation -> CMS/Media -> Programs/Projects -> Grants/Partners -> MEAL/Forms -> Operations -> Sensitive domains
```

Each phase: migration -> API -> authorization -> tests -> UI.

---

## 15. Inventory Summary

```
Project
  Application    NOT STARTED (modular monolith planned)
  Frontend       PLANNED (Next.js/React/TS)
  Backend        PLANNED — CONTRADICTED
  Database       PLANNED (PG+PostGIS, no schema file)
  APIs           PLANNED (/api/v1, no OpenAPI)
  Services       PLANNED (52 modules)
  Infrastructure PLANNED (Docker Compose)
  Testing        PLANNED (none yet)
  Documentation  PARTIAL (specs exist, not reorganized)
  Deployment     PLANNED (Ubuntu/Docker)
```

## 16. Documentation ↔ Implementation Gap Analysis

Every declared capability is documentation-only. This table is the canonical
gap register (Gap = requirement listed in specs but with no code, no test, and
no acceptance criterion executable today).

| Feature | Documentation | Implementation | Tests | Prod Ready | Gap | Action |
|---------|---------------|----------------|-------|-----------|-----|--------|
| Tenancy isolation | Detailed (must isolate across all surfaces incl. search/export/jobs/AI) | None | None | No | Mechanism (RLS vs app-layer) undecided | ADR-004 + tenant guard + negative tests |
| Authentication | email/password, MFA, OIDC, SAML, passkeys | None | None | No | No phased plan | Phase 2: email/password + sessions; MFA in Phase 7 |
| Authorization RBAC+ABAC | Naming `resource:action`, attribute rules | None | None | No | No permission catalog artifact | Generate permission catalog + policy engine |
| Audit logging | Fields defined (actor, tenant, old/new, request_id) | None | None | No | No schema, no immutability rule | Append-only audit table + helper |
| CMS content lifecycle | 8-9 states, editorial, versioning, scheduling | None | None | No | State set contradictory (C-03) | Freeze states in docs/02-domain |
| Media / DAM | Upload pipeline, signed URLs, renditions | None | None | No | No storage abstraction | Storage adapter interface + MinIO impl |
| Documents / DMS | Versioning, retention, classification | None | None | No | No classification enforcement | Data-governance labels per entity |
| Programs/Projects/Activities/Indicators | Full entity lists | None | None | No | No schema | Phase 3 vertical slice |
| Grants/Donors/Partners | Workflows, due diligence | None | None | No | No schema | Phase 4 |
| MEAL / Forms | Framework, indicators, form lifecycle | None | None | No | No schema | Phase 5 |
| Beneficiaries | Strict perms, field-level privacy, pseudonymous IDs | None | None | No | Enforcement pattern missing | Phase 6 with field-level policy |
| Cases / Referrals | Lifecycle defined | None | None | No | No schema | Phase 6 |
| Safeguarding | Restricted, separate audit, no global search | None | None | No | Exclusion rule unimplemented | Phase 6 + search exclusion test |
| Complaints/Feedback | Public intake, anonymous option | None | None | No | No schema | Phase 6 |
| Workflow engine | Definitions, steps, SLA, bypass prevention | None | None | No | No engine design | Phase 7 |
| Automation engine | Trigger->Condition->Action | None | None | No | No engine design | Phase 7 |
| Notifications | 5 channels, templates, preferences | None | None | No | No provider adapters | Phase 7 |
| Search | PG FTS abstraction -> OpenSearch later | None | None | No | Interface contract missing | Phase 3 (abstraction) |
| Reporting/Dashboards | Templates, async generation, exports | None | None | No | No query/policy layer | Phase 8 |
| GIS | PostGIS, MapLibre, privacy layers | None | None | No | Map source + layer policy undecided | Phase 8 |
| API platform + keys + webhooks | Endpoints, scopes, retry/backoff | None | None | No | No OpenAPI artifact | Phase 1 scaffold, Phase 3 fill |
| Integrations | 15+ providers behind adapters | None | None | No | No adapter contract | Phase 9 |
| Portals | 9 portals on shared API | None | None | No | No route-guard design | Phase 9 |
| Offline Field | Sync engine, conflict resolution | None | None | No | Correctly deferred | Phase 10 |
| AI platform | Providers, agents, governance, human review | None | None | No | Correctly deferred | Phase 10 (bounded, human-approved) |
| Observability | Logs, request IDs, health/ready/live | None | None | No | No SLOs | Phase 1 (health) + Phase 8 (metrics) |
| Backup/DR | Daily/weekly/offsite/restore-verify | None | None | No | No runbook script | Phase 11 |

## 17. Legacy Components

**None.** No application code, configuration, database, migration, test, or
deployment artifact exists in `I:\YNGO`. Nothing to Keep, Refactor, Replace,
Migrate, Remove, or Defer at the code level.

| Item | Classification | Reason |
|------|----------------|--------|
| `YNGO-CMS Blueprint.md` | KEEP (frozen source) | Product vision baseline; authority for principles |
| `YNGO-CMS_Engineering_Documentation_Package_v2.md` | KEEP (frozen source) | Most complete product/domain spec |
| `YNGO-CMS_TECHNICAL_ARCHITECTURE_INSTRUCTIONS.md` | KEEP (frozen source) | Mandatory stack/layering constraints |
| `YNGO-CMS_Enterprise_Master_Build_Prompt_v2.md` | KEEP (frozen source) | Extra UX/API detail; overlaps package |
| `YNGO-CMS_Enterprise_Master_Build_Prompt_v2 (1).md` | REMOVE (duplicate) | Byte-identical duplicate |
| `YNGO-CMS Master Engineering Build Prompt.md` | KEEP (superseded) | Retained for provenance |
| `YNGO-CMS_OpenCode_Final_Build_Instructions.md` | KEEP (process) | Local-first build/verification rules |

Originals are **frozen inputs**; the `docs/` tree becomes the living single
source of truth citing them.

---

## 18. Risks (summary — details in RISK_REGISTER.md)

| Risk | Prob | Impact | Severity |
|------|------|--------|----------|
| Stack contradiction unresolved → divergent codebases | High | High | Critical |
| Tenant isolation bypass (cross-tenant read/export) | Medium | Critical | Critical |
| Beneficiary / Safeguarding data exposure | Medium | Critical | Critical |
| Secrets committed to source control | Medium | Critical | Critical |
| Scope creep (52 modules attempted at once) | High | High | High |
| Sensitive location exposed via public GIS layer | Low | Critical | High |
| No automated tests → silent authz regressions | High | High | High |
| Upload abuse (MIME spoof / malware) | Medium | High | High |
| Documentation drift from code | High | Medium | High |
| Reporting/search latency at scale | Medium | Medium | Medium |
| Vendor lock-in (storage, search, AI) | Medium | Medium | Medium |
| Single-node deployment failure (no HA) | Medium | High | High |

## 19. Open Questions (details in OPEN_QUESTIONS.md)

| ID | Question | Class |
|----|----------|-------|
| Q-01 | Canonical backend: NestJS (recommended) or FastAPI? | BLOCKING |
| Q-02 | ORM/migration tool: Prisma, TypeORM, or Drizzle? | BLOCKING |
| Q-03 | Tenant resolution: subdomain, header, or path? | BLOCKING |
| Q-04 | Tenancy data model: shared-schema+tenant_id, RLS, or schema-per-tenant? | BLOCKING |
| Q-05 | Bootstrap: CLI or first-run wizard for first org+admin? | BLOCKING |
| Q-06 | Reference object storage: MinIO self-hosted? | BLOCKING |
| Q-07 | GIS base map source: OSM public or self-hosted tiles? | IMPORTANT |
| Q-08 | Package manager: pnpm (recommended) or npm? | IMPORTANT |
| Q-09 | Queue: BullMQ (Redis) vs pg-boss (Postgres-only)? | IMPORTANT |
| Q-10 | i18n framework: next-intl vs i18next? | IMPORTANT |
| Q-11 | Search abstraction interface shape? | IMPORTANT |
| Q-12 | AI provider(s) for Phase 10 + data-privacy routing? | OPTIONAL |
| Q-13 | Licensing model for the platform itself? | OPTIONAL |
| Q-14 | SMS/WhatsApp provider preferences for Yemen context? | OPTIONAL |

---

## 20. Readiness Summary

| Question | Answer |
|----------|--------|
| Do we understand the project? | YES |
| Do we understand the domain? | YES (entity-name normalisation pending) |
| Is target architecture defined? | YES (see ARCHITECTURE.md) |
| Can an agent execute without guessing? | YES — after Q-01..Q-06 are answered |
| Are requirements contradictory? | YES — C-01..C-04 |
| Are there critical questions? | YES — 6 BLOCKING |

**Audit verdict:** Well-specified but not engineered. Proceed to Phase 0
Foundation only after recording ADR-001..ADR-005 and answering the BLOCKING
questions.

*End of PROJECT_AUDIT.md*
