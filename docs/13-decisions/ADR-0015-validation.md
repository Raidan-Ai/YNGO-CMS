# ADR-0015 — Validation Strategy

> **Status:** PROPOSED | **Owner:** Lead Architect | **Date:** 2026-10-02
> **Canonical rationale:** `../../TECHNICAL_DECISIONS.md` (ADR-015)
> **Blocks:** DTOs and forms | **Related:** ADR-006 (API style)

## Context

The frozen specifications require schema-aligned validation on both ends: Zod on the
frontend and a server-side validator on the API, with the backend remaining
authoritative. Duplicating trust between two validators risks the frontend being
mistaken for a security control (R-18), while a single source avoids drift but must
still serve both form ergonomics and server enforcement.

## Options

| Option | Pros | Cons |
|--------|------|------|
| A. `class-validator` DTOs only | One authoritative validator; native NestJS | No rich client-side form feedback |
| B. Zod everywhere (shared) | One schema for UI + API | Weak fit with NestJS DTO pipeline; couples UI and API |
| C. `class-validator` authoritative + aligned Zod on UI | Server-authoritative; great UX feedback | Two schema definitions to keep aligned |

## Decision

**The API owns validation with `class-validator` DTOs (the source of truth for
accepted input). The frontend uses Zod schemas aligned to those contracts for
immediate UX feedback and form ergonomics. Client validation is never treated as a
security control; server responses (including `VALIDATION_ERROR` details) drive the
final form state.**

## Mandatory Rules

1. Every API input is validated by a `class-validator` DTO through the global
   validation pipe (whitelist + forbid unknown properties).
2. Frontend Zod schemas mirror the DTO contracts and are used only for pre-submit UX;
   they never gate authorization or persistence.
3. The server response is authoritative: `VALIDATION_ERROR` payloads map to field
   errors in the form (`../05-api/errors.md`).
4. Alignment is maintained by generating TypeScript types from OpenAPI where
   practical; drift is caught by contract tests (R-17).
5. No validation logic is duplicated in controllers or components beyond the DTO
   and its Zod mirror.

## Consequences

- Positive: one authoritative validator plus a UX-friendly mirror; no duplicated trust.
- Negative / accepted cost: two schema definitions must stay aligned; mitigated by
  generated types and contract tests.
- Follow-up work: define the DTO + Zod mirror convention in `../07-backend/architecture.md`
  and `../06-frontend/architecture.md` during Phase 0/1.

## Migration Impact

None — greenfield.

## Status & Approval

- Status: `PROPOSED` → `ACCEPTED` only with a recorded approver and date.
- Register update: `index.md` and `TECHNICAL_DECISIONS.md`.

## Related

- Canonical rationale: `../../TECHNICAL_DECISIONS.md`
- API errors: `../05-api/errors.md` · Backend: `../07-backend/architecture.md`
- Risk: R-17, R-18

*End of ADR-0015*
