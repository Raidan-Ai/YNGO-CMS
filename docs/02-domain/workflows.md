# docs/02-domain/workflows.md

> **Status:** Current (state machines `PLANNED`) | **Owner:** Lead Architect
> **Last Updated:** 2026-10-02 | **Source:** Engineering_Documentation_Package_v2.md
> **Depends on:** `domain-model.md`, `business-rules.md`
> **Rule:** status changes happen only through defined transitions (BR-021).

## 1. Workflow Engine Model

```text
WorkflowDefinition ──1:N──▶ WorkflowVersion ──1:N──▶ WorkflowStep
                                        │                │
                                        │                ├── WorkflowCondition
                                        │                └── WorkflowTransition
                                        │
                             WorkflowInstance (pins one version)
                                        │
                                        ├── WorkflowTask   (human action)
                                        ├── Approval       (approve / reject)
                                        └── Escalation     (SLA timer fired)
```

**Guarantees**
- A `WorkflowInstance` records the `WorkflowVersion` it started with; publishing a
  new version never mutates in-flight instances (BR-080).
- Every transition writes an audit record with actor, from-state, to-state, and reason.
- Conditions are evaluated server-side against the entity's current state and the
  actor's permissions — never trusted from the client.
- SLA timers run in `apps/worker` and are idempotent.

## 2. Content Publishing Workflow

```text
Draft ──submit──▶ InReview ──requestChanges──▶ ChangesRequested
                     │                                  │
                     │                           resubmit│
                     │                                  ▼
                     │◀──────────────────────────── InReview
                     ├──approve──▶ Approved ──publish──▶ Published
                     │                              (or Scheduled → Published)
                     └──reject───▶ Rejected          Published ──unpublish──▶ Unpublished
                                                                 │
                                                            archive ──▶ Archived
```

| Transition | Permission | Guard |
|-----------|-----------|-------|
| submit | `content:update` | author or editor of the item |
| approve / reject | `content:approve` | reviewer ≠ author where four-eyes is configured |
| publish / unpublish / schedule | `content:publish` | requires status `Approved` (BR-020) |
| archive | `content:archive` | any non-published state or after unpublish |

## 3. Project Lifecycle

```text
Draft ──▶ Planned ──▶ Active ──▶ Suspended ──▶ Active
                        │
                        ├──▶ Completed   (guard: no open mandatory milestones, BR-033)
                        └──▶ Cancelled
                                 Completed ──▶ Closed
```

## 4. Grant Lifecycle

```text
Identified ──▶ Applied ──▶ Awarded ──▶ Active ──▶ Reporting ──▶ Closed
                                 │                    │
                             amendment            report submitted
                                 ▼                    (guard: milestones complete, BR-040)
                          GrantAmendment
```

## 5. Case Management Lifecycle

```text
Opened ──▶ Assessment ──▶ Active ──▶ Referral / FollowUp ──▶ Resolved ──▶ Closed
```
Restricted workflow: every transition captures actor + reason and is audited;
visits to the record are audited (BR-060/BR-061).

## 6. Safeguarding Lifecycle (separate audit stream)

```text
Reported ──▶ Triage ──▶ Investigating ──▶ Actioned ──▶ Closed
```
- Only `safeguarding:*` roles may read/write.
- Excluded from generic search, activity feeds, and generic exports (BR-062).
- Exports require elevated permission + justification (BR-064).

## 7. Complaints / Feedback Lifecycle

```text
Received ──▶ Triage ──▶ Investigating ──▶ Actioned ──▶ Resolved ──▶ Closed
                                      └──▶ Escalated ──▶ Investigating
```
Anonymous intake is allowed; anonymity is preserved in audit (actor recorded as
`anonymous`, with the intake channel, not the identity).

## 8. Approval & Escalation Model

| Concept | Behaviour |
|---------|-----------|
| Approval | `Pending → Approved / Rejected / Delegated / Escalated / TimedOut` |
| Delegation | Time-bounded, audited, and cannot delegate beyond the delegator's permissions |
| Escalation | Fired by SLA timer in the worker; escalates to a configured role/assignee |
| Timeout | Recorded explicitly — never silently auto-approved for sensitive flows |
| Four-eyes | Configurable for content approval, safeguarding, and payment-adjacent flows |

## 9. Automation Model

```text
Trigger (entity event / schedule / webhook)
   → Conditions (state, permission, tenant, time)
     → Actions (notify, assign, transition, create record, call webhook)
        → AutomationExecution + AutomationLog (always recorded)
```
Limits: automations run with the configuring actor's permission set (BR-082), are
rate-limited, and cannot perform destructive operations without a configured
approval step.

## 10. Status Change Enforcement

1. Modules exposing status fields MUST route changes through the workflow service.
2. Direct `PATCH status` in generic update endpoints is rejected with
   `WORKFLOW_TRANSITION_INVALID` (`../05-api/errors.md`).
3. Transition coverage is proven per module by tests that attempt an illegal
   direct status write and expect a refusal (R-16).

## Related

- Rules: `business-rules.md` · Entities: `entities.md`
- Error codes: `../05-api/errors.md` · Phase 7 plan: `../../IMPLEMENTATION_PLAN.md`

*End of docs/02-domain/workflows.md*