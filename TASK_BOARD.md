# Task Board — aws-agentic-platform-standard

Status legend: [Now] [Next] [Later] [Blocked] [Done]

## [Now]
- [x] Treat `docs/agentcore-prototype-spec-v1.md` as the current authoritative prototype spec.
- [x] Add Section 2 to the prototype spec for tool/data-plane, external platform agents, and gateway-vs-governance posture.
- [x] Add repo reconciliation artifact identifying stale/misaligned docs.
  - Artifact: `docs/agentcore-repo-reconciliation-v1.md`
- [x] Rewrite public entry-point docs to match the authoritative spec:
  - `README.md`
  - `docs/one-page-architecture.md`
- [x] Rewrite architecture baseline sections that previously implied planner/worker-first, Step-Functions-first, or ROSA-first defaults:
  - `docs/aws-agentic-platform-architecture.md`
  - `docs/agentcore-capabilities-implementation-guide.md`
- [x] Patch deployment blueprint with external agent inventory, control tiers, delegated-auth checks, and SaaS-native compensating controls.
  - Artifact: `docs/deployment-blueprint-v1.md`
- [x] Add Workato/iPaaS/Pega/SaaS examples and control-tier language to risk taxonomy and autonomy overview.
- [x] Remove non-enterprise/private multi-agent docs from this enterprise reference repo:
  - `docs/pro-upgrade-multi-agent-implementation-packet.md`
  - `docs/radiant-mode-multi-agent-spec.md`
- [x] Include local `docs/acp-*.md` implementation annex drafts in the PR for total review.

## [Next]
- [ ] Review PR #1 and decide whether ACP annex docs should remain under `acp-*` names or be renamed to AgentCore governance subcontracts.
- [ ] Add validation/tests gate for final doc package if implementation artifacts follow.

## [Later]
- [ ] Add publication checklist once review feedback lands.

## [Blocked]
- none

## [Done]
- Initial AgentCore prototype draft added.
- Repo reconciliation pass completed against current authoritative direction.
- Open PR package prepared for total review.
