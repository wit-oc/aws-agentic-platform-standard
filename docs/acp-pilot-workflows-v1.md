# ACP Pilot Workflows v1

## Scope

Defines the MVP pilot pack with one low-risk and one high-risk workflow using the ACP classification and approval model.

## Flow Contract

### Workflow A (Low risk)
- Name: internal status digest post to approved ops channel
- Action class: `AC1`
- Risk: `R1`
- Lane: `auto`
- Required controls:
  - allowlisted destination,
  - trace/audit receipt.

### Workflow B (High risk)
- Name: external account profile update
- Action class: `AC3`
- Risk: `R3`
- Lane: `manual`
- Required controls:
  - explicit approval,
  - rollback plan,
  - immutable evidence chain.

## Decision Contract

For both workflows, policy output must include:
- `action_class`,
- `risk_class`,
- `lane`,
- `approval_required`,
- `classification_reasons[]`,
- `evidence_requirements[]`.

Exit criteria:
- Workflow A completes in `auto` path with deterministic audit fields.
- Workflow B blocks without approval and resumes successfully only with valid approval token.

## Failure/Blocker Handling

- Missing workflow context => elevate class and block for operator input.
- Approval SLA missed on Workflow B => close as `approval_expired`.
- Audit field missing => validation failure; workflow cannot be marked complete.

## Evidence/Audit Requirements

For each pilot run:
- policy decision artifact,
- execution result artifact,
- audit event chain with `trace_id/run_id/policy_decision_id`,
- rollback evidence (high-risk workflow only).
