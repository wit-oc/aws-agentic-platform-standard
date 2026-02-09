# AgentCore Capabilities and Implementation Guide

## Purpose
Translate the earlier AWS architecture decisions into a practical **AgentCore-oriented implementation stack**, while preserving:
- Terraform-first delivery,
- Lambda-first to ROSA scale path,
- strong agent-to-agent auth,
- explicit audit + observability split,
- layered prompt/security inheritance.

---

## 1) Working Definition: What AgentCore Should Be

Treat AgentCore as the platform layer that sits above raw AWS primitives and provides:

1. **Agent lifecycle management**
   - register/version/deploy/rollback agent definitions
2. **Execution control**
   - runtime policy checks and approval hooks
3. **Inter-agent governance**
   - identity propagation, signed handoffs, trust boundaries
4. **Memory governance**
   - scoped memory connectors with retention/lineage controls
5. **Evaluation + audit integration**
   - release gates, runtime scoring, immutable evidence trails

If a given AgentCore product surface does not fully provide this, implement missing pieces in your control plane.

---

## 2) Capability Map: AgentCore -> AWS Building Blocks

| AgentCore Capability | AWS-Native Backing Pattern |
|---|---|
| Agent registry + versions | DynamoDB/Aurora metadata + ECR artifacts + signed manifests in S3 |
| Runtime execution API | API Gateway + Lambda/ECS/ROSA runtime service |
| Deterministic orchestration | Step Functions state machines |
| Async agent coordination | EventBridge + SQS/SNS envelopes |
| Tool invocation governance | Tool Gateway (private API), IAM/IRSA, Verified Permissions |
| Prompt/policy layering | Versioned config bundles in S3/AppConfig + manifest resolver |
| Session + run state | DynamoDB |
| Semantic memory | OpenSearch Serverless or Aurora pgvector |
| Artifacts/evidence | S3 (+ Object Lock if required) |
| Observability | OTel -> CloudWatch/X-Ray/OpenSearch |
| Audit trail | Append-only decision/action logs in immutable storage |

---

## 3) Implementing Earlier Design Decisions in AgentCore

## 3.1 Agent-to-Agent Authentication and Trust

### Required pattern
- Runtime identity per agent service (IAM role / IRSA role).
- Inter-agent API calls are SigV4 signed.
- Event handoffs use signed envelope with:
  - `agent_id`, `tenant_id`, `trace_id`, `policy_context`, `nonce`, `exp`.

### Verification steps at receiver
1. Verify caller identity + role mapping.
2. Verify tenant and environment boundaries.
3. Verify action is allowed by policy.
4. Verify nonce/TTL to prevent replay.

### Why this matters
This is the minimum to safely run multi-agent systems without implicit trust between services.

---

## 3.2 Event Bus + Orchestration Model

### Recommended hybrid
- **Step Functions** for stateful, deterministic control flow.
- **EventBridge/SQS** for decoupled inter-agent communication and background tasks.

### Practical split
- Keep critical business transitions in Step Functions.
- Use EventBridge for notifications, fanout, and independent worker execution.
- Persist all handoff envelopes and outcomes for replay/debug.

---

## 3.3 Runtime Path: Lambda -> ROSA

### Stage A: Lambda baseline
- rapid iteration, lower ops overhead, low-to-medium traffic.

### Stage B: Mixed runtime
- Lambda for burst/simple tasks,
- container workers for long-running or memory-heavy steps.

### Stage C: ROSA scale
- ROSA-hosted agent pools for sustained/high-throughput workloads,
- queue-driven autoscaling,
- stricter pod/network policy and workload identity.

**Key requirement:** agent contract and policy model must remain identical across all runtime tiers.

---

## 3.4 Terraform-First Control

Model AgentCore stacks as Terraform modules:
- `agentcore-control-plane`
- `agent-runtime-lambda`
- `agent-runtime-rosa`
- `agent-event-fabric`
- `agent-tool-gateway`
- `agent-memory`
- `agent-observability-audit`

CI checks:
- `terraform validate` / `fmt` / lint,
- security checks (`tfsec`/`checkov`),
- plan review + drift detection,
- environment promotion with approvals.

---

## 3.5 Auditing vs Observability

### Observability (ops)
- latency, error rate, throughput, saturation, token/cost telemetry.

### Auditing (compliance/forensics)
- immutable chain of:
  - model/tool/policy decisions,
  - approval checkpoints,
  - side-effect actions,
  - actor identity and timestamps.

CloudWatch is necessary for runtime operations, but not sufficient as the sole audit system for enterprise-grade governance.

---

## 3.6 Layered Prompt/Security Inheritance (Base Image Model)

Use three composable layers:

1. `base-policy-pack` (platform/security owned)
   - non-overridable rules: data handling, auth constraints, mandatory logging/citation, tool risk policies.
2. `domain-pack` (domain/platform owned)
   - domain terminology, domain guardrails, approved tool defaults.
3. `agent-intent-pack` (product team owned)
   - intent/objective and use-case behavior.

### Merge policy
- base layer is immutable from downstream teams.
- domain + intent can extend but not weaken base controls.
- final merged system prompt/config is signed into deployment manifest.

### Update inheritance
- when base policy updates, dependent agent packs receive a compatibility/eval check and staged rollout.

---

## 4) Suggested AgentCore API Surface (Minimal)

- `POST /agent-definitions` (register/update versioned definition)
- `POST /agent-deployments` (deploy definition to env/tenant)
- `POST /agent-runs` (start run with policy context)
- `POST /agent-runs/{id}/approve` (approval callback)
- `POST /agent-handoffs` (signed inter-agent transfer)
- `GET /agent-runs/{id}/trace` (execution + policy + audit views)

This gives a clean contract that product teams can target without needing to know underlying AWS internals.

---

## 5) Agent Definition Schema (Template)

```yaml
agentDefinition:
  id: customer-support-triage
  version: 1.3.0
  runtime:
    mode: lambda|rosa
    timeoutSeconds: 120
  modelPolicy:
    profile: balanced-cost
    fallback: true
  promptLayers:
    basePolicyPack: base-security-v7
    domainPack: support-domain-v3
    intentPack: triage-intent-v12
  tools:
    - id: kb.search
      risk: read
    - id: ticket.create
      risk: write-low
    - id: account.update
      risk: write-high
      requiresApproval: true
  memory:
    sessionStore: dynamodb
    vectorStore: opensearch
    retentionDays: 30
  observability:
    emitOtel: true
    auditLevel: full
```

---

## 6) Rollout Sequence for an AgentCore Program

1. Standardize agent definition + prompt layer schema.
2. Implement control plane APIs and signing/manifest resolver.
3. Deploy Lambda baseline runtime + event/orchestration fabric.
4. Add tool gateway with risk policies and approvals.
5. Add immutable audit evidence pipeline.
6. Introduce ROSA runtime class for high-throughput workloads.
7. Enforce release gates (eval + policy + security) for all agent updates.

---

## 7) Decision Guidance for Your Team

- If AgentCore gives strong lifecycle/governance primitives, adopt it as the control abstraction.
- If AgentCore is mostly convenience wrappers, keep your control plane authoritative and use AgentCore selectively.
- Keep Strands/agent templates as a packaging layer, not your only governance boundary.

The strategic goal is to keep enterprise controls stable even if agent frameworks evolve.
