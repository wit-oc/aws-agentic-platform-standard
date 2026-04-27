# ACP Implementation Runner Setup v1

## Purpose

Run ACP implementation-phase action items as deterministic initiative steps using the workspace initiative-runner contract.

## Initiative Artifact

- Path: `initiatives/aws-agentic-platform-acp-implementation-2026-03-08.md`
- Source of truth for:
  - task state,
  - progress evidence,
  - blockers and next steps.

## Runner Contract

Use canonical contract from:
- `docs/internal/INITIATIVE_RUNNER.md`

Every run returns exactly one line:
- `ARTIFACT: <concrete delta>`
- `BLOCKED: <reason> | NEXT: <smallest unblocked step>`

## One-Shot Scheduling Pattern

Use one-shot jobs at 15-minute intervals, one job per unchecked task.

Template:

```bash
at now + 15 minutes
```

Runner prompt template:

```text
[cron:<job-id> acp-impl-Tn] Initiative runner turn for `initiatives/aws-agentic-platform-acp-implementation-2026-03-08.md`.
Execute exactly one next unchecked task, update the initiative file, and return exactly one line:
- ARTIFACT: <concrete delta>
- or BLOCKED: <reason> | NEXT: <smallest unblocked step>
Rules: proof-first status reporting; no reminder/coaching output; keep changes reversible.
```

## Blocker Policy

- On blocker:
  - set initiative state to `blocked`,
  - record blocker evidence and smallest unblock step,
  - cancel remaining scheduled jobs for this initiative.

## Validation Before Marking Done

- Check initiative task status lines match actual file changes.
- Ensure each completed task has:
  - touched file path,
  - command/output proof marker.
- Ensure `TASK_BOARD.md` reflects completed setup and remaining tasks.
