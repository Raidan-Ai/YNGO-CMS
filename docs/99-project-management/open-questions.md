# docs/99-project-management/open-questions.md

> **Status:** Pointer | **Owner:** Lead Architect | **Last Updated:** 2026-10-02
> **This file intentionally contains no question data.** Single source of truth:
> **`OPEN_QUESTIONS.md`** (repo root).

## What lives there

| Class | Count | Effect |
|-------|-------|--------|
| BLOCKING (Q-01 … Q-06) | 6 | Phase 0 may not start until answered and the matching ADR is `ACCEPTED` |
| IMPORTANT (Q-07 … Q-11) | 5 | Must be answered before the affected phase begins |
| OPTIONAL (Q-12 … Q-14) | 3 | May be deferred with a documented default |

Each entry carries: question, options considered, recommended answer, the ADR it
decides, and what it blocks. Approved answers are written into the decision log at
the bottom of that file.

## Agent behaviour when a question is unanswered

1. **Do not guess.** Do not implement a variant and document it as the requirement.
2. If the question is **OPTIONAL**, adopt the documented default and record it in
   your task report.
3. If the question is **IMPORTANT** and the affected phase has not started, proceed
   with other tasks and flag the blocker.
4. If the question is **BLOCKING**, stop, report `BLOCKED` with the question id, and
   take no code action (`AGENTS.md` §13).

## Related

- Canonical questions: `../../OPEN_QUESTIONS.md`
- Decisions: `../../TECHNICAL_DECISIONS.md`, `../13-decisions/index.md`
- Audit context: `../../PROJECT_AUDIT.md` §19

*End of docs/99-project-management/open-questions.md*