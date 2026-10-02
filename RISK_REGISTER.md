# RISK_REGISTER.md — YNGO-CMS / YemenNGO-CMS

> **Status:** Active | **Owner:** Lead Architect | **Last Updated:** 2026-10-02
> **Scoring:** Probability (1-5) × Impact (1-5) = Score; 20-25 Critical, 12-19 High, 6-11 Medium, 1-5 Low.
> **Review cadence:** end of every phase and before each release.

## Risk Matrix

| ID | Risk | Category | P | I | Score | Severity | Mitigation | Owner | Status |
|----|------|----------|:-:|:-:|:-----:|----------|------------|-------|--------|
| R-01 | Backend stack contradiction unresolved → two divergent codebases | Architecture | 4 | 5 | 20 | Critical | ADR-001 (NestJS primary + optional Python AI sidecar); update all docs to one stack | Lead Architect | OPEN |
| R-02 | Tenant isolation bypass (cross-tenant read/list/search/export/report/file/webhook/AI) | Security | 3 | 5 | 15 | Critical | Mandatory tenant resolution + scoped repository base + RLS defence-in-depth + negative tests per surface | Backend Lead | OPEN |
| R-03 | Beneficiary / Safeguarding / Case exposure (field-level privacy) | Security/Privacy | 3 | 5 | 15 | Critical | Field-level policy engine, search exclusion, elevated permission for export, separate audit, no public GIS | Security Lead | OPEN |
| R-04 | Secrets committed to source control | Security | 3 | 5 | 15 | Critical | `.env.example` only; `.gitignore`; pre-commit secret scan; no default credentials; rotation runbook | DevOps | OPEN |
| R-05 | Scope creep — attempting 52 modules at once → stubs | Delivery | 4 | 4 | 16 | High | Phase gates; V1 = Core+Tenancy+Identity+Access+CMS+Media+Programs/Projects | Product Owner | OPEN |
| R-06 | No automated tests → silent authz/tenant regressions | Quality | 4 | 4 | 16 | High | Test pyramid mandatory from Phase 0; negative tests per security boundary; CI blocks merges | QA Lead | OPEN |
| R-07 | Upload abuse (MIME spoof, path traversal, malware) | Security | 3 | 4 | 12 | High | MIME+magic-byte, allow-list, size limits, normalised keys, quarantine, AV adapter, signed URLs | Backend Lead | OPEN |
| R-08 | Sensitive location exposed via public GIS layer | Privacy | 2 | 5 | 10 | High | Layer policy + precision reduction + exclusion tests | GIS Lead | OPEN |
| R-09 | Documentation drift (docs diverge from code) | Process | 4 | 3 | 12 | High | Single source of truth in `docs/`; DoD requires doc update; OpenAPI generated | Doc Architect | OPEN |
| R-10 | Single-node deployment failure (no HA) | Operations | 3 | 4 | 12 | High | Backup/restore drill, WAL archiving, off-site copy, tested restore | DevOps | OPEN |
| R-11 | Migration failure / destructive schema change | Data | 3 | 4 | 12 | High | Append-only migrations, tested rollback, backup first, ADR for breaking changes | DB Architect | OPEN |
| R-12 | Reporting/search latency at scale | Performance | 3 | 3 | 9 | Medium | GIN/GiST indexes, materialised views, async reports, cursors, tenant-scoped cache | Backend Lead | OPEN |
| R-13 | N+1 queries in relational graphs | Performance | 3 | 3 | 9 | Medium | Eager-load discipline, query budget review, perf integration tests | Backend Lead | OPEN |
| R-14 | Vendor lock-in (storage/search/AI/email) | Architecture | 3 | 3 | 9 | Medium | Ports + adapters; swappable without domain changes | Lead Architect | OPEN |
| R-15 | AI output treated as authoritative / provider data leakage | AI/Privacy | 2 | 4 | 8 | Medium | Human-approval lifecycle, privacy routing, redaction, no silent overwrite | AI Lead | OPEN |
| R-16 | Workflow bypass via direct CRUD status edit | Integrity | 3 | 3 | 9 | Medium | Status changes only via guarded transitions; workflow tests; transition audit | Backend Lead | OPEN |
| R-17 | API contract drift (spec vs implementation) | Quality | 3 | 3 | 9 | Medium | OpenAPI generated in CI; contract tests; versioning rules | API Owner | OPEN |
| R-18 | Frontend permission display mistaken for security | Security | 2 | 4 | 8 | Medium | Backend is the only boundary; authz tests hit the API directly | Security Lead | OPEN |
| R-19 | Dependency vulnerabilities | Security | 3 | 3 | 9 | Medium | Dependabot/audit in CI; pinned versions; monthly review | DevOps | OPEN |
| R-20 | Arabic/RTL quality regressions | UX | 3 | 2 | 6 | Medium | RTL lint/tests in CI; RTL checklist per UI task | Frontend Lead | OPEN |
| R-21 | Offline sync conflict / data loss (future field app) | Data | 2 | 4 | 8 | Medium | Deferred to Phase 10; real sync engine before claiming offline | Field Lead | ACCEPTED (deferred) |
| R-22 | Duplicate/legacy spec files confuse agents | Process | 4 | 2 | 8 | Medium | Archive originals as frozen inputs; `docs/` authoritative; remove duplicate | Doc Architect | OPEN |
| R-23 | Missing environment contract blocks local run | Delivery | 3 | 3 | 9 | Medium | `.env.example` + local-development doc in Phase 0 | DevOps | OPEN |
| R-24 | Licensing/IP ambiguity for distribution | Legal | 2 | 3 | 6 | Medium | Q-13; choose licence before public release | Product Owner | OPEN |

## Top 5 Risks Requiring Immediate Action

1. **R-01** — resolve the stack contradiction (blocker for all work).
2. **R-02 / R-03** — tenant + sensitive-data isolation design before any module code.
3. **R-04** — secrets hygiene from the very first commit.
4. **R-05** — enforce phase gates to prevent stub sprawl.
5. **R-06** — tests are part of Definition of Done, not a later phase.

## Risk Acceptance Policy

- No **Critical** risk may be accepted silently; requires a documented ADR entry.
- **High** risks must have a named owner and a mitigation task in the active phase.
- **Medium / Low** risks are reviewed at phase boundaries.
- Risks are re-scored at each phase exit; closed risks are recorded in CHANGELOG.

## Traceability

| Risk | Linked ADR / Task |
|------|-------------------|
| R-01 | ADR-001, TASK-001 |
| R-02 | ADR-004, TASK-010, TASK-011 |
| R-03 | TASK-060 (People module), docs/11-security/security.md |
| R-04 | TASK-002, TASK-004 |
| R-05 | IMPLEMENTATION_PLAN.md phase gates |
| R-06 | TASK-006, TASK-007 |
| R-07 | TASK-030 (Media), TASK-031 (Documents) |
| R-09 | AGENTS.md documentation rules |
| R-11 | TASK-008 (migration baseline) |

*End of RISK_REGISTER.md*