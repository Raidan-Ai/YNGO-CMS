# docs/README.md — Documentation Map (Single Source of Truth)

> **Status:** Current | **Owner:** Documentation Architect | **Last Updated:** 2026-10-02
> **Source:** AGENTS.md, PROJECT_AUDIT.md | **Purpose:** one map for every doc file
> **Rule:** if a fact lives in a root canonical file, link to it — never copy it.

---

## 1. Authority Model

| Domain | Canonical file (repo root) | `docs/` role |
|--------|---------------------------|--------------|
| System design | `ARCHITECTURE.md` | detail only under `03-architecture/` |
| Delivery plan | `IMPLEMENTATION_PLAN.md` | detail under `99-project-management/` |
| Decisions | `TECHNICAL_DECISIONS.md` | per-ADR records in `13-decisions/` |
| Risks | `RISK_REGISTER.md` | pointer file only |
| Open questions | `OPEN_QUESTIONS.md` | pointer file only |
| Execution rules | `AGENTS.md` | referenced by every task |
| Audit findings | `PROJECT_AUDIT.md` | never duplicated |

Frozen input specifications at the repo root (`YNGO-CMS_*.md`) are **historical
inputs**, not canonical — see `00-overview/source-documents.md`.

## 2. File Map

| Path | Content | Status |
|------|---------|--------|
| `00-overview/vision.md` | Product vision, mission, principles | Current |
| `00-overview/goals.md` | Goals, non-goals, success metrics | Current |
| `00-overview/scope.md` | In/out of scope per release (V1→V3) | Current |
| `00-overview/glossary.md` | Canonical vocabulary + naming conventions | Current |
| `00-overview/source-documents.md` | Frozen input index + precedence rules | Current |
| `01-product/prd.md` | Product requirements, audiences, module intent | Current |
| `01-product/requirements.md` | Numbered FR/NFR with traceability | Current |
| `01-product/personas.md` | Personas, permissions, constraints | Current |
| `01-product/user-flows.md` | End-to-end flows per persona | Current |
| `02-domain/domain-model.md` | Bounded contexts + context map | Current |
| `02-domain/entities.md` | Entity catalogue (purpose, relations, owner) | Current |
| `02-domain/business-rules.md` | Rule catalogue (BR-NNN) | Current |
| `02-domain/workflows.md` | State machines and approval paths | Current |
| `03-architecture/architecture.md` | Layering + module boundary rules | Current |
| `03-architecture/system-context.md` | Actors, systems, trust boundaries | Current |
| `03-architecture/components.md` | Container/component inventory | Current |
| `03-architecture/data-flow.md` | Request, job, file, search, export flows | Current |
| `03-architecture/security.md` | Enforcement points in the stack | Current |
| `03-architecture/decisions/` | ADR index + ADR-0001…0015 records | Current |
| `04-data/database.md` | Storage strategy, PG+PostGIS, conventions | Current |
| `04-data/schema.md` | Core table catalogue + tenancy rules | Current |
| `04-data/migrations.md` | Migration policy, naming, rollback | Current |
| `05-api/api.md` | REST conventions, versioning, pagination | Current |
| `05-api/endpoints.md` | Endpoint catalogue by module (planned) | Current |
| `05-api/errors.md` | Error envelope + code catalogue | Current |
| `05-api/openapi.md` | Generation policy (no hand-written spec) | Current |
| `06-frontend/architecture.md` | App Router, data layer, state rules | Current |
| `06-frontend/routes.md` | Route catalogue + access requirements | Current |
| `06-frontend/components.md` | Component taxonomy | Current |
| `06-frontend/design-system.md` | Tokens, RTL, empty/loading/error states | Current |
| `07-backend/architecture.md` | Module anatomy, DI, guards, interceptors | Current |
| `07-backend/modules.md` | Module inventory + phase + V1 flag | Current |
| `07-backend/services.md` | Cross-cutting services and ports | Current |
| `08-integrations/integrations.md` | Adapters, credentials, failure isolation | Current |
| `09-testing/strategy.md` | Test pyramid, tooling, CI gates | Current |
| `09-testing/acceptance-criteria.md` | Acceptance criteria per phase | Current |
| `10-devops/local-development.md` | Workstation → running stack | Current |
| `10-devops/deployment.md` | Environments, release, rollback | Current |
| `10-devops/monitoring.md` | Signals, SLOs, alert thresholds | Current |
| `10-devops/backup-recovery.md` | Backup policy, RTO/RPO, restore drill | Current |
| `11-security/security.md` | Threat model, controls, RBAC/ABAC, audit | Current |
| `11-security/privacy.md` | Sensitive-data classification + handling | Current |
| `12-operations/runbook.md` | Operational procedures, incident playbooks | Current |
| `13-decisions/*` | ADR-0001 … ADR-0015 records | Current |
| `99-project-management/roadmap.md` | Release-level sequencing | Current |
| `99-project-management/tasks.md` | Task index TASK-001 … TASK-115 | Current |
| `99-project-management/backlog.md` | Deferred items and candidates | Current |
| `99-project-management/risk-register.md` | Pointer to root risk register | Current |
| `99-project-management/open-questions.md` | Pointer to root open questions | Current |
| `99-project-management/documentation-qa.md` | Coverage QA checklist + result | Current |

> **`PROPOSED` / `PLANNED` markers inside `docs/` are deliberate.** Nothing may be
> described as `Implemented` until code and tests exist (`AGENTS.md` §6).

## 3. Reading Order for an AI Agent

```text
AGENTS.md                                  (rules — always first)
  ↓
IMPLEMENTATION_PLAN.md                     (phase + TASK id)
  ↓
docs/99-project-management/tasks.md        (task definition)
  ↓
ARCHITECTURE.md + docs/03-architecture/*   (boundaries)
  ↓
docs/02-domain/* + docs/04-data/schema.md  (model)
  ↓
docs/05-api/* + docs/07-backend/modules.md (contract + placement)
  ↓
docs/09-testing/*                          (how it is proven)
  ↓
docs/10-devops/*                           (how it runs)
```

## 4. Maintenance Rules

1. **Update, don't duplicate** — change the canonical file, then fix links.
2. **Honest statuses** — `Implemented` requires code + tests.
3. **No invented requirements** — use `UNKNOWN` / `RECOMMENDATION` /
   `DECISION REQUIRED` (`AGENTS.md` §6).
4. **Every task updates docs** when behaviour, contracts, schema, or env vars change.
5. **Coverage is verified** by `99-project-management/documentation-qa.md`.

## Related

- Execution contract: `AGENTS.md`
- Audit: `PROJECT_AUDIT.md` · Architecture: `ARCHITECTURE.md`
- Plan: `IMPLEMENTATION_PLAN.md` · Decisions: `TECHNICAL_DECISIONS.md`

*End of docs/README.md*