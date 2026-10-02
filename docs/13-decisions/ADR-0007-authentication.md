# ADR-0007 — Authentication

> **Status:** PROPOSED | **Owner:** Security Lead | **Date:** 2026-10-02
> **Canonical rationale:** `../../TECHNICAL_DECISIONS.md` (ADR-007)
> **Blocks:** identity module, portals, API keys, MFA timeline

## Context

Three client classes need authentication: a first-party browser SPA (admin), API
consumers (integrations, SDKs, scripts), and external portals. The product must
avoid the classic failure of storing long-lived bearer tokens in browser storage,
and must support revocation, session visibility, and later MFA.

## Options

| Option | Pros | Cons |
|--------|------|------|
| A. httpOnly session cookie for the SPA + bearer tokens for API clients | No token in JS; CSRF-defendable; simple revocation; separate API path | Two mechanisms to maintain; CSRF handling required |
| B. Bearer access token in browser memory + refresh | Stateless | XSS exposure, silent revoke gaps, refresh complexity in every client |
| C. Long-lived API tokens for everything | Simplest | No browser-safe story; unacceptable token lifetime |

## Decision

**A: the first-party web app authenticates with an `httpOnly`, `Secure`,
`SameSite` session cookie bound to the tenant; API consumers authenticate with
short-lived bearer access tokens plus rotating refresh tokens. Password hashing
uses Argon2id (bcrypt acceptable if Argon2id is unavailable). MFA (TOTP) is
introduced in Phase 7 and hooks into the same session/token model.**

## Mandatory Rules

1. No credential is ever stored in `localStorage`/`sessionStorage`.
2. Refresh tokens rotate on use; reuse of a rotated token revokes the family and is
   logged as a security event.
3. Sessions and tokens are revocable per user and per device; users can list and
   revoke their own sessions.
4. Login success/failure, lockout, password reset, MFA enrol/verify, and session
   revocation are all audited (`../11-security/security.md`).
5. Cookies are `httpOnly`, `Secure`, `SameSite=Lax` (or `Strict` where the flow
   allows), scoped to the tenant host/path.
6. Rate limiting and progressive lockout apply to login, reset, and MFA endpoints.
7. Password policy, reset token TTL, and token TTLs are configuration, not code.

## Consequences

- Positive: browser-safe by default; revocation is real; API clients unaffected by
  CSRF concerns.
- Negative: two credential pathways and CSRF protection must be tested explicitly.
- Follow-up: MFA recovery codes and step-up rules documented in Phase 7 (TASK-074).

## Related

- Identity tasks: TASK-008, TASK-012, TASK-013, TASK-074
- Security docs: `../11-security/security.md` · Errors: `../05-api/errors.md`

*End of ADR-0007*