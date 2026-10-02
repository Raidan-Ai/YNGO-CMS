# ADR-0010 — Search

> **Status:** PROPOSED | **Owner:** Backend Lead | **Date:** 2026-10-02
> **Canonical rationale:** `../../TECHNICAL_DECISIONS.md` (ADR-0010)
> **Decides:** `../../OPEN_QUESTIONS.md` Q-11 | **Blocks:** search, indexes, cross-entity lookup

## Context

Users search content, projects, partners, grants, documents, and (permission
permitting) people records, in Arabic and English. Search must never leak across
tenants or beyond permissions. Running a dedicated search cluster at v1 conflicts
with the small-team operability goal, but a future swap to OpenSearch must not
require domain rewrites.

## Options

| Option | Pros | Cons |
|--------|------|------|
| A. PostgreSQL FTS (`tsvector`, GIN) behind `SearchPort` | No new infrastructure; transactional consistency; Arabic config exists | Weaker relevance/facets at scale |
| B. OpenSearch/Elasticsearch | Rich relevance, facets, scale | Separate cluster to operate, sync lag, second source of truth |
| C. `LIKE`/`ILIKE` queries | Trivial | Poor quality, no ranking, slow |

## Decision

**PostgreSQL full-text search behind a `SearchPort` interface
(`index`, `remove`, `search`) with mandatory tenant and permission predicates on
every call. OpenSearch may replace the adapter later without changing domain code.**

## Mandatory Rules

1. Every search call takes `tenant_id` plus the caller's permission filter; the
   port must reject a call without them (`TENANT_REQUIRED` / `FORBIDDEN`).
2. Restricted data classes are excluded from generic indexes by policy:
   safeguarding and case records are never indexed for general search; internally
   they remain reachable only through their own permissioned modules.
3. Soft-deleted records are removed from the index.
4. Ranking and filtering are deterministic and tested (no "search is magic").
5. Index updates happen in the same transaction/outbox as the source change so the
   index cannot silently diverge.
6. Arabic and English configurations are both created; Arabic tokenisation is
   tuned in Phase 8 (TASK-085) with tests.
7. Adding a search backend is a new adapter + ADR, not a rewrite of modules.

## Consequences

- Positive: zero new infrastructure at v1; consistent with tenancy/permission rules.
- Negative: facet-heavy analytics may need materialised projections.
- Follow-up: query budgets and index review in TASK-110.

## Related

- Ports: `../07-backend/services.md` · Data: `../04-data/schema.md`
- Privacy exclusions: `../11-security/privacy.md` · Risk: R-12

*End of ADR-0010*