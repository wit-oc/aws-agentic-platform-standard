# ACP Pilot Build-Out Runbook v1

## Scope

Defines operational build-out for ACP pilot deployment across `dev -> stg -> prod` with explicit go/no-go controls.

## Flow Contract

### Dev
1. Deploy policy gateway and approval broker contracts.
2. Run validation matrix V1..V9.
3. Validate audit propagation on one low-risk and one high-risk scenario.

### Stg
1. Promote only if all required dev checks pass.
2. Re-run validation matrix.
3. Perform rollback drill and approval-SLA drill.

### Prod Pilot
1. Enable low-risk workflow lane first.
2. Enable high-risk lane with mandatory approvals.
3. Monitor SLOs and audit completeness for pilot window.

## Decision Contract

Go/no-go gates:
- `GO_DEV_TO_STG`: all required dev checks `PASS`.
- `GO_STG_TO_PROD`: all required stg checks `PASS` + rollback drill success.
- `GO_PILOT_EXPAND`: pilot SLO and audit-completeness thresholds met.

## Failure/Blocker Handling

Rollback triggers:
- policy decision mismatch against contract,
- approval resume failure above threshold,
- missing correlation IDs on required events,
- audit sink failure without successful replay.

Rollback steps:
1. Freeze new high-risk actions.
2. Revert ACP runtime to prior validated release.
3. Preserve and export incident evidence bundle.
4. Open blocker artifact with exact failure markers.

## Evidence/Audit Requirements

Per environment, operator proof pack must include:
- deployed version identifiers,
- validation matrix result summary,
- sample trace IDs for low/high-risk paths,
- approval SLA metrics,
- rollback test output (stg/prod-prep).

Required completion format:
- `ARTIFACT: env=<dev|stg|prod> <concrete validated delta>`
- `BLOCKED: <reason with evidence> | NEXT: <smallest unblocked step>`
