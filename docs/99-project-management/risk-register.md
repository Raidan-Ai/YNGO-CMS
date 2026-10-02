# docs/99-project-management/risk-register.md

> **Status:** Pointer | **Owner:** Lead Architect | **Last Updated:** 2026-10-02
> **This file intentionally contains no risk data.** Single source of truth:
> **`RISK_REGISTER.md`** (repo root).

## Why a pointer instead of a copy

Duplicated risk tables drift. The canonical register holds:

- Scoring method (Probability × Impact; Critical 20-25, High 12-19, Medium 6-11, Low 1-5).
- R-01 … R-24 with category, owner, mitigation, status.
- Top-5 risks requiring immediate action.
- Acceptance policy (no Critical risk accepted silently).
- Traceability map to ADRs and TASK ids.

## How risks are used during implementation

| Moment | Action |
|--------|--------|
| Task start | Read the risks whose traceability row references your TASK id; apply the mitigation as an acceptance criterion |
| Task end | State which risks your change reduced or affected (see `AGENTS.md` §14 reporting format) |
| Phase exit | Re-score affected risks; a phase gate cannot pass with an unmitigated **Critical** risk |
| Release | Publish open risks in the release notes; newly accepted risks require an ADR |

## Related

- Canonical register: `../../RISK_REGISTER.md`
- Plan and gates: `../../IMPLEMENTATION_PLAN.md`
- Threats and controls: `../11-security/security.md`, `../11-security/privacy.md`

*End of docs/99-project-management/risk-register.md*