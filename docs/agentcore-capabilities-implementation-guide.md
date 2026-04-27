# AgentCore Capabilities and Implementation Guide

## Purpose

Translate the current AgentCore prototype specification into a practical implementation stack while preserving:
- code-first orchestration,
- Terraform-first delivery for AWS assets,
- strong identity and delegated authorization,
- explicit audit + observability split,
- layered prompt/security inheritance,
- cross-platform governance for AWS, iPaaS/proprietary, and SaaS-native agent surfaces.

Primary authority:
- `docs/agentcore-prototype-spec-v1.md`

## 1) Working Definition: What AgentCore Should Be

Treat AgentCore as a managed runtime and lifecycle surface that participates in the enterprise governance spine.

AgentCore should ideally provide or integrate with:

1. **Agent lifecycle management**
   - register/version/deploy/rollback agent definitions
2. **Managed runtime hosting**
   - production agent execution with stable operational controls
3. **Policy and approval hooks**
   - runtime checkpoints before side effects
4. **Identity propagation**
   - user, agent, tenant/business context, environment, delegation mode
5. **Tool contract enforcement**
   - registered actions, risk classes, execution authority checks
6. **Memory governance**
   - scoped memory connectors with retention and lineage metadata
7. **Evaluation + audit integration**
   - release gates, runtime scoring, immutable evidence trails

AgentCore Runtime should not force provider-native orchestration lock-in. Prompts, workflow logic, tool-selection behavior, and state transitions should remain portable in application/framework code unless a provider-native feature is explicitly justified.

## 2) Capability Map: Governance Contract -> Implementation Options

| Governance capability | AWS / AgentCore option | iPaaS / proprietary option | SaaS-native option |
|---|---|---|---|
| Agent inventory | Control plane registry + signed manifests | Registry entry for workflow/agent/app | Registry entry for native agent/app feature |
| Runtime execution | AgentCore Runtime, Lambda, ECS/EKS/ROSA | Workato/Pega-like execution | Vendor-hosted execution |
| Orchestration | Application/framework code; Step Functions for durable checkpoints | Platform workflow engine where controls fit | Vendor-native workflow controls |
| Tool/action governance | Tool Gateway, PEP/PDP, IAM, Verified Permissions | Connector policy hooks, scoped credentials, approval integrations | Admin settings, OAuth scopes, vendor audit/export |
| Identity/delegation | Entra/AWS federation + propagated request context | Delegated auth where platform supports it; otherwise marked service-authorized | Vendor-supported delegated auth or documented service-account gap |
| Approval | Approval broker / Step Functions / workflow pause | Platform approval task or external broker callback | Native approval feature or compensating manual control |
| Audit | Immutable evidence stream + OTel/CloudWatch correlation | Audit export + evidence correlation | Native audit export/admin logs + evidence correlation |
| Replay/idempotency | Gateway idempotency keys + nonce/TTL | Platform-supported idempotency or wrapper controls | Vendor capability or documented gap |

## 3) Runtime and Orchestration Pattern

### Required posture

- Keep orchestration logic in application/framework code by default.
- Use AgentCore Runtime as a preferred managed hosting plane where it supports the required controls.
- Use Step Functions for durable checkpoints, approvals, long-running business transitions, and external workflow coordination—not as the default agent brain.
- Use EventBridge/SQS for decoupled async work and event choreography.

### Why this matters

This prevents expensive lock-in and keeps the enterprise policy/identity/audit contract stable if runtime providers, agent frameworks, or integration platforms change.

## 4) Agent-to-Agent Authentication and Trust

### Required pattern

- Runtime identity per agent service or platform integration.
- Signed inter-agent or inter-platform handoff envelopes where supported.
- Handoff context includes:
  - `agent_id`
  - `tenant_id` or business context
  - `environment`
  - `trace_id`
  - `run_id`
  - `policy_context`
  - `delegation_mode`
  - `nonce`/`exp` where supported

### Verification steps at receiver

1. Verify caller identity + role/platform mapping.
2. Verify tenant/business and environment boundaries.
3. Verify action is registered and allowed by policy.
4. Verify nonce/TTL/idempotency where available.
5. Preserve trace/audit correlation IDs.

No agent platform should implicitly trust another agent just because both are “enterprise tools.”

## 5) Tool and External Action Governance

Tool governance must distinguish:
- **capability availability**: whether an agent/platform has a registered action surface
- **execution authority**: whether this invocation is allowed for this user/context/resource

Every governed action should have a contract containing:
- action name and description
- target system and resource type
- read/write/destructive classification
- risk class
- required identity mode
- required request context fields
- approval requirements
- audit event requirements
- rollback/compensation notes

## 6) Workato / iPaaS Position

Workato can be valuable, but it should not be declared the universal gateway because the organization wants one gateway.

Use Workato as a **Tier 1 mediated integration plane** when it can provide:
- connector coverage for the target system,
- enforceable policy or approval hooks,
- scoped credential management,
- delegated user authorization where required or clear service-account labeling where not,
- idempotency/replay protection or wrapper controls,
- audit export with trace correlation,
- operational ownership and support.

Do not use Workato as the single default gateway when:
- the agent runs inside AgentCore/application runtime and needs low-latency/native tool control,
- the action occurs inside a SaaS-native vendor agent surface that Workato cannot truly front,
- Workato would collapse delegated user authorization into a broad service account,
- approval/audit context would be weaker than a direct platform/Gateway path,
- connector semantics hide the target resource authorization decision,
- routing everything through Workato creates an operational choke point without stronger controls.

Architecture anchor: **one governance spine, multiple enforcement points**.

## 7) Integration Control Tiers

| Tier | Pattern | Examples | Expected control posture |
|---|---|---|---|
| Tier 0 | Platform-owned agent/runtime/tool gateway | AgentCore-hosted domain agent, AWS-hosted app agent | Full request context, policy evaluation, approval, audit, replay protection |
| Tier 1 | Mediated iPaaS/proprietary platform | Workato, Pega-like workflow engine | Registered contracts, policy hooks where available, audit correlation, scoped credentials |
| Tier 2 | SaaS-native/constrained vendor surface | Workday/Zoom native agent features | Inventory, native admin controls, least-privilege scopes, audit ingestion, documented gaps |
| Tier 3 | Legacy/manual exception | Manual runbooks, unwrapped scripts | Explicit risk acceptance, owner review, migration target |

## 8) Terraform-First Control

Model AWS-hosted pieces as Terraform modules where practical:
- `agentcore-control-plane`
- `agent-runtime-agentcore`
- `agent-runtime-lambda`
- `agent-runtime-container`
- `agent-event-fabric`
- `agent-tool-gateway`
- `agent-memory`
- `agent-observability-audit`
- `integration-registry`

CI checks:
- `terraform validate` / `fmt` / lint,
- security checks,
- plan review + drift detection,
- environment promotion with approvals.

External platform configuration may not be fully Terraform-manageable. Where it is not, the platform still needs exportable evidence of configuration, owners, scopes, and audit settings.

## 9) Auditing vs Observability

### Observability

Operational signals:
- latency,
- error rate,
- throughput,
- saturation,
- token/cost telemetry,
- runtime quality.

### Auditing

Immutable evidence chain:
- request context,
- model/tool/policy decisions,
- approval checkpoints,
- side-effect actions,
- actor identity,
- delegation mode,
- timestamps,
- target resource outcome.

CloudWatch is necessary for runtime operations, but not sufficient as the sole audit system for enterprise governance.

## 10) Layered Prompt/Security Inheritance

Use three composable layers:

1. `base-policy-pack` (platform/security owned)
   - non-overridable rules: data handling, auth constraints, mandatory logging/citation, tool risk policies.
2. `domain-pack` (domain/platform owned)
   - domain terminology, domain guardrails, approved tool defaults.
3. `agent-intent-pack` (product team owned)
   - intent/objective and use-case behavior.

### Merge policy

- Base layer is immutable from downstream teams.
- Domain + intent can extend but not weaken base controls.
- Final merged system prompt/config is signed into deployment manifest.

### Update inheritance

When base policy updates, dependent agent packs receive a compatibility/eval check and staged rollout.

## 11) Suggested AgentCore API Surface

Minimal platform-owned API surface:

- `POST /agent-definitions` — register/update versioned definition
- `POST /agent-deployments` — deploy definition to env/tenant
- `POST /agent-runs` — start run with request/policy context
- `POST /agent-runs/{id}/approve` — approval callback
- `POST /agent-handoffs` — signed inter-agent transfer
- `GET /agent-runs/{id}/trace` — execution + policy + audit view
- `POST /tool-actions/register` — register governed action contract
- `GET /external-agent-surfaces` — inventory external/iPaaS/SaaS-native surfaces

## 12) Agent Definition Schema (Template)

```yaml
agentDefinition:
  id: customer-support-triage
  version: 1.3.0
  runtime:
    mode: agentcore|lambda|container|external-platform|saas-native
    hostPlatform: bedrock-agentcore
    timeoutSeconds: 120
  modelPolicy:
    provider: bedrock
    profile: balanced-cost
    fallback: true
  promptLayers:
    basePolicyPack: base-security-v7
    domainPack: support-domain-v3
    intentPack: triage-intent-v12
  identity:
    supportsDelegatedUser: true
    serviceAccountFallbackAllowed: false
  tools:
    - id: kb.search
      risk: read
      identityMode: delegated-user
    - id: ticket.create
      risk: write-low
      identityMode: delegated-user
    - id: account.update
      risk: write-high
      identityMode: delegated-user
      requiresApproval: true
  externalSurfaces:
    - platform: workato
      controlTier: tier-1
      auditExport: enabled
    - platform: workday
      controlTier: tier-2
      auditExport: native-admin-log
  memory:
    sessionStore: dynamodb
    vectorStore: opensearch
    retentionDays: 30
  observability:
    emitOtel: true
    auditLevel: high
```

## 13) Rollout Sequence

1. Standardize agent/action inventory schema.
2. Inventory AWS, iPaaS/proprietary, and SaaS-native agent surfaces.
3. Assign control tiers and document gaps.
4. Standardize request context and tool/action contracts.
5. Implement AgentCore Runtime pilot for one bounded-purpose domain agent.
6. Add policy, approval, and immutable audit pipeline.
7. Add Workato/iPaaS integration where technically justified.
8. Add SaaS-native compensating controls and audit ingestion.
9. Enforce release gates for all agent updates and action-surface changes.

## 14) Decision Guidance

- If AgentCore gives strong runtime/lifecycle primitives, use it as the preferred managed runtime.
- If AgentCore or any vendor surface weakens governance, keep the control in the platform governance spine.
- If Workato provides the strongest enforceable control for a workflow, use it as a Tier 1 enforcement point.
- If Workato is only organizationally convenient but weakens identity, approval, audit, or target authorization, do not make it the gateway for that workflow.
- Keep enterprise controls stable even when frameworks, vendors, and runtime surfaces evolve.
