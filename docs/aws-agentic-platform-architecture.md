# Enterprise Agentic AI Platform on AWS

> Canonical architecture standard for an enterprise-grade, multi-tenant, agentic AI platform on AWS.

This document captures practical implementation patterns for:
- control plane / data plane split,
- multi-agent orchestration,
- tenancy/isolation,
- memory/state,
- external tool integration,
- security/governance,
- observability/evals,
- CI/CD promotion,
- phased rollout.

## Quick Start
1. Start with the single-agent template + strict runtime contract.
2. Add Step Functions + EventBridge for durable multi-agent flows.
3. Enforce tool risk tiers + approval gates before production write actions.
4. Instrument traces/metrics from day 1 and gate releases on evals.

## Core Architecture Pattern
- **Control plane:** global metadata, policy, evals, onboarding, release controls.
- **Data plane:** execution runtime, memory, tools, model calls, side effects.

## Recommended AWS Baseline
- Bedrock (model access)
- Step Functions (durable orchestration)
- EventBridge + SQS/SNS (event choreography)
- Lambda first for low-volume, ROSA/EKS/ECS for sustained/high-volume runtime
- DynamoDB + S3 + OpenSearch/Aurora pgvector (state + memory)
- IAM + KMS + Secrets Manager + Verified Permissions (security)
- CloudWatch + X-Ray + OTel/OpenSearch (observability + auditable traces)

## Multi-Agent Default
- Start with **planner + worker** pattern for predictability.
- Use event-driven fanout for background and long-running tasks.
- Add human-gated checkpoints for high-risk actions.
- Keep orchestration intent in Step Functions; use EventBridge for decoupled inter-agent messaging.

## Agent-to-Agent Authentication (A2A)
Use identity-backed service-to-service auth, never shared static keys between agents.

- Each agent runtime gets its own IAM role (or IRSA role on ROSA/EKS).
- Inter-agent calls are signed (SigV4) via API Gateway/App Mesh/private service endpoints.
- Event messages carry a signed envelope with `agent_id`, `tenant_id`, `trace_id`, `policy_context`, and short TTL.
- Receiver validates:
  1) caller identity/role,
  2) tenant boundary,
  3) allowed action contract,
  4) replay protection (`nonce`/idempotency key).
- High-risk action delegation requires approval token binding (scope + expiry).

This gives non-repudiation and clear provenance across agent hops.

## Tenancy Models
- Shared runtime (logical isolation) -> fastest/cheapest.
- Service/namespace-per-tenant -> moderate isolation.
- Account-per-tenant -> strongest isolation for regulated workloads.

## Tool Governance
- `read` => allow + log
- `write-low` => guarded allow
- `write-high` => approval required
- `critical` => dual approval, default deny

## Observability vs Auditing (Both Required)
Observability and auditing overlap, but they are not the same system.

- **Observability (CloudWatch/X-Ray/OTel):** operational health, latency, errors, throughput, cost.
- **Auditing (immutable decision trail):** who/what/when/why for model/tool/policy decisions and side effects.

Recommended split:
- Send runtime telemetry to CloudWatch/X-Ray dashboards.
- Write an append-only audit stream (S3 object lock / immutable store) for regulated review.
- Correlate both using shared `trace_id` and `run_id`.

So yes: CloudWatch is part of the observability construct, but enterprise agent auditing should be explicitly modeled as a separate immutable evidence plane.

## Release Gates
- tests pass
- security scans clean
- eval thresholds met
- staged canary healthy
- explicit production approval

## Terraform-First Delivery Model
Treat all deployed assets as Terraform-managed, including control plane and data plane primitives.

- Terraform modules:
  - `platform-control-plane`
  - `agent-runtime-lambda`
  - `agent-runtime-rosa`
  - `event-bus-core`
  - `tool-gateway`
  - `memory-tier`
  - `observability-audit`
- Promote via environment workspaces/stacks (`dev` -> `stg` -> `prod`).
- Enforce policy checks in CI (`terraform validate`, `tflint`, `tfsec/checkov`, drift detection).
- Keep app/runtime release decoupled from infra release, but pin compatible versions in an agent manifest.

## Lambda -> ROSA Runtime Path
Default path for your stated operating model:

1. **Low volume / fast iteration:** Lambda runtime + Step Functions + EventBridge.
2. **Medium scale:** mixed mode (Lambda for burst, containers for long-running workers).
3. **High volume / specialized runtime:** ROSA-hosted agent pools with IRSA/workload identity, queue-backed autoscaling, and stricter network policy.

Use the same agent manifest and policy contracts across both runtimes to prevent platform drift.

## What AgentCore Should Extend (over baseline)
Assuming Strands is primarily an agent definition/template construct, a practical AgentCore value layer should add:

- managed agent lifecycle (registration, versioning, rollout, rollback),
- first-class policy hooks and approval checkpoints,
- standardized tool contract enforcement,
- built-in identity propagation and signed handoff envelopes,
- memory connectors with governance metadata,
- native eval/audit integration.

If AgentCore does not provide these strongly, keep them in your platform control plane and treat AgentCore as optional acceleration, not a dependency.

## Layered Prompt/Security Inheritance Model
Your base-layer idea is solid. Implement prompt/security inheritance like a base image model:

- `base-policy-pack` (owned by platform/security):
  - global safety and compliance constraints,
  - mandatory logging/citation/tool-use rules,
  - tenant and data handling constraints.
- `domain-pack` (owned by domain/platform team):
  - business vocabulary, domain guardrails, tool defaults.
- `agent-intent-pack` (owned by product team):
  - task-specific objectives and behavior.

Composition rule: `base-policy-pack` cannot be overridden by lower layers; only extended.

Operationally:
- Version each layer independently.
- Resolve layers into a signed final manifest at deploy time.
- Require eval + policy regression checks for any base layer change.

## Roadmap (Condensed)
- Phase 0: architecture decisions + standards.
- Phase 1: single-agent production pilot.
- Phase 2: multi-agent orchestration + approvals.
- Phase 3: enterprise hardening + self-service onboarding.

## Detailed Design Source
For expanded design rationale and deeper tables, see the workspace doc:
- `../docs/AWS_ENTERPRISE_AGENTIC_PLATFORM_STANDARD.md`
