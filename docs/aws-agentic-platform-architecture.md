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
- ECS/EKS/Lambda (agent runtime + tool adapters)
- DynamoDB + S3 + OpenSearch/Aurora pgvector (state + memory)
- IAM + KMS + Secrets Manager + Verified Permissions (security)
- CloudWatch + X-Ray + OTel/OpenSearch (observability)

## Multi-Agent Default
- Start with **planner + worker** pattern for predictability.
- Use event-driven fanout for background and long-running tasks.
- Add human-gated checkpoints for high-risk actions.

## Tenancy Models
- Shared runtime (logical isolation) -> fastest/cheapest.
- Service/namespace-per-tenant -> moderate isolation.
- Account-per-tenant -> strongest isolation for regulated workloads.

## Tool Governance
- `read` => allow + log
- `write-low` => guarded allow
- `write-high` => approval required
- `critical` => dual approval, default deny

## Release Gates
- tests pass
- security scans clean
- eval thresholds met
- staged canary healthy
- explicit production approval

## Roadmap (Condensed)
- Phase 0: architecture decisions + standards.
- Phase 1: single-agent production pilot.
- Phase 2: multi-agent orchestration + approvals.
- Phase 3: enterprise hardening + self-service onboarding.

## Detailed Design Source
For expanded design rationale and deeper tables, see the workspace doc:
- `../docs/AWS_ENTERPRISE_AGENTIC_PLATFORM_STANDARD.md`
