# AgentCore Prototype Specification v1

> Working spec for the initial AWS AgentCore prototype. This document captures currently locked decisions and open follow-on design work.

## Status
- **State:** draft for review
- **Scope:** prototype architecture and governance decisions
- **Out of scope for this revision:** Terraform implementation details, full API contracts, and final production rollout sequencing

## 1. Agent Runtime and Orchestration

### 1.A. Portability Requirement
Portability **MUST** be treated as a hard requirement.

The preferred pattern **MUST** keep orchestration logic in application code rather than making Bedrock Agents the primary architectural abstraction. AgentCore Runtime **MAY** be the default hosting plane, but prompts, orchestration graphs, tool-selection logic, and workflow state **SHOULD** remain portable enough to move to another approved runtime if required.

### 1.B. Primary Orchestration Model
The platform **SHOULD** use framework-native orchestration in code as the default execution model.

AgentCore-native surfaces **MAY** be used where they create clear operational value, but the core agent contract **MUST NOT** depend on a provider-specific orchestration model when a portable code-first pattern is available.

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

## 2. Reserved for Forthcoming Tool and Data-Plane Section
This revision leaves Section 2 open so the numbering of the identity decisions remains aligned with the review thread. The next draft should use this section for tool/data-plane exposure, gateway boundaries, and resource access flow.

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

## 4. Next Section to Draft
The next revision **SHOULD** define the request evaluation and execution flow in normative terms.

Recommended follow-on topics:
1. canonical claim set passed to runtime and tools
2. delegation model for service identity vs delegated user identity
3. policy decision point placement and enforcement order
4. audit event contract for every tool invocation and side effect

## 5. Open Notes
- This document intentionally separates **agent capability registration** from **per-request execution authority**.
- This document intentionally separates **user-proxy autonomy** from **platform-native automation**.
- Prototype bridges using non-sensitive data are acceptable for architecture validation, but they do not satisfy the end-state requirement for federated authorization.
