# ADR-0001 — Backend Framework

> **Status:** ACCEPTED | **Owner:** Lead Architect | **Date:** 2026-10-02
> **Canonical rationale:** `../../TECHNICAL_DECISIONS.md` (ADR-001)
> **Decides:** `../../OPEN_QUESTIONS.md` Q-01 | **Blocks:** every backend task

## Context

Frozen inputs contradict each other: `Engineering_Documentation_Package_v2`
documents a FastAPI/Python backend, while
`TECHNICAL_ARCHITECTURE_INSTRUCTIONS` mandates NestJS + Fastify on Node/TypeScript.
The platform must enforce 50+ module boundaries, tenant isolation, RBAC/ABAC, and
audit on every request — this is the deciding pressure, not raw HTTP performance.

## Options

| Option | Pros | Cons |
|--------|------|------|
| A. NestJS + Fastify (TS) | First-class modules/DI/guards; one language with the frontend; generated OpenAPI; mature guard/interceptor model | Node CPU-bound work must move to workers; DI boilerplate |
| B. FastAPI (Python) | Excellent validation/OpenAPI; strong data/AI ecosystem | No equivalent opinionated module boundary system; duplicated contracts across two languages |
| C. Hybrid | Right tool per job | Two runtimes and two contract models to maintain |

## Decision

**NestJS + Fastify (TypeScript) is the canonical runtime for `apps/api` and
`apps/worker`. Python is permitted only as an optional, bounded AI/data sidecar
(ADR-0014) and never owns canonical data.**

## Mandatory Rules

1. No Python service inside `apps/api`; no Python in the transactional path.
2. Every module is a Nest module declaring `imports / controllers / providers /
   exports`; cross-module access only through exported services or events.
3. Guards (tenant, auth, permission) are global, deny-by-default; controllers stay thin.
4. `strict` TypeScript everywhere; no `any` in domain code.

## Consequences

- Positive: one language for API + web contracts; guards map 1:1 to security
  requirements; OpenAPI generated from decorators.
- Negative: two skill sets when the AI sidecar is enabled; CPU-heavy jobs must be
  queued to `worker`.
- Follow-up: fold every conflicting FastAPI instruction into this decision and mark
  the frozen input as superseded on that point.

## Migration Impact

None — greenfield, no code exists.

## Related

- Stack table: `../../README.md` §3 · Layering: `../03-architecture/architecture.md`
- AI boundary: `ADR-0014-ai-sidecar.md` · Audit: `../../PROJECT_AUDIT.md` C-01

*End of ADR-0001*
