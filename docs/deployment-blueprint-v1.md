# Deployment Blueprint v1

## 0) Before You Start
- [ ] Define two candidate workflows (low-risk + high-risk)
- [ ] Assign data classification and compliance scope
- [ ] Select tenancy mode (shared / segmented / account-per-tenant)

## 1) Platform Foundations
- [ ] Create `shared-platform`, `dev`, `stg`, `prod` accounts
- [ ] Baseline IAM boundaries and SCP guardrails
- [ ] Enable KMS, Secrets Manager, CloudTrail organization trail

## 2) Runtime and Orchestration
- [ ] Deploy Agent Runtime (Lambda baseline, ROSA/EKS for scale)
- [ ] Deploy Step Functions workflow scaffold
- [ ] Add EventBridge bus + SQS queues for async tasks
- [ ] Implement idempotency keys + retry budgets
- [ ] Standardize inter-agent signed envelope (`agent_id`,`tenant_id`,`trace_id`,`policy_context`,`ttl`)

## 3) Model and Memory
- [ ] Bedrock model routing config (cost/latency/risk aware)
- [ ] DynamoDB for session/run state
- [ ] S3 for transcripts/artifacts with retention policies
- [ ] Vector memory store (OpenSearch/pgvector) + retrieval filters

## 4) Tooling and Integrations
- [ ] Stand up Tool Gateway service
- [ ] Register tool contracts (schema, auth, risk level)
- [ ] Enforce policy checks pre/post invocation
- [ ] Add human approvals for write-high/critical actions

## 5) Security and Governance
- [ ] Verify least privilege role-per-service
- [ ] Configure network boundaries + VPC endpoints
- [ ] Enable model/content guardrails
- [ ] Set audit trail to immutable retention where required
- [ ] Enforce A2A auth via IAM/IRSA + SigV4/private endpoints (no static shared keys)
- [ ] Enable replay protection/nonces on inter-agent commands

## 6) Observability and Evals
- [ ] Emit standardized telemetry (`trace_id`, `tenant_id`, `agent_id`)
- [ ] Create SLO dashboard (availability, latency, tool success)
- [ ] Define eval datasets + pass/fail thresholds
- [ ] Configure rollback triggers on quality/safety regression
- [ ] Build separate immutable audit evidence stream correlated by `trace_id/run_id`

## 7) CI/CD Promotion
- [ ] Build immutable runtime artifact + signed manifest
- [ ] Gate promotion on tests/scans/evals
- [ ] Canary in staging then prod
- [ ] Require manual approval for high-risk production changes

## 8) Go-Live Exit Criteria
- [ ] One low-risk workflow in production with stable SLOs
- [ ] End-to-end audit trace validated
- [ ] Cost variance within budget threshold
- [ ] Incident runbook tested (rollback + failover drill)
