# Enterprise Agentic AI Platform on AWS

> Architecture standard for an enterprise-grade, multi-tenant, governed agentic AI platform, reconciled with the AgentCore prototype direction.

This document captures practical implementation patterns for:
- control plane / data plane split,
- single-domain-agent-first runtime topology,
- external platform agent inventory,
- tenancy/isolation,
- memory/state,
- tool and action-surface governance,
- identity, delegation, and authorization,
- observability/evals and immutable audit,
- CI/CD promotion,
- phased rollout.

## Current Authority

The current prototype authority is:
- `docs/agentcore-prototype-spec-v1.md`

This document is the broader AWS/platform baseline and should be read underneath that spec. If there is a conflict, the prototype spec wins.

## Quick Start

1. Inventory all agent and agent-like automation surfaces, including AWS-hosted agents, AgentCore Runtime agents, Workato/iPaaS workflows, proprietary automation platforms, and SaaS-native agents such as Workday or Zoom features.
2. Start with one bounded-purpose domain agent and a strict runtime/tool contract.
3. Use framework-native/code-first orchestration as the default source of workflow truth.
4. Put nontrivial external actions behind Gateway-managed tool contracts or equivalent policy-enforced execution boundaries where feasible.
5. For constrained SaaS-native agents, apply compensating controls: inventory, least-privilege scopes, native audit/admin controls, owner review, and documented gaps.
6. Instrument traces, policy decisions, approvals, and audit evidence from day 1.

## Core Architecture Pattern

- **Governance spine:** portable policy, identity context, approval routing, audit correlation, registry, and release controls.
- **Execution lanes:** AWS/AgentCore runtime, iPaaS/proprietary automation, SaaS-native agent surfaces, and legacy/manual exceptions.
- **Target resources:** enterprise systems that must continue enforcing their own native authorization.

The governance spine is logical. It must not be confused with a single vendor gateway product.

## Recommended AWS Baseline

- Amazon Bedrock as primary model access path for the prototype.
- Bedrock AgentCore Runtime as a preferred managed hosting plane where it fits.
- Application/framework code as the preferred orchestration source of truth.
- Step Functions for durable workflow checkpoints, approvals, or long-running business transitions when useful.
- EventBridge + SQS/SNS for event choreography and async integration.
- Lambda/ECS/EKS/ROSA as runtime classes selected by workload shape, not as the core architectural abstraction.
- DynamoDB + S3 + OpenSearch/Aurora pgvector for state, artifacts, and memory.
- IAM + KMS + Secrets Manager + Verified Permissions/Cedar-style PDP for security and policy.
- CloudWatch + X-Ray + OTel/OpenSearch for observability, plus immutable audit evidence storage.

## Agent Topology Default

Start with a **single bounded-purpose domain agent** per capability.

Supervisor, planner/worker, or multi-agent choreography may be introduced later where workflow complexity justifies it, but it is not the default starting topology. This reduces governance ambiguity, prompt/tool drift, and unclear accountability.

## External Platform Agent Inventory

All enterprise agent surfaces must be registered, including:
- AWS-hosted application/framework agents,
- Bedrock AgentCore Runtime agents,
- Workato or other iPaaS automations that act agentically,
- proprietary automation platforms such as Pega-like systems,
- SaaS-native agent features in systems such as Workday and Zoom,
- legacy scripts or manually operated exception paths.

Inventory fields should include owner, platform, environment, data domains, tool/action surface, identity mode, delegated-user support, approval support, audit support, and enforcement tier.

## Integration Control Tiers

| Tier | Pattern | Expected control posture |
|---|---|---|
| Tier 0 | Platform-owned agent/runtime/tool gateway | Full request context, policy evaluation, approval, audit, replay protection |
| Tier 1 | Mediated iPaaS or proprietary automation platform | Registered contracts, policy hooks where available, audit correlation, scoped credentials |
| Tier 2 | SaaS-native agent or constrained vendor surface | Inventory, native admin controls, least-privilege app scopes, audit ingestion, documented gaps |
| Tier 3 | Legacy/manual exception | Explicit risk acceptance, compensating controls, owner review, migration target |

## Workato / iPaaS Gateway Position

There may be an organizational desire to make Workato the single enterprise gateway. That desire is understandable: one intake plane, one operating model, one place to govern integrations. It is not sufficient as an architecture decision.

Technical reality should drive gateway placement:
- Workato can be a strong **Tier 1 mediated integration plane** where it has connector coverage, execution visibility, policy hooks, approval integration, scoped credentials, and audit export.
- Workato is not automatically the best control point for AgentCore-hosted agents, application-native agent runtime, SaaS-native agents, or vendor features that execute entirely inside their own admin/control surfaces.
- A single product gateway can become a brittle choke point if it cannot preserve delegated identity, least privilege, idempotency/replay controls, approval binding, and immutable audit correlation for the action surface.
- The architecture should anchor on a portable governance contract, then choose the right enforcement point per platform.

Recommended framing: **one primary governance spine, multiple enforcement points**. Workato may be one major enforcement point, not the universal answer by default.

## Tool Governance

Tool and resource contracts must distinguish capability availability from execution authority.

Recommended risk tiers:
- `read` => allow + log when policy permits
- `write-low` => guarded allow
- `write-high` => approval required
- `critical` => dual approval or default deny

The effective decision model is layered:
1. agent/Gateway/platform policy decides whether the action may be attempted,
2. tool or platform enforcement point checks invocation conditions and risk gates,
3. target resource enforces native authorization.

A denial at any layer denies the request.

## Identity and Delegated Authorization

Human-originated access to protected data or governed business functions should use delegated user authorization when supported by the downstream system.

Where a platform only supports service-account execution, the action must be marked as service-authorized, policy-bounded, and audit-enhanced. Service-account execution must not be represented as proof of user entitlement.

## Observability vs Auditing

Observability and auditing overlap, but they are not the same system.

- **Observability:** operational health, latency, errors, throughput, cost, runtime quality.
- **Auditing:** immutable evidence of who/what/when/why for model, tool, policy, approval, and side-effect decisions.

Recommended split:
- Send runtime telemetry to CloudWatch/X-Ray/OTel dashboards.
- Write append-only audit evidence for regulated review.
- Correlate both with shared `trace_id`, `run_id`, and policy/approval IDs.

## Release Gates

- tests pass
- security scans clean
- eval thresholds met
- policy contract checks pass
- delegated-auth behavior validated where required
- audit evidence validated end-to-end
- staged canary healthy
- explicit production approval for high-risk actions or platform changes

## Terraform-First Delivery Model

Treat deployed AWS assets as Terraform-managed where practical, including control plane and data plane primitives.

Potential Terraform modules:
- `platform-control-plane`
- `agent-runtime-agentcore`
- `agent-runtime-lambda`
- `agent-runtime-container`
- `event-bus-core`
- `tool-gateway`
- `memory-tier`
- `observability-audit`
- `integration-registry`

Promote via environment workspaces/stacks (`dev` -> `stg` -> `prod`). Keep app/runtime release decoupled from infra release, but pin compatible versions in an agent manifest.

## Runtime Path

Select runtime by workload shape:

1. **AgentCore Runtime:** preferred managed hosting plane for interactive production agents when it supports the required control contract.
2. **Lambda:** low-volume, bursty, simple operations.
3. **ECS/EKS/ROSA:** sustained/high-throughput, long-running, memory-heavy, or specialized runtime needs.
4. **iPaaS/proprietary platforms:** integration-heavy workflows where the platform provides adequate governance hooks.
5. **SaaS-native agents:** constrained vendor surfaces governed through inventory, admin controls, scopes, and audit ingestion.

Use the same policy, identity, request-context, and audit contracts across runtime classes to prevent platform drift.

## What AgentCore Should Provide

A practical AgentCore layer should provide or integrate with:
- managed agent lifecycle: registration, versioning, rollout, rollback,
- first-class policy hooks and approval checkpoints,
- standardized tool contract enforcement,
- identity propagation and signed handoff envelopes,
- memory connectors with governance metadata,
- native eval/audit integration,
- runtime hosting without forcing orchestration lock-in.

If a product surface does not provide these strongly, keep the missing pieces in the platform governance spine and use that product selectively.

## Layered Prompt/Security Inheritance Model

Implement prompt/security inheritance like a base image model:

- `base-policy-pack` (platform/security owned): global safety/compliance constraints, mandatory logging/citation/tool-use rules, tenant and data handling constraints.
- `domain-pack` (domain/platform team owned): business vocabulary, domain guardrails, approved tool defaults.
- `agent-intent-pack` (product team owned): task-specific objectives and behavior.

Composition rule: `base-policy-pack` cannot be overridden by lower layers; only extended.

Operationally:
- Version each layer independently.
- Resolve layers into a signed final manifest at deploy time.
- Require eval + policy regression checks for any base layer change.

## Roadmap

- Phase 0: inventory, architecture decisions, and governance contracts.
- Phase 1: single-domain-agent production pilot with one low-risk and one approval-gated high-risk workflow.
- Phase 2: external platform integration tiers and SaaS-native compensating controls.
- Phase 3: multi-agent orchestration only where justified by workflow complexity.
- Phase 4: enterprise hardening + self-service onboarding.
