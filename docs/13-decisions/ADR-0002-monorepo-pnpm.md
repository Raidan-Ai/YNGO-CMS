# ADR-0002 — Monorepo with pnpm Workspaces

> **Status:** PROPOSED | **Owner:** Lead Architect | **Date:** 2026-10-02
> **Canonical rationale:** `../../TECHNICAL_DECISIONS.md` (ADR-002)
> **Decides:** `../../OPEN_QUESTIONS.md` Q-08

## Context

The platform needs a single source for API contracts shared by the API, the web
app, and generated SDKs, plus one CI pipeline that can lint/typecheck/test all
surfaces atomically. Separate repositories would force premature versioning of
internal packages.

## Decision

**One monorepo managed with pnpm workspaces: `apps/*` (api, web, worker) and
`packages/*` (contracts, config, ui).**

## Mandatory Rules

1. `pnpm` is the only package manager; no `npm install`/`yarn` artefacts
   (`package-lock.json`, `yarn.lock`) may be committed.
2. Workspace packages are consumed by name (`@yngo/contracts`), never by relative
   path crossing app boundaries.
3. Root scripts (`pnpm lint`, `pnpm typecheck`, `pnpm test`, `pnpm build`) run
   across all workspaces and are the CI entrypoints.
4. Dependency versions are pinned; shared configuration lives in `packages/config`.
5. `tests/` holds cross-app integration/E2E only; unit tests stay next to the code.

## Consequences

- Positive: atomic contract changes, one lockfile, simple CI, no internal publishing.
- Negative: CI must be scoped (`--filter`) to stay fast; workspace-wide bumps need care.
- Follow-up: enforce scope tags so `apps/web` cannot import `apps/api` internals.

## Migration Impact

None — greenfield.

## Related

- Repo layout: `../../AGENTS.md` §2 · Git rules: `../../AGENTS.md` §8
- CI: `../09-testing/strategy.md`

*End of ADR-0002*