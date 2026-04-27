# AgentCore Prototype Specification v1

> Working spec for the initial AWS AgentCore prototype. This document captures currently locked decisions and open follow-on design work.

## Status
- **State:** draft for review
- **Scope:** prototype architecture and governance decisions
- **Out of scope for this revision:** Terraform implementation details, full API contracts, and final production rollout sequencing

## 1. Agent Runtime and Orchestration

### 1.A. Portability Requirement
Portability **MUST** be treated as a hard requirement.

The preferred pattern **MUST** keep durable orchestration state, approval checkpoints, retry policy, and cross-system workflow control externalized from any single agent runtime where practical. AgentCore Runtime **MAY** be the default hosting plane for agent execution, but prompts, agent contracts, orchestration state, tool-selection boundaries, and workflow definitions **SHOULD** remain portable enough to move to another approved runtime or orchestration surface if required.

Agent application code **MAY** own short-lived task-local reasoning and bounded intra-agent flow. It **SHOULD NOT** become the only place where enterprise workflow state, HITL state, retry behavior, or cross-agent/cross-system process state can be observed or controlled.

### 1.B. Primary Orchestration Model
The platform **SHOULD** use an externalized, durable orchestration/control-plane model for business workflow state and long-running execution.

Framework-native or application-code orchestration **MAY** be used for agent-local behavior, prompt flow, and bounded task execution. Durable business process orchestration, HITL pause/resume, approval routing, retries, escalation, fanout, and cross-system coordination **SHOULD** be represented in explicit orchestration or control-plane constructs that operators can inspect and govern.

AgentCore-native surfaces **MAY** be used where they create clear operational value, but the core agent contract **MUST NOT** depend on a provider-specific orchestration model when a portable externalized pattern is available.

### 1.C. Agent Topology
The initial implementation **MUST** prefer a single bounded-purpose agent per domain capability.

A supervisor or multi-agent pattern **MAY** be introduced later where justified by workflow complexity, but the default starting point **MUST** be a single domain agent with a shared underlying tool fabric.

### 1.D. Runtime Boundary
Agents **SHOULD** interact with enterprise systems through Gateway-managed tools by default.

Direct calls from agents to enterprise systems **MUST NOT** be the default integration pattern for nontrivial external actions. Gateway-managed tools **SHOULD** provide the primary control point for authorization, policy enforcement, auditing, and future portability.

### 1.E. Autonomy and Failure Handling
The default runtime posture **MUST** be bounded retries with fallback.

The platform **MUST NOT** assume aggressive autonomous recovery by default for high-consequence workflows. Where retries are allowed, retry bounds, fallback behavior, and failure closure **MUST** be explicit.

### 1.F. Runtime Standardization
The platform **SHOULD** standardize on one preferred agent framework for v1.

Additional frameworks **MAY** be supported later, but the prototype **MUST** optimize for one preferred implementation path to reduce fragmentation in runtime behavior, prompts, tooling, and operational support.

### 1.G. Model Provider Posture
Amazon Bedrock **SHOULD** be the primary model access path for the prototype.

Other model providers **MAY** be added in later revisions where justified, but multi-provider support **MUST NOT** weaken the initial governance, identity, and runtime contracts.

## 2. Tool, Data-Plane, and External Agent Platform Governance

### 2.A. Enterprise Agent Inventory
All enterprise agent surfaces **MUST** be inventoried in a central registry or registry-backed control plane, regardless of runtime location.

The inventory **MUST** include, where applicable:
- agent or automation name
- owning team
- runtime or host platform
- environment
- data domains accessed
- tool or action surface
- identity mode
- delegated-user support status
- approval and audit capabilities
- gateway or policy-enforcement control mode

This includes agents and agent-like automations running in AWS-hosted application code, Bedrock AgentCore Runtime, Workato or other iPaaS platforms, proprietary automation platforms such as Pega, and SaaS-native agent surfaces such as Workday or Zoom.

### 2.B. Logical Governance Spine, Not Single Product Gateway
The platform **MUST** preserve a single logical governance spine for policy, identity context, audit correlation, approval routing, and platform reporting.

That governance spine **MUST NOT** be equated with a single vendor gateway product. Workato, a proprietary iPaaS, an AWS-hosted tool gateway, or a SaaS-native admin/control surface **MAY** each serve as an enforcement point for the resources they can actually mediate, but no single integration platform **SHOULD** be assumed to front every agent, tool, or SaaS action.

Workato currently publishes AI gateway and MCP capabilities that are relevant to this pattern, including AI gateway collections, MCP servers, connector-backed skills/tools, configurable authorization between Workato and AI agents, and audit/governance positioning. Those capabilities make Workato a credible candidate enforcement point for mediated integration workflows. They do **not**, by themselves, prove that Workato is the technically correct universal gateway for AgentCore-hosted agents, proprietary agent platforms, or SaaS-native agent surfaces that execute inside vendor control planes.

The decision to use Workato as an enforcement point **MUST** be made per workflow/action surface based on technical control fit: delegated identity support, connector semantics, approval hooks, audit export, idempotency/replay controls, operational ownership, latency, failure behavior, and target-resource authorization preservation. Organizational preference for one primary gateway **MUST NOT** override those technical constraints.

### 2.C. Gateway Fronting Expectations
For platform-owned agents and integration platforms where requests can be mediated, nontrivial external actions **SHOULD** route through Gateway-managed tool contracts or an equivalent policy-enforced execution boundary.

For SaaS-native agents or constrained vendor surfaces where full gateway fronting is not feasible, the platform **MUST** still apply compensating controls:
- inventory the agent surface and owner
- restrict OAuth scopes, app permissions, and admin entitlements
- collect native audit/admin events where available
- classify exposed actions by risk
- document unsupported delegation or approval capabilities
- define exception handling, disablement, and review cadence

The Product Owner or accountable platform owner for each external agent platform **MUST** own evidence that their platform aligns to these standards. That ownership **SHOULD** include a recurring review/reporting process covering inventory completeness, scope and permission posture, audit availability, exceptions, open gaps, and remediation status.

Limited fronting **MUST NOT** be treated as invisibility. SaaS-native agents remain governed assets even when the platform can only observe, configure, or constrain them indirectly.

### 2.D. Tool and Resource Contract Registration
Every governed tool or external action surface **SHOULD** have a registered contract containing:
- action name and description
- target system and resource type
- read/write/destructive classification
- risk class
- required identity mode
- required request context fields
- approval requirements
- audit event requirements
- rollback or compensation notes where applicable

The default operating assumption **SHOULD** be that if a tool is registered to an agent, users authorized to use that agent can invoke that tool through the agent. The platform **SHOULD NOT** create a default spiderweb of independent per-user entitlements at the gateway, agent, tool, and resource layers.

Data contracts and target-resource authorization **MUST** still be preserved. The tool should pass through the required caller, delegation, and request context so the system behind the tool can enforce access to the underlying data or action. If tool use must be limited, the preferred control is to gate the tool's registration/availability to the agent or agent profile, rather than layering ad hoc per-user tool entitlements by default.

Per-user or per-group tool constraints **MAY** be introduced for exceptional high-risk cases, but they **SHOULD** be explicit exceptions with owner, rationale, auditability, and troubleshooting guidance.

### 2.E. Integration Control Modes
The prototype **SHOULD** classify each external platform or tool surface into one of these control modes:

| Mode | Pattern | Expected control posture |
|---|---|---|
| Full Feature | Platform-owned agent/runtime/tool gateway | Full request context, policy evaluation, approval, audit, replay protection |
| Partial Feature | Mediated iPaaS or proprietary automation platform | Registered contracts, policy hooks where available, audit correlation, scoped credentials |
| Best Effort | SaaS-native agent or constrained vendor surface | Inventory, native admin controls, least-privilege app scopes, audit ingestion, documented gaps |
| Exception | Legacy/manual exception | Explicit risk acceptance, compensating controls, owner review, migration target |

Promotion from prototype to production **MUST** identify the control mode for every in-scope action surface and document any Best Effort or Exception gaps.

### 2.F. Delegated Authorization Preservation Across Platforms
Where a request originates from a human user and touches data or governed business functions, the integration path **MUST** preserve delegated user authorization when the downstream system supports it.

If an external platform only supports service-account execution, the use case **MUST** be explicitly marked as service-authorized, policy-bounded, and audit-enhanced. Service-account execution **MUST NOT** be used to imply user entitlement where the downstream system did not actually enforce it.

### 2.G. Portability and Vendor Neutrality for Integration Control
The control model **SHOULD** avoid making any one integration vendor the irreversible center of the architecture.

The preferred pattern is a portable policy, identity, request-context, and audit contract that can be implemented by multiple enforcement points: an AWS-hosted gateway, Workato or another iPaaS, proprietary platforms, and SaaS-native controls. This preserves governance consistency without forcing every workflow through one product-shaped choke point.

The governance contract **SHOULD** be declarative where practical. Platform-specific adapters or harnesses **SHOULD** translate the declarative policy intent into native controls, configuration, evidence collection, and reports for each enforcement surface. This allows new platforms to come online by mapping their native capabilities to the shared governance pattern rather than rewriting the enterprise policy model for each platform.

## 3. Identity, Delegation, and Authorization

### 3.A. Authoritative Identity Source
Azure Entra **MUST** be treated as the authoritative identity provider for human users. Where permitted by platform and integration constraints, federated access into AWS services **SHOULD** be mediated through AWS SSO / IAM Identity Center using the approved enterprise federation path.

### 3.B. Human and Service Identity Separation
Human access **MUST** resolve back to the authoritative enterprise identity provider. Service and agent identities **SHOULD** default to IAM roles, workload identities, or equivalent non-user service principals. IAM users **MUST NOT** be the default pattern for agent or service access.

### 3.C. Federated Permission Requirement
For use cases involving protected data or governed business functions, federated authorization **MUST** be supported. Prototype implementations **MAY** use non-sensitive data or reduced-scope substitutes to validate architecture, but the use case **MUST NOT** be considered complete until federated permissions are enforced end-to-end.

### 3.D. Initial Identity Propagation Scope
Initial implementation **MUST** propagate user identity sufficient to support user-scoped authorization decisions. Group or role propagation **MAY** be deferred where not immediately required by downstream systems, but the design **MUST** preserve a clear extension path for future propagation.

### 3.E. Canonical User Identifier
User identity **SHOULD** be human-readable in audit and access records. UPN **MAY** be used as the primary displayed identifier for auditability and operator review. However, the system **SHOULD** retain a stable immutable subject identifier for canonical correlation where available, to prevent ambiguity caused by rename or alias changes.

### 3.F. Human-in-the-Loop Boundaries
A Human-in-the-Loop (HITL) control framework **MUST** exist for actions crossing defined business, security, or operational boundaries. HITL review **MUST** be enforceable for actions with downstream business impact, exception handling, or execution outside a self-contained approved use case.

### 3.G. Data Classification and Exception Awareness
Protected data, sensitive tools, and exception-bearing resources **MUST** support explicit tagging or equivalent classification metadata. Authorization and policy enforcement **MUST** account for such tags. Where feasible, exception awareness **SHOULD** be visible through the service or agent registry.

### 3.H. Environment Boundary Controls
Identities **MAY** hold entitlements across multiple SDLC environments according to approved policy. However, service-to-service and agent-to-service interactions **MUST** enforce environment boundaries by default. Cross-environment access, including development-to-production interaction, **MUST NOT** be assumed or implicitly trusted.

### 3.I. Tool Availability vs. Tool Execution Authority
Tool availability **MUST** be determined by agent registration, environment, and platform policy. Tool execution authority **MUST** be evaluated per request against the applicable caller context, delegated identity context where present, target resource policy, and HITL or other policy gates. User claims **MUST NOT** directly expand the registered capability surface of an agent, but **MAY** further constrain or authorize execution within that surface.

### 3.J. Delegated User Entitlement and Agent Credentials
User-originated requests for protected data or business functions **MUST** execute under the user’s delegated entitlement when such delegation is supported. Agent or service credentials **MUST NOT** be used to bypass user authorization boundaries for actions expected to be performed under user entitlement. Agents **MUST NOT** be granted standing privileges for user-governed actions when delegated user authorization is available. Autonomous agent actions on behalf of a user, without user context, **MUST** be explicitly bounded, tagged, and policy-governed. Autonomous agent actions not operating on behalf of a user **MAY** execute under agent or service identity, but **MUST** remain within explicitly defined service scope, environment boundaries, and policy constraints.

## 4. Authorization and Execution Flow

### 4.1. Canonical Claim and Request Context Set
Each agent request that reaches an execution boundary **MUST** carry a canonical request context sufficient to support authorization, auditing, and policy evaluation.

The baseline context **MUST** include, where applicable:
- immutable subject identifier
- UPN or equivalent human-readable user identifier
- agent identity
- tenant or business context
- environment identifier
- session or invocation identifier
- trace identifier
- delegation mode
- group, role, or entitlement attributes when required for policy evaluation

Where supported, the context **SHOULD** also include additional security-relevant attributes such as authentication strength, token issuer, token age or expiry context, source application, approval state, data-classification hints, and purpose-of-use or business-justification metadata for sensitive workflows.

Claims and request attributes **MUST NOT** be propagated beyond what is required for enforcement, auditing, and downstream authorization.

### 4.2. Delegation Model
The platform **MUST** distinguish between actions executed under delegated user identity and actions executed under agent or service identity.

As the default operating rule:
- access to governed or protected data **MUST** use delegated user entitlement when a user is the originating requester and the downstream system supports delegated authorization
- system, API, infrastructure, and platform-control operations **SHOULD** execute under agent or service identity unless a stronger user-bound control is explicitly required

Where a workflow mixes both data access and platform actions, the implementation **MUST** preserve the identity mode used for each step and **MUST NOT** silently widen a data-access step from delegated user authorization to standing service authorization.

### 4.3. Policy Decision and Enforcement Layers
Access control **SHOULD** be enforced primarily at the agent runtime and/or Gateway boundary, where request context, delegation mode, tool policy, and HITL state can be evaluated together.

Tools **MUST** receive authorization context from upstream rather than inventing or mutating it locally.

Protected resources **MUST** continue to enforce their own native access controls, connection policy, and resource ACLs independent of the agent or tool path. Agent or Gateway authorization **MUST NOT** be treated as a substitute for downstream resource authorization.

The effective decision model **MUST** behave as layered enforcement:
1. agent or Gateway policy determines whether the requested action may be attempted
2. tool boundary determines whether invocation conditions and risk gates are satisfied
3. target resource determines whether the presented identity is authorized for the requested operation

A request **MUST** be denied when any applicable layer denies it.

### 4.4. Audit Contract and Audit Levels
The platform **MUST** define a standard audit contract in agent templates and shared execution surfaces.

Every tool invocation or side-effecting action **MUST** emit audit data sufficient to reconstruct:
- requesting user identity when present
- agent or service identity
- delegation mode
- tool or action invoked
- target system or resource
- environment and tenant context
- policy decision result
- HITL state and outcome where applicable
- execution timestamp and trace correlation identifiers

The platform **SHOULD** define default audit levels at the template level.

At minimum:
- **medium audit** **MUST** be the default baseline for standard enterprise actions
- **high audit** **MUST** be required for PHI, PII, or similarly sensitive workflows

Higher audit levels **MAY** require additional event detail, stronger retention controls, stricter immutable storage requirements, and enhanced approval evidence.

## 5. Open Notes
- This document intentionally separates **agent capability registration** from **per-request execution authority**.
- This document intentionally separates **user-proxy autonomy** from **platform-native automation**.
- Prototype bridges using non-sensitive data are acceptable for architecture validation, but they do not satisfy the end-state requirement for federated authorization.
