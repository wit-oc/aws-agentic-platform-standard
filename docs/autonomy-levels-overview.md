# Autonomy Levels Overview (Public)

**Status:** Public-facing working spec  
**Audience:** Operators, stakeholders, and design partners evaluating agent governance

## Why this exists
This document explains how we scale agent autonomy **without** losing human accountability.

Our approach combines two controls:
- **Risk class** (`R0`–`R4`) to classify action impact.
- **Execution lane** (`auto`, `queued`, `manual`) to enforce the right level of oversight.

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
- **Auditability:** immutable run/decision trace.
- **Fail-safe default:** when context is missing, route to stricter lane.
- **Least privilege:** agent/tool access scoped to required minimum.

## What this enables

- Faster execution on low-risk work.
- Predictable human checkpoints for consequential work.
- Clear governance posture for teams adopting agentic systems.

---

For implementation details and architecture context, see the other documents in this repository's `docs/` folder.
