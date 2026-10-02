# ADR-0009 — Object Storage

> **Status:** ACCEPTED | **Owner:** Lead Architect | **Date:** 2026-10-02
> **Canonical rationale:** `../../TECHNICAL_DECISIONS.md` (ADR-009)
> **Decides:** `../../OPEN_QUESTIONS.md` Q-06 | **Blocks:** media, documents, uploads, exports

## Context

Media/DAM, Documents/DMS, form submission attachments, avatars, report exports, and
import files all need durable object storage. The reference deployment is a
self-hosted Linux server, and deployments in the target region may have limited
external connectivity; at the same time, larger deployments may prefer a cloud
provider. Vendor lock-in must be avoided (R-14).

## Options

| Option | Pros | Cons |
|--------|------|------|
| A. MinIO self-hosted (S3 API) | Runs in compose, offline-capable, identical API to cloud S3 | Operator owns durability/backup |
| B. AWS S3 / Azure Blob | Managed durability | Connectivity/cost/legal exposure; needs different adapters |
| C. Local filesystem only | Simplest | Not portable, breaks horizontal scaling, no signed URLs |

## Decision

**MinIO (S3-compatible) is the v1 reference implementation behind a `StoragePort`
interface. Cloud S3/Azure Blob adapters are supported by the same port and may be
selected by configuration.**

## Mandatory Rules

1. No application code may call an S3 SDK directly — only through `StoragePort`.
2. Object keys are tenant-prefixed and normalised: `t/<tenant_id>/<module>/<yyyy>/<mm>/<uuid>`.
   Original filenames are metadata, never paths.
3. Signed, time-limited URLs are used for downloads; buckets stay private by default.
4. Uploads enforce allow-listed MIME + magic-byte verification, size limits, and a
   quarantine step before an object is considered usable (R-07).
5. Deletion is soft for metadata; object lifecycle/retention follows the module's
   retention policy (`../11-security/privacy.md`).
6. Metadata is committed only after the object is durably stored; no dangling rows.
7. Backup/restore covers the bucket as well as the database
   (`../10-devops/backup-recovery.md`).

## Consequences

- Positive: offline-capable reference deployment, swappable provider, one port.
- Negative: MinIO durability is the operator's responsibility (documented drill).
- Follow-up: storage adapter contract documented in `../07-backend/services.md`.

## Related

- Media/Documents: TASK-023, TASK-024 · Ports: `../07-backend/services.md`
- Security: `../11-security/security.md` §uploads · Risk: R-07, R-14

*End of ADR-0009*
