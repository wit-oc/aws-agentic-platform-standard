# ACP Approval Broker API Contract v1

## Scope

Defines the implementation contract for creating, deciding, and resuming approval-required ACP actions.

## Flow Contract

### Endpoint: create approval
- `POST /v1/approvals`

Request:
```json
{
  "approval_request_id": "uuid",
  "tenant_id": "string",
  "trace_id": "string",
  "run_id": "string",
  "policy_decision_id": "uuid",
  "action_summary": "string",
  "risk_class": "HIGH|CRITICAL",
  "approval_scope": "single|dual",
  "evidence_bundle": {
    "intent_snapshot": {},
    "policy_reasons": ["string"],
    "required_evidence": ["string"]
  },
  "expires_at": "ISO8601"
}
```

Response:
```json
{
  "approval_id": "uuid",
  "status": "PENDING",
  "review_url": "string",
  "resume_token": "string",
  "expires_at": "ISO8601"
}
```

### Endpoint: decision submission
- `POST /v1/approvals/{approval_id}/decision`

Request:
```json
{
  "decision": "APPROVE|REJECT",
  "reviewer_id": "string",
  "reviewer_signature": "string",
  "reason": "string",
  "decided_at": "ISO8601"
}
```

Response:
```json
{
  "approval_id": "uuid",
  "status": "APPROVED|REJECTED",
  "decision_id": "uuid"
}
```

### Endpoint: workflow resume
- `POST /v1/approvals/{approval_id}/resume`

Request:
```json
{
  "resume_token": "string",
  "run_id": "string",
  "trace_id": "string"
}
```

Response:
```json
{
  "approval_id": "uuid",
  "resume_status": "RESUMED|DENIED|EXPIRED"
}
```

## Decision Contract

- `APPROVE` resumes paused workflow.
- `REJECT` terminates workflow with policy denial outcome.
- Expired approvals must not be resumable.

## Failure/Blocker Handling

- `404 APPROVAL_NOT_FOUND`
- `409 APPROVAL_ALREADY_DECIDED`
- `410 APPROVAL_EXPIRED`
- `401 INVALID_REVIEWER_SIGNATURE`
- `503 APPROVAL_QUEUE_UNAVAILABLE`

Timeout/escalation:
- If `PENDING` exceeds SLA, emit escalation event and notify fallback reviewer group.
- If still unresolved at `expires_at`, auto-mark `EXPIRED` and block action.

## Evidence/Audit Requirements

Required events:
- `approval.created`
- `approval.reviewed`
- `approval.resumed`
- `approval.expired`

All events must include:
- `trace_id`, `run_id`, `approval_id`, `policy_decision_id`, `tenant_id`, `reviewer_id` (if present), `timestamp`.
