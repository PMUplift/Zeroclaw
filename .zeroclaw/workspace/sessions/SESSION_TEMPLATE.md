# Session Log Template
**Location:** `/Users/agent/.zeroclaw/workspace/sessions/SESSION_TEMPLATE.md`
**Purpose:** Reference document describing the TOML format for all agent session logs.
**Actual log format:** TOML — files named `[agent-lowercase]-[YYYYMMDD]-[NNN].toml`

Session logs are written by agents at the RALPH Log phase and parsed by `src/rag/` to extract
memory nodes, quality scores, and flags. **The Rust RAG layer requires valid TOML. Free-form
text will not be parsed and the memory signal is silently lost.**

The `[session]`, `[reflect]`, `[act]`, `[quality]`, and `[flags]` sections are required on
every log. `[[memory_nodes]]` is optional — only include genuine new lessons.

---

## Canonical Naming Conventions

The Rust parser is case-sensitive. Follow these exactly. This is the single source of truth.

| Element | Format | Example |
|---|---|---|
| Session log filename | `[agent-lowercase]-[YYYYMMDD]-[NNN].toml` | `priya-20260221-001.toml` |
| FORGE log filename | `forge-[YYYYMMDD]-[NNN].toml` | `forge-20260221-001.toml` |
| `task_id` field (TOML) | `AGENT-UPPERCASE-YYYYMMDD-NNN` | `PRIYA-20260221-001` |
| `agent` field (TOML) | lowercase | `"priya"` |
| `source_task_ids` (TOML) | UPPERCASE | `["PRIYA-20260221-001"]` |
| Task files (/shared/tasks/) | `[TASK-ID].md` | `PRIYA-20260221-001.md` |
| NNN sequence | Zero-padded, resets per agent per day | `001`, `002`, `003` |
| folder_monitor agent name | `folder_monitor` (lowercase, underscore) | `FM-20260221-001` (task_id uses FM) |

**Rule:** The `agent` TOML field is always lowercase. Task IDs referenced anywhere (task_id,
source_task_ids, filenames) are always UPPERCASE agent prefix. Never mix these.

---

## TOML Template

```toml
[session]
agent = ""              # lowercase: axel | steph | clive | jamie | priya | marcus | folder_monitor
task_id = ""            # UPPERCASE: AGENT-YYYYMMDD-NNN  e.g. "PRIYA-20260221-003"
task_type = ""          # frontend | backend | devops | copy | calendar | orchestration | context-check
timestamp = ""          # ISO8601  e.g. "2026-02-21T14:32:00+10:00"
quality_score = 0       # integer 1–10
delegated_by = ""       # axel | clive | self
phase = ""              # complete | blocked | in-progress
forge_cycle = 0         # which FORGE cycle this task is on (1, 2, or 3)

[reflect]
notes = """
What memory nodes were surfaced. Which were applied. Any conflicts between nodes.
If no relevant nodes found: "No relevant nodes surfaced. Proceeding from first principles."
For Axel: note any block_history nodes surfaced for this task.
"""

[act]
summary = """
What was done. Tools called and their results. Do not fabricate tool output.
Key decisions made and why. If a tool failed, note it.
For Steph: include edit_delta_outcome field (see below).
"""

# Steph only — include this in [act] summary:
# edit_delta_outcome = "explicit_feedback" | "silent_approval_24h" | "revised_version_received"
# edit_delta_notes = "What changed, or 'No delta received — silent approval assumed after 24h'"

[quality]
score = 0           # integer 1–10 — must match session.quality_score
rationale = """
Why this score. Be specific.
Good: "Clean scoping on all queries, auth dependency applied, 12 tests pass."
Bad: "It went well."
"""

# [[memory_nodes]] is a TOML array of tables — repeat the block for each node.
# Omit entirely if this session adds no new lesson. Do not write nodes to fill the section.

[[memory_nodes]]
agent = ""          # lowercase — owner of this node
category = ""       # skill | preference | error | lesson | style
content = ""        # single precise actionable statement an agent can apply
confidence = 0.0    # 0.6 new | 0.72 reinforced once | 0.75 Clive-approved | 0.85 owner-confirmed
source_task_ids = [""]  # UPPERCASE task IDs

[flags]
content = ""        # Specific issues, blockers, scope violations, security concerns.
                    # "None" if clean. Axel reads flags at next Reflect.
                    # If phase = "blocked": describe block reason precisely — Axel includes
                    # this in the next task brief if the task is resumed.
```

---

## Field Rules

**agent** — always **lowercase** in the TOML field. `"priya"`, not `"PRIYA"`.

**task_id** — `AGENT-UPPERCASE-YYYYMMDD-NNN` where NNN is zero-padded and increments per agent
per day. `JAMIE-20260221-001`, `JAMIE-20260221-002`. folder_monitor uses `FM` as the prefix.

**forge_cycle** — integer. Which FORGE cycle this task is currently on. Starts at 1. If a task
is retried after a failed gate, this increments. Maximum is 3 (from config.toml [forge]).
If forge_cycle = 3 and gate fails: Clive writes "❌ Blocked — max iterations reached",
not a standard revision request.

**quality_score** — integer 1–10. Must appear in both `[session]` and `[quality]` and must match.
Self-assessed. Do not round up. Axel (via FORGE) triggers a skill gap review if any agent
scores below 6 on the same task type for three consecutive sessions. Axel also writes a
[GLOBAL] promotion node if any agent scores 9+ for three consecutive sessions on the same type.

**phase** — `complete` means output delivered. `blocked` means escalated. Do not write a final
session log with `in-progress` — only interim logs for long-running tasks use that value.
If `phase = "blocked"`: the flags.content field must describe the block reason precisely.
This becomes part of the next task brief if the task is resumed.

**memory_nodes.content** — the most important field. One precise statement an agent can act on
next time. Not a description of what happened.

| Bad content | Good content |
|---|---|
| "Priya had a scoping issue" | "GET /tasks endpoints inside project context require both organization_id and project_id scoping — not just organization_id" |
| "Component needed accessibility work" | "Interactive drag handles require aria-grabbed and aria-label — not just tabIndex" |
| "Steph's copy was edited" | "Amii removes exclamation marks from professional emails — use a period and let the sentence carry the warmth" |

**confidence starting values:**

| Source | Value |
|---|---|
| Agent self-identified | 0.60 |
| Recurred once | 0.72 |
| Clive-approved pattern | 0.75 |
| Owner-confirmed rule | 0.85 |
| Hardened (3+ reinforcements) | 0.85–1.0 |
| Steph: silent approval (no edit in 24h) | +0.05 to existing node (does not exceed 0.95) |

---

## Example — Priya (complete backend session, Forge cycle 1)

```toml
[session]
agent = "priya"
task_id = "PRIYA-20260221-003"
task_type = "backend"
timestamp = "2026-02-21T16:45:00+10:00"
quality_score = 8
delegated_by = "clive"
phase = "complete"
forge_cycle = 1

[reflect]
notes = """
Framework surfaced 2 nodes:
- [skill, 0.87] "All task queries require organization_id AND project_id scoping when inside project context."
- [error, 0.72] "PRIYA-20260218-001: forgot auth dependency on /tasks/reorder — caught by Clive."
Applied both. Checked auth dependency explicitly before submitting.
"""

[act]
summary = """
Added GET /api/v1/projects/{project_id}/tasks endpoint.
- Route registered in tasks.py router
- Auth dependency applied via Depends(get_current_user)
- organization_id + project_id scoping on query
- Pydantic response model: TaskListResponse (excludes internal fields)
- Tests: happy path, 401 auth failure, 403 wrong org, 200 empty project
- ruff check: 0 issues. pytest: 12 passed.
"""

[quality]
score = 8
rationale = """
All checklist items passed. Response model clean. Scoping correct.
Minor: no pagination — will need it when task count grows. Not flagging as
tech debt yet — Phase 1 scope does not require it.
"""

[[memory_nodes]]
agent = "priya"
category = "lesson"
content = "GET endpoints returning lists in project context require both organization_id and project_id scoping — not just organization_id."
confidence = 0.72
source_task_ids = ["PRIYA-20260221-003"]

[flags]
content = "None"
```

---

## Example — Steph (complete copy session with edit delta)

```toml
[session]
agent = "steph"
task_id = "STEPH-20260221-001"
task_type = "copy"
timestamp = "2026-02-21T10:12:00+10:00"
quality_score = 8
delegated_by = "axel"
phase = "complete"
forge_cycle = 1

[reflect]
notes = """
Read AMII.md and style guide. Framework surfaced 1 node:
- [style, 0.91] "Amii uses em dashes for asides — not parentheses."
Applied. Reviewed prior empty state draft STEPH-20260218-002 for tone reference.
"""

[act]
summary = """
Wrote 4 empty state messages: Projects list, Tasks list, Time log, Dashboard.
Style guide and AMII.md read before drafting. Self-scored before delivery: 8.
Revised "Nothing here yet" to "You haven't logged any time this week." on second pass.
edit_delta_outcome = "silent_approval_24h"
edit_delta_notes = "No edit feedback received within 24 hours. Silent approval assumed. Incremented relevant style node confidence by 0.05."
"""

[quality]
score = 8
rationale = """
Warm tone, verb-first, no corporate phrases. Em dash used correctly throughout.
One phrase — "ready to go" — felt slightly generic but rhythm worked. Left it.
"""

[[memory_nodes]]
agent = "steph"
category = "style"
content = "Empty states work best with a specific timeframe or number: 'this week', '30 seconds', 'yet' — not generic 'nothing here'."
confidence = 0.62
source_task_ids = ["STEPH-20260221-001"]

[flags]
content = "None"
```

---

## Example — Clive (gate failed, forge cycle 2)

```toml
[session]
agent = "clive"
task_id = "CLIVE-20260221-002"
task_type = "backend"
timestamp = "2026-02-21T15:30:00+10:00"
quality_score = 5
delegated_by = "axel"
phase = "blocked"
forge_cycle = 2

[reflect]
notes = """
Forge cycle 2 — this task failed gate once already (CLIVE-20260219-001: missing auth dependency).
Applying deeper scrutiny. Surfaced prior error node. Checking auth dependency explicitly.
"""

[act]
summary = """
Reviewed PRIYA-20260221-002 (GET /api/v1/tasks bulk endpoint).
Ran check_unscoped_queries tool. Result: 2 unscoped queries found in tasks.py lines 87, 112.
Point 7 (Scoping) check: FAILED.
db.query(Task).filter(Task.id == task_id) — missing organization_id filter.
Gate: ❌ Blocked. Returning to Axel with specific revision notes.
"""

[quality]
score = 5
rationale = """
Core logic correct. Tests pass. But Point 7 failure is a hard stop — cannot score higher
while a Critical scoping issue exists regardless of other quality. Second cycle failure on
this task suggests the brief needs to more explicitly call out scoping requirements.
"""

[[memory_nodes]]
agent = "clive"
category = "lesson"
content = "When a task fails the gate twice on scoping issues, add an explicit scoping requirement to the task brief at Assign — do not assume the agent will catch it from SOUL.md alone."
confidence = 0.72
source_task_ids = ["CLIVE-20260221-002", "CLIVE-20260219-001"]

[flags]
content = "GATE FAILED — Cycle 2. Point 7 (Scoping): FAILED. 2 unscoped queries in tasks.py lines 87 and 112. Axel: increment Forge-Cycle to 3 if retrying. If this is cycle 3, escalate to owner — do not retry."
```

---

## folder_monitor (blocked session only)

folder_monitor only writes a session log when it issues a block or hold. Clean proceed manifests
are not logged — they are included in the task brief and discarded.

```toml
[session]
agent = "folder_monitor"
task_id = "FM-20260221-001"
task_type = "context-check"
timestamp = "2026-02-21T09:55:00+10:00"
quality_score = 0       # N/A for gatekeeper — always 0
delegated_by = "axel"
phase = "blocked"
forge_cycle = 1

[reflect]
notes = "Task brief: 'Build Gantt chart view'. SOUL.md read. Phase 3 feature."

[act]
summary = "Scope check triggered at step 2. Remaining steps not run. Out-of-scope block issued."

[quality]
score = 0
rationale = "Gatekeeper function — quality score not applicable."

[flags]
content = "OUT OF SCOPE BLOCK issued. Task: 'Gantt chart view'. SOUL.md: 'Phase 3 — Deferred'. Axel to execute Blocked Task Protocol: freeze task file, notify owner, await decision. Do not retry without owner approval."
```
