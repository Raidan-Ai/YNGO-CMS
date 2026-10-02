# docs/09-testing/acceptance-criteria.md

> **Status:** Current (criteria) — none verified yet | **Owner:** QA Lead
> **Last Updated:** 2026-10-02 | **Source:** `docs/01-product/requirements.md`,
> `IMPLEMENTATION_PLAN.md`, `AGENTS.md` §12
> **Rule:** a criterion is `VERIFIED` only when its automated test exists and passes
> in CI. Until then it is `PLANNED`.

## 1. How to Read

```text
Requirement (FR/NFR)  →  Phase  →  Criterion (Given/When/Then)  →  Test file
```

Every criterion below is written so it becomes an executable test directly
(`docs/09-testing/strategy.md` §"Acceptance Criteria Convention").

## 2. Phase 0 — Foundation

| # | Criterion | Test type |
|---|-----------|-----------|
| P0-1 | Given a clean checkout, when `docker compose up` runs, then all services report healthy and `/health` and `/ready` return 200 | integration |
| P0-2 | Given the repo, when `pnpm lint && pnpm typecheck && pnpm test` run in CI, then all pass | CI |
| P0-3 | Given two tenants A and B, when a user of A requests an entity owned by B by id, then the response is 404 and no data is returned | tenant negative |
| P0-4 | Given an empty database, when `pnpm bootstrap:org` runs with supplied values, then tenant + organization + admin exist and an audit record is written | integration |
| P0-5 | Given an existing tenant, when bootstrap runs again without a token, then it fails and creates nothing | integration |
| P0-6 | Given the request pipeline, when a request without tenant context hits a tenant-scoped route, then it is rejected with `TENANT_REQUIRED` | authz |
| P0-7 | Given the source tree, when the CI scan runs, then no `mockData|demoData|sampleData|fakeData|dummyData` or `admin/admin` pattern exists outside `tests/` | CI scan |
| P0-8 | Given a mutation, when it succeeds, then an append-only audit row exists with actor, entity, before/after, and `request_id` | integration |

## 3. Phase 1 — Core, Identity, Access

| # | Criterion | Test type |
|---|-----------|-----------|
| P1-1 | Given a protected route, when the caller lacks its permission, then 403 `PERMISSION_DENIED` is returned | authz |
| P1-2 | Given a record outside the caller's ABAC scope, when it is requested, then it is invisible and non-mutable | authz |
| P1-3 | Given a login, when it succeeds, then the refresh token rotates and the previous token is rejected on replay | API |
| P1-4 | Given an invitation, when it is used twice or after expiry, then it is rejected | API |
| P1-5 | Given an API key creation, when the response is returned, then the raw secret appears once and is unresolvable afterwards | API |
| P1-6 | Given a revoked API key, when it is used, then the request fails immediately | API |
| P1-7 | Given a suspended user, when they attempt to authenticate or refresh, then both fail | API |
| P1-8 | Given audit grants, when an `UPDATE`/`DELETE` is attempted on `audit.audit_log`, then the database refuses it | integration |
| P1-9 | Given every Phase 1 handler, when the permission matrix is evaluated, then each declares at least one required permission (deny-by-default) | static + authz |

## 4. Phase 2 — Content & Media

| # | Criterion | Test type |
|---|-----------|-----------|
| P2-1 | Given a draft, when publish is requested without an approved revision, then it is refused (`WORKFLOW_TRANSITION_INVALID`, BR-020) | API + workflow |
| P2-2 | Given a published item, when its status is edited via generic PATCH, then it is rejected | workflow |
| P2-3 | Given an upload with a forged MIME type, when it is submitted, then it is rejected with `UPLOAD_TYPE_NOT_ALLOWED` | API |
| P2-4 | Given an oversized upload, when it is submitted, then it is rejected with `UPLOAD_TOO_LARGE` | API |
| P2-5 | Given a stored asset, when a download is requested, then only a time-limited signed URL is issued and the object path is never public | API |
| P2-6 | Given tenant B's content, when tenant A searches a matching term, then nothing from B is returned | tenant negative |
| P2-7 | Given a restricted document, when public search runs, then it never appears | authz + search |
| P2-8 | Given two concurrent editors on one item, when the second saves, then a lock conflict is reported rather than a silent overwrite | integration |
| P2-9 | Given `ar` locale, when content pages render, then no physical left/right utility is used and focus order is correct | RTL + component |

## 5. Phase 3 — Programs, Projects, Operations, Reporting

| # | Criterion | Test type |
|---|-----------|-----------|
| P3-1 | Given real data, when the program → project → milestone → indicator → report chain is followed, then every step is queryable and audited | E2E |
| P3-2 | Given an async report, when it is created, then it is durable, cancellable, permission-checked, and progress is observable | integration |
| P3-3 | Given report download, when the requester lacks permission at download time, then access is refused even though the run exists (BR-073) | authz |
| P3-4 | Given a project with open mandatory milestones, when completion is requested, then it is refused (BR-033) | API |
| P3-5 | Given a restricted project location, when the public layer is requested, then the location is absent (BR-034) | API + GIS |
| P3-6 | Given an import where some rows are invalid, when it commits, then valid rows are stored, each failure is reported per row, and no committed row is corrupted | integration |
| P3-7 | Given a webhook endpoint that keeps failing, when retries exhaust, then attempts are logged and the endpoint is disabled and visible | integration |
| P3-8 | Given an export, when it runs, then it is permission-checked, rate-limited, and audited | authz |
| P3-9 | Given a project with no measurements, when a report is produced, then it states the absence of data and fabricates nothing (BR-035) | unit + API |

## 6. Phase 6 — Sensitive Domains (highest scrutiny)

| # | Criterion | Test type |
|---|-----------|-----------|
| P6-1 | Given a user without `cases:read`, when cases are listed/searched/exported, then nothing is returned | authz + tenant negative |
| P6-2 | Given a readable case, when a restricted field is requested, then it is denied (`FIELD_FORBIDDEN`) while the record stays readable | authz |
| P6-3 | Given restricted access or export, when the reason is missing, then the request fails with `REASON_REQUIRED` | API |
| P6-4 | Given a safeguarding record, when generic search, feeds, or generic exports run, then it never appears (BR-062) | authz + search |
| P6-5 | Given safeguarding activity, when audit is inspected, then writes appear in the separate stream readable only by designated roles (BR-063) | integration |
| P6-6 | Given a sensitive export, when it is produced, then it is permission-checked, rate-limited, watermarked, and audited (BR-064) | API |
| P6-7 | Given an anonymous complaint, when audit is inspected, then the actor is recorded as `anonymous` with the channel, never the identity | integration |
| P6-8 | Given a beneficiary list, when it is displayed, then direct identifiers are masked unless the actor holds the field grant (BR-066) | API |

## 7. Cross-Cutting Criteria (every phase)

| # | Criterion | Test type |
|---|-----------|-----------|
| X-1 | Given any data surface (read, list, search, export, report, file, webhook, job, AI), when a foreign tenant id is used, then access is denied and never leaks existence | tenant negative |
| X-2 | Given any protected route, when analyzed, then it has an authorization test proving denial by default | authz |
| X-3 | Given any mutation, when it completes, then an audit record exists and traces to the same `request_id` | integration |
| X-4 | Given any list view, when rendered with no data, then an honest empty state appears with a single next action and no fabricated rows | E2E + component |
| X-5 | Given any localized surface, when rendered in `ar`, then RTL layout and logical spacing hold | RTL |
| X-6 | Given an AI-assisted step, when it produces output, then it is a draft requiring human approval and never overwrites a record (BR-090/092) | API + integration |
| X-7 | Given an error response, when inspected, then it contains no stack trace, SQL, secret, or PII | API |

## 8. Non-Functional Verification

| Requirement | Verification | Phase |
|-------------|--------------|:-----:|
| NFR-001 API p95 < 300 ms | load test at target scale | 11 |
| NFR-002 Search p95 < 500 ms | load test | 11 |
| NFR-003 99.5 % monthly availability | monitoring + alert thresholds | 8/11 |
| NFR-004 Arabic RTL + English LTR quality | RTL checks in CI + review checklist | every UI phase |
| NFR-005 TS `strict`, no `any` in domain code | lint/typecheck gate | every phase |
| NFR-006 100 % of modules have an authz negative test | CI coverage of the authz suite | every phase |
| NFR-007 WCAG 2.1 AA on core flows | accessibility audit | 11 |
| NFR-008 Structured logs, metrics, tracing, `/health`, `/ready` | observability review | 8 |
| NFR-009 Backup/restore within documented RTO/RPO | restore drill | 11 |
| NFR-010 Same images across environments | compose + env contract review | 0/11 |
| NFR-011 Argon2id password hashing, no secrets logged | security review | 11 |
| NFR-012 No fabricated data; professional empty states | ADR-013 CI scan | every phase |

## 9. Status Tracking Rule

```text
PLANNED    criterion written, no test yet
COVERED    test exists and passes in CI
VERIFIED   covered + reviewed by the phase gate evidence
FAILED     test exists and fails — the phase gate is closed until fixed
```

Do not mark any criterion `COVERED` or `VERIFIED` in this file without a passing CI
run. Do not delete or weaken a criterion to close a phase (`AGENTS.md` §5).

## Related

- Strategy and suites: `strategy.md` · Requirements: `../01-product/requirements.md`
- Plan and gates: `../../IMPLEMENTATION_PLAN.md`
- Rules: `../../AGENTS.md` §12

*End of docs/09-testing/acceptance-criteria.md*