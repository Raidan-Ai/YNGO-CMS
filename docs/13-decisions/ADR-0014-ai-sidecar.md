# ADR-0014 — Optional Bounded Python AI Sidecar

> **Status:** ACCEPTED | **Owner:** Lead Architect | **Date:** 2026-10-02
> **Canonical rationale:** `../../TECHNICAL_DECISIONS.md` (ADR-014)
> **Blocks:** Phase 10 AI work | **Decides:** `../../OPEN_QUESTIONS.md` Q-12 (default)

## Context

AI assistance (translation, summarisation, classification, document extraction,
content assistance, reporting help) is valuable but optional. Python offers the
strongest OCR/NLP/embedding ecosystem, yet the platform's canonical stack forbids
Python in `apps/api` (ADR-001). AI must never become the source of truth or silently
overwrite human decisions, and sensitive data must be protected from provider
leakage (R-15).

## Options

| Option | Pros | Cons |
|--------|------|------|
| A. No AI at all | Simplest; zero privacy surface | Loses genuine productivity value |
| B. AI embedded in `apps/api` (Node libs) | One runtime | Weak OCR/NLP; bloats the API; harder to isolate |
| C. Optional bounded Python sidecar behind `AiPort` | Best ML ecosystem; isolated; swappable | A second runtime to operate and secure |

## Decision

**AI is accessed only through `AiPort`. The optional sidecar is a separate FastAPI
service with no database ownership and no credentials to the canonical store beyond
read-only, permission-scoped retrieval. Every AI output is a draft requiring
explicit human review before it becomes a record. AI inherits the requesting user's
permissions and may never broaden them.**

## Mandatory Rules

1. `apps/api` contains no Python; the sidecar is a separate deployable (compose
   profile `ai`) and is disabled by default.
2. `AiPort` exposes a bounded contract; the default binding is a no-op so core flows
   never depend on AI availability.
3. Every AI request/response records provider, model, prompt version, timestamp,
   actor, source references, review status, and usage metadata.
4. AI output is stored as a draft and requires an explicit human approval step before
   it can influence a canonical record; AI never silently overwrites (BR-090/092).
5. Sensitive data is redacted or blocked before any provider call per the privacy
   matrix (`../11-security/privacy.md`); AI calls carry the originating tenant id.
6. Evaluation thresholds gate enabling any AI feature (Phase 10 harness).

## Consequences

- Positive: strongest ML tooling without contaminating the canonical API; AI is
  genuinely optional and failure-isolated.
- Negative / accepted cost: a second runtime to secure, patch, and monitor; Python
  operational skills required when AI is enabled.
- Follow-up work: AI governance + review UI (TASK-101), evaluation harness (TASK-102),
  sidecar (TASK-103) in Phase 10.

## Migration Impact

None — greenfield. Implementation deferred to Phase 10.

## Status & Approval

- Status: `ACCEPTED`. Enabling AI in any environment is an operational decision, not
  an architecture change.
- Register update: `index.md` and `TECHNICAL_DECISIONS.md`.

## Related

- Canonical rationale: `../../TECHNICAL_DECISIONS.md`
- Privacy routing: `../11-security/privacy.md` · Rules: `../02-domain/business-rules.md` §10
- Risk: R-15 · Sidecar module: `../07-backend/modules.md` (#48 `ai`)

*End of ADR-0014*
