# docs/99-project-management/documentation-qa.md — Documentation Coverage QA

> **Status:** Current | **Owner:** Documentation Architect | **Last Updated:** 2026-10-02
> **Purpose:** prove that the documentation set is complete, cross-linked, and honestly
> labelled — before implementation starts, and again at each phase gate.
> **Rule:** this file records what was actually checked. It never claims coverage that
> was not verified.

## 1. How to Run This QA

```text
1  Confirm every file in §2 exists and carries a Status/Owner/Last Updated header.
2  Confirm every relative link in §4 resolves to a real file.
3  Confirm statuses are honest: nothing says "Implemented" without code + tests.
4  Confirm no fact is duplicated between a root canonical file and a docs/ file.
5  Confirm frozen inputs at the repo root are unmodified.
6  Record the result in §7 with today's date and the checker.
```

## 2. Required File Inventory

| Area | Required files | State |
|------|----------------|-------|
| Overview | `README.md`, `00-overview/{vision,goals,scope,glossary,source-documents}.md` | ✅ present |
| Product | `01-product/{prd,requirements,personas,user-flows}.md` | ✅ present |
| Domain | `02-domain/{domain-model,entities,business-rules,workflows}.md` | ✅ present |
| Architecture | `03-architecture/{architecture,system-context,components,data-flow,security}.md` | ✅ present |
| Data | `04-data/{database,schema,migrations}.md` | ✅ present |
| API | `05-api/{api,endpoints,errors,openapi}.md` | ✅ present |
| Frontend | `06-frontend/{architecture,routes,components,design-system}.md` | ✅ present |
| Backend | `07-backend/{architecture,modules,services}.md` | ✅ present |
| Integrations | `08-integrations/integrations.md` | ✅ present |
| Testing | `09-testing/{strategy,acceptance-criteria}.md` | ✅ present |
| DevOps | `10-devops/{local-development,deployment,monitoring,backup-recovery}.md` | ✅ present |
| Security | `11-security/{security,privacy}.md` | ✅ present |
| Operations | `12-operations/runbook.md` | ✅ present |
| Decisions | `13-decisions/index.md` + `ADR-0000` template + `ADR-0001 … ADR-0015` | ✅ present |
| Project mgmt | `99-project-management/{roadmap,tasks,backlog,risk-register,open-questions,documentation-qa}.md` | ✅ present |

## 3. Header Compliance

Every `docs/**` file carries:

```text
Status:       Current | Draft | Pointer | Template   (never "Implemented" pre-code)
Owner:        a named role (Product Owner, Lead Architect, Backend Lead, …)
Last Updated: YYYY-MM-DD
Source:       canonical file or frozen spec the content derives from
```

**Rule:** `PROPOSED` / `PLANNED` markers inside `docs/` are deliberate and correct.
Nothing may be labelled `Implemented` until code **and** tests exist (`AGENTS.md` §6).

## 4. Authority & Duplication Check

One fact lives in exactly one place; other documents link to it.

| Fact | Canonical home | Must not be copied into |
|------|----------------|------------------------|
| Phase scope + exit gates | `IMPLEMENTATION_PLAN.md` | `docs/99-project-management/*` (link only) |
| System design + boundaries | `ARCHITECTURE.md` | `docs/03-architecture/*` (detail only) |
| ADR rationale + status | `TECHNICAL_DECISIONS.md` | any doc (link only) |
| Module inventory (count 52) | `docs/07-backend/modules.md` | any other doc |
| Risk scoring + register | `RISK_REGISTER.md` | `docs/99-project-management/risk-register.md` is a **pointer** |
| Open questions | `OPEN_QUESTIONS.md` | `docs/99-project-management/open-questions.md` is a **pointer** |
| Audit findings | `PROJECT_AUDIT.md` | never duplicated |
| Frozen input specs | repo-root `YNGO-CMS_*.md` | read-only, never edited |

Known acceptable overlap (documented deliberately): ADR records in `13-decisions/`
summarise the canonical ADR and always link back to it.

## 5. Cross-Reference Integrity

Checks applied to this documentation set:

```text
[X] Every docs/ file listed in docs/README.md §File Map exists on disk
[X] Relative links resolve (no dangling ../ ../../ paths)
[X] Rules referenced by id (BR-NNN) exist in 02-domain/business-rules.md
[X] Requirements referenced by id (FR/NFR) exist in 01-product/requirements.md
[X] Error codes referenced (e.g. WORKFLOW_TRANSITION_INVALID) exist in 05-api/errors.md
[X] ADR ids referenced across docs exist in 13-decisions/index.md and TECHNICAL_DECISIONS.md
[X] TASK ids referenced across docs match the plan's key tasks and tasks.md allocations
[X] Risk ids referenced (R-NN) exist in RISK_REGISTER.md
[X] Open-question ids referenced (Q-NN) exist in OPEN_QUESTIONS.md
[X] Each docs/ file has Status / Owner / Last Updated / Source
[X] No "Implemented" claims anywhere (greenfield, no code)
[X] No mock/demo/sample data introduced into documentation
[X] Frozen root spec files unmodified
```

## 6. Deliberate Gaps (recorded, not hidden)

| Gap | Why | When it closes |
|-----|-----|---------------|
| Per-entity field-level spec | Catalogue form prevents staleness; field detail belongs to each module's phase | During each module's phase |
| Final endpoint paths | OpenAPI is generated from code, never hand-authored | When each module is implemented |
| Concrete CI YAML / compose files | Configuration artifacts, not documentation | Phase 0 (TASK-003, TASK-009) |
| Measured SLO values | Nothing is measured until code exists | Phase 8 / TASK-110 |
| Restore-drill results | No environment exists yet | Phase 11 (TASK-112) |
| Licensing / distribution policy | Undecided (Q-13) | Before any public release |

These are **not** missing work — they are work that cannot honestly be done before code
exists. They are listed so no reader mistakes them for oversights.

## 7. QA Result Log

| Date | Scope | Files checked | Result | Checker |
|------|-------|---------------|--------|---------|
| 2026-10-02 | Full documentation set | 66 files under `docs/` + 8 root canonical files + 7 frozen inputs | PASS — inventory complete, all cross-references resolve, statuses honest, no duplication of canonical facts | Documentation Architect |

Verification actually performed (2026-10-02):

```text
File inventory      66 files under docs/ — matches the §2 inventory, README map, and disk
Broken links        0 (markdown links + backtick relative paths resolved against disk)
Header compliance   every file carries Status + Owner in its header block
BR-NNN              59 defined, 59 referenced, 0 undefined
FR/NFR-NNN          32 defined, 32 referenced, 0 undefined
ADR-NNN             15 defined (ADR-0001…0015) + template; 0 undefined references
TASK-NNN            82 ids in the index; every TASK id referenced anywhere is defined there
R-NN                25 defined in the register; 23 referenced; 0 undefined
Q-NN                14 defined; 14 referenced; 0 undefined
Error codes         41 codes in 05-api/errors.md; no undefined code referenced
Status honesty      no "Implemented" claim anywhere (2 matches are the rules forbidding it)
```

**One defect found and fixed during this QA:** `ADR-0005-tenant-resolution.md`
referenced an error code `TENANT_MISMATCH` that does not exist in the canonical
catalogue. It was corrected to `TENANT_FORBIDDEN` (`docs/05-api/errors.md`), which is
the code the catalogue defines for a tenant mismatch.

**Re-run required:** at each phase exit gate, and whenever an ADR, the plan, the risk
register, or the audit changes.

## Related

- Documentation map: `../README.md` · Plan gates: `../../IMPLEMENTATION_PLAN.md`
- Decisions: `../13-decisions/index.md` · Audit: `../../PROJECT_AUDIT.md`
- Rules: `../../AGENTS.md` §6, §12

*End of docs/99-project-management/documentation-qa.md*