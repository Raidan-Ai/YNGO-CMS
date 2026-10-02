# ADR-0003 — ORM and Migrations

> **Status:** ACCEPTED | **Owner:** DB Architect | **Date:** 2026-10-02
> **Canonical rationale:** `../../TECHNICAL_DECISIONS.md` (ADR-003)
> **Decides:** `../../OPEN_QUESTIONS.md` Q-02 | **Blocks:** schema, migrations, API scaffolding

## Context

Canonical storage is PostgreSQL + PostGIS with tenant-scoped relational graphs,
row-level security as a defence layer, and spatial types/operators. The tool must
express those without escaping to raw SQL for every spatial query, and must be
native to NestJS dependency injection.

## Options

| Option | Pros | Cons |
|--------|------|------|
| A. Prisma | Best DX, generated types | Weaker PostGIS/RLS story; partial escape hatch |
| B. TypeORM | Native Nest integration, entities + migrations, spatial column support | Decorator-heavy, verbose criteria API |
| C. Drizzle | SQL-close, typed, light | Smaller NestJS integration surface, less familiar |

## Decision

**TypeORM is the default ORM and migration runner, paired with PostgreSQL 16+ and
PostGIS. Raw SQL is allowed for spatial/analytical queries and must live in
reviewed repository methods.**

## Mandatory Rules

1. Migrations are **append-only**, generated as files, reviewed, and never edited
   after being applied to a shared environment.
2. Every tenant-owned entity carries `tenant_id`; global tables are explicitly
   marked as such in `../04-data/database.md`.
3. Repository base class applies the tenant predicate automatically; domain code
   must not hand-write unscoped `find` calls on tenant-owned entities.
4. Spatial columns use PostGIS types (`geometry(Point,4326)` etc.) with GiST indexes.
5. An index is added in the same migration as the query that needs it (query budget rule).
6. Prisma may only be adopted by a later ADR that proves PostGIS + RLS parity.

## Consequences

- Positive: native DI, single migration path, spatial support, explicit SQL escape hatch.
- Negative: more boilerplate than Prisma; must enforce repository discipline by review.
- Follow-up: document the repository base contract in `../07-backend/architecture.md`.

## Migration Impact

None — greenfield.

## Related

- Data docs: `../04-data/database.md`, `../04-data/schema.md`, `../04-data/migrations.md`
- Rule: `../../AGENTS.md` §9 · Risk: `../../RISK_REGISTER.md` R-11

*End of ADR-0003*
