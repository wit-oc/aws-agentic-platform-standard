# AgentCore Prototype Specification v1

> Working spec for the initial AWS AgentCore prototype. This document captures currently locked decisions and open follow-on design work.

## Status
- **State:** draft for review
- **Scope:** prototype architecture and governance decisions
- **Out of scope for this revision:** Terraform implementation details, full API contracts, and final production rollout sequencing

## 1. Agent Runtime and Orchestration

### 1.A. Portable, Externalized Orchestration
Portability **MUST** be treated as a hard requirement.

Durable orchestration state, approval checkpoints, retry policy, HITL pause/resume, and cross-system workflow control **SHOULD** be externalized from any single agent runtime where practical. AgentCore Runtime **MAY** be the default hosting plane for agent execution, but prompts, agent contracts, tool-selection boundaries, workflow definitions, and orchestration state **SHOULD** remain portable enough to move to another approved runtime or orchestration surface if required.

Agent application code **MAY** own short-lived task-local reasoning, prompt flow, and bounded intra-agent behavior. It **SHOULD NOT** be the only place where enterprise workflow state, HITL state, retry behavior, or cross-agent/cross-system process state can be observed or controlled.

AgentCore-native surfaces **MAY** be used where they create clear operational value, but the core agent contract **MUST NOT** depend on a provider-specific orchestration model when a portable externalized pattern is available.

### 1.B. Agent Topology
The initial implementation **MUST** prefer a single bounded-purpose agent per domain capability.

A supervisor or multi-agent pattern **MAY** be introduced later where justified by workflow complexity, but the default starting point **MUST** be a single domain agent with a shared underlying tool fabric.

### 1.C. Runtime Boundary
Agents **SHOULD** interact with enterprise systems through Gateway-managed tools by default.

Direct calls from agents to enterprise systems **MUST NOT** be the default integration pattern for nontrivial external actions. Gateway-managed tools **SHOULD** provide the primary control point for authorization, policy enforcement, auditing, and future portability.

### 1.D. Autonomy and Failure Handling
The default runtime posture **MUST** be bounded retries with fallback.

The platform **MUST NOT** assume aggressive autonomous recovery by default for high-consequence workflows. Where retries are allowed, retry bounds, fallback behavior, and failure closure **MUST** be explicit.

### 1.E. Runtime and Model Standardization
The platform **SHOULD** standardize on one preferred agent framework and Amazon Bedrock as the primary model access path for v1.

Additional frameworks or model providers **MAY** be supported later where justified, but the prototype **MUST** optimize for one preferred implementation path and **MUST NOT** weaken the initial governance, identity, and runtime contracts.

## 2. Tool, Data-Plane, and External Agent Platform Governance

### 2.A. Enterprise Agent and Action-Surface Inventory
All enterprise agent surfaces **MUST** be inventoried in a central registry or registry-backed control plane, regardless of runtime location.

The inventory **MUST** include, where applicable: agent or automation name, owning team, runtime or host platform, environment, data domains accessed, tool or action surface, identity mode, delegated-user support status, approval and audit capabilities, and gateway or policy-enforcement control mode.

This includes agents and agent-like automations running in AWS-hosted application code, Bedrock AgentCore Runtime, Workato or other iPaaS platforms, proprietary automation platforms such as Pega, and SaaS-native agent surfaces such as Workday or Zoom.

### 2.B. Governance Spine and Vendor Neutrality
The platform **MUST** preserve a single logical governance spine for policy, identity context, audit correlation, approval routing, platform reporting, and declarative control intent.

That governance spine **MUST NOT** be equated with a single vendor gateway product. Workato, a proprietary iPaaS, an AWS-hosted tool gateway, or a SaaS-native admin/control surface **MAY** each serve as an enforcement point for the resources they can actually mediate, but no single integration platform **SHOULD** be assumed to front every agent, tool, or SaaS action.

Workato currently publishes AI gateway and MCP capabilities relevant to this pattern, including AI gateway collections, MCP servers, connector-backed skills/tools, configurable authorization between Workato and AI agents, and audit/governance positioning. Those capabilities make Workato a credible candidate enforcement point for mediated integration workflows. They do **not**, by themselves, prove that Workato is the technically correct universal gateway for AgentCore-hosted agents, proprietary agent platforms, or SaaS-native agent surfaces that execute inside vendor control planes.

The decision to use Workato or any other platform as an enforcement point **MUST** be made per workflow/action surface based on technical control fit: delegated identity support, connector semantics, approval hooks, audit export, idempotency/replay controls, operational ownership, latency, failure behavior, and target-resource authorization preservation. Organizational preference for one primary gateway **MUST NOT** override those technical constraints.

The governance contract **SHOULD** be declarative where practical. Platform-specific adapters or harnesses **SHOULD** translate declarative policy intent into native controls, configuration, evidence collection, and reports for each enforcement surface.

### 2.C. Gateway Fronting and Platform Owner Accountability
For platform-owned agents and integration platforms where requests can be mediated, nontrivial external actions **SHOULD** route through Gateway-managed tool contracts or an equivalent policy-enforced execution boundary.

For SaaS-native agents or constrained vendor surfaces where full gateway fronting is not feasible, the platform **MUST** still apply compensating controls: inventory, owner assignment, OAuth/app/admin scope restriction, native audit/admin event capture where available, action risk classification, unsupported delegation or approval capability documentation, exception handling, disablement path, and review cadence.

The Product Owner or accountable platform owner for each external agent platform **MUST** own evidence that their platform aligns to these standards. That ownership **SHOULD** include recurring review/reporting for inventory completeness, permission posture, audit availability, exceptions, open gaps, and remediation status.

Limited fronting **MUST NOT** be treated as invisibility. SaaS-native agents remain governed assets even when the platform can only observe, configure, or constrain them indirectly.

### 2.D. Tool Registration and Availability
Every governed tool or external action surface **SHOULD** have a registered contract containing action name, target system/resource, read/write/destructive classification, risk class, identity mode, required request context, approval requirements, audit requirements, and rollback or compensation notes where applicable.

The default operating assumption **SHOULD** be that if a tool is registered to an agent, users authorized to use that agent can invoke that tool through the agent. The platform **SHOULD NOT** create a default spiderweb of independent per-user entitlements at the gateway, agent, tool, and resource layers.

Data contracts and target-resource authorization **MUST** still be preserved. The tool should pass through required caller, delegation, and request context so the system behind the tool can enforce access to the underlying data or action. If tool use must be limited, the preferred control is to gate the tool's registration/availability to the agent or agent profile. Per-user or per-group tool constraints **MAY** be introduced for exceptional high-risk cases, but they **SHOULD** be explicit exceptions with owner, rationale, auditability, and troubleshooting guidance.

### 2.E. Integration Control Modes
The prototype **SHOULD** classify each external platform or tool surface into one of these control modes:

| Mode | Pattern | Expected control posture |
|---|---|---|
| Full Feature | Platform-owned agent/runtime/tool gateway | Full request context, policy evaluation, approval, audit, replay protection |
| Partial Feature | Mediated iPaaS or proprietary automation platform | Registered contracts, policy hooks where available, audit correlation, scoped credentials |
| Best Effort | SaaS-native agent or constrained vendor surface | Inventory, native admin controls, least-privilege app scopes, audit ingestion, documented gaps |
| Exception | Legacy/manual exception | Explicit risk acceptance, compensating controls, owner review, migration target |

Promotion from prototype to production **MUST** identify the control mode for every in-scope action surface and document any Best Effort or Exception gaps.

## 3. Identity, Delegation, and Authorization

### 3.A. Identity Sources and Separation
Azure Entra **MUST** be treated as the authoritative identity provider for human users. Where permitted by platform and integration constraints, federated access into AWS services **SHOULD** be mediated through AWS SSO / IAM Identity Center using the approved enterprise federation path.

Human access **MUST** resolve back to the authoritative enterprise identity provider. Service and agent identities **SHOULD** default to IAM roles, workload identities, or equivalent non-user service principals. IAM users **MUST NOT** be the default pattern for agent or service access.

User identity **SHOULD** be human-readable in audit and access records. UPN **MAY** be used as the primary displayed identifier, but the system **SHOULD** retain a stable immutable subject identifier for canonical correlation where available.

### 3.B. Delegated Authorization and Hard Controls
Where a request originates from a human user and touches data or governed business functions, the integration path **MUST** preserve delegated user authorization when the downstream system supports it. Prototype implementations **MAY** use non-sensitive data or reduced-scope substitutes to validate architecture, but the use case **MUST NOT** be considered complete until required federated/delegated permissions are enforced end-to-end.

Agent or service credentials **MUST NOT** be used to bypass user authorization boundaries for actions expected to be performed under user entitlement. Agents **MUST NOT** be granted standing privileges for user-governed actions when delegated user authorization is available.

Autonomous agent actions on behalf of a user, without user context, **MUST** be explicitly bounded, tagged, and policy-governed. Autonomous agent actions not operating on behalf of a user **MAY** execute under agent or service identity, but **MUST** remain within explicitly defined service scope, environment boundaries, and policy constraints.

These boundaries **MUST** be enforced by hard controls such as IAM, workload identity, OAuth scopes, gateway policy, resource policy, network controls, approval gates, or equivalent platform mechanisms. Prompt steering, system instructions, or agent self-discipline **MUST NOT** be treated as sufficient enforcement.

### 3.C. Identity Propagation and Context
Initial implementation **MUST** propagate user identity sufficient to support user-scoped authorization decisions. Group or role propagation **MAY** be deferred where not immediately required by downstream systems, but the design **MUST** preserve a clear extension path for future propagation.

Protected data, sensitive tools, and exception-bearing resources **MUST** support explicit tagging or equivalent classification metadata. Authorization and policy enforcement **MUST** account for such tags. Where feasible, exception awareness **SHOULD** be visible through the service or agent registry.

### 3.D. Human-in-the-Loop and Boundary Controls
A Human-in-the-Loop (HITL) control framework **MUST** exist for actions crossing defined business, security, or operational boundaries. HITL review **MUST** be enforceable for actions with downstream business impact, exception handling, or execution outside a self-contained approved use case.

Identities **MAY** hold entitlements across multiple SDLC environments according to approved policy. However, service-to-service and agent-to-service interactions **MUST** enforce environment and account boundaries by default.

Cross-environment and cross-account access, including development-to-production interaction, **MUST** be deny-by-default and allow-by-explicit-exception. Exceptions **MUST** be documented, owner-approved, scope-bounded, time-bounded where practical, and auditable. Cross-account access **MUST NOT** be assumed from broad platform membership, agent registration, or shared operational ownership.

## 4. Authorization, Execution, and Audit Flow

### 4.A. Canonical Claim and Request Context Set
Each agent request that reaches an execution boundary **MUST** carry a canonical request context sufficient to support authorization, auditing, and policy evaluation.

The baseline context **MUST** include, where applicable: immutable subject identifier, UPN or equivalent human-readable user identifier, agent identity, tenant or business context, environment identifier, session or invocation identifier, trace identifier, delegation mode, and group/role/entitlement attributes when required for policy evaluation.

Additional attributes **SHOULD** be attached only when required by policy, the target resource, or the registered tool/capability contract. Examples include authentication strength, token issuer, token age or expiry context, source application, approval state, and data-classification hints.

Purpose-of-use and business-justification metadata **SHOULD** generally be defined at the workflow, tool, or capability level and referenced by execution/audit records rather than repeated chattily on every invocation. Per-execution purpose or justification **MAY** be required for sensitive, exceptional, or approval-gated actions.

Claims and request attributes **MUST NOT** be propagated beyond what is required for enforcement, auditing, and downstream authorization.

### 4.B. Layered Enforcement
The delegation mode defined in Section 3 **MUST** be preserved across execution boundaries. Where a workflow mixes data access and platform actions, the implementation **MUST NOT** silently widen a data-access step from delegated user authorization to standing service authorization.

Access control **SHOULD** be enforced primarily at the agent runtime and/or Gateway boundary, where request context, delegation mode, tool policy, and HITL state can be evaluated together. Tools **MUST** receive authorization context from upstream rather than inventing or mutating it locally.

Target resources **MUST** continue to enforce their own native access controls, connection policy, and resource ACLs independent of the agent or tool path. Agent or Gateway authorization **MUST NOT** be treated as a substitute for downstream resource authorization.

The effective decision model **MUST** behave as layered enforcement:
1. agent or Gateway policy determines whether the requested action may be attempted
2. tool boundary determines whether invocation conditions and risk gates are satisfied
3. target resource determines whether the presented identity is authorized for the requested operation

A request **MUST** be denied when any applicable layer denies it. These enforcement outcomes **MUST** be implemented as hard controls in the relevant enforcement layer.

### 4.C. Audit Contract and Audit Levels
The platform **MUST** define a standard audit contract in agent templates and shared execution surfaces.

Every tool invocation or side-effecting action **MUST** emit audit data sufficient to reconstruct requesting user identity when present, agent or service identity, delegation mode, tool/action invoked, target system or resource, environment and tenant context, policy decision result, HITL state/outcome where applicable, execution timestamp, and trace correlation identifiers.

The platform **SHOULD** define default audit levels at the template, tool, or capability level.

At minimum:
- **medium audit** **MUST** be the default baseline for standard enterprise actions. It should capture the actor, agent, delegation mode, action, target system/resource reference, environment, policy decision, HITL outcome where applicable, timestamp, and trace identifiers. It **SHOULD NOT** capture full prompts, payloads, or data values by default unless required for troubleshooting or compliance.
- **high audit** **MUST** be required for PHI, PII, regulated data, high-risk writes, exception paths, or similarly sensitive workflows. It should add policy version/reason codes, approval evidence, reviewer identity where applicable, data-classification context, purpose-of-use or business-justification reference, stronger retention/immutability requirements, and request/response metadata or hashes sufficient to support review.

Full payload capture **MUST** remain data-minimized and policy-governed. High audit does not automatically mean storing sensitive content verbatim when a reference, hash, redacted payload, or downstream system audit record is sufficient.
