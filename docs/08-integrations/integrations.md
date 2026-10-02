# docs/08-integrations/integrations.md

> **Status:** Current (design) — all adapters `PLANNED` | **Owner:** Integrations Lead
> **Last Updated:** 2026-10-02 | **Source:** `ARCHITECTURE.md` §10; ADR-001/009/014
> **Purpose:** how external systems are integrated, configured, and isolated.

## 1. Integration Principles

```text
Ports first        every integration is reached through a port interface
Configuration      credentials and endpoints come from configuration, never code
Least privilege    the smallest credential scope that the use-case needs
Failure isolation  a provider outage degrades one capability, never the platform
Auditability       outbound actions that change state or export data are audited
No reverse control an integration can never write canonical data except through
                   the normal validated API/domain path
```

## 2. Adapter Catalogue

| Integration | Port | Direction | Phase | Default |
|-------------|------|-----------|:-----:|---------|
| Email (SMTP) | `NotifierPort` | outbound | 3 | Enabled |
| In-app notifications | `NotifierPort` | internal | 3 | Enabled |
| SMS gateway | `NotifierPort` | outbound | 9 | Disabled (Q-14) |
| WhatsApp Business | `NotifierPort` | outbound | 9 | Disabled (Q-14) |
| Telegram | `NotifierPort` | outbound | 9 | Disabled |
| Web push | `NotifierPort` | outbound | 9 | Disabled |
| Outbound webhooks | `WebhookPort` (internal) | outbound | 3 | Per subscription |
| Object storage (MinIO/S3/Azure/GCS) | `StoragePort` | both | 2 | MinIO |
| Search backend | `SearchPort` | both | 2 | PostgreSQL FTS |
| Map tiles | `MapPort` | inbound | 3 | Public OSM |
| Identity providers (OIDC/SAML) | `IdpPort` | inbound | 9 | Local credentials |
| Microsoft 365 / Entra ID | `IdpPort`, `CalendarPort`, `StoragePort` | both | 9 | Disabled |
| Google Workspace | `IdpPort`, `CalendarPort`, `StoragePort` | both | 9 | Disabled |
| Slack / Teams | `NotifierPort` | outbound | 9 | Disabled |
| Accounting/ERP | `AccountingPort` | outbound | 9 | Disabled |
| Payment gateway | `PaymentPort` | outbound | 9 | Disabled |
| GIS services (geocoding/routing) | `MapPort`, `GeoPort` | both | 9 | Disabled |
| AI provider(s) | `AiPort` | outbound | 10 | Disabled (`ADR-014`) |
| Antivirus | `AntivirusPort` | internal | 2 | No-op (logged) |

## 3. Configuration Model

```text
integration_provider    platform catalog: code, name, kind, required credential keys
integration_config      per tenant: provider_id, credentials (encrypted), endpoint,
                        options (jsonb, validated), enabled, last_tested_at, health
```

Rules:

1. A tenant enables an integration explicitly; nothing is enabled by default.
2. Credentials are encrypted at rest and never returned by the read API (only a
   masked hint such as the last four characters or key fingerprint).
3. `POST /integrations/{id}/test` performs a connectivity check and returns a
   **redacted** result (no credential echo, no provider stack trace).
4. Health and last-success are visible in the admin UI so silent failure is
   impossible.
5. Per-tenant configuration means one tenant cannot observe another's endpoints,
   credentials, or delivery logs.

## 4. Failure Isolation and Retry

| Dependency | Failure behaviour | Retry |
|------------|-------------------|-------|
| Email/SMS/WhatsApp | notification retries then dead-letters; in-app still delivered | exponential backoff, max attempts, then visible dead-letter |
| Webhooks | delivery attempts recorded; endpoint auto-disabled after repeated failure | exponential backoff + jitter; manual redelivery available |
| Object storage | upload/list return 503 with a clear error; metadata not committed without a stored object | client retry; no partial record |
| Search backend | writes still succeed; projection rebuilt by queue when available | queue retry; rebuild from source |
| Map tiles | map shows an explicit unavailable state; the page still functions | client retry |
| Identity provider | local credentials remain available for designated admins | n/a — fail closed for the IdP path only |
| Accounting/ERP export | job fails visibly in the job list; no partial postings | operator-triggered rerun |
| AI provider | AI features degrade; core flows never block | bounded retry, then disabled with a status message |

Rules: **no integration failure may corrupt canonical data or block a core flow**
(`ARCHITECTURE.md` §13); every failure is recorded with `tenant_id` + `request_id`
and surfaced in the admin console.

## 5. Outbound Data Rules

```text
[ ] outbound payloads contain only the fields the integration needs
[ ] sensitive/restricted entities are never sent to an integration by default
[ ] AI calls pass through the privacy filter (redact or block, BR-094)
[ ] PII is not written to integration logs or delivery records
[ ] every export-style integration action is audited with actor + reason
[ ] webhook payloads are HMAC-signed with a timestamp and replay window
[ ] a tenant may never subscribe to an event outside its own scope
```

## 6. Webhook Contract (outbound)

```text
Headers    X-YNGO-Event · X-YNGO-Delivery · X-YNGO-Timestamp
           X-YNGO-Signature: HMAC-SHA256(timestamp + "." + body, endpoint secret)
Body       { event, occurred_at, tenant_id, data: { <minimal fields> }, request_id }
Retries    exponential backoff with jitter; attempts recorded; dead-letter visible
Security   secret stored hashed; replay rejected outside the timestamp window;
           endpoint auto-disabled after repeated failure with an admin notification
```

## 7. Inbound Integration Rules

1. Inbound integrations authenticate with an API key or signed request; they never
   receive a user session.
2. Inbound writes follow the normal validated API path — no side-door tables.
3. Rate limits and idempotency keys are mandatory on inbound write endpoints.
4. An inbound request can only act within the tenant and permissions its credential
   grants.
5. Imported data passes the same validation as UI-entered data, and import failures
   are reported per row (`BR-071`).

## 8. Adding a New Integration

```text
1  Identify the port the capability belongs to (add a port only if none fits)
2  Record an ADR if it changes architecture, data flow, or an external assumption
3  Implement the adapter behind the port; keep provider SDKs inside the adapter
4  Add configuration keys to .env.example with safe placeholders and comments
5  Encrypt credentials at rest; never echo them in responses or logs
6  Implement failure isolation (timeouts, retries, backoff, dead-letter)
7  Surface health/last-success in the admin UI
8  Add tests: adapter unit tests, failure-path tests, tenant-scoping tests on config
9  Update integration docs and the adapter catalogue above
```

## Related

- Root architecture: `../../ARCHITECTURE.md` §10 · Ports: `../07-backend/services.md`
- Security: `../11-security/security.md` · Privacy: `../11-security/privacy.md`
- Deploy/env contract: `../10-devops/local-development.md`

*End of docs/08-integrations/integrations.md*