# ACP Audit Propagation Contract v1

## Scope

Defines canonical event propagation and correlation requirements across ACP policy, approval, and execution hops.

## Flow Contract

Required IDs on every ACP hop:
- `trace_id`
- `run_id`
- `policy_decision_id`
- `approval_id` (when approval path is used)
- `event_id`

Required context fields:
- `tenant_id`
- `agent_id`
- `action`
- `environment`
- `timestamp`

Canonical event order (high-risk path):
1. `intent.received`
2. `policy.evaluate.requested`
3. `policy.evaluate.decided`
4. `approval.created`
5. `approval.reviewed`
6. `approval.resumed`
7. `action.executed` or `action.blocked`
8. `audit.committed`

## Decision Contract

- If decision is `ALLOW`, `approval_id` may be null.
- If decision is `APPROVAL_REQUIRED`, `approval_id` is mandatory before execution.
- If decision is `DENY`, execution events are prohibited; `action.blocked` is required.

## Failure/Blocker Handling

Failure events are first-class and must be emitted for:
- policy backend failures,
- approval queue failures,
- execution failures,
- audit sink failures.

If audit sink write fails:
- mark flow `BLOCKED_AUDIT_WRITE_FAILED`,
- enqueue replay with idempotent `event_id`,
- prevent irreversible side effects until audit write is acknowledged.

## Evidence/Audit Requirements

Immutability requirements:
- Append-only audit log storage.
- No in-place mutation of committed events.
- Hash chain or equivalent integrity proof per event batch.

Retention requirements:
- Control-plane events: minimum 365 days hot retention.
- Compliance archive: policy-defined cold retention target.

Validation requirements:
- 100% of high-risk flows must contain all required IDs.
- Missing correlation IDs fail release promotion checks.
