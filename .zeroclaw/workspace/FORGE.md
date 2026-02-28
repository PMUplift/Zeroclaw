# FORGE Loop — Multi-Agent Orchestration
**Location:** `/Users/agent/.zeroclaw/workspace/FORGE.md`
**Read by:** Axel at every orchestration cycle. Clive at every Audit phase.
**Relationship to RALPH:** FORGE is the outer loop — coordinates tasks across the team.
RALPH is the inner loop — each agent's individual execution cycle.
Every agent runs RALPH. Only FORGE coordinates between them.
**Last Updated:** February 2026

---

## What FORGE Is

FORGE applies heat and pressure to a task until it is hard enough to ship — or escalates
when it isn't. It asks one question: **has this task passed the release gate, or does it
need another cycle?**

FORGE wraps RALPH. When Axel assigns a task, one FORGE cycle begins. Inside it, each
agent runs their own RALPH cycle. FORGE reads the results, applies the gate, and decides:
**ship, retry, or escalate.**

```
FORGE Cycle (outer — owned by Axel):
  F — Focus       → Axel scopes the task, runs folder_monitor, checks agent fitness
  O — Orchestrate → Axel assigns; agents execute their RALPH cycles
  R — Review      → Clive applies the Seven-Point Release Gate
  G — Gate        → Pass → Ship. Fail → retry (max 3). Fail at cycle 3 → escalate.
  E — Evolve      → Axel extracts lessons, writes [GLOBAL] nodes, updates ETA heuristics

RALPH Cycle (inner — runs inside O above):
  R — Reflect → A — Act → L — Log → P — Prune → H — Harden
```

---

## Loop Parameters

`config.toml [forge]` — keep in sync with this file.

| Parameter | Value | Meaning |
|---|---|---|
| `max_iterations` | 3 | FORGE cycles before auto-escalation to owner |
| `iteration_cooldown_minutes` | 0 | Delay between cycles (0 = immediate retry) |
| `escalation_channel` | "axel" | Who receives escalation when max_iterations is hit |
| `gate_pass_required` | true | Cannot ship without Clive's explicit sign-off |
| `blocked_task_timeout_hours` | 24 | Owner response window before Axel re-flags |
| `success_promotion_threshold` | 3 | Consecutive 9+ scores to promote agent as preferred assignee |
| `forge_log_enabled` | true | Axel writes a FORGE log per cycle (separate from RALPH logs) |

---

## Phase Definitions

### F — Focus

**Owner:** Axel.

1. Read the task brief.
2. Invoke folder_monitor. If it returns `blocked` → execute **Blocked Task Protocol**. Stop.
3. Check agent's recent quality scores. If below 6 on this task type for 3 consecutive
   sessions → flag to owner before assigning.
4. Write task file to `/Users/agent/Projex-Axels-Team/shared/tasks/[TASK-ID].md`
   with this header:

```
Task-ID:       [AGENT-YYYYMMDD-NNN]
Forge-Cycle:   1                    ← increment on every retry, max 3
Assigned-To:   [agent]
Status:        assigned             ← assigned | in-progress | blocked | approved-by-clive | shipped | escalated
Created:       [ISO8601]
Last-Updated:  [ISO8601]
```

---

### O — Orchestrate

**Owner:** Axel (assigns). Each agent (executes RALPH cycle).

- Delegate task with full brief including any prior block reasons or failure notes.
- Agents run RALPH and write session logs.
- If any session log shows `phase = "blocked"` → execute **Blocked Task Protocol** immediately.
- If a session log is missing past 2× the agent's historical average → flag to owner via FLAG format.

---

### R — Review

**Owner:** Clive.

Clive applies the **Seven-Point Release Gate** (see clive-SKILL.toml).

- All 7 points pass → verdict `✅ Ready to ship`. Update task: `Status: approved-by-clive`.
- Any point fails → verdict `⚠️ Issues flagged` or `❌ Blocked`. Return to Axel with
  specific revision notes. Axel increments `Forge-Cycle` and returns to Focus.
- If `Forge-Cycle` is already 3 and gate fails → verdict `❌ Blocked — max cycles reached`.
  Axel executes **Max Cycles Escalation Protocol**. Do not retry.

---

### G — Gate

**Owner:** Axel (routing decision after Clive's verdict).

| Gate result | Forge-Cycle | Action |
|---|---|---|
| ✅ Pass | Any | Proceed to Evolve → Ship |
| ❌ Fail | 1 or 2 | Increment Forge-Cycle → return to Focus |
| ❌ Fail | 3 | Execute Max Cycles Escalation Protocol |
| 🔴 Blocked | Any | Execute Blocked Task Protocol |

---

### E — Evolve

**Owner:** Axel, after Ship or escalation.

1. Read all session logs from this FORGE cycle.
2. **Skill gap check:** Agent below 6 on same task_type for 3 consecutive sessions
   → trigger skill gap review (see axel-SKILL.toml).
3. **Positive promotion:** Agent at 9+ on same task_type for 3 consecutive sessions
   → write [GLOBAL] skill node. Update agent profile: `preferred_for = [task_type]`.
4. **Cross-agent lessons:** Any lesson applying to 2+ agents → write [GLOBAL] node.
5. **ETA heuristic:** Log actual vs estimated time. Update routing heuristics.
6. Write FORGE session log. Increment FORGE cycle count in `memory_hygiene_state.json`.

---

## Blocked Task Protocol

Activates when folder_monitor returns `blocked`, or any agent session log has
`phase = "blocked"`.

**Axel's steps:**

1. **Freeze** — update task file: `Status: blocked`, record blocker, reason, and
   `Owner-Response-Required-By: [now + 24h]`

2. **Notify** — draft owner notification via Steph:
   ```
   ⚠️ FLAG — Task Blocked: [task_id]
   Blocked by: [agent or folder_monitor]
   Reason: [plain English]
   Impact: [what cannot proceed]
   Options: [descope / defer / approve exception]
   Response needed by: [datetime]
   ```
   Steph sends directly if recipient is `requested2019@gmail.com`. Otherwise Steph drafts,
   Axel reviews before sending.

3. **Wait.** No response in 24h → re-flag. Repeat at 48h.

4. **On owner decision:**
   - Exception approved → unfreeze, increment `Forge-Cycle`, return to Focus
   - Deferred → move to `/shared/tasks/deferred/`, log `phase = "deferred"`
   - Cancelled → delete task file, log `phase = "cancelled"`, write memory node

5. **On resumption** — include block reason in new task brief so agent's Reflect phase
   surfaces it automatically.

---

## Max Cycles Escalation Protocol

Triggered when `Forge-Cycle` would exceed 3. The task has failed the release gate three
times — the brief, scope, assignment, or architecture is wrong.

**Axel's steps:**

1. Set `Status: escalated`.
2. Write FORGE session log with `phase = "escalated"`.
3. Draft owner notification via Steph:
   ```
   ⚠️ FLAG — Escalation Required: [task_id]
   Issue: Failed release gate [N] times.
   Failure pattern: [what Clive flagged each cycle]
   Root cause hypothesis: [ambiguous brief / skill gap / scope conflict / architecture mismatch]
   Recommended action: [rewrite brief / reassign / decompose / defer]
   Blocking: [what cannot ship until resolved]
   ```
4. Do not retry without explicit owner approval.
5. After owner decision → write memory node, `confidence = 0.85`, `category = "lesson"`.

---

## FORGE Session Log Format

One log per completed FORGE cycle, written by Axel.
**Filename:** `forge-[YYYYMMDD]-[NNN].toml`
**Location:** `/Users/agent/.zeroclaw/workspace/sessions/`

```toml
[forge_session]
task_id = ""
forge_cycle = 0           # 1, 2, or 3
timestamp_start = ""      # ISO8601
timestamp_end = ""        # ISO8601
agents_involved = []      # e.g. ["clive", "priya", "marcus"]
gate_result = ""          # passed | failed | escalated | blocked | deferred | cancelled
phase = ""                # shipped | escalated | blocked | deferred | cancelled

[forge_learn]
quality_scores = {}       # e.g. { priya = 8, clive = 9, marcus = 8 }
skill_gap_triggers = []   # agent names where gap review was triggered
global_nodes_written = [] # content of [GLOBAL] nodes written this cycle
eta_variance = ""         # "estimate: 2h, actual: 3h 20m"

[forge_flags]
content = ""              # escalations, blocks, owner decisions needed — or "None"
```

---

## Naming Conventions — Canonical Reference

Single source of truth. The Rust parser is case-sensitive.

| Element | Format | Example |
|---|---|---|
| Session log filename | `[agent-lowercase]-[YYYYMMDD]-[NNN].toml` | `priya-20260221-001.toml` |
| FORGE log filename | `forge-[YYYYMMDD]-[NNN].toml` | `forge-20260221-001.toml` |
| `task_id` TOML field | `AGENT-YYYYMMDD-NNN` (UPPERCASE) | `PRIYA-20260221-001` |
| `agent` TOML field | lowercase | `"priya"` |
| `source_task_ids` | UPPERCASE | `["PRIYA-20260221-001"]` |
| Task files | `[TASK-ID].md` | `PRIYA-20260221-001.md` |
| Sequence (NNN) | Zero-padded, resets per agent per day | `001`, `002`, `003` |
| folder_monitor task_id prefix | `FM` | `FM-20260221-001` |

---

## brain.db Seed Nodes

Seed before first autonomous run via `src/memory/seed.rs` or equivalent bootstrap script.
These rules must not be learned from failure.

```toml
[[seed_nodes]]
agent = "priya"
category = "skill"
content = "Every database query returning user data MUST filter by organization_id. No exceptions."
confidence = 0.85
reinforcement_count = 5
source_task_ids = ["SEED-001"]

[[seed_nodes]]
agent = "axel"
category = "skill"
content = "Phase 2 work must not begin until the owner has explicitly confirmed all 10 Phase 1 gate checks pass."
confidence = 0.85
reinforcement_count = 5
source_task_ids = ["SEED-002"]

[[seed_nodes]]
agent = "steph"
category = "skill"
content = "send_email routes to requested2019@gmail.com only. All other recipients: draft only, flag to Axel. Cannot be overridden."
confidence = 0.85
reinforcement_count = 5
source_task_ids = ["SEED-003"]

[[seed_nodes]]
agent = "axel"
category = "skill"
content = "[GLOBAL] Nothing deploys without Clive's explicit sign-off. Task Status must be 'approved-by-clive' before Marcus begins."
confidence = 0.85
reinforcement_count = 5
source_task_ids = ["SEED-004"]

[[seed_nodes]]
agent = "axel"
category = "skill"
content = "[GLOBAL] After 3 failed gate attempts, escalate to owner. Persistent failure means the brief or assignment is wrong — not that another iteration will fix it."
confidence = 0.85
reinforcement_count = 5
source_task_ids = ["SEED-005"]
```

---

## FORGE vs RALPH — Quick Reference

| | FORGE | RALPH |
|---|---|---|
| Scope | Outer — cross-agent orchestration | Inner — single agent execution |
| Owner | Axel | Each agent individually |
| Log file | `forge-[YYYYMMDD]-[NNN].toml` | `[agent]-[YYYYMMDD]-[NNN].toml` |
| Memory writes | [GLOBAL] nodes, ETA heuristics, skill gap/promotion flags | Agent skill/error/lesson nodes |
| Max cycles | 3 (then escalate) | 1 per task (blocked tasks escalate to FORGE) |
| Exit | Gate passed + shipped, OR max cycles, OR blocked/deferred | Complete OR blocked |

---

*FORGE.md maintained by Axel and the owner. Changes must be reflected in both FORGE.md
and `config.toml [forge]`.*
