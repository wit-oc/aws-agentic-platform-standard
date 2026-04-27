# ACP Implementation Validation Matrix v1

## Scope

Defines required validation checks for ACP implementation-phase artifacts prior to promotion.

## Decision Contract

Promotion decision is deterministic:
- `PASS`: all required checks pass.
- `FAIL`: any required check fails.
- `BLOCKED`: environment dependency unavailable; release cannot proceed.

## Validation Matrix

| Check ID | Category | Scenario | Expected Result | Gate |
|---|---|---|---|---|
| V1 | Contract | Policy evaluate returns `ALLOW` for low-risk allowed action | Response contains deterministic decision fields + `policy_decision_id` | Required |
| V2 | Contract | Policy evaluate returns `DENY` for blocked action | Denied response includes reasons + no execution | Required |
| V3 | Contract | Policy evaluate returns `APPROVAL_REQUIRED` for high-risk action | Approval path triggered with `approval_id` | Required |
| V4 | Replay | Duplicate request with same idempotency key/payload | Same decision, no duplicate side effects | Required |
| V5 | Replay | Duplicate request with same key/different payload | `409 REPLAY_DETECTED` | Required |
| V6 | Approval | Approval submitted before expiry | Workflow resumes successfully | Required |
| V7 | Approval | Approval times out | Status `EXPIRED`, execution blocked | Required |
| V8 | Audit | Correlation IDs propagated end-to-end | All events contain required IDs | Required |
| V9 | Audit | Audit sink unavailable | Flow blocked with failure event + replay entry | Required |
| V10 | Promotion | Dev -> Stg promotion gate | Block if any required check is FAIL/BLOCKED | Required |

## Flow Contract

Execution order:
1. Run V1..V9 in `dev`.
2. Remediate any failures.
3. Re-run full suite.
4. If all required checks pass, allow promotion gate V10.

## Failure/Blocker Handling

- `FAIL`: open defect artifact with check ID, evidence, and owner.
- `BLOCKED`: capture dependency blocker, timestamp, and smallest unblocked step.
- Promotion is denied until all required checks are `PASS`.

## Evidence/Audit Requirements

For each check, evidence bundle must include:
- command/request payload,
- observed response/output snippet,
- timestamp and environment,
- traceable artifact path.

Minimum reporting line:
- `ARTIFACT: Vx PASS <artifact-path-or-output-marker>`
- or `BLOCKED: Vx <reason> | NEXT: <smallest unblocked step>`
