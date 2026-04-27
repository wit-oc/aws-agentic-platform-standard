# Deployment Blueprint v1

## 0) Before You Start

- [ ] Define two candidate workflows (low-risk + high-risk)
- [ ] Assign data classification and compliance scope
- [ ] Select tenancy mode (shared / segmented / account-per-tenant)
- [ ] Identify the originating user populations and authoritative identity source
- [ ] Identify whether protected-data actions require delegated user authorization

## 1) Agent and Action-Surface Inventory

- [ ] Inventory AWS-hosted and AgentCore Runtime agents
- [ ] Inventory Workato / iPaaS / proprietary automation surfaces
- [ ] Inventory SaaS-native agent surfaces (for example Workday, Zoom, vendor-native copilots/agents)
- [ ] Inventory legacy scripts/manual runbooks that perform agent-like actions
- [ ] Assign owner, platform, environment, data domains, action surface, and support team for each entry
- [ ] Identify whether each surface supports delegated user authorization, service-account-only execution, approvals, audit export, and idempotency/replay controls

## 2) Control Tier Assignment

- [ ] Assign each action surface a control tier:
  - Tier 0: platform-owned runtime/tool gateway
  - Tier 1: mediated iPaaS/proprietary platform
  - Tier 2: SaaS-native/constrained vendor surface
  - Tier 3: legacy/manual exception
- [ ] Document gaps for Tier 2 and Tier 3 surfaces
- [ ] Define compensating controls for non-frontable SaaS-native surfaces
- [ ] Confirm that Workato/iPaaS use is technically justified per workflow, not assumed as the default gateway by organizational preference alone

## 3) Platform Foundations

- [ ] Create `shared-platform`, `dev`, `stg`, `prod` accounts where AWS assets are in scope
- [ ] Baseline IAM boundaries and SCP guardrails
- [ ] Enable KMS, Secrets Manager, CloudTrail organization trail
- [ ] Stand up or select the agent/tool/action registry
- [ ] Define request context schema: user, agent, tenant/business context, environment, trace/run IDs, delegation mode, approval state

## 4) Runtime and Orchestration

- [ ] Deploy preferred AgentCore Runtime or application/runtime pilot for one bounded-purpose domain agent
- [ ] Keep orchestration logic in application/framework code unless a provider-native construct is explicitly justified
- [ ] Add Step Functions only for durable checkpoints, approval waits, or long-running business transitions where useful
- [ ] Add EventBridge bus + SQS queues for async tasks where useful
- [ ] Implement idempotency keys + retry budgets
- [ ] Standardize inter-agent signed envelope (`agent_id`, `tenant_id`, `environment`, `trace_id`, `run_id`, `policy_context`, `delegation_mode`, `ttl`)

## 5) Model and Memory

- [ ] Bedrock model routing config (cost/latency/risk aware)
- [ ] DynamoDB or equivalent for session/run state
- [ ] S3 or equivalent for transcripts/artifacts with retention policies
- [ ] Vector memory store (OpenSearch/pgvector) + retrieval filters where needed
- [ ] Memory access rules mapped to data classification and tenant/business context

## 6) Tooling and Integrations

- [ ] Stand up Tool Gateway service or equivalent policy-enforced execution boundary for Tier 0 actions
- [ ] Register tool/action contracts: schema, auth mode, risk class, approval requirement, audit obligation
- [ ] Enforce policy checks pre/post invocation
- [ ] Add human approvals for write-high/critical actions
- [ ] For Workato/iPaaS Tier 1 actions, validate connector scopes, approval hooks, audit export, and trace correlation
- [ ] For SaaS-native Tier 2 actions, configure least-privilege OAuth/app scopes, admin controls, audit export, disablement path, and review cadence

## 7) Security and Governance

- [ ] Verify least privilege role-per-service or platform-equivalent credential boundary
- [ ] Validate delegated user authorization for protected data where downstream systems support it
- [ ] Mark service-account-only paths explicitly; do not imply user entitlement where not enforced downstream
- [ ] Configure network boundaries + private endpoints where applicable
- [ ] Enable model/content guardrails
- [ ] Set audit trail to immutable retention where required
- [ ] Enforce A2A auth via IAM/IRSA/SigV4/private endpoints or equivalent platform controls; no static shared keys
- [ ] Enable replay protection/nonces on inter-agent commands where supported

## 8) Observability and Evals

- [ ] Emit standardized telemetry (`trace_id`, `run_id`, `tenant_id`, `agent_id`, `platform`, `environment`)
- [ ] Create SLO dashboard (availability, latency, tool success)
- [ ] Define eval datasets + pass/fail thresholds
- [ ] Configure rollback triggers on quality/safety regression
- [ ] Build separate immutable audit evidence stream correlated by `trace_id/run_id/policy_decision_id/approval_id`
- [ ] Validate native audit ingestion for Workato/iPaaS and SaaS-native surfaces

## 9) CI/CD and Promotion

- [ ] Build immutable runtime artifact + signed manifest where platform-owned
- [ ] Capture external platform configuration evidence where Terraform/control-plane management is not available
- [ ] Gate promotion on tests/scans/evals
- [ ] Gate promotion on policy contract checks and delegated-auth behavior
- [ ] Canary in staging then prod
- [ ] Require manual approval for high-risk production changes

## 10) Go-Live Exit Criteria

- [ ] One bounded-purpose domain agent in production with stable SLOs
- [ ] At least one low-risk workflow and one approval-gated high-risk workflow validated
- [ ] Every in-scope action surface has an inventory owner and control tier
- [ ] End-to-end audit trace validated across runtime, policy, approval, and target-resource outcome
- [ ] Delegated authorization validated for protected-data use cases or service-account exception documented
- [ ] Cost variance within budget threshold
- [ ] Incident runbook tested (rollback, disablement, and failover drill)
