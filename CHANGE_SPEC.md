# Change Spec v1.0

This spec defines the change description that bridges intent to execution.
It is human-first and does NOT need to be machine-readable. Agents convert
it into TODOs or implementation plans.

---

## Required Sections

```
# Change

## Targets (Mandatory)
List what is being changed.
- targetType: workflow | entity-schema | ui | api | database | integration | ai
- targetName: the specific thing (e.g., request, approval, fulfillment)

## Current State (Mandatory)
Describe the current behavior per tech stack:
- Next.js (UI)
- API (routes/handlers)
- Postgres (tables/columns)
- AI (if applicable)
- Integrations (if applicable)

## Expected State (Mandatory)
Describe the expected behavior per tech stack:
- Next.js (UI)
- API (routes/handlers)
- Postgres (tables/columns)
- AI (if applicable)
- Integrations (if applicable)
```

---

## Recommended Sections

- Risks and rollback
- Plan vs Accept mode (risk gating)
- Validation notes (how to verify)

---

## Example

```markdown
# Change

## Targets
- targetType: workflow
  targetName: request
- targetType: api
  targetName: request-approval

## Current State
- Next.js: Request detail page shows "Fulfill" for any status.
- API: POST /request/fulfill allows fulfillment without approval.
- Postgres: request.workflow_state has no "approved" gate.
- AI: no checks for approval status.

## Expected State
- Next.js: "Fulfill" action is disabled until status is "approved".
- API: POST /request/fulfill rejects if not approved.
- Postgres: request.workflow_state must be "approved" before fulfill.
- AI: suggests approval step when users try to fulfill early.
```

