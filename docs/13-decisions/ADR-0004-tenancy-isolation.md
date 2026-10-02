# ADR-0004 — Multi-Tenancy Data Isolation

> **Status:** ACCEPTED | **Owner:** Lead Architect | **Date:** 2026-10-02
> **Canonical rationale:** `../../TECHNICAL_DECISIONS.md` (ADR-004)
> **Decides:** `../../OPEN_QUESTIONS.md` Q-04 | **Blocks:** every table, query, test, review

## Context

One deployment must serve multiple organizations with provable isolation while
keeping migrations, backups, and operational cost manageable. Isolation failures
are the single most severe risk in the product (R-02): they affect every data
surface — reads, lists, search, exports, reports, files, webhooks, jobs, AI calls.

## Options

| Option | Pros | Cons |
|--------|------|------|
| A. Shared schema + `tenant_id` | One migration path, cheap operations, easy cross-tenant platform features | Requires discipline + tests; a single missed predicate leaks data |
| B. Shared schema + Postgres RLS | Database-enforced defence in depth | Session-variable plumbing; harder debugging; still not a substitute for A |
| C. Schema-per-tenant | Strong separation | Migration fan-out, connection/pool complexity, cross-tenant analytics pain |
| D. Database-per-tenant | Strongest separation | Highest ops cost; incompatible with small-team operations goal |

## Decision

**Shared schema with a mandatory `tenant_id` on every tenant-owned table, enforced
by a tenant-scoped repository base and guards, with Postgres RLS enabled as
defence-in-depth where the table is tenant-owned.**

## Mandatory Rules

1. `tenant_id uuid NOT NULL` + index on every tenant-owned table; the omission on a
   global table must be documented as intentional.
2. The tenant predicate is applied by the repository layer, never by ad-hoc code.
3. Tenant context is established once per request/job and is immutable thereafter;
   it may never be read from user-supplied body fields.
4. Jobs, webhooks, exports, reports, and AI calls carry the originating tenant id.
5. Every module ships a cross-tenant negative test proving 403/404 on foreign ids.
6. Platform-admin cross-tenant access is a distinct, explicitly audited code path.

## Consequences

- Positive: simple operations, single migration set, cheap onboarding.
- Negative: isolation depends on discipline — mitigated by the repository base,
  RLS, and mandatory negative tests per module.
- Follow-up: define the RLS policy template in `../04-data/database.md`.

## Migration Impact

None — greenfield.

## Related

- Tenant resolution: `ADR-0005-tenant-resolution.md` · Data docs: `../04-data/*`
- Security: `../11-security/security.md` · Risk: `../../RISK_REGISTER.md` R-02

*End of ADR-0004*
