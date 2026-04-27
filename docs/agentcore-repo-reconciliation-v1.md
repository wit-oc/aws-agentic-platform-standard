# AgentCore Repo Reconciliation v1

## Purpose

Reconcile the existing `aws-agentic-platform-standard` repository against the current authoritative AgentCore prototype direction captured in:

- `docs/agentcore-prototype-spec-v1.md`

This document identifies repo artifacts that align, partially align, or conflict with the current vision, and defines the cleanup path before calling the spec/architecture package finished.

## Current Authoritative Direction

The current prototype direction is:

1. **Code-first orchestration.** Keep prompts, orchestration graphs, tool-selection behavior, and workflow state in portable application/framework code where possible. AgentCore Runtime may host the workload, but provider-native agent orchestration must not become the core abstraction by default.
2. **Single bounded-purpose domain agent first.** Start with one bounded-purpose agent per domain capability; add supervisor or multi-agent topology only when workflow complexity justifies it.
3. **Gateway-managed external actions by default.** Nontrivial enterprise actions should pass through Gateway-managed tool contracts or equivalent policy-enforced execution boundaries.
4. **Federated/delegated authorization for governed data.** User-originated protected-data or business-function actions must use delegated user entitlement when downstream systems support it.
5. **Layered enforcement.** Agent/Gateway policy, tool invocation controls, and target-resource native authorization must all remain active. Any one layer denying a request denies the request.
6. **Single logical governance spine, not a single gateway product.** Workato, proprietary iPaaS, AWS-hosted Gateway, Pega-like platforms, and SaaS-native controls can all be enforcement points, but no one product should be assumed to front every agent/action surface.
7. **External agent inventory is mandatory.** Agents in Workato, proprietary iPaaS/Pega-like systems, Workday, Zoom, and other SaaS surfaces must be inventoried and assigned a control tier even when they cannot be fully front-ended by the platform Gateway.

## Alignment Summary

| Artifact | State | Reconciliation note |
|---|---:|---|
| `docs/agentcore-prototype-spec-v1.md` | Authoritative draft | Now contains the missing Section 2 for tool/data-plane, external platform agents, and gateway-vs-governance posture. |
| `docs/autonomy-action-taxonomy-risk-classes-v1.md` | Mostly aligned | Good action/risk/lane taxonomy. Needs examples expanded to cover Workato/iPaaS/SaaS-native agent surfaces. |
| `docs/autonomy-levels-overview.md` | Mostly aligned | Good public one-pager. Needs cross-platform wording so readers do not assume all automation lives inside AWS runtime. |
| `docs/acp-*.md` implementation annex drafts | Included for PR review | Policy gateway, approval broker, audit propagation, validation matrix align with the governance spine. Naming may still need review: keep `acp-*` as implementation annexes or rename to AgentCore governance subcontracts. |

## Artifacts That Do Not Yet Align

### 1. `README.md`

**Prior problem:** The README presented a single AWS-shaped architecture diagram: clients -> API Gateway/ALB -> AWS data plane -> Tool Gateway -> enterprise systems.

**Why it conflicted:** The current vision must include agents and agent-like automations outside AWS-hosted runtime: Workato, proprietary iPaaS/Pega-like platforms, Workday, Zoom, and other SaaS-native agent surfaces. The README also did not link the current authoritative AgentCore prototype spec.

**Resolution in this PR:** The README now links the prototype spec and reconciliation doc, reframes around one logical governance spine with multiple enforcement points, and includes a cross-platform architecture diagram.

### 2. `docs/aws-agentic-platform-architecture.md`

**Prior problem:** Several sections reflected an older baseline: planner/worker-first, Step-Functions-first, Lambda/ROSA-first, and AgentCore as optional acceleration over a generic AWS platform.

**Why it conflicted:** Current authority says start with a single bounded-purpose domain agent, keep orchestration logic portable/code-first, and allow AgentCore Runtime as a hosting plane without making provider-native orchestration the architectural abstraction.

**Resolution in this PR:** The architecture doc now follows single-domain-agent-first, code-first orchestration, AgentCore Runtime as managed hosting where appropriate, runtime classes selected by workload shape, external platform inventory, and integration control tiers.

### 3. `docs/agentcore-capabilities-implementation-guide.md`

**Prior problem:** The guide said to treat AgentCore as the platform layer above raw AWS primitives and mapped deterministic orchestration directly to Step Functions.

**Why it conflicted:** Current authority distinguishes AgentCore Runtime from the broader governance/control plane. The application/framework orchestration contract should remain portable. Step Functions may coordinate durable transitions, but should not own core prompt/tool-selection state by default.

**Resolution in this PR:** The guide now centers AgentCore Runtime as a managed hosting surface, application/framework code as orchestration source of truth, central governance contracts across multiple enforcement points, and external platform agents as first-class registry entries.

### 4. `docs/deployment-blueprint-v1.md`

**Prior problem:** The deployment checklist assumed the main job was to deploy AWS runtime, Step Functions, EventBridge, and a Tool Gateway.

**Why it conflicted:** The prototype must begin with inventory and classification of in-scope agent/action surfaces, including SaaS and iPaaS platforms that may already exist and may not be fully frontable.

**Resolution in this PR:** The blueprint now starts with agent/action-surface inventory, control-tier assignment, delegated-auth checks, SaaS-native audit/admin controls, and explicit Workato/iPaaS gateway role decisioning.

### 5. `docs/one-page-architecture.md`

**Prior problem:** The diagram was too narrow: Client/API/ChatOps -> API Gateway -> Agent Runtime -> Tool Gateway -> Enterprise Systems.

**Why it conflicted:** It hid the most important enterprise reality: agents exist in several platforms and must be governed even when not hosted behind the primary AWS gateway.

**Resolution in this PR:** The one-page architecture now shows the governed agent registry/control plane, AWS/AgentCore runtime lane, iPaaS/proprietary platform lane, SaaS-native lane, common policy/approval/audit spine, and target-resource native authorization.

### 6. `docs/pro-upgrade-multi-agent-implementation-packet.md`

**Problem:** Household ROI, Cosmere-themed personal-agent design, and model-upgrade planning are not part of an enterprise AWS/AgentCore architecture repo.

**Why it conflicts:** This is private/operator design material, not a public/reference enterprise AgentCore artifact. It also pushes a coordinator-first multi-agent topology that conflicts with the current single-domain-agent-first prototype default.

**Resolution in this PR:** Removed from the enterprise reference repo.

### 7. `docs/radiant-mode-multi-agent-spec.md`

**Problem:** This is a Wit/Radiant internal multi-agent operating model with household roles and thematic labels.

**Why it conflicts:** It is not enterprise AgentCore reference architecture and could confuse readers about recommended product topology. It also makes multi-agent delegation feel default, while current authority says single bounded-purpose domain agent first.

**Resolution in this PR:** Removed from the enterprise reference repo.

### 8. `TASK_BOARD.md`

**Prior problem:** The local task board included ACP execution artifacts and marked several docs complete, while those docs were not reconciled to the AgentCore prototype vocabulary.

**Why it conflicted:** It implied the repo was further along than the authoritative AgentCore spec actually was.

**Resolution in this PR:** The task board now reflects the reconciliation work, PR review state, ACP annex decision point, and removal of out-of-scope private docs.

## Workato / iPaaS Gateway Position

Do **not** make Workato the assumed single enterprise gateway by default.

Recommended posture:

- Workato can be a **Tier 1 mediated integration plane** where it has strong connector coverage and can preserve required controls.
- The enterprise should maintain a **logical governance spine** for policy, identity context, approval routing, and audit correlation independent of Workato.
- For platform-owned and iPaaS/proprietary automation, front actions with Gateway-managed contracts or equivalent PEP/PDP hooks where feasible.
- For SaaS-native agents such as Workday or Zoom, use compensating governance: inventory, least-privilege OAuth/app scopes, native admin controls, audit ingestion, and documented delegation/approval gaps.
- Treat any service-account-only path as service-authorized, policy-bounded, and audit-enhanced; never imply delegated user authorization if the downstream platform did not enforce it.

## Remaining Review Questions

1. Should the ACP implementation annex docs stay under `acp-*` names, or should they be renamed into AgentCore governance subcontracts?
2. Should Workato/iPaaS integration examples become a separate dedicated decision record, or are the current sections in the spec/architecture/guide sufficient?
3. Which first pilot workflow should be used to validate Tier 0 vs Tier 1 vs Tier 2 enforcement behavior?

## Current Repo State Note

This reconciliation was prepared on branch `agentcore-prototype-spec-v1`, which backs PR #1. The intended PR package includes the authoritative spec updates, cross-platform architecture rewrites, Workato/iPaaS gateway positioning, ACP implementation annex drafts, task board reset, and removal of out-of-scope private multi-agent docs.
