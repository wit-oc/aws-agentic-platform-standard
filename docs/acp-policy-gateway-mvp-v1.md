# ACP Policy Gateway MVP v1

## Scope

Defines the minimal policy gateway (PEP + PDP integration) to control A2A and tool-bound actions with deterministic approval routing.

## Flow Contract

1. Caller submits Intent Envelope to gateway.
2. PEP validates identity, signature, tenant boundary, nonce/TTL.
3. PEP sends normalized context to PDP.
4. PDP returns `ALLOW|DENY|APPROVAL_REQUIRED` + obligations.
5. If `ALLOW`, gateway forwards invocation with bound obligations.
6. If `APPROVAL_REQUIRED`, gateway creates approval task and pauses/resumes only with signed approval token.
7. Emit audit event for request, decision, and final execution result.

## AWS Mapping (MVP)

- Ingress: API Gateway.
- Policy Gateway/PEP: Lambda or ECS service.
- PDP: policy service (Cedar-style or Verified Permissions backed adapter).
- Approval broker: Step Functions + DynamoDB + EventBridge/SQS.
- Audit stream: append-only S3 evidence objects plus CloudWatch operational telemetry.

## Decision Contract

Gateway request:
- Intent Envelope v1 (from `acp-intent-envelope-contract-v1.md`).

Gateway response:
- policy decision payload + status:
  - `allowed` with obligations,
  - `denied` with reason/rule ids,
  - `pending_approval` with `approval_request_id`.

## Failure/Blocker Handling

- PDP unavailable: fail closed for `R2+`; optional bounded retry for `R0/R1`.
- Approval timeout: action canceled, emit `approval_expired`.
- Resume token mismatch/expired: reject resume, require new approval.
- Duplicate intent id: treat as idempotent lookup; never execute twice.

## Evidence/Audit Requirements

Must record immutable events for:
- `intent_received`,
- `policy_decided`,
- `approval_requested` (if required),
- `approval_granted|rejected|expired` (if required),
- `execution_forwarded|execution_blocked|execution_completed`.

Required correlation fields on every event:
- `trace_id`, `run_id`, `intent_id`, `policy_decision_id`, `tenant_id`, timestamp.
