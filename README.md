# YNGO-CMS — YemenNGO-CMS

**An open, modular, API-first digital operating platform for civil-society and NGO organizations.**

> Content · Programs · People · Impact

---

## Status

| Item | State |
|------|-------|
| Stage | **Specification complete — implementation not started** |
| Application code | None yet (greenfield) |
| Documentation | Complete planning package (this repo) |
| Next milestone | Phase 0 — Foundation (see `IMPLEMENTATION_PLAN.md`) |
| Blocking decisions | `OPEN_QUESTIONS.md` (Q-01 … Q-06) |

This repository currently contains the **engineering planning package** for
YNGO-CMS: the audit, target architecture, decisions, risks, execution contract,
and the phased implementation plan. No application code exists yet — by design.

---

## What YNGO-CMS Is

A configurable platform that combines a professional **content management system**
with NGO **operational modules** — programs, projects, grants, MEAL, people,
partners, documents, workflows, reporting, GIS, integrations, automation, and
optional human-controlled AI — on a single multi-tenant, Arabic-first, API-first
foundation.

**Core principles:** API-first · modular · multi-tenant · Arabic-first (RTL) ·
headless-capable · configurable · secure by design · self-hostable · open
integration model · human-controlled AI.

**Explicitly not:** a WordPress clone, a website builder only, a donor CRM only,
an ERP replacement on day one, or a data-collection app only.

---

## Documentation Map

| Document | Purpose |
|----------|---------|
| `AGENTS.md` | **Execution contract** — read first before any change |
| `PROJECT_AUDIT.md` | Audit of current state, gaps, contradictions, findings |
| `ARCHITECTURE.md` | Target architecture, boundaries, security, deployment |
| `IMPLEMENTATION_PLAN.md` | Phased plan with exit gates and task breakdown |
| `TECHNICAL_DECISIONS.md` | ADR log (stack, tenancy, API, storage, auth, …) |
| `RISK_REGISTER.md` | Risk matrix, owners, mitigations |
| `OPEN_QUESTIONS.md` | Blocking / important / optional decisions |
| `docs/` | Structured living documentation (single source of truth) |

Original specification files (`YNGO-CMS_*.md`) are **frozen inputs** retained for
provenance; `docs/` is authoritative going forward.

---

## Planned Stack

```text
Frontend    Next.js + React + TypeScript + Tailwind + shadcn/ui + MapLibre
Backend     Node.js + TypeScript + NestJS + Fastify
Database    PostgreSQL + PostGIS (canonical source of truth)
Cache/Queue Redis + BullMQ (worker)
Storage     S3-compatible (MinIO reference)
API         REST /api/v1 + generated OpenAPI
Optional AI FastAPI + Python sidecar (bounded, human-reviewed)
Deployment  Docker + Docker Compose, reverse proxy, TLS
```

Confirming the backend framework choice is **Q-01 / ADR-001** in
`OPEN_QUESTIONS.md` and `TECHNICAL_DECISIONS.md`.

---

## Getting Started (once Phase 0 is approved)

```bash
# 1. Review and accept the blocking ADRs and open questions
#    see TECHNICAL_DECISIONS.md and OPEN_QUESTIONS.md

# 2. Install dependencies
pnpm install

# 3. Create your local environment file from the template
cp .env.example .env

# 4. Start the development stack
docker compose up -d

# 5. Run database migrations
pnpm db:migrate

# 6. Create the first organisation and administrator (no default credentials)
pnpm bootstrap:org
```

These commands describe the **target** developer experience defined in
`IMPLEMENTATION_PLAN.md`; they become runnable after Phase 0 is implemented.

---

## Non-Negotiables

```text
No mock, demo, or fake business data in runtime code
No default credentials (no admin/admin)
No secrets committed to source control
No hard-coded tenant ids or production domains
Tenant isolation enforced and tested on every data surface
Authorization enforced by the backend, never by the UI alone
A clean installation is genuinely empty, with professional empty states
```

---

## Contributing

Read `AGENTS.md` first — it is the binding execution contract covering coding,
testing, documentation, security, git, migration, and environment rules, plus
the Definition of Done every change must satisfy.

---

## Licence

Not yet determined — see Q-13 in `OPEN_QUESTIONS.md`. No public distribution
until decided.