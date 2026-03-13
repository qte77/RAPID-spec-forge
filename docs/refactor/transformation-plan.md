# Transformation Plan

## Overview

Transform CABIO-test from manual `make` commands invoking `claude -p` into a
Python orchestrator that manages multiple projects, auto-chains pipeline stages,
tracks costs, and provides dashboard visibility.

**Architectural inversion**: Human runs make -> Python orchestrator drives CC
agents per project automatically.

**Scope boundary**: CABIO handles the business pipeline only (BRD->PRD->FRD).
The `execute` stage hands off to Ralph WT for TDD implementation. Market
intelligence is sourced from `agentic-market-research-to-gtm`. See
[ecosystem.md](ecosystem.md) for full repo relationships.

## Phase 0: Foundation (Week 1)

### Goal

Data models, state persistence, CC bridge, workspace isolation.

### Files to Create

```
src/__init__.py                 (edit: keep version)
src/models/__init__.py          (new)
src/models/project.py           (new: PipelineStage, StageStatus, StageRecord, Project, StateFile)
src/models/pipeline.py          (new: STAGE_COMMANDS registry, STAGE_ORDER)
src/store/__init__.py           (new)
src/store/state_store.py        (new: JSON persistence, atomic writes)
src/runner/__init__.py          (new)
src/runner/cc_bridge.py         (new: subprocess claude -p wrapper)
src/runner/workspace.py         (new: project isolation, path resolvers)
tests/test_models.py            (new: Hypothesis round-trip serialization)
tests/test_state_store.py       (new: Hypothesis upsert/get, atomic writes)
tests/test_cc_bridge.py         (new: mocked subprocess)
tests/test_workspace.py         (new: path resolution)
```

### pyproject.toml Changes

- Add `pydantic>=2.0.0` to `dependencies`

### Verification

- `state/` directory created (first visible artifact)
- `make ruff` passes
- `make check_types` passes
- `make test_all` passes

## Phase 1: Single-Project Orchestration (Weeks 2-3)

### Goal

Pipeline runner, quality gates, FRD fan-out, CLI.

### Files to Create

```
src/pipeline/__init__.py        (new)
src/pipeline/runner.py          (new: run_pipeline -- iterate stages in order)
src/pipeline/stages.py          (new: execute_stage -- build args, call bridge)
src/pipeline/gate.py            (new: quality gate via claude -p scoring prompt)
src/pipeline/frd_fan_out.py     (new: extract features from PRD, one FRD per feature)
src/cli/__init__.py             (new)
src/cli/main.py                 (new: argparse CLI)
tests/test_pipeline_runner.py   (new)
tests/test_pipeline_stages.py   (new)
tests/test_gate.py              (new)
tests/test_frd_fan_out.py       (new)
```

### Execute Stage: Ralph WT Handoff

The `execute` stage does NOT call `claude -p` directly. It:
1. Transforms the FRD into Ralph's `docs/PRD.md` format
2. Runs `make ralph_init_loop` (generates `prd.json` from PRD)
3. Runs `make ralph_run` (TDD loop in git worktrees)
4. Polls Ralph's `progress.txt` for story completion and cost
5. Updates cockpit state with aggregated results

Each feature FRD can spawn its own Ralph WT instance. This is the natural
parallelism point -- independent features run in parallel worktrees.

See [cc-integration.md](cc-integration.md) for command mapping details and
[ecosystem.md](ecosystem.md) for the CABIO->Ralph boundary.

### CLI Interface

```
cabio project create --name "My SaaS" --company "Acme"
cabio project list
cabio run --project my-saas [--stage brd|prd|frd|execute]
cabio status --project my-saas
cabio status --all
```

### pyproject.toml Changes

- Add `[project.scripts] cabio = "src.cli.main:main"`

### Makefile Additions (append)

```makefile
# MARK: orchestration cockpit
cockpit_status:  ## Show all projects dashboard
    uv run python -m src.cli.main status --all

project_create:  ## Create project "ARGS=--name 'My SaaS' --company Acme"
    uv run python -m src.cli.main project create $(ARGS)

project_run:  ## Run pipeline "ARGS=--project my-saas"
    uv run python -m src.cli.main run $(ARGS)
```

### Verification

- End-to-end with mocked CCBridge
- `make test_all` passes
- Manual: `cabio project create --name Test && cabio status --all`

## Phase 2: Multi-Project Cockpit (Weeks 3-4)

### Goal

Dashboard, parallel FRD, multi-company workspace, Vibe Kanban integration.

### Files to Create

```
src/dashboard/__init__.py       (new)
src/dashboard/table.py          (new: Rich terminal table)
src/dashboard/summary.py        (new: cost aggregation, company grouping)
src/pipeline/parallel.py        (new: ThreadPoolExecutor for FRD fan-out)
src/integrations/__init__.py    (new)
src/integrations/vibe_kanban.py (new: optional REST push, urllib.request)
tests/test_dashboard.py         (new)
tests/test_parallel.py          (new: thread-safety with Hypothesis)
tests/test_vibe_kanban.py       (new)
```

### Dashboard Output

```
CABIO Cockpit -- 2026-03-13 14:30 UTC
COMPANY       PROJECT        BRD       PRD       FRD       RALPH     COST
Acme Corp     my-saas        passed    passed    3/5       2/3       $0.42
StartupXYZ    analytics      running   .         .         .         $0.09
Total: 2 projects | 2 companies | $0.51 total cost
```

### pyproject.toml Changes

- Add optional dep group: `[dependency-groups] dashboard = ["rich>=13.0.0"]`
- Graceful fallback to plain text if rich not installed

### Verification

- Dashboard renders for 3+ projects fixture
- Parallel FRD: 5 features complete, all state records correct
- Thread-safety: Hypothesis test with `threading.Barrier`
- Cost sums match

## Phase 3: Intelligence Layer (Future -- Design Only)

Not implemented. Architecture accommodates:

- **Automated market intelligence in pipeline**: The `market-research` cc-utils-plugin
  is available now (`claude plugin install`), but cockpit automation of invoking it
  as a pre-BRD stage belongs here. See [ecosystem.md](ecosystem.md).
- **Triple memory**: Long-term, working, episodic per project
- **Cost budgets**: `CostBudget` model with auto-throttle at 80%
- **Agent runtime abstraction**: `AgentRuntime(Protocol)` with CC and Gemini
- **Claude-as-Judge**: Compare parallel FRD implementations, pick best
- **Claude remote/mobile**: `claude --remote` for async pipeline execution

## Files Modified (Not Created)

| File | Change | Phase |
|------|--------|-------|
| `pyproject.toml` | Add pydantic dep, CLI entry point, optional dashboard group | 0, 1, 2 |
| `Makefile` | Append orchestration recipes | 1 |
| `context/config/paths.md` | Add project workspace paths | 0 |
| `.claude/settings.json` | Add headless optimization env vars | 1 |
| `README.md` | Update with cockpit usage | 2 |
| `CHANGELOG.md` | Document changes | 0-2 |
