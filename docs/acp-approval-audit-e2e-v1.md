# ACP Approval + Audit End-to-End v1

## Scope

Defines one high-risk (`R3`) workflow path from intent submission through approval and immutable audit completion.

## Flow Contract

Representative action:
- `account.update` on external system with broad customer impact.

Path:
1. Agent emits Intent Envelope (`AC3`, candidate `R3`).
2. PEP validates identity/tenant/replay fields and forwards to PDP.
3. PDP returns `APPROVAL_REQUIRED` + evidence requirements.
4. Approval task created with full context card:
  - target system/object,
  - change summary,
  - blast radius estimate,
  - rollback strategy.
5. Authorized approver signs decision (`approve` or `reject`).
6. If approved, gateway issues short-lived resume token bound to:
  - `intent_id`,
  - `policy_decision_id`,
  - `approval_id`,
  - expiry.
7. Execution proceeds once, with idempotency key.
8. Final disposition emitted to immutable audit stream.

## Decision Contract

Approval record:
- `approval_id`
- `approval_scope` (`approver` or `owner`)
- `decision` (`approved|rejected`)
- `approver_identity`
- `expires_at`
- `required_evidence[]`

Execution record:
- `execution_id`
- `intent_id`
- `policy_decision_id`
- `approval_id` (nullable only when no approval needed)
- `result` (`completed|failed|canceled`)

## Failure/Blocker Handling

- Approval rejected => final `canceled` disposition.
- Approval expired => new approval required.
- Resume attempted without valid token => hard block.
- Side-effect API timeout => bounded retry with same idempotency key; if exhausted, final `failed` with retry status.

## Evidence/Audit Requirements

Immutable event chain (minimum):
1. `intent_received`
2. `policy_decided`
3. `approval_requested`
4. `approval_decided`
5. `execution_started`
6. `execution_completed|execution_failed|execution_canceled`

Required fields on each event:
- `trace_id`
- `run_id`
- `intent_id`
- `policy_decision_id`
- `approval_id` (if applicable)
- `actor_id`
- timestamp
