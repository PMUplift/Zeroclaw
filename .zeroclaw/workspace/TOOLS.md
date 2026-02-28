# Tools Registry
**Location:** `/Users/agent/.zeroclaw/workspace/TOOLS.md`
**Read by:** Axel (routing), Clive (delegation), agents at Reflect
**Last Updated:** February 2026 — aligned to SKILL.toml v0.4.0

All tools are registered in each agent's SKILL.toml. This document is the human-readable registry.
If a tool command changes in SKILL.toml, update this file to match.

---

## axel

| Tool | Command | Purpose |
|---|---|---|
| `git_log` | `git -C /Users/agent/Projex-Axels-Team/app-project log --oneline -10` | Last 10 commits |
| `git_status` | `git -C /Users/agent/Projex-Axels-Team/app-project status` | Current git status |
| `git_diff` | `git -C /Users/agent/Projex-Axels-Team/app-project diff HEAD~1` | Latest diff |
| `list_tasks` | `ls /Users/agent/Projex-Axels-Team/shared/tasks/` | All active task files |
| `read_soul` | `cat /Users/agent/.zeroclaw/workspace/skills/SOUL.md` | SOUL.md product context |
| `read_forge` | `cat /Users/agent/.zeroclaw/workspace/FORGE.md` | FORGE orchestration loop — read at every cycle |
| `read_ralph` | `cat /Users/agent/.zeroclaw/workspace/RALPH.md` | RALPH individual agent cycle config |
| `read_amii` | `cat /Users/agent/.zeroclaw/workspace/AMII.md` | Amii's contact profile and notification preferences |
| `list_sessions` | `ls -lt /Users/agent/.zeroclaw/workspace/sessions/ \| head -20` | List recent session log files by date |
| `read_session` | `cat /Users/agent/.zeroclaw/workspace/sessions/%filename%` | Read a specific session log |

---

## steph

| Tool | Command | Purpose |
|---|---|---|
| `create_calendar_event` | `osascript -e 'tell application "Calendar" to tell calendar "Home" to make new event with properties {summary:"%title%", start date:date "%start%", end date:date "%end%"}'` | Create Apple Calendar event |
| `send_email` | `mail -s "%subject%" requested2019@gmail.com` | Send email — authorised recipient only |
| `read_style_guide` | `cat /Users/agent/Projex-Axels-Team/Steph/style-guide.md` | Read Amii's voice style guide |
| `compare_document` | `diff /Users/agent/Projex-Axels-Team/Steph/%original% /Users/agent/Projex-Axels-Team/Steph/%revised%` | Extract edit delta when Amii returns a revised version |
| `read_soul` | `cat /Users/agent/.zeroclaw/workspace/skills/SOUL.md` | SOUL.md product context |
| `read_amii` | `cat /Users/agent/.zeroclaw/workspace/AMII.md` | Amii's current voice profile and edit history — read before every task |
| `read_ralph` | `cat /Users/agent/.zeroclaw/workspace/RALPH.md` | RALPH individual agent cycle config |
| `list_steph_docs` | `ls -lt /Users/agent/Projex-Axels-Team/Steph/` | List Steph's document folder |

**Email restriction:** `send_email` hard-routes to `requested2019@gmail.com` only. All other recipients receive a draft.

---

## clive

| Tool | Command | Purpose |
|---|---|---|
| `git_log` | `git -C /Users/agent/Projex-Axels-Team/app-project log --oneline -10` | Last 10 commits |
| `git_diff` | `git -C /Users/agent/Projex-Axels-Team/app-project diff HEAD~1` | Latest code changes |
| `run_tests` | `cd /Users/agent/Projex-Axels-Team/app-project && python -m pytest && npm run test` | Full test suite |
| `check_unscoped_queries` | `grep -rn 'db.query' /Users/agent/Projex-Axels-Team/app-project --include='*.py' \| grep -v 'organization_id'` | **Mandatory on every code review** — flag any results as Critical |
| `open_claude` | `open -a Safari https://claude.ai` | Open Claude for senior peer review |
| `read_soul` | `cat /Users/agent/.zeroclaw/workspace/skills/SOUL.md` | SOUL.md product context |
| `read_forge` | `cat /Users/agent/.zeroclaw/workspace/FORGE.md` | FORGE loop — check max_iterations and gate protocols |
| `read_ralph` | `cat /Users/agent/.zeroclaw/workspace/RALPH.md` | RALPH individual agent cycle config |
| `list_tasks` | `ls /Users/agent/Projex-Axels-Team/shared/tasks/` | Dev team task queue |

---

## jamie

| Tool | Command | Purpose |
|---|---|---|
| `lint_frontend` | `cd /Users/agent/Projex-Axels-Team/app-project && npm run lint` | ESLint on frontend |
| `test_frontend` | `cd /Users/agent/Projex-Axels-Team/app-project && npm run test` | Frontend test suite |
| `build_frontend` | `cd /Users/agent/Projex-Axels-Team/app-project && npm run build` | Production frontend build |
| `read_soul` | `cat /Users/agent/.zeroclaw/workspace/skills/SOUL.md` | SOUL.md product context |
| `read_ralph` | `cat /Users/agent/.zeroclaw/workspace/RALPH.md` | RALPH individual agent cycle config |

---

## priya

| Tool | Command | Purpose |
|---|---|---|
| `run_tests` | `cd /Users/agent/Projex-Axels-Team/app-project && python -m pytest` | Backend test suite |
| `run_migrations` | `cd /Users/agent/Projex-Axels-Team/app-project && alembic upgrade head` | Run pending migrations |
| `lint_backend` | `cd /Users/agent/Projex-Axels-Team/app-project && ruff check .` | Ruff linter on backend |
| `read_soul` | `cat /Users/agent/.zeroclaw/workspace/skills/SOUL.md` | SOUL.md product context |
| `read_ralph` | `cat /Users/agent/.zeroclaw/workspace/RALPH.md` | RALPH individual agent cycle config |

---

## marcus

| Tool | Command | Purpose |
|---|---|---|
| `check_health` | `curl -s http://localhost:8000/api/health` | Backend health check — http for local dev ✅ |
| `check_unscoped_queries` | `grep -rn 'db.query' /Users/agent/Projex-Axels-Team/app-project --include='*.py' \| grep -v 'organization_id'` | **Mandatory pre-deployment** — stop deployment if any results |
| `check_secrets` | `grep -rn 'SECRET\|PASSWORD\|API_KEY\|TOKEN' /Users/agent/Projex-Axels-Team/app-project --include='*.py' --include='*.ts' --include='*.tsx' \| grep -v '.env' \| grep -v 'process.env' \| grep -v 'os.environ' \| grep -v 'config\.' \| grep -v '#'` | Check for hardcoded secrets before deployment |
| `run_migrations` | `cd /Users/agent/Projex-Axels-Team/app-project && alembic upgrade head` | Run pending migrations |
| `build_frontend` | `cd /Users/agent/Projex-Axels-Team/app-project && npm run build` | Production frontend build |
| `run_all_tests` | `cd /Users/agent/Projex-Axels-Team/app-project && python -m pytest && npm run test` | Full test suite |
| `tail_logs` | `tail -f /Users/agent/Projex-Axels-Team/app-project/logs/backend.log` | Live backend log stream |
| `read_soul` | `cat /Users/agent/.zeroclaw/workspace/skills/SOUL.md` | SOUL.md product context |
| `read_forge` | `cat /Users/agent/.zeroclaw/workspace/FORGE.md` | FORGE Ship phase protocols |
| `read_ralph` | `cat /Users/agent/.zeroclaw/workspace/RALPH.md` | RALPH individual agent cycle config |
| `read_amii` | `cat /Users/agent/.zeroclaw/workspace/AMII.md` | Amii's notification preferences — read before every deployment notification |

---

## folder_monitor

| Tool | Command | Purpose |
|---|---|---|
| `scan_tasks` | `ls /Users/agent/Projex-Axels-Team/shared/tasks/` | List all task files |
| `scan_project_frontend` | `find /Users/agent/Projex-Axels-Team/app-project -path '*/components*' -name '*.tsx' \| head -40` | Frontend component inventory |
| `scan_project_backend` | `find /Users/agent/Projex-Axels-Team/app-project -name '*.py' -not -path '*/__pycache__/*' \| head -40` | Backend Python file inventory |
| `read_soul` | `cat /Users/agent/.zeroclaw/workspace/skills/SOUL.md` | SOUL.md — scope check |
| `check_style_guide` | `test -f /Users/agent/Projex-Axels-Team/Steph/style-guide.md && echo 'FOUND' \|\| echo 'MISSING'` | Confirm style guide exists |
| `scan_steph_docs` | `ls /Users/agent/Projex-Axels-Team/Steph/` | Prior copy/docs for context |

---

## Tool Availability Summary

| Tool | axel | steph | clive | jamie | priya | marcus | folder_monitor |
|---|---|---|---|---|---|---|---|
| git_log | ✅ | — | ✅ | — | — | — | — |
| git_status | ✅ | — | — | — | — | — | — |
| git_diff | ✅ | — | ✅ | — | — | — | — |
| list_tasks | ✅ | — | ✅ | — | — | — | — |
| read_soul | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| read_forge | ✅ | — | ✅ | — | — | ✅ | — |
| read_ralph | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | — |
| read_amii | ✅ | ✅ | — | — | — | ✅ | — |
| run_tests | — | — | ✅ | ✅ | ✅ | ✅ | — |
| run_migrations | — | — | — | — | ✅ | ✅ | — |
| lint_frontend | — | — | — | ✅ | — | — | — |
| lint_backend | — | — | — | — | ✅ | — | — |
| build_frontend | — | — | — | ✅ | — | ✅ | — |
| check_health | — | — | — | — | — | ✅ | — |
| check_unscoped_queries | — | — | ✅ | — | — | ✅ | — |
| check_secrets | — | — | — | — | — | ✅ | — |
| tail_logs | — | — | — | — | — | ✅ | — |
| open_claude | — | — | ✅ | — | — | — | — |
| send_email | — | ✅ | — | — | — | — | — |
| create_calendar_event | — | ✅ | — | — | — | — | — |
| read_style_guide | — | ✅ | — | — | — | — | — |
| compare_document | — | ✅ | — | — | — | — | — |
| list_sessions | ✅ | — | — | — | — | — | — |
| read_session | ✅ | — | — | — | — | — | — |
| scan_project_frontend | — | — | — | — | — | — | ✅ |
| scan_project_backend | — | — | — | — | — | — | ✅ |
| check_style_guide | — | — | — | — | — | — | ✅ |
| scan_steph_docs | — | — | — | — | — | — | ✅ |

---

## Notes

- `read_soul` is on every agent — always read before every task.
- `read_forge` is on Axel, Clive, and Marcus — the three agents who touch the FORGE outer loop directly.
- `read_ralph` is on all agents except folder_monitor — read at every Reflect phase.
- `read_amii` is on Axel, Steph, and Marcus — the agents who communicate with or write for Amii.
- `check_unscoped_queries` is on both Clive (code review) and Marcus (pre-deployment). Both must run it independently. Commands are identical.
- `check_secrets` is on Marcus only — part of the pre-deployment checklist.
- `compare_document` is on Steph only — used to extract edit deltas from Amii's revised documents.
- `check_health` uses `http://localhost:8000` (not https). Change to https in marcus-SKILL.toml only if local TLS is configured. The default is http to prevent false negatives on local dev setups.
- `run_migrations` is on both Priya (development) and Marcus (deployment). Command is identical.
- `build_frontend` is on both Jamie (pre-submission check) and Marcus (deployment).
