# Autonomy Action Taxonomy + Risk Classes (v1)

**Status:** Public working draft for implementation  
**Audience:** Platform architects, policy owners, and operations leads

---

## 1) MVP posture

This v1 taxonomy is intentionally biased toward safe autonomy:
- explicit lanes (`auto` / `queued` / `manual`),
- deterministic pre-action policy decisions,
- high-risk boundaries stay human-owned,
- conservative default behavior when context is missing.

This is not a final enterprise policy language; it is an implementation-ready contract for early-stage governance.

---

## 2) Scope and terms

### In scope
- Classifying proposed actions before execution.
- Assigning a risk class used by policy evaluator and approval routing.
- Defining lane mapping and approval expectations by risk class.
- Applying the same classification model across AWS-hosted agents, AgentCore Runtime agents, Workato/iPaaS/proprietary automation, SaaS-native agents, and legacy/manual exception paths.

### Core terms
- **Action:** a concrete side-effect intent (or privileged read) the system proposes.
- **Action surface:** the runtime or platform where the action is exposed, such as AgentCore Runtime, an AWS-hosted tool gateway, Workato, a proprietary iPaaS/Pega-like platform, or a SaaS-native agent surface.
- **Risk class:** normalized severity bucket (`R0`…`R4`) assigned pre-execution.
- **Lane:** execution path (`auto`, `queued`, `manual`) chosen from risk class + policy.
- **Approval expectation:** who must approve, within what SLA, with what evidence.

---

## 3) Action taxonomy (primary action classes)

### AC0 — Non-side-effect internal compute
Examples:
- local analysis/summarization
- draft generation not sent externally
- internal planning artifacts

Default risk tendency: `R0` unless sensitive handling rules elevate.

### AC1 — Internal communication egress (bounded)
Examples:
- posting to approved internal channels
- internal thread replies in approved ops surfaces

Default risk tendency: `R1` with strict allowlisting.

### AC2 — External/public communication egress
Examples:
- comments/posts on external social/web systems
- outbound messages to non-internal recipients

Default risk tendency: `R2`→`R3` depending on audience, permanence, and auth surface.

### AC3 — Credentialed state-changing write
Examples:
- API writes using tokens/secrets
- modifying third-party SaaS objects
- workflow executions with durable external side effects

Default risk tendency: `R2`→`R3`.

### AC4 — Governance/security boundary change
Examples:
- policy allowlist changes
- auth scope expansion
- secret lease scope/duration expansion
- changes to constitutional governance controls

Default risk tendency: `R4`.

### AC5 — Destructive/irreversible operations
Examples:
- deleting/closing/merging irreversible records
- mass updates with broad blast radius
- production-impacting destructive infra actions

Default risk tendency: `R3`→`R4`.

---

## 4) Risk classes (`R0`–`R4`)

### R0 — Trivial / no external side effect
- No external write, no credential use, reversible/internal only.
- Typical lane: `auto`.

### R1 — Low-risk bounded side effect
- Limited internal side effect in pre-approved surfaces.
- Reversible, small blast radius, no privileged scope expansion.
- Typical lane: `auto` (with audit receipt).

### R2 — Moderate risk
- External or credentialed write but constrained blast radius and clear rollback.
- Typical lane: `queued` (single approver).

### R3 — High risk
- Significant external impact, weak reversibility, or broad blast radius.
- Typical lane: `manual` (explicit human approval + stronger evidence).

### R4 — Critical / governance boundary
- Governance boundary change, privileged scope expansion, severe irreversible risk.
- Typical handling: `manual` with owner-level approval; often default-`deny` without explicit exception.

---

## 5) Deterministic classification rules

## 5.1 Required factors
1. **Surface:** internal approved / external trusted / external public.
2. **Action surface control tier:** Tier 0 platform-owned / Tier 1 mediated iPaaS-proprietary / Tier 2 SaaS-native constrained / Tier 3 legacy exception.
3. **Auth level:** none / delegated user / standard service token / high-privilege credential.
4. **Data sensitivity:** public / internal / sensitive.
5. **Blast radius:** single object / small batch / broad or unknown.
6. **Reversibility:** easy rollback / partial rollback / irreversible.
7. **Governance impact:** no / indirect / direct boundary change.
8. **Auditability:** full immutable evidence / native audit export / partial or unknown.

## 5.2 Hard-elevation rules (must apply)
- If governance impact is direct => `R4`.
- If operation is destructive and irreversible with broad/unknown blast radius => at least `R3`.
- If credential scope expansion is requested => `R4`.
- If required context is missing (unknown target/audience/scope) => elevate one class or route `manual`.
- If a protected-data action cannot preserve delegated user authorization where required => route `manual` or deny until exception is approved.
- If the action surface is Tier 2 or Tier 3 and native audit coverage is partial/unknown => elevate one class unless compensating controls are documented.

## 5.3 Defaulting rules
- Unknowns default conservative: do not classify below `R2`.
- Conflicts resolve to the highest resulting class.
- Policy denylist rules override lane mapping (`deny` beats `auto/queued/manual`).

---

## 6) Lane mapping + approval expectations (v1)

| Risk | Default lane | Approval expectation | Evidence minimum |
|---|---|---|---|
| R0 | `auto` | none | action intent + policy decision id |
| R1 | `auto` | none (notify/audit only) | intent, allowlist match, audit receipt |
| R2 | `queued` | 1 authorized approver | context card, target, rollback plan |
| R3 | `manual` | explicit human approval (owner/delegate) | full context, risk rationale, rollback/mitigation |
| R4 | `manual` + owner gate (or `deny` by default) | owner explicit approval only | governance/security impact statement + exception record |

### Additional lane forcing conditions
- Quiet-hours, incident mode, or elevated threat mode may force `queued`/`manual` for `R1`/`R2`.
- Missing rollback for non-trivial writes forces at least `manual` for `R3` candidates.
- Actions touching constitutional governance controls are always `R4`.

---

## 7) Implementation examples

1. **Post draft status update in approved internal ops channel**
   - Class: AC1
   - Risk: `R1`
   - Lane: `auto`

2. **Reply externally on social platform from monitored account**
   - Class: AC2
   - Risk: `R2` (or `R3` if high-visibility/sensitive topic)
   - Lane: `queued` by default; `manual` when elevated

3. **Write to third-party API with standard scoped token (single object)**
   - Class: AC3
   - Risk: `R2`
   - Lane: `queued`

4. **Bulk external API mutation across many records**
   - Class: AC3/AC5
   - Risk: `R3`
   - Lane: `manual`

5. **Change secret lease scope or duration policy**
   - Class: AC4
   - Risk: `R4`
   - Lane: `manual` owner gate

6. **Trigger Workato workflow that updates one governed SaaS record with scoped credentials and approval hook**
   - Class: AC3
   - Risk: `R2` or `R3` depending on data sensitivity and reversibility
   - Lane: `queued`/`manual`
   - Note: Workato is the Tier 1 enforcement point, not proof that the target resource authorized the user.

7. **SaaS-native agent changes Workday/Zoom configuration where gateway fronting is not available**
   - Class: AC3/AC4 depending on setting
   - Risk: `R3` or `R4`
   - Lane: `manual`
   - Note: requires owner, native audit evidence, least-privilege scope review, and documented compensating controls.

---

## 8) Output contract for evaluator/policy engines

Each classified intent should emit:
- `action_class` (AC0..AC5)
- `risk_class` (R0..R4)
- `lane` (`auto`/`queued`/`manual`)
- `approval_required` (boolean)
- `approval_scope` (`none`/`approver`/`owner`)
- `classification_reasons[]` (deterministic rule ids)
- `evidence_requirements[]`

This creates a stable interface between planning, policy, and approval-routing layers.
