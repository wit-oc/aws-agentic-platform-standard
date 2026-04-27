# ACP Intent Envelope Contract v1

## Scope

Defines the minimum deterministic contract for ACP classification and policy decisions before agent or tool side effects.

## Decision Contract

## Intent Envelope (request)

```json
{
  "intent_id": "uuid",
  "agent_id": "domain-agent",
  "tenant_id": "tenant-123",
  "trace_id": "trace-uuid",
  "run_id": "run-uuid",
  "policy_context": {
    "environment": "dev|stg|prod",
    "quiet_hours": false,
    "incident_mode": false
  },
  "action": {
    "action_class": "AC0|AC1|AC2|AC3|AC4|AC5",
    "target": "resource identifier",
    "surface": "internal|external_trusted|external_public",
    "auth_level": "none|standard|high_privilege",
    "data_sensitivity": "public|internal|sensitive",
    "blast_radius": "single|small_batch|broad|unknown",
    "reversibility": "easy|partial|irreversible",
    "governance_impact": "none|indirect|direct"
  },
  "handoff_security": {
    "nonce": "uuid",
    "exp": 1731000000
  }
}
```

## Policy Decision (response)

```json
{
  "decision": "ALLOW|DENY|APPROVAL_REQUIRED",
  "policy_decision_id": "pd-uuid",
  "risk_class": "R0|R1|R2|R3|R4",
  "lane": "auto|queued|manual",
  "approval_required": true,
  "approval_scope": "none|approver|owner",
  "classification_reasons": ["rule_id"],
  "evidence_requirements": ["list of required evidence"],
  "obligations": {
    "max_tool_scope": "ticket.write",
    "max_budget_tokens": 8000,
    "redaction_profile": "sensitive-default"
  }
}
```

Deterministic rules:
- `governance_impact=direct` => `R4`.
- irreversible destructive + broad/unknown blast radius => `>=R3`.
- unknown mandatory fields => elevate at least one class; never below `R2`.
- denylist beats any lane.

## Failure/Blocker Handling

- Missing `trace_id`, `tenant_id`, `nonce`, or expired `exp` => hard `DENY`.
- Policy engine timeout => fail closed (`DENY`) for `R2+`; queue retry for `R0/R1` if policy allows.
- Nonce replay detected => `DENY`, emit security event.

## Evidence/Audit Requirements

For every decision, persist:
- `intent_id`, `trace_id`, `run_id`, `policy_decision_id`,
- evaluated risk/lane and rule ids,
- approval id if decision was `APPROVAL_REQUIRED`,
- final execution disposition (`executed|canceled|expired`).
