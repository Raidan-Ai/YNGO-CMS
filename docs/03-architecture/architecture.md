# docs/03-architecture/architecture.md

> **Status:** Index | **Owner:** Lead Architect | **Last Updated:** 2026-10-02
> **Purpose:** navigation entry point. **The authoritative architecture document is
> the repository root [`ARCHITECTURE.md`](../../ARCHITECTURE.md).**

This folder holds architecture detail that supports the root document. Do not
duplicate content here — link to it.

## Reading Order

1. [`ARCHITECTURE.md`](../../ARCHITECTURE.md) — system overview, principles,
   context, containers, components, boundaries, data, API, security, integration,
   deployment, observability, scalability, change control.
2. [`TECHNICAL_DECISIONS.md`](../../TECHNICAL_DECISIONS.md) — why each choice was made.
3. This folder — supporting detail.

## Documents in this Folder

| File | Contents | Status |
|------|----------|--------|
| `system-context.md` | C4 Level 1 context + external actors | Planned (Phase 0) |
| `components.md` | C4 Level 3 component detail per module | Planned (incremental) |
| `data-flow.md` | End-to-end flows (content publish, project report, case handling) | Planned (Phase 1) |
| `security.md` | Threat model, guard pipeline, isolation proofs | Planned (Phase 1) |
| `decisions/` | Individually filed ADR copies | Ongoing |

## Section Map (root ARCHITECTURE.md)

| Topic | Section |
|-------|---------|
| Overview | §1 |
| Principles | §2 |
| System context | §3 |
| Containers | §4 |
| Components / modules | §5 |
| Domain boundaries | §6 |
| Data architecture | §7 |
| API architecture | §8 |
| Security architecture | §9 |
| Integration architecture | §10 |
| Deployment architecture | §11 |
| Observability | §12 |
| Scalability & failure handling | §13 |
| Change control | §14 |

## Diagram Conventions

- Mermaid `flowchart` for context, containers, dependencies, and flows.
- Mermaid `erDiagram` for data relationships.
- Sequence diagrams (`sequenceDiagram`) for multi-step flows in `data-flow.md`.
- Diagrams live next to the text they explain; the root document keeps the
  high-level views only.

## Related

- Decisions: [`TECHNICAL_DECISIONS.md`](../../TECHNICAL_DECISIONS.md)
- Data: `docs/04-data/database.md`
- Security: `docs/11-security/security.md`
- Backend: `docs/07-backend/architecture.md`
- Frontend: `docs/06-frontend/architecture.md`