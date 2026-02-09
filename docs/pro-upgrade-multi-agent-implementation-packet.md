# Pro Upgrade: Multi-Agent Implementation Packet

## Objective
Use higher-capacity model access to run deeper autonomous workflows that produce measurable household cost reduction, with strict governance and clear ROI checkpoints.

---

## 1) Recommended Operating Model

## Should coordinator remain primary?
**Yes.** Keep one top-level coordinator as the only default ingress for most requests.

Why:
- Consistent policy enforcement
- Better context routing
- Lower duplicate work across agents
- Cleaner auditability

Coordinator responsibilities:
- intent classification
- agent routing
- memory boundary checks
- approval gating
- synthesis of final output

---

## 2) Agent Lineup (with Cosmere-aligned names)

| Role | Working Name | Theme Fit | Core Duties |
|---|---|---|---|
| Coordinator | **Navani** | Systems architect, integrator | Route tasks, orchestrate handoffs, enforce policy/order |
| Food planning + procurement | **Rock** | Food, practicality, provisioning | Meal plans, shopping lists, prep fallback options |
| Calendar + routine optimization | **Steris** | Structured planning, reliability | Schedule-aware planning and anti-chaos reminders |
| Receipt + leakage analysis | **Jasnah** | Analytical rigor, hard truth | Parse receipts, detect spend leakage, weekly recommendations |
| Safety/approval gate | **Taln** | Discipline, guardrail under pressure | Enforce critical-action approvals and policy boundaries |

Optional later additions:
- Home automation tuning: **Raboniel** (systems experimentation)
- Family coordination concierge: **Syl** (lightweight nudges and reminders)

---

## 3) Routing Model: Coordinator-first + selective direct channels

## Default
All requests route through **Navani (Coordinator)**.

## Direct channels (allowed)
Allow direct routing only when all are true:
1. task is narrow/specialized,
2. low-risk (no critical side effects),
3. caller is authorized,
4. output still emits trace metadata.

Examples:
- Direct to Rock: “generate this week’s grocery list from these notes”
- Direct to Steris: “optimize this week schedule + reminders”

Anything cross-domain or with side effects routes back through coordinator.

---

## 4) Agent Definitions (v1)

## Navani (Coordinator)
- Input: user intent, context, constraints, budget/risk profile
- Output: orchestrated plan + consolidated response
- Memory: read global + read metadata indexes, minimal direct writes
- Tool rights: orchestration only, no critical external side effects

## Rock (FoodOps)
- Input: calendar windows, meal preferences, pantry notes, recent food receipts
- Output: weekly plan, shopping list, daily prep/fallback prompts
- Memory: `memory_food`
- Tool rights: notes/calendar read; write shopping/meal notes; no purchasing actions

## Steris (CalendarOps)
- Input: calendar events, routines, known constraints
- Output: schedule-aware reminders and prep windows
- Memory: `memory_calendar`
- Tool rights: calendar read; create/update reminders only if approved policy allows

## Jasnah (ReceiptOps)
- Input: email receipts/confirmations, category rules
- Output: spend leakage digest, renewal warnings, suggested actions
- Memory: `memory_spend_signals`
- Tool rights: email read/parse, note/report generation; no financial transactions

## Taln (SafetyGate)
- Input: proposed action + risk classification + actor identity
- Output: allow/deny/needs-approval token
- Memory: policy/audit metadata only
- Tool rights: policy checks + audit logging only

---

## 5) Workflow Triggers and Schedules

## Weekly (Sunday 6pm ET)
- Rock builds week meal plan + grocery list draft.
- Steris overlays calendar reality and marks busy-night fast meals.

## Daily (4:30pm ET)
- Rock sends “tonight plan + fallback” message.

## Daily (7:30pm ET)
- Jasnah ingests day receipts from email and updates leakage counters.

## Weekly (Friday 5pm ET)
- Jasnah publishes “money leakage + actions” digest.

## On-demand
- “Plan this week” -> coordinator runs Rock + Steris.
- “Where did money leak this week?” -> coordinator runs Jasnah.

---

## 6) KPI Dashboard Template

Track weekly:
- takeout orders (count)
- takeout spend ($)
- grocery adherence (%)
- unplanned convenience buys (count)
- renewal catches before charge (count)
- estimated avoidable spend recovered ($)
- automation acceptance rate (% suggestions followed)
- human time saved (hours)

ROI formula:
`net_value = avoided_spend + value_of_time_saved - incremental_pro_cost`

---

## 7) Go / No-Go Criteria

## Day 14 checkpoint
**Go** if:
- at least 2 workflows running consistently,
- 20%+ reduction in takeout count OR clear early trend,
- no major policy/safety incidents.

**No-go / adjust** if:
- automation quality is low,
- too many false/annoying reminders,
- savings signal unclear.

## Day 30 checkpoint
**Go (scale)** if:
- net positive ROI for month,
- user satisfaction high,
- stable cadence with low manual overhead.

**No-go (pause/reshape)** if:
- ROI flat/negative,
- operational burden outweighs value,
- quality/safety misses persist.

---

## 8) Implementation Sequence (48-hour start)

1. Define agent configs + memory boundaries.
2. Wire coordinator routing rules.
3. Launch Week 1 food loop (Rock + Steris).
4. Add receipt parser baseline (Jasnah).
5. Start KPI logging sheet.
6. Run Day-14 review.

---

## Recommendation Summary
- Keep coordinator-led topology.
- Permit selective direct channels for low-risk specialist asks.
- Use segmented memory + role-specific prompts together.
- Start with food + receipt loops for fastest high-probability ROI.
