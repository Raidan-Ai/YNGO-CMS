# ADR-0012 — First-Run Bootstrap of Organization and Admin

> **Status:** ACCEPTED | **Owner:** Lead Architect | **Date:** 2026-10-02
> **Canonical rationale:** `../../TECHNICAL_DECISIONS.md` (ADR-012)
> **Decides:** `../../OPEN_QUESTIONS.md` Q-05 | **Blocks:** local dev, first deploy, onboarding

## Context

A clean install is genuinely empty (ADR-013): no tenant, no organization, no admin
user. The platform must still be usable by a non-technical operator on first run,
while honouring the absolute rules "no default credentials" and "no automatic demo
organization" (`AGENTS.md` §7, `PROJECT_AUDIT.md`). There must be exactly one
controlled path that creates the first tenant/org/admin and it must not be replayable.

## Options

| Option | Pros | Cons |
|--------|------|------|
| A. Seeded default admin (`admin/admin`) | Zero-friction first login | Violates security rules; credential is guessable; forbidden |
| B. CLI `pnpm bootstrap:org` | Scriptable, auditable, no exposed endpoint | Requires shell access (acceptable for operators) |
| C. One-time-token web wizard | Usable without a shell | Needs a guarded endpoint and a token from env |
| D. Auto-create a demo org on boot | Instant content to look at | Fabricated business data; forbidden |

## Decision

**A CLI command (`pnpm bootstrap:org`) is the primary provisioning path, with an
optional web setup wizard that is enabled only while the platform has zero tenants
and requires a one-time bootstrap token supplied through the environment. The
operator supplies the organization name, admin email, and admin password at
runtime — nothing is defaulted. The bootstrap path self-disables after the first
successful run and every attempt is audited.**

## Mandatory Rules

1. The bootstrap command reads operator-provided values from prompt/env; it never
   falls back to a default email, name, or password.
2. The web wizard route is reachable only while `tenant_count = 0`; it requires the
   `BOOTSTRAP_TOKEN` env value and is rate-limited.
3. After the first success, both the CLI path and the wizard refuse to create a
   second root tenant; re-running without a valid token fails and creates nothing.
4. The created password is never logged, never echoed, and is stored only as an
   Argon2id hash (`docs/11-security/security.md` §3).
5. Bootstrap writes an audit record (actor = `bootstrap`, action, resulting ids).
6. No demo organization, sample pages, or seed business records are ever created
   (ADR-013).

## Consequences

- Positive: satisfies "no `admin/admin`" and "no automatic demo org" while staying
  usable for non-technical operators; one controlled, auditable entry point.
- Negative / accepted cost: first run requires either a shell step or possession of
  the bootstrap token; documented in `../10-devops/local-development.md` and the
  runbook (`../12-operations/runbook.md`).
- Follow-up work: expose bootstrap status on the admin status surface so an operator
  can confirm provisioning completed.

## Migration Impact

None — greenfield. The bootstrap command is created in Phase 0 (TASK-010).

## Status & Approval

- Status: `PROPOSED` → `ACCEPTED` only with a recorded approver and date.
- Register update: `index.md` and `TECHNICAL_DECISIONS.md`.

## Related

- Canonical rationale: `../../TECHNICAL_DECISIONS.md`
- Plan impact: `../../IMPLEMENTATION_PLAN.md` (Phase 0, TASK-010)
- Onboarding flow: `../01-product/user-flows.md` §1 · Risk: R-04

*End of ADR-0012*
