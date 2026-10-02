# docs/11-security/security.md

> **Status:** Current (model) — controls `PLANNED` | **Owner:** Security Lead
> **Last Updated:** 2026-10-02 | **Source:** `ARCHITECTURE.md` §9; ADR-004/007; `AGENTS.md` §7
> **Scope:** threat model, controls, RBAC/ABAC, and audit. Enforcement points live in
> `../03-architecture/security.md`; data classes live in `privacy.md`.

## 1. Assets to Protect

| Asset | Why it matters | Primary controls |
|-------|----------------|------------------|
| Tenant data (all modules) | isolation is the product's core promise | TenantGuard, scoped repositories, RLS, negative tests |
| Beneficiary / case / safeguarding data | harm to vulnerable people if exposed | field-level policy, reason capture, separate audit, search/export exclusion |
| Credentials and secrets | platform-wide compromise | hashing/encryption, no logging, secret scanning, rotation |
| Audit trail | accountability and investigation | append-only grants, redaction, separate sensitive stream |
| Content integrity | public trust | workflow transitions, versions, four-eyes for approval |
| Availability | service delivery for field operations | backups, health gating, graceful degradation |

## 2. Threat Model (STRIDE-style)

| # | Threat | Vector | Control |
|---|--------|--------|---------|
| T-1 | Cross-tenant read | guessed/leaked uuid, missing predicate, job/webhook misrouting | mandatory tenant context, scoped repositories, RLS, 404-on-foreign-id, negative tests per surface (R-02) |
| T-2 | Privilege escalation | role/API-key misuse, IDOR on admin routes | deny-by-default guards, permission catalog review, API-key scopes, audited platform-admin path |
| T-3 | Sensitive-data exposure | unscoped list/search/export/report, screenshotting | field-level policy, search/export exclusion, reason capture, watermarked audited exports (R-03) |
| T-4 | Credential theft | weak hashing, token replay, leaked logs | Argon2id, rotating refresh tokens, hashed tokens/keys, no secrets in logs, rate limits |
| T-5 | Upload abuse | MIME spoof, path traversal, malware, decompression bomb | allow-list + magic bytes + size limits + server-generated keys + quarantine + AV adapter (R-07) |
| T-6 | Injection | SQL/NoSQL/command/template | parameterised queries, DTO whitelisting, no raw SQL outside justified PostGIS cases, escaped templates |
| T-7 | Workflow bypass | direct status edit, replaying a transition | transitions only, guarded state machines, transition audit (R-16) |
| T-8 | Webhook forgery/replay | unsigned or replayed delivery | HMAC signature + timestamp window + delivery log; secrets hashed (BR-084) |
| T-9 | Audit tampering | UPDATE/DELETE on audit rows | append-only grants, separate schema, integrity verification |
| T-10 | Denial of service | brute force, bulk export, expensive queries | rate limits per IP/actor/key/tenant, export thresholds, cursor pagination, query budgets |
| T-11 | Insider misuse | legitimate access used improperly | least privilege, ABAC scope, reason capture, anomaly alerts on denials/exports |
| T-12 | Supply chain | vulnerable dependency, malicious package | pinned versions, audit scan in CI, no unreviewed postinstall, dependency review (R-19) |
| T-13 | AI data leakage | sensitive data sent to a provider | privacy filter (redact/block), disabled by default, draft-only outputs (R-15) |
| T-14 | Misconfiguration | public storage bucket, debug mode in prod, permissive CORS | config validation at boot, env contract, review gates, no debug in prod |

## 3. Authentication

```text
Passwords            Argon2id with per-user salt; strength policy enforced at set-time;
                     no default credentials; reset tokens single-use and expiring
Sessions (SPA)       httpOnly + Secure + SameSite cookie; server-side session record;
                     revocation is immediate
Tokens (API)         short-lived access token + rotating refresh token; rotation
                     invalidates the predecessor (BR-011); replay rejected
API keys (M2M)       hashed at rest; prefix shown for identification; raw secret
                     returned once; scoped; revocable immediately (BR-006)
MFA                  TOTP-first, optional per user and enforceable per role (Phase 7)
External IdP         OIDC/SAML adapters behind IdpPort; local credentials remain
                     available to designated admins so an IdP outage cannot lock out
                     the platform
Login events         success/failure recorded with ip + user agent; brute-force
                     throttling per IP and per account
```

## 4. Authorization Model

```text
RBAC   permissions are `resource:action` (e.g. projects:approve); roles are
       per-tenant compositions of permissions; effective permissions = union of
       assigned roles minus explicit denials (BR-014)
ABAC   attribute scope narrows RBAC: organization unit subtree, project assignment,
       donor/grant scope, case assignment. ABAC can never widen access (BR-004)
FIELD  field-level policy is independent of record readability: a readable record
       may still have denied/masked attributes (BR-015/BR-066)
DENY   a route without an explicit permission declaration is denied (BR-003)
```

Permission naming rules:

```text
resource:action      read · create · update · delete · approve · publish · export ·
                     verify · manage · execute · record · report · decide
scope suffix         only where a distinct capability exists (e.g. documents:manage-access)
forbidden            ad-hoc permissions invented per endpoint without catalog review
```

The permission catalog is platform reference data; adding a permission is a reviewed
change that updates the catalog, the roles that use it, and the authz test matrix.

## 5. Audit Model

| Aspect | Rule |
|--------|------|
| Coverage | every mutation writes an append-only record in the same transaction (BR-005) |
| Contents | actor, tenant, action, entity type/id, before/after (redacted), ip, user agent, `request_id`, timestamp |
| Immutability | no `UPDATE`/`DELETE` grants; separate `audit` schema (`../04-data/database.md`) |
| Sensitive records | access with reason is audited; safeguarding writes to a separate stream readable only by designated roles (BR-063) |
| Exports | sensitive exports are audited with actor, scope, row count, and reason (BR-064) |
| Denials | permission/scope/field denials write a **security event**, not a business audit row |
| Redaction | before/after values are redacted per data classification; no secrets ever recorded |
| Query | readable only through the authorized audit endpoint with filters; export is audited and rate-limited |
| Retention | per data-governance policy; never shortened without a documented decision |

## 6. Related

- Enforcement points: `../03-architecture/security.md` · Data classes: `privacy.md`
- Rules: `../02-domain/business-rules.md` §7, §10 · Acceptance: `../09-testing/acceptance-criteria.md` §6
- Risks: `../../RISK_REGISTER.md` R-02, R-03, R-08, R-15, R-18

