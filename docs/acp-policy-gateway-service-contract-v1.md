# ACP Policy Gateway Service Contract v1

## Scope

Defines the implementation contract for the ACP Policy Gateway service used by PEPs to classify intents and obtain deterministic policy decisions.

## Decision Contract

### Endpoint
- `POST /v1/policy/evaluate`

### Request (required fields)
```json
{
  "request_id": "uuid",
  "tenant_id": "string",
  "agent_id": "string",
  "trace_id": "string",
  "run_id": "string",
  "intent": {
    "action": "string",
    "resource": "string",
    "risk_hints": ["string"]
  },
  "policy_context": {
    "environment": "dev|stg|prod",
    "actor_type": "human|agent|system",
    "claims": {"key": "value"}
  },
  "idempotency_key": "string",
  "nonce": "string",
  "exp": "unix_epoch_seconds"
}
```

### Response (deterministic)
```json
{
  "request_id": "uuid",
  "policy_decision_id": "uuid",
  "decision": "ALLOW|DENY|APPROVAL_REQUIRED",
  "action_class": "string",
  "risk_class": "LOW|MEDIUM|HIGH|CRITICAL",
  "lane": "autonomous|approval|blocked",
  "approval_scope": "none|single|dual",
  "obligations": ["string"],
  "reasons": ["string"],
  "required_evidence": ["string"],
  "ttl_seconds": 300
}
```

Determinism rules:
- Same normalized input + active policy version returns same `decision`, `risk_class`, `lane`, and `approval_scope`.
- Every response must include `policy_decision_id` and at least one `reason`.

## Flow Contract

1. PEP normalizes inbound intent.
2. Gateway validates schema + token/expiry + replay guard.
3. PDP evaluation executes against policy bundle.
4. Gateway returns deterministic decision payload.
5. PEP either executes, blocks, or routes to approval broker.

## Failure/Blocker Handling

- `400 INVALID_REQUEST`: schema validation failure.
- `401 UNAUTHORIZED`: identity/token validation failed.
- `409 REPLAY_DETECTED`: duplicate `idempotency_key` with mismatched payload.
- `422 POLICY_INPUT_INCOMPLETE`: required policy fields missing.
- `503 POLICY_BACKEND_UNAVAILABLE`: fail closed (`DENY`) at PEP.

Operational blocker rule:
- If PDP backend is unavailable for > configured timeout budget, service returns `503`; callers must not bypass policy.

## Evidence/Audit Requirements

Emit immutable audit events for:
- `policy.evaluate.requested`
- `policy.evaluate.decided`
- `policy.evaluate.failed`

Each event must include:
- `trace_id`, `run_id`, `tenant_id`, `agent_id`, `request_id`, `policy_decision_id`, `timestamp`, `policy_version`.
