# docs/03-architecture/security.md

> **Status:** Current (design) — implementation `PLANNED` | **Owner:** Security Lead
> **Last Updated:** 2026-10-02 | **Source:** `ARCHITECTURE.md` §9; ADR-004/007
> **Scope:** **where** security is enforced in the stack. Threat model, controls,
> privacy classes, and RBAC/ABAC detail live in `../11-security/security.md` and
> `../11-security/privacy.md`.

## 1. Enforcement Pipeline (the only trusted boundary)

```text
Request
 1  proxy            TLS, HSTS, header hygiene, body/rate limits at the edge
 2  RequestId        attach/propagate X-Request-Id (audit + trace correlation)
 3  Logging          structured JSON, no PII/secrets
 4  RateLimitGuard   per IP / per actor / per API key / per tenant
 5  Authentication   session cookie | bearer access token | API key
 6  TenantGuard      resolve tenant (ADR-005) + verify membership; reject otherwise
 7  PermissionsGuard deny-by-default `resource:action` (BR-003)
 8  PolicyGuard      ABAC attribute scope — narrows, never widens (BR-004)
 9  ValidationPipe   DTO whitelist + forbidNonWhitelisted (ADR-015)
10  FieldPolicy      strip/mask restricted attributes on the way out (BR-015)
11  Service/Domain   invariants; state changes only via workflow transitions
12  Repository       tenant predicate applied by base class (+ RLS in the DB)
13  Audit            append-only row in the same transaction (BR-005)
14  ExceptionFilter  stable error envelope; no internals leaked (errors.md)
```

Rules: a route with no permission declaration is **denied**; guards are registered
globally so a new module cannot opt out by forgetting; the frontend is never a
security boundary (`R-18`).

## 2. Where Each Control Lives

| Control | Frontend | API | Database | Worker | Infra |
|---------|:--------:|:---:|:--------:|:------:|:-----:|
| Authentication | session handling | guards + token signing | hashed tokens | job context | TLS |
| Tenant isolation | hint only (`X-Tenant-Id`) | TenantGuard + scoped repos | `tenant_id` + RLS | payload tenant | network policy |
| RBAC | hide/disable (UX) | PermissionsGuard | — | permission reload per job | — |
| ABAC | — | PolicyGuard | indexed scope columns | same policy service | — |
| Field-level policy | masking display | FieldPolicyInterceptor | encrypted columns | redaction before send | — |
| Audit | — | AuditInterceptor | append-only grants | job audit rows | log retention |
| Upload safety | client hints only | MIME + magic bytes + size | metadata rows | scan + renditions | quarantine bucket |
| Secrets | never | config validation | hashed/encrypted at rest | same config | `.env`/secret store |
| Rate limiting | — | guard + Redis | — | per-queue limits | edge limits |

## 3. Isolation Proof Points

```text
Read        GET foreign id            → 404 <ENTITY>_NOT_FOUND
List        filter by foreign tenant  → empty result, never foreign rows
Search      query term from tenant B  → nothing returned to tenant A
Export      id from tenant B          → 404/403, audited denial
Report      parameters from tenant B  → no cross-tenant rows in output
File        key from tenant B         → 404; signed URL never issued
Webhook     endpoint from tenant B    → 404; no delivery attempted
Job         payload tenant mismatch   → job rejected; security event written
AI request  data from tenant B        → blocked before the provider call
```

Each point requires a negative test per module (`AGENTS.md` §5, `BR-001`/`BR-060`).

## 4. Sensitive Data Paths (extra controls)

| Path | Extra controls |
|------|----------------|
| Beneficiary / household | Pseudonymous reference codes in lists, encrypted direct identifiers, field-level masking, export requires `beneficiaries:export` + reason + audit (BR-064/066) |
| Case records | `cases:read` + ABAC; reason captured on access; every read of a restricted record audited (BR-061) |
| Safeguarding | `safeguarding:*` roles only; separate audit stream (BR-063); excluded from search, feeds, and generic exports (BR-062); export needs elevated permission + justification |
| Complaints | Anonymity preserved in audit (actor recorded as `anonymous` + channel, never identity) |
| Locations | `is_public=false` or `is_sensitive` excluded from public layers; precision reduced by role (BR-034) |
| Documents | Classification label gates search/export eligibility (BR-024); downloads audited |

Rule: the **absence** of a record is never distinguishable from a denial
(`errors.md` §4 rules 1–2).

## 5. Secret and Credential Handling

```text
Passwords          Argon2id, per-user salt, never logged, never defaulted
Session tokens     opaque, stored hashed, rotating refresh (ADR-007)
API keys           hashed at rest, prefix-only display, raw value shown once (BR-006)
Integration creds  encrypted at rest; decrypted only in the adapter call path
Webhook secrets    stored hashed; HMAC per delivery with timestamp + replay window
JWT/signing keys   from configuration/secret store; rotation documented in the runbook
Env vars           validated at boot; fail-fast; .env git-ignored; .env.example lists keys
```

No secret is ever written to logs, error payloads, audit before/after values,
traces, or webhook bodies.

## 6. Security Events (distinct from business audit)

```text
login.success · login.failure · mfa.challenge · session.revoked
permission.denied · scope.denied · field.denied · tenant.forbidden
export.denied · sensitive.read (with reason) · api_key.created|revoked
webhook.disabled (repeated failure) · rate_limit.triggered
platform_admin.action (cross-tenant, always audited)
```

Security events carry `request_id`, actor (or `anonymous`), tenant, route, and
outcome — never the sensitive payload that caused the denial.

## 7. Residual Risks Accepted at Design Time

| Risk | Why accepted | Compensating control |
|------|--------------|----------------------|
| Shared-schema isolation depends on discipline | Costs/ops of schema-per-tenant (ADR-0004) | Scoped repository base + RLS + mandatory negative tests (R-02) |
| Redis unavailable weakens rate limiting | Canonical data is unaffected | In-process fallback limits + alerting |
| AntivirusPort is a no-op by default | No AV dependency at V1 | Quarantine + allow-list + logged no-op + documented enablement (R-07) |
| Public OSM tiles are a third-party dependency | No self-hosting cost at V1 | `MapPort` abstraction; self-host path (Q-07) |
| AI sidecar availability | Optional capability | Disabled by default; core flows never depend on it (R-15) |

Any **Critical** risk may not be accepted silently (`RISK_REGISTER.md`); acceptance
requires a named ADR entry.

## 8. Review Gates

```text
[ ] every new route declares permissions and appears in the authz matrix
[ ] every tenant-scoped surface has a cross-tenant negative test
[ ] field-level policy covered for every sensitive/restricted entity
[ ] uploads pass allow-list + magic byte + size + quarantine tests
[ ] no secret or PII in logs, errors, audit values, or tracing
[ ] error paths return the documented codes without leaking internals
[ ] dependency scan shows no known critical vulnerability
```

## Related

- Root architecture: `../../ARCHITECTURE.md` §9
- Threat model + controls: `../11-security/security.md` · Privacy: `../11-security/privacy.md`
- Tenancy: `ADR-0004` · Auth: `ADR-0007` · Flow detail: `data-flow.md`

*End of docs/03-architecture/security.md*