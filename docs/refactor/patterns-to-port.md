# Patterns to Port

Reusable patterns from Agents-eval, Ralph Loop, and cc-utils-plugin that inform
the cockpit design. These are reference material, not copy-paste targets.

**Source hierarchy** (see [ecosystem.md](ecosystem.md)):
- **cc-research**: Read-only reference. Informs design. Never imported.
- **cc-utils-plugin**: Consumed via `claude plugin install`. No code porting.
- **Ralph WT**: Coding engine. CABIO delegates to it. Patterns inform bridge design.
- **Agents-eval**: Sibling project. Shares patterns via AGENT_LEARNINGS.

## From Agents-eval

### Subprocess CC Bridge

**Source**: `src/app/engines/cc_engine.py`

Key elements to reuse:
- `claude -p "prompt" --output-format json` via `subprocess.run()`
- Check `shutil.which("claude")` at startup
- JSON output parsing for `total_cost_usd`
- CWE-78 sanitization (no empty, no dash-prefix, length cap)
- Timeout handling

### Registry Dict Dispatch

**Source**: AGENT_LEARNINGS -- "Repeated Dispatch Chains Inflate File Complexity"

Replace repeated `if/elif/else` over stage types with a single registry dict
(`STAGE_COMMANDS`). Validates once at entry point.

### Parallel CC Invocation

**Source**: `polyforge/scripts/cc-parallel.sh` (formerly Agents-eval)

Key elements to reuse:
- `claude -p` with presets (`validate`, `status`, `security`)
- JSON output parsing with `jq` for `cost_usd`/`session_cost`
- Cost aggregation across parallel subprocesses
- PID tracking + wait-based result collection

Directly informs `cc_bridge.py`'s parallel FRD execution in Phase 2.

## From Ralph Loop

### Atomic JSON State

**Source**: `ralph/scripts/generate_prd_json.py`

Key elements:
- Write to tmp file, then `os.replace()` for atomicity
- Immutable passed stages (safety rule)
- Version field in state file for migration

### Wave-Based DAG Execution

**Source**: `ralph/scripts/lib/teams.sh:get_unblocked_stories()`

BFS dependency resolution for parallel execution. In cockpit: parallel FRD
generation where features are independent.

### Vibe Kanban REST Client

**Source**: `ralph/scripts/lib/vibe.sh`

REST API patterns:
- POST/PUT tasks with status lifecycle mapping
- Health check at startup, silent no-op if unavailable
- Maps stage status -> Vibe Kanban task lifecycle

### Claude-as-Judge

**Source**: `ralph/scripts/lib/judge.sh`

Phase 3 pattern: compare N parallel FRD implementations, score and pick best.
Not needed until intelligence layer.

## From cc-utils-plugin

### Plugins to Install

Installed via `claude plugin install <plugin>@qte77-claude-code-utils`.
CABIO and Ralph WT use different plugin subsets:

**CABIO-test plugins** (business pipeline):

| Plugin | Skills | Cockpit Role |
|--------|--------|-------------|
| `workspace-setup` | SessionStart deploy hook | Deploy rules, statusline |
| `codebase-tools` | `researching-codebase`, `compacting-context` | Isolated exploration |
| `market-research` | Source analysis, landscape, PMF, GTM | Data-driven BRD input |
| `mas-design` | `designing-mas-plugins`, `securing-mas` | Reducer pattern, MAESTRO |
| `docs-generator` | `generating-tech-spec`, `generating-report` | ADR/RFC, reports |

**Ralph WT plugins** (coding pipeline -- NOT installed in CABIO):

| Plugin | Skills | Ralph Role |
|--------|--------|-----------|
| `python-dev` | `implementing-python`, `testing-python`, `reviewing-code` | TDD workflow |
| `ralph` | `generating-prd-json-from-prd-md` | PRD-to-JSON |

### Stateless Reducer Pattern

**Source**: `plugins/mas-design/skills/designing-mas-plugins/SKILL.md`

Each pipeline stage as a pure function: `evaluate(context) -> result` with
`get_context_for_next_tier()` chaining. Maps directly to
`execute_stage(project, stage, ...) -> StageRecord`.

### Copy-if-not-exists Deploy

**Source**: `plugins/workspace-setup/hooks/scripts/setup-workspace.sh`

Idempotent workspace initialization. Port to `ensure_workspace()` -- only create
dirs that don't exist.

### Statusline JSON Schema

**Source**: `plugins/workspace-setup/scripts/statusline.sh`

jq field paths for cost, context window, lines changed, agent type, model ID.
Becomes the cockpit's data ingestion schema for dashboard metrics.

### MAESTRO 7-Layer Checklist

**Source**: `plugins/mas-design/skills/securing-mas/SKILL.md`

L1-L7 security validation. Use as security review gate template for cockpit
components.

## Pattern Summary

| Pattern | Source | Target | Phase |
|---------|--------|--------|-------|
| Subprocess CC bridge | Agents-eval cc_engine | `src/runner/cc_bridge.py` | 0 |
| Parallel CC invocation | polyforge cc-parallel.sh | `src/runner/cc_bridge.py` | 2 |
| Atomic JSON state | Ralph prd.json | `src/store/state_store.py` | 0 |
| Registry dict dispatch | AGENT_LEARNINGS | `src/models/pipeline.py` | 0 |
| Workspace isolation | Paperclip multi-company | `src/runner/workspace.py` | 0 |
| Stateless reducer | cc-utils MAS design | `src/pipeline/stages.py` | 1 |
| Quality thresholds | AGENTS.md 8/7/8/7 | `src/pipeline/gate.py` | 1 |
| Wave DAG execution | Ralph teams.sh | `src/pipeline/parallel.py` | 2 |
| Vibe Kanban REST | Ralph vibe.sh | `src/integrations/vibe_kanban.py` | 2 |
| Statusline metrics | cc-utils workspace-setup | `src/dashboard/metrics.py` | 2 |
| Claude-as-Judge | Ralph judge.sh | Phase 3 design | 3 |
| MAESTRO security | cc-utils securing-mas | Security review gates | 3 |
