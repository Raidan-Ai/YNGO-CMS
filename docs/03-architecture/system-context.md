# docs/03-architecture/system-context.md

> **Status:** Current (design) — implementation `PLANNED` | **Owner:** Lead Architect
> **Last Updated:** 2026-10-02 | **Source:** `ARCHITECTURE.md` §3; personas from `../01-product/personas.md`
> **Purpose:** C4 Level 1 — who and what the system talks to, and where trust changes.

## 1. Actors

| Actor | Kind | Interface | Trust |
|-------|------|-----------|-------|
| Public visitor (X-01) | Person | HTTPS → public site / `/public/*` | Untrusted |
| Beneficiary / complainant (X-04, X-06) | Person | Public forms, optional anonymity | Untrusted |
| Donor, partner, volunteer, member (X-02/03/05) | Person | Portal (authenticated, own records) | Authenticated, narrow scope |
| NGO staff (P-01 … P-15) | Person | Admin SPA (session cookie) | Authenticated, RBAC + ABAC |
| Platform (system) admin (P-16) | Person | Admin console, cross-tenant | Highly privileged, heavily audited |
| Developer / integrator (P-17) | System actor | `/api/v1` with API key | Scoped to granted permissions |
| External systems | System actor | REST + webhooks + OIDC/SAML | Per-integration credentials |
| AI provider (optional) | System actor | Outbound only, via `AiPort` | Bounded, redacted payloads (ADR-014) |

## 2. System Context (C4 Level 1)

```mermaid
flowchart TD
    Visitor[Public Visitor] -->|HTTPS| Web[YNGO-CMS Web]
    Staff[NGO Staff] -->|HTTPS session| Web
    Portal[Donor / Partner / Volunteer / Member] -->|HTTPS| Web
    Dev[Developer / Integrator] -->|Bearer API key| Api[YNGO-CMS API /api/v1]
    Ext[External Systems] -->|REST + webhooks| Api

    Web --> Api
    Api --> PG[(PostgreSQL + PostGIS)]
    Api --> Redis[(Redis cache/queue)]
    Api --> Store[(S3 / MinIO objects)]
    Api --> Jobs[Worker]
    Jobs --> PG
    Jobs --> Redis
    Jobs --> Store

    Api -.optional.-|AiPort| AI[AI sidecar - FastAPI]
    Jobs -.->|email / SMS / WhatsApp| Providers[External providers]
    Api -.->|HMAC webhooks| Ext
    Api -.->|OIDC / SAML| IdP[Identity providers]
```

## 3. Trust Boundaries

| Boundary | Crossing | Enforcement |
|----------|----------|-------------|
| Internet → edge | Public HTTP(S) | TLS termination at the reverse proxy, header hygiene, WAF/rate limits |
| Edge → `web` | Page/asset requests | CSP, HSTS, no server secrets in the browser bundle |
| Internet → `api` (public routes) | `/public/*`, `POST /public/complaints` | Strict rate limits, published-content-only rule, no tenant leakage |
| Internet → `api` (authenticated) | Session cookie, bearer token, API key | Authentication → TenantGuard → PermissionsGuard → ABAC policy |
| `api` → database | SQL | Least-privilege role, parameterised queries, RLS session binding |
| `api` → object storage | S3 API | Server-generated keys, scoped credentials, signed URLs only |
| `api`/`worker` → providers | Outbound HTTPS | Egress allow-list, redacted payloads, per-provider credentials |
| `api` → AI sidecar (optional) | Internal HTTP | Bounded contract, no canonical-store credentials beyond scoped reads |
| Operator → infrastructure | SSH/console | Documented runbook, audit of privileged actions, no shared credentials |

## 4. External Dependencies and Failure Posture

| Dependency | Required at V1 | If unavailable |
|------------|:--------------:|----------------|
| PostgreSQL + PostGIS | Yes | `/ready` fails; 503 on writes; workers pause |
| Redis | Yes (cache/queue/rate limit) | Rate limits degrade in-process; async jobs delay; canonical data unaffected |
| S3-compatible storage | Yes | Uploads/list return 503; metadata not committed without a stored object |
| SMTP | Yes (notifications) | Delivery retries then dead-letters; in-app still works |
| SMS/WhatsApp/push | No | Adapter disabled; notifier falls back to in-app/email |
| Identity provider (OIDC/SAML) | No | Local credentials remain the default path |
| Map tiles (OSM) | No (public map only) | Map degrades with an explicit unavailable state |
| AI provider | No | AI features disabled; core flows never blocked |

Failure handling detail: `ARCHITECTURE.md` §13.

## 5. Boundaries the Architecture Must Preserve

1. **No direct client access** to the database, Redis, or object storage.
2. **No portal-specific backend logic** — portals call the same API with narrower
   tokens (`../01-product/personas.md` §3).
3. **No cross-tenant path** except an explicit, audited platform-admin code path
   (`ADR-0004` rule 6).
4. **No AI path** that writes to the canonical store without human approval
   (`ADR-014`).
5. **No public path** to unpublished content, restricted locations, or any
   sensitive/restricted entity (`BR-024`, `BR-034`, `BR-062`).

## Related

- Root architecture: `../../ARCHITECTURE.md` §3, §9
- Containers/components: `components.md` · Flows: `data-flow.md` · Security: `security.md`
- Personas: `../01-product/personas.md`

*End of docs/03-architecture/system-context.md*