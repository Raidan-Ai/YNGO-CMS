# docs/00-overview/source-documents.md

> **Status:** Current | **Owner:** Documentation Architect | **Last Updated:** 2026-10-02
> **Purpose:** index the frozen specification inputs and define precedence.

## 1. Frozen Inputs (repo root — read-only)

| File | Size | Role | Standing |
|------|------|------|----------|
| `YNGO-CMS_Blueprint.md` | ~61 KB | Product vision, principles, module intent | **Authority** for product principles |
| `YNGO-CMS_Engineering_Documentation_Package_v2.md` | ~171 KB | Most complete product/domain/entity specification | **Authority** for domain/entity intent |
| `YNGO-CMS_TECHNICAL_ARCHITECTURE_INSTRUCTIONS.md` | ~27 KB | Mandatory stack, layering, and engineering constraints | **Authority** for stack/layering |
| `YNGO-CMS_Enterprise_Master_Build_Prompt_v2.md` | ~115 KB | Extra UX/API/ops detail | Supporting detail |
| `YNGO-CMS_Enterprise_Master_Build_Prompt_v2 (1).md` | ~115 KB | Byte-identical duplicate | **REMOVE** (duplicate, R-22) |
| `YNGO-CMS  YemenNGO-CMS  Master Engineering Build Prompt.md` | ~49 KB | Earlier build prompt | Superseded; retained for provenance |
| `YNGO-CMS_OpenCode_Final_Build_Instructions.md` | ~24 KB | Local-first build + verification process rules | Process reference |

These files are **inputs, not canonical**. They are not edited. Changes to
requirements are recorded in `docs/` with a citation back to the source.

## 2. Precedence Rules

Conflicts between frozen inputs are resolved in this order:

```text
1. TECHNICAL_ARCHITECTURE_INSTRUCTIONS.md   (stack, layering, engineering law)
2. TECHNICAL_DECISIONS.md (ADRs)            (recorded, dated resolutions)
3. Engineering_Documentation_Package_v2.md  (domain/entity semantics)
4. Blueprint.md                             (product principles)
5. Enterprise_Master_Build_Prompt_v2.md     (supporting detail)
6. Superseded/duplicate build prompts       (no authority)
```

A conflict that is not yet covered by an ADR is a **contradiction finding** and
must be recorded in `PROJECT_AUDIT.md` §13 and raised in `OPEN_QUESTIONS.md`.

## 3. Known Contradictions (resolved / pending)

| ID | Contradiction | Resolution |
|----|---------------|-----------|
| C-01 | FastAPI/Python vs NestJS/Node/Fastify backend | **Resolved** by ADR-001 (NestJS primary; Python only as optional AI sidecar) |
| C-02 | Duplicate build-prompt file | **Resolved** by removal instruction (R-22); documented only |
| C-03 | Module count quoted as 40 / 52 / 53 | **Resolved** in `../07-backend/modules.md` as a single canonical inventory |
| C-04 | "No demo data" vs "sample screens" in UX specs | **Resolved** by ADR-013 (fixtures in `tests/**` only) |

## 4. Citation Convention

When a `docs/` file derives a fact from a frozen input, it cites it:

```text
Source: YNGO-CMS_Engineering_Documentation_Package_v2.md §<section>
```

When a fact is an engineering recommendation rather than a spec statement, it is
labelled `RECOMMENDATION` or `DECISION REQUIRED` — never presented as a requirement.

## Related

- Audit + contradictions: `../../PROJECT_AUDIT.md`
- Decisions: `../../TECHNICAL_DECISIONS.md` · `../13-decisions/`
- Open questions: `../../OPEN_QUESTIONS.md`

*End of docs/00-overview/source-documents.md*