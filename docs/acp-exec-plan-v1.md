# ACP Execution Plan v1

## Scope

Implementation plan for the AWS Agentic Platform baseline, aligned to existing control-plane guidance and the current AWS docs set.

Primary references:
- `aws-agentic-platform-standard/docs/aws-agentic-platform-architecture.md`
- `aws-agentic-platform-standard/docs/deployment-blueprint-v1.md`
- `aws-agentic-platform-standard/docs/agentcore-capabilities-implementation-guide.md`
- `aws-agentic-platform-standard/docs/agentcore-prototype-spec-v1.md`

## Outcome Target

Deliver a publish-ready ACP MVP definition that is implementation-credible for:
- policy-gated A2A routing,
- approval-required high-risk actions,
- immutable audit correlation,
- AgentCore/application-runtime pilot with AWS runtime classes selected by workload shape.

## Sprint Deliverables (Top Execution Tasks)

1. Intent + Decision Contract (T1)
- Output: `aws-agentic-platform-standard/docs/acp-intent-envelope-contract-v1.md`
- Must include:
  - intent envelope schema (`agent_id`, `tenant_id`, `trace_id`, `policy_context`, `nonce`, `exp`, action intent),
  - deterministic policy response contract (`ALLOW`, `DENY`, `APPROVAL_REQUIRED`),
  - required evaluator output fields (`action_class`, `risk_class`, `lane`, `approval_scope`, reasons, evidence requirements).

2. Policy Gateway MVP Design (T2)
- Output: `aws-agentic-platform-standard/docs/acp-policy-gateway-mvp-v1.md`
- Must include:
  - PEP/PDP interaction sequence,
  - hard-elevation/default-deny rules,
  - approval broker hook and resume token contract,
  - replay/idempotency handling,
  - AWS mapping (API Gateway plus AgentCore Runtime/Lambda/ECS as appropriate, Step Functions for durable checkpoints, EventBridge/SQS, Verified Permissions/Cedar-style policy service).

3. Approval + Audit E2E Path (T3)
- Output: `aws-agentic-platform-standard/docs/acp-approval-audit-e2e-v1.md`
- Must include:
  - one representative high-risk action walkthrough,
  - approval evidence bundle,
  - immutable audit event list,
  - correlation model (`trace_id`, `run_id`, `policy_decision_id`, approval id).

## Extended Build Tasks

4. IaC + Promotion Gate Map (T4)
- Output: `aws-agentic-platform-standard/docs/acp-iac-promotion-gates-v1.md`

5. Pilot Workflow Pack (T5)
- Output: `aws-agentic-platform-standard/docs/acp-pilot-workflows-v1.md`

## Validation Gate (Doc/Test Contract)

Release gate passes only when all checks below are true:

- Required deliverable files (T1..T5) exist.
- Each deliverable contains required headings:
  - `Scope`
  - `Decision Contract` or `Flow Contract`
  - `Failure/Blocker Handling`
  - `Evidence/Audit Requirements`
- `TASK_BOARD.md` status and references match actual files.
- Initiative artifact contains at least one concrete proof entry per completed task.

Reference validation commands:

```bash
cd /Users/wit/.openclaw/workspace
test -f aws-agentic-platform-standard/docs/acp-intent-envelope-contract-v1.md
test -f aws-agentic-platform-standard/docs/acp-policy-gateway-mvp-v1.md
test -f aws-agentic-platform-standard/docs/acp-approval-audit-e2e-v1.md
rg -n "^## (Scope|Decision Contract|Flow Contract|Failure/Blocker Handling|Evidence/Audit Requirements)" \
  aws-agentic-platform-standard/docs/acp-*.md
rg -n "acp-.*\\.md" aws-agentic-platform-standard/TASK_BOARD.md
```

## Deployment/Runbook Step

Operational execution runs through initiative runner artifact:
- `initiatives/aws-agentic-platform-acp-mvp-2026-03-07.md`

Operator runbook contract:
- `docs/internal/INITIATIVE_RUNNER.md`

Runner outcomes must remain one-line deterministic:
- `ARTIFACT: <concrete delta>`
- `BLOCKED: <reason> | NEXT: <smallest unblocked step>`

## Risks

- AgentCore feature ambiguity can create control-plane ownership drift.
- Approval latency can degrade end-to-end workflow SLAs.
- Audit completeness can fail if IDs are not propagated at every hop.

## Mitigations

- Keep governance-spine APIs authoritative even if AgentCore or iPaaS wrappers evolve.
- Treat approval broker as first-class dependency in the execution path.
- Enforce propagation of `trace_id/run_id/policy_decision_id` in all contracts.
