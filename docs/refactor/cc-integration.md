# Claude Code Integration Details

## Bridge Design

The cockpit invokes Claude Code via `claude -p` (headless/print mode) through
a Python subprocess wrapper.

### Invocation Pattern

```bash
claude -p "/project:{command} {args}" --output-format json
```

- `/project:` prefix routes to `.claude/commands/{command}.md`
- `--output-format json` returns structured output with `total_cost_usd`
- Working directory set to project workspace via `subprocess.run(cwd=...)`

### JSON Output Parsing

The `--output-format json` flag returns:

```json
{
  "result": "...",
  "total_cost_usd": 0.042,
  ...
}
```

Cost is extracted from `total_cost_usd` field and recorded in `StageRecord`.

## Settings Enhancements (`.claude/settings.json`)

### Env Vars for Headless Optimization

Add to existing `env` section:

```json
{
  "env": {
    "CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS": "1",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1"
  }
}
```

- `DISABLE_GIT_INSTRUCTIONS`: Save tokens in automated loops
- `DISABLE_NONESSENTIAL_TRAFFIC`: Reduce noise in headless runs

### Additional Env Vars (when needed)

```json
{
  "env": {
    "CLAUDE_CODE_SUBAGENT_MODEL": "claude-sonnet-4-20250514",
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

- `SUBAGENT_MODEL`: Route sub-agents to cheaper models for cost control
- `AGENT_TEAMS`: Enable parallel agent execution

## Hook Events for Observability

### Useful Hook Events

| Event | Hook Type | Cockpit Use |
|-------|-----------|-------------|
| `SessionStart` | command | Initialize cockpit state |
| `SessionEnd` | prompt | Generate session summary |
| `PostToolUse` | command | Emit tool events for monitoring |
| `PreCompact` | command | Save critical state before compaction |

### Limitation: `claude -p` Hook Support

Hooks are designed for interactive sessions. The cockpit should handle events
on the Python side for headless runs rather than relying on CC hooks.

## Cross-Repo Access (User-Level Settings)

Use `permissions.additionalDirectories` + `sandbox.filesystem.allowWrite` in
`~/.claude/settings.json` (user-level, not per-project). This gives any CC
session cross-repo access without per-project config or session restarts.

- `additionalDirectories`: Expands Read/Write/Edit tool scope beyond CWD
- `allowWrite` (additive): Expands Bash sandbox write access, merges with per-project configs

```json
{
  "permissions": { "additionalDirectories": ["/workspaces"] },
  "sandbox": { "filesystem": { "allowWrite": ["/workspaces"] } }
}
```

**DRY consolidation**: Since `allowWrite` merges across scopes, project-level
`sandbox.filesystem` is redundant. User-level is the single source of truth.
Project-level settings should only contain project-specific concerns (`env`,
`permissions`, `plugins`, `statusLine`).

**Credential friction**: Best long-term fix is Codespaces encrypted secrets
via `containerEnv`. The PAT is injected at container start, visible to all
processes including CC sandbox. No `source`, no `.env`, no read permission issues.

## Known Gotchas

### 1. Teams Artifacts Ephemeral in Print Mode

`~/.claude/teams/` and `~/.claude/tasks/` are empty after `claude -p` completes.
Artifacts only persist during interactive sessions.

**Implication**: Cannot rely on filesystem artifacts for trace data. Parse
`raw_stream.jsonl` or use Python-side event tracking.

### 2. OTel Has No Trace Spans

CC OTel integration exports only metrics and logs, not distributed trace spans.
For execution analysis, use artifact collection, not OTel.

### 3. Auto-Compact at ~78-85%

Context auto-compacts earlier than documented (~78-85%, not 95%). Plan context
budgets conservatively for long pipeline runs.

### 4. Plugin Hooks Duplicate Error

Never list `hooks` field in `plugin.json` when using standard
`hooks/hooks.json` path. Causes silent duplication.

### 5. Cost Parsing May Vary

The `total_cost_usd` field format may change between CC versions. Always wrap
cost parsing in try/except with fallback to 0.0.

## Dynamic Context Injection

CC commands support dynamic context via `` !`cmd` `` syntax. The cockpit can
inject live project state into command prompts:

```markdown
# In .claude/commands/generate-brd.md
Current project state:
!`cat state/projects.json | jq '.projects["$PROJECT_ID"]'`
```

This allows commands to adapt based on cockpit state without code changes.

## Command Mapping

### CABIO Stages (CC Commands)

| Pipeline Stage | CC Command | Input Path | Output Path |
|---------------|------------|------------|-------------|
| `brd` | `generate-brd` | `business_inputs/$FILE` | `BRDs/$FILE` |
| `prd` | `generate-prd-from-brd` | `BRDs/$FILE` | `PRDs/$FILE` |
| `frd` | `generate-frd-from-prd` | `PRDs/$FILE` + feature name | `FRDs/${FILE}_${FEATURE}.md` |

The `STAGE_COMMANDS` registry in `src/models/pipeline.py` codifies this mapping.

### Execute Stage (Ralph WT -- NOT a CC Command)

| Pipeline Stage | Tool | Input | Output |
|---------------|------|-------|--------|
| `execute` | `make ralph_init_loop && make ralph_run` | FRD transformed to `docs/PRD.md` | Working code in `src/` + `tests/` |

The execute stage:
1. Copies/transforms FRD into Ralph's `docs/PRD.md` format
2. Runs `make ralph_init_loop` (generates `prd.json`)
3. Runs `make ralph_run` (TDD loop in git worktrees)
4. Parses Ralph's `progress.txt` for cost and story status
5. Updates cockpit state with aggregated results

CABIO never sees or generates application code. Ralph WT handles all
implementation via its own CC invocations (`claude -p --dangerously-skip-permissions`).

### Market Research Integration (via cc-utils-plugin)

| Stage | Tool | Input | Output |
|-------|------|-------|--------|
| Pre-BRD | `market-research` plugin (installed from cc-utils-plugin) | `config/sources.md` + `config/targets.md` | Market intelligence documents |

The market research plugin (source: `agentic-market-research-to-gtm`, distributed
via cc-utils-plugin) provides skills/commands for a 7-phase pipeline: source
analysis, landscape, market research, PMF, GTM, contradiction analysis, synthesis.

Install: `claude plugin install market-research@qte77-claude-code-utils`

Outputs feed into CABIO's BRD generation for data-driven business requirements.
No separate repo invocation needed -- the plugin runs within CABIO's CC session.
