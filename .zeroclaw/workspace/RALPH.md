# RALPH Loop — Individual Agent Task Cycle Configuration
**Location:** `/Users/agent/.zeroclaw/workspace/RALPH.md`
**Read by:** Agents at Reflect phase via `read_ralph` tool. Not parsed by the Rust binary.
**Framework config:** Loop parameters live in `config.toml [ralph]` — that is what the Rust engine reads.
**Relationship to FORGE:** RALPH is the inner loop. Each agent runs RALPH for their own task.
FORGE is the outer loop — it coordinates across agents, applies the release gate, and decides
whether to ship or retry. See FORGE.md for the full orchestration picture.
**Last Updated:** February 2026

---

## What RALPH Is

RALPH is each agent's **individual task execution cycle**. When Axel assigns a task, the
receiving agent runs RALPH from start to finish. When Clive assigns a subtask to Priya,
Priya runs RALPH. When Steph drafts copy, she runs RALPH.

RALPH does not know about other agents. It is a single-agent loop. The coordination layer
above it is FORGE — Axel's orchestration loop that wraps all RALPH cycles together.

```
RALPH Cycle (per agent):
  R — Reflect   → Load memory nodes. Review prior errors on this task type.
  A — Act       → Execute the task. Use tools. Do not fabricate output.
  L — Log       → Write a valid TOML session log. Quality score required.
  P — Prune     → (Framework-triggered every 10 cycles) Clean memory.
  H — Harden    → (Framework-triggered after Prune) Promote proven nodes.
```

---

## Loop Parameters

These mirror the `[ralph]` section in `config.toml`. Both must stay in sync.

| Parameter | Value | Meaning |
|---|---|---|
| `prune_cycle_frequency` | 10 | Prune triggers after every 10 task cycles per agent |
| `harden_confidence_threshold` | 0.85 | Minimum confidence for a node to graduate to standing rule |
| `harden_reinforcement_count` | 3 | Times a pattern must be reinforced before Harden promotes it |
| `session_retention_days` | 90 | Raw session logs archived after 90 days |
| `cross_agent_learning` | true | [GLOBAL] tagged nodes are surfaced to all agents at Reflect |
| `quality_score_logging` | true | Agents log quality scores (1–10) to every session |
| `folder_monitor_enabled` | true | folder_monitor runs before every task delegation |

---

## Phase Definitions — What Happens at Each Phase

### R — Reflect

**Framework action:** `memory_loader.rs` queries `brain.db` via `src/rag/` semantic search
using the current task brief as the query. Returns ranked memory nodes relevant to this
task type, agent, and recent error history.

**Agent obligation:** Use the surfaced nodes. Do not ignore them. If a [GLOBAL] node is
present, it takes priority over agent-specific nodes of equal confidence. If no nodes are
surfaced, note: "No relevant nodes surfaced. Proceeding from first principles."

**What to look for (per agent type):**

| Agent | Priority memory categories at Reflect |
|---|---|
| axel | routing_history, skill_gap_flags, eta_heuristics, block_history, [GLOBAL] |
| steph | style_patterns, prior_drafts, edit_deltas, rejected_phrases, [GLOBAL] |
| clive | architecture_decisions, tech_debt_flags, dev_quality_profiles, [GLOBAL] |
| jamie | component_patterns, clive_feedback, accessibility_lessons, [GLOBAL] |
| priya | scoping_patterns, auth_lessons, migration_patterns, security_flags, [GLOBAL] |
| marcus | deployment_history, incident_postmortems, infrastructure_changes, [GLOBAL] |
| folder_monitor | out_of_scope_flags, missing_dependency_history |

**If a `block_history` node is surfaced (Axel only):** This means the current task was
previously blocked. Read the block reason before assigning. Include it in the task brief
so the assigned agent's Reflect phase surfaces it.

---

### A — Act

**Framework action:** Executes the agent's SKILL.toml prompt with full context (memory
nodes + task brief + tool results). Calls registered tools via `src/tools/`.

**Agent obligation:** Execute the task. Apply all rules in SKILL.toml. Use tools — do not
fabricate tool output. Invoke folder_monitor (Axel and Clive only) before delegating.

**Exit conditions from Act:**
- Task complete → proceed to Log with `phase = "complete"`
- Task blocked → proceed to Log with `phase = "blocked"`, then FORGE Blocked Task Protocol activates
- Task requires owner input → proceed to Log with `phase = "blocked"`, flag clearly

---

### L — Log

**Framework action:** Captures agent output to `workspace/sessions/[agent]-[YYYYMMDD]-[NNN].toml`.
The `src/rag/` module parses this TOML file to extract memory nodes and index them in `brain.db`.

**Agent obligation:** Write a valid TOML session log using the structure in `SESSION_TEMPLATE.md`.
Include quality score, any new memory nodes, and flags. A non-TOML or malformed session log
cannot be parsed by `src/rag/` — the learning signal is lost silently.

**Filename format:** `[agent-lowercase]-[YYYYMMDD]-[NNN].toml`
**Task ID format:** `AGENT-UPPERCASE-YYYYMMDD-NNN`
See FORGE.md Naming Conventions section for the full canonical reference.

---

### P — Prune

**Framework action:** Triggered automatically every 10 task cycles per agent. Reads
`state/memory_hygiene_state.json` to determine when each agent last pruned. Calls
`src/memory/` hygiene routines.

**What gets pruned:**

| Category | Prune rule |
|---|---|
| `style` | Merge nodes with identical or near-identical content into single node with reinforcement_count summed |
| `error` | Archive errors that have not recurred in 30+ days |
| `lesson` | Merge lessons covering the same failure pattern |
| `skill` | Consolidate overlapping architectural/pattern nodes |
| `preference` | Keep all — owner preferences are never pruned |
| `[GLOBAL]` | Merge global nodes where scope overlaps; never delete without Axel review |

**Hard rule:** Never delete a node with `confidence >= 0.85` or `reinforcement_count >= 5`.
Archive it instead.

---

### H — Harden

**Framework action:** After Prune, promotes provisional nodes to standing rules in `brain.db`
where confidence and reinforcement thresholds are met.

**Promotion criteria:**
```
confidence >= 0.85
AND reinforcement_count >= 3
AND no contradicting node exists with higher confidence
```

**What hardening means:** A hardened node is surfaced to the agent at every Reflect phase
for this task type — not just when semantically similar tasks arrive. It becomes a standing rule.

**Demotion:** If a hardened node is contradicted by 3+ subsequent sessions (agent acts contrary
to it and scores well), confidence drops and the node returns to provisional status for Axel's review.

---

## Session Log Format

Logs are **TOML**, named `[agent-lowercase]-[YYYYMMDD]-[NNN].toml`. Malformed logs are
silently dropped by `src/rag/` — the learning signal is lost.

Full template, field rules, and worked examples:
`/Users/agent/.zeroclaw/workspace/sessions/SESSION_TEMPLATE.md`

```toml
[session]        # agent (lowercase), task_id (UPPERCASE), forge_cycle, task_type, timestamp, quality_score, delegated_by, phase
[reflect]        # memory nodes surfaced and applied
[act]            # what was done, tools called, decisions made
[quality]        # score (integer 1–10), rationale (specific)
[[memory_nodes]] # optional — genuine new lessons only
[flags]          # specific issues or "None"
```

---

## Memory Categories — Full Reference

| Category | Used by | What it stores |
|---|---|---|
| `skill` | All agents | Approved patterns, techniques, architectural decisions |
| `preference` | steph, axel | Owner voice patterns, communication preferences, routing preferences |
| `error` | All agents | Mistakes made, how they were caught, what the fix was |
| `lesson` | All agents | Generalised rules derived from errors or feedback |
| `style` | steph | Voice patterns, phrasing, edit deltas, exclusion list items |
| `[GLOBAL]` | axel (writes), all (reads) | Cross-agent lessons surfaced at all agents' Reflect phases |

---

## Cross-Agent Learning Protocol

When Axel's Evolve phase surfaces a lesson applying to 2+ agents, he writes it tagged `[GLOBAL]`.

**Format:**
```
[GLOBAL] [single precise actionable statement]
Example: "[GLOBAL] Kanban drag-and-drop tasks require 40% longer ETA estimates than equivalent UI tasks."
```

**Rules:** Written by Axel only. Must be actionable, not an observation. Must apply to 2+ agents.
The RAG layer indexes [GLOBAL] nodes separately — all agents see them at Reflect regardless of author.

**Positive promotion:** Agent scores 9+ on same task type for 3 consecutive sessions → Axel writes
a [GLOBAL] skill node and marks agent as `preferred_for` that task type. See FORGE.md E — Evolve.

---

## folder_monitor Integration

folder_monitor runs as a pre-task gate. It is not a member of the RALPH loop in the same
way as the six agents — it does not maintain a quality score or a long-term memory profile.
It runs synchronously and returns before the task begins.

**Where in the loop it sits:**
```
R (Reflect) → folder_monitor invoked → context manifest returned → A (Act) begins
```

folder_monitor's output is included in the acting agent's task brief. The agent reads the
manifest during their own Reflect phase. If folder_monitor issues a block, the RALPH loop
does not proceed to Act. The block event is emitted to the FORGE layer — Axel executes the
**Blocked Task Protocol** defined in FORGE.md.

**folder_monitor does not handle its own blocks.** It detects and reports. FORGE resolves.

---

## Per-Agent RALPH Summary

| Agent | Reflect priority | Log fields | Prune target | Harden target |
|---|---|---|---|---|
| axel | Routing history, skill gaps, ETA accuracy, block_history, [GLOBAL] | task_id, agent, ETA, rationale, quality_score | Redundant routing patterns | Routing rules >85% success |
| steph | Style guide, edit deltas, voice profile, [GLOBAL] | task_type, word_count, style_score, revisions | Redundant style nodes | Patterns reinforced 3+ times with no edits |
| clive | Architecture decisions, dev quality, tech debt, [GLOBAL] | task_id, dev_assigned, verdict, issues, quality_score | Duplicate architectural lessons | Review criteria that catch same error 3+ times |
| jamie | Component patterns, Clive feedback, accessibility, [GLOBAL] | component_name, breakpoints, a11y_pass, quality_score | Duplicate component patterns | Patterns approved without revision 3+ times |
| priya | Scoping patterns, auth lessons, migrations, security, [GLOBAL] | endpoints, migrations, test_coverage, security_flags, quality_score | Duplicate scoping rules | Query patterns with zero scoping issues |
| marcus | Deployment history, incidents, infrastructure, [GLOBAL] | version, checklist_pass, issues, rollback_available, quality_score | Duplicate deployment patterns | Deployment patterns with zero incidents |

---

## Quality Score Reference

Used by all agents. Logged to every session. Feeds Axel's efficiency monitoring via FORGE.

| Score | Meaning |
|---|---|
| 9–10 | Exceptional. No revisions needed. Promote patterns to memory immediately. |
| 7–8 | Good. Minor improvements possible. Standard memory write. |
| 5–6 | Acceptable but sub-target. Revise before delivering if possible. Flag patterns. |
| 3–4 | Below standard. Delivered under constraint. Write error node. |
| 1–2 | Failed output. Write error node. Axel flags to owner if pattern repeats. |

Axel (via FORGE) triggers a skill gap review when any agent scores below 6 on the same task
type for three consecutive sessions. Axel also writes a [GLOBAL] promotion node when any
agent scores 9+ on the same task type for three consecutive sessions.

---

## Hygiene State Tracking

The framework tracks RALPH loop cycle counts in:
`/Users/agent/.zeroclaw/workspace/state/memory_hygiene_state.json`

Schema:
```json
{
  "agents": {
    "axel":   { "cycle_count": 0, "last_prune": null, "last_harden": null },
    "steph":  { "cycle_count": 0, "last_prune": null, "last_harden": null },
    "clive":  { "cycle_count": 0, "last_prune": null, "last_harden": null },
    "jamie":  { "cycle_count": 0, "last_prune": null, "last_harden": null },
    "priya":  { "cycle_count": 0, "last_prune": null, "last_harden": null },
    "marcus": { "cycle_count": 0, "last_prune": null, "last_harden": null }
  },
  "global_cycle_count": 0,
  "last_global_prune": null
}
```

`cycle_count` increments on every completed RALPH task. When `cycle_count % prune_cycle_frequency == 0`,
Prune and Harden run automatically. FORGE cycle counts are tracked separately in the FORGE
session logs and are not reflected here.

---

*RALPH.md is maintained by the owner and Axel. Changes to loop parameters must be reflected
in both RALPH.md and `config.toml [ralph]` to stay in sync. For outer orchestration loop
parameters, see FORGE.md and `config.toml [forge]`.*
