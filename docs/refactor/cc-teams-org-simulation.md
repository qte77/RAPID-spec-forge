# CC Teams as Org Chart Simulation

## The Idea

Use Claude Code Teams mode (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`) to simulate
Paperclip's org chart behavior natively -- without Node.js, React, or PostgreSQL.
Each teammate is a role in the org, with its own skills loaded via cc-utils-plugin.

## How It Would Work

```
CABIO cockpit (Python orchestrator)
  ↓ spawns CC Teams session
CC Teams (org chart)
  ├── Product Strategist (teammate)
  │     Skills: market-research plugin
  │     → runs PMF/GTM analysis, feeds BRD
  │
  ├── Requirements Architect (teammate)
  │     Skills: docs-generator plugin
  │     → generates BRD, PRD, FRD
  │
  ├── Quality Reviewer (teammate)
  │     Skills: reviewing-code, securing-mas plugins
  │     → quality gates between stages
  │
  └── SWE Lead (teammate)
        Skills: python-dev, ralph plugins
        → delegates to Ralph WT for implementation
        → hyperfocused on code execution only
```

## Plugin-to-Role Mapping

Each role installs only the cc-utils-plugins it needs:

| Role | Plugins | Responsibility |
|------|---------|---------------|
| Product Strategist | `market-research` | Source analysis, landscape, PMF, GTM |
| Requirements Architect | `docs-generator`, `codebase-tools` | BRD, PRD, FRD generation |
| Quality Reviewer | `mas-design` (MAESTRO) | Gate validation, security review |
| SWE Lead | `python-dev`, `ralph` | Ralph WT delegation, implementation oversight |
| All roles | `workspace-setup`, `codebase-tools` | Common infrastructure |

## CC Teams Mechanics

### Task-Based Delegation

```python
# From CABIO orchestrator
TaskCreate(subject="Market research for Project X",
           description="Run market-research plugin phases 0-5",
           assignee="product-strategist")

TaskCreate(subject="Generate BRD from market research",
           description="Use docs-generator to create BRD",
           assignee="requirements-architect",
           blockedBy=["market-research-task-id"])

TaskCreate(subject="Quality gate: BRD review",
           description="Score BRD against 8/7/8/7 thresholds",
           assignee="quality-reviewer",
           blockedBy=["brd-task-id"])
```

### Wave Execution (from Ralph pattern)

Independent tasks run in parallel. Dependent tasks wait for blockers:

```
Wave 1: [Market Research]  ← parallel with any other Wave 1 tasks
Wave 2: [BRD Generation]  ← blocked by Wave 1
Wave 3: [BRD Quality Gate] ← blocked by Wave 2
Wave 4: [PRD Generation]  ← blocked by Wave 3
Wave 5: [PRD Quality Gate] ← blocked by Wave 4
Wave 6: [FRD Fan-out]     ← blocked by Wave 5 (parallel per feature)
Wave 7: [Ralph WT x N]    ← blocked by Wave 6 (parallel per feature)
```

## Ralph WT as Hyperfocused SWE

The SWE Lead teammate doesn't write code itself. It:

1. Receives FRD from Requirements Architect (via task completion)
2. Transforms FRD into Ralph's `docs/PRD.md` format
3. Invokes `make ralph_run` in project workspace
4. Monitors Ralph's `progress.txt` for completion
5. Reports results back to CABIO cockpit

Ralph WT stays hyperfocused: it only does TDD in isolated worktrees. It doesn't
know about org charts, market research, or business requirements. It just
receives a PRD and produces working code.

## Advantages Over Paperclip

| Aspect | CC Teams | Paperclip |
|--------|----------|-----------|
| **Infrastructure** | Zero -- runs in CC session | Node.js + React + PostgreSQL |
| **Agent runtime** | Native CC (same process) | External agents via heartbeat API |
| **Plugin ecosystem** | cc-utils-plugin (existing) | Custom extensions |
| **Setup** | `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | `npx paperclipai onboard` + DB |
| **Context sharing** | CC Teams shared context | Ticket-based message passing |
| **Cost** | CC API costs only | CC API + infrastructure costs |

## Claude Remote / Mobile

`claude --remote` enables async pipeline execution from any device. Combined with
CC Teams, the cockpit could:

- Start a pipeline from mobile, monitor progress remotely
- Run long pipelines (multi-company, full BRD->PRD->FRD->Ralph) without keeping
  a terminal open
- Resume sessions after disconnection

This complements the CC Teams org simulation by making the cockpit accessible
beyond the developer workstation.

## Limitations

| Limitation | Impact | Mitigation |
|-----------|--------|------------|
| **Teams artifacts ephemeral in print mode** | Can't use `claude -p` for teams | Use interactive mode or parse `raw_stream.jsonl` |
| **Token cost scales linearly** | N teammates = N CC instances | Use cheaper models for simple roles (`CLAUDE_CODE_SUBAGENT_MODEL`) |
| **No persistent org state** | Teams session doesn't survive restart | CABIO state file tracks progress; teams are re-created per run |
| **Experimental feature** | API may change | Abstract behind `AgentRuntime(Protocol)` |
| **No heartbeats/scheduling** | Can't wake on schedule | CABIO orchestrator handles scheduling (Phase 3) |

## Implementation Path

### Phase 1 (current plan): No teams

Single `claude -p` calls per stage. Simple, reliable, no experimental features.

### Phase 2: Optional teams mode

```python
# In CABIO orchestrator
if config.use_teams:
    run_pipeline_with_teams(project, stages)
else:
    run_pipeline_sequential(project, stages)
```

Teams mode parallelizes independent stages (market research + BRD prep).

### Phase 3: Full org simulation

CC Teams as the default execution model. Each role is a teammate with
role-specific plugins. CABIO cockpit manages the org chart, cost budgets,
and state persistence that CC Teams doesn't provide natively.

## Key Insight

CC Teams + cc-utils-plugins gives us Paperclip's org chart behavior **without
Paperclip's infrastructure**. The tradeoff: less polished UI (terminal vs React),
less persistent state (JSON vs PostgreSQL), but zero additional dependencies and
native CC integration.

For CABIO's use case (business pipeline, not 24/7 autonomous company), this
tradeoff is favorable. CABIO doesn't need heartbeats or always-on agents -- it
needs quality-gated document generation with role-based expertise.
