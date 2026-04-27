# ACP Implementation Execution Plan v1

## Scope

Execution plan for the ACP implementation phase (runtime service wiring), building on completed MVP spec artifacts.

Primary references:
- `aws-agentic-platform-standard/docs/acp-exec-plan-v1.md`
- `aws-agentic-platform-standard/docs/acp-intent-envelope-contract-v1.md`
- `aws-agentic-platform-standard/docs/acp-policy-gateway-mvp-v1.md`
- `aws-agentic-platform-standard/docs/acp-approval-audit-e2e-v1.md`
- `aws-agentic-platform-standard/docs/agentcore-prototype-spec-v1.md`

## Outcome Target

Deliver implementation-ready ACP runtime contracts and operator runbooks that can be turned into code with minimal ambiguity:
- policy gateway service contract,
- approval broker API contract,
- audit propagation requirements,
- deployment and validation workflow for `dev -> stg -> prod`.

## Implementation Action Items

1. T6: Policy Gateway Service Contract
- Output: `aws-agentic-platform-standard/docs/acp-policy-gateway-service-contract-v1.md`
- Must include:
  - request/response JSON schema for policy evaluation,
  - deterministic decision mapping (`ALLOW`, `DENY`, `APPROVAL_REQUIRED`),
  - idempotency and replay fields,
  - failure response model.

2. T7: Approval Broker API + Resume Contract
- Output: `aws-agentic-platform-standard/docs/acp-approval-broker-api-contract-v1.md`
- Must include:
  - approval request payload and evidence bundle shape,
  - reviewer decision payload and signature metadata,
  - resume token contract for paused workflows,
  - timeout/escalation behavior.

3. T8: Audit/Event Propagation Contract
- Output: `aws-agentic-platform-standard/docs/acp-audit-propagation-contract-v1.md`
- Must include:
  - required IDs on every hop (`trace_id`, `run_id`, `policy_decision_id`, `approval_id`),
  - canonical event types and required fields,
  - retention and immutability expectations,
  - failure/blocker event requirements.

4. T9: Implementation Validation Matrix
- Output: `aws-agentic-platform-standard/docs/acp-implementation-validation-matrix-v1.md`
- Must include:
  - contract tests (policy decisions, approval-required flow, denied flow),
  - replay/idempotency tests,
  - audit completeness checks,
  - promotion gate pass/fail criteria.

5. T10: Pilot Build-Out Runbook
- Output: `aws-agentic-platform-standard/docs/acp-pilot-buildout-runbook-v1.md`
- Must include:
  - stepwise deployment order for dev/stg/prod,
  - rollback triggers and rollback steps,
  - operator checklists per environment,
  - go/no-go criteria for production pilot.

## Validation Gate

Release gate passes only when all checks below are true:

- Required T6..T10 files exist.
- Each deliverable contains required headings:
  - `Scope`
  - `Decision Contract` or `Flow Contract`
  - `Failure/Blocker Handling`
  - `Evidence/Audit Requirements`
- `TASK_BOARD.md` status and references match produced artifacts.
- Initiative artifact includes proof-first `ARTIFACT` lines for each completed task.

Reference validation commands:

```bash
cd /Users/wit/.openclaw/workspace
test -f aws-agentic-platform-standard/docs/acp-policy-gateway-service-contract-v1.md
test -f aws-agentic-platform-standard/docs/acp-approval-broker-api-contract-v1.md
test -f aws-agentic-platform-standard/docs/acp-audit-propagation-contract-v1.md
test -f aws-agentic-platform-standard/docs/acp-implementation-validation-matrix-v1.md
test -f aws-agentic-platform-standard/docs/acp-pilot-buildout-runbook-v1.md
rg -n "acp-.*(service-contract|api-contract|propagation-contract|validation-matrix|buildout-runbook).*\.md" aws-agentic-platform-standard/TASK_BOARD.md
```

## Deployment/Runbook Step

Operational execution runs through initiative runner artifact:
- `initiatives/aws-agentic-platform-acp-implementation-2026-03-08.md`

Operator runbook contract:
- `docs/internal/INITIATIVE_RUNNER.md`

Runner outcomes must remain one-line deterministic:
- `ARTIFACT: <concrete delta>`
- `BLOCKED: <reason> | NEXT: <smallest unblocked step>`
