# Autonomy Levels Overview (Public)

**Status:** Public-facing working spec  
**Audience:** Operators, stakeholders, and design partners evaluating agent governance

## Why this exists
This document explains how we scale agent autonomy **without** losing human accountability.

Our approach combines three controls:
- **Risk class** (`R0`–`R4`) to classify action impact.
- **Execution lane** (`auto`, `queued`, `manual`) to enforce the right level of oversight.
- **Action-surface control tier** to account for where the agent actually runs: platform-owned runtime, Workato/iPaaS/proprietary platform, SaaS-native agent surface, or legacy exception.

## Autonomy model at a glance

### Level 0 — Observe
- Agent can read context and produce analysis.
- No side effects.
- Typical use: summarization, diagnostics, planning drafts.

### Level 1 — Recommend
- Agent proposes actions and prepares artifacts.
- Human decides before execution.
- Typical use: suggested replies, candidate workflows, implementation plans.

### Level 2 — Auto (bounded)
- Agent may execute **low-risk, reversible** actions automatically.
- Guardrails: policy checks, audit logging, rollback path.
- Typical use: internal housekeeping, routine deterministic operations.

### Level 3 — Queued Approval
- Agent can prepare/queue higher-impact actions.
- Execution requires explicit approver decision in a governed queue.
- Typical use: external messaging, permission changes, cross-system updates.

### Level 4 — Manual / Human-owned
- High-risk or irreversible actions remain human-owned.
- Agent provides evidence and options, but does not execute.
- Typical use: sensitive data operations, destructive changes, high-blast-radius actions.

## Risk class mapping (R0–R4)

- **R0** — informational, no side effects
- **R1** — low-risk internal, reversible
- **R2** — moderate-risk, controlled side effects
- **R3** — high-risk, approval required
- **R4** — critical/high-blast-radius, human-owned

## Lane mapping (default)

- `R0–R1` -> `auto`
- `R2` -> `auto` or `queued` (policy-dependent)
- `R3` -> `queued`
- `R4` -> `manual`

## Non-negotiable controls

- **Policy-first execution:** every action classified before execution.
- **Deterministic approvals:** clear approver identity, decision trail, and timestamps.
- **Auditability:** immutable run/decision trace, or documented native audit/compensating control where full gateway fronting is not technically available.
- **Fail-safe default:** when context is missing, route to stricter lane.
- **Least privilege:** agent/tool/app access scoped to required minimum.
- **Delegated authorization clarity:** do not represent service-account execution as user entitlement.
- **Governance spine over product preference:** Workato or another iPaaS can be an enforcement point, but the autonomy model must not assume one vendor gateway can govern every agent surface.

## What this enables

- Faster execution on low-risk work.
- Predictable human checkpoints for consequential work.
- Clear governance posture for teams adopting agentic systems.
- A practical way to govern agents across AWS/AgentCore, Workato or other iPaaS platforms, proprietary automation, and SaaS-native surfaces such as Workday or Zoom.

---

For implementation details and architecture context, see the other documents in this repository's `docs/` folder.
