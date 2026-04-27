# ACP IaC + Promotion Gates v1

## Scope

Defines Terraform-first module mapping and release promotion controls for ACP services across `dev -> stg -> prod`.

## Flow Contract

1. Merge to main triggers infra validation pipeline.
2. `dev` plan/apply runs with policy/security checks.
3. `stg` promotion requires:
  - successful dev validation,
  - architecture contract checks,
  - approval from platform owner.
4. `prod` promotion requires:
  - successful staging canary,
  - no unresolved high-severity findings,
  - explicit manual approval.

## Decision Contract

Promotion status per env:
- `pending`
- `blocked`
- `approved`
- `deployed`

Gate checks:
- `terraform fmt -check`
- `terraform validate`
- static IaC security scan (`tfsec`/equivalent)
- drift check against active workspace state
- ACP contract doc check (required docs present and versioned)

## Failure/Blocker Handling

- Any failed gate => env status `blocked`.
- Blocked promotions must include:
  - failing check id,
  - failing artifact/log pointer,
  - smallest unblock step.
- Prod rollback path must be declared before promotion.

## Evidence/Audit Requirements

Per promotion, capture:
- commit SHA,
- environment,
- gate results,
- approver identity,
- deployment timestamp,
- rollback reference.
