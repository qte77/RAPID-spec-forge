# Current State Inventory

## What Exists (v1.0.0)

### Pipeline Commands (`.claude/commands/`)

5 proven prompt-engineering commands driving the BRD->PRD->FRD->Execute workflow
(legacy files still use `frp` naming -- to be renamed during refactor):

| Command | File | Input | Output |
|---------|------|-------|--------|
| `generate-brd` | `generate-brd.md` | `context/business_inputs/$FILE` or interactive | `context/BRDs/$FILE` |
| `generate-prd-from-brd` | `generate-prd-from-brd.md` | `context/BRDs/$FILE` | `context/PRDs/$FILE` |
| `generate-frp-from-prd` | `generate-frp-from-prd.md` | `context/PRDs/$FILE` + `context/BRDs/$FILE` | `context/FRPs/${FILE}_${FEATURE}.md` |
| `execute-frp` | `execute-frp.md` | `context/FRPs/$FILE` | Implementation + logs |
| `generate-frp` (legacy) | `generate-frp.md` | `context/features/$FILE` | `context/FRPs/$FILE` |

### Templates (`context/templates/`)

| Template | Purpose |
|----------|---------|
| `business_input_base.md` | Stakeholder initial business info |
| `brd_base.md` | Business Requirements Definition |
| `prd_base.md` | Product Requirements Document |
| `frp_base.md` | Feature Requirements Document (legacy name: "Prompt") |
| `feature_base.md` | Legacy feature description |

### Sub-Agents (`.claude/agents/`)

| Agent | Role |
|-------|------|
| `backend-agents.md` | Backend architect -- API, DB, scaling |
| `frontend-developer.md` | React/Tailwind frontend |
| `code-reviewer.md` | Quality, security, review |

### Configuration

| File | Content |
|------|---------|
| `.claude/settings.json` | Permissions (allow/deny), statusline, telemetry off |
| `context/config/paths.md` | All path variable definitions ($CTX_BRD_PATH etc.) |
| `AGENTS.md` | Quality thresholds 8/7/8/7, decision framework, agent behavior |
| `CLAUDE.md` | Redirects to AGENTS.md |

### Makefile Recipes (current)

**Context engineering**: `brd_gen_claude`, `prd_gen_claude`, `frp_gen_claude`,
`frp_exe_claude`, `frp_gen_legacy_claude`, `frp_exe_legacy_claude`

**Dev tooling**: `setup_dev`, `ruff`, `check_types`, `test_all`,
`test_hypothesis_*`, `coverage_all`

**Examples**: `run_example_gui`, `run_example_server`, `run_example_client`

### Source Code (`src/`)

**Effectively empty.** Only:
- `src/__init__.py` -- version string `__version__ = "1.0.0"`
- `src/main.py` -- docstring + `pass`

### Tests (`tests/`)

- `tests/test_tests.py` -- Hypothesis hello-world (integers, ordered pairs)

### pyproject.toml

- **Runtime deps**: none (`dependencies = []`)
- **Dev groups**: `dev` (pyright, ruff), `test` (hypothesis, coverage), `docs` (mkdocs stack)
- **Build**: hatchling
- **Python**: >=3.13

### Documentation (`docs/`)

| File | Purpose |
|------|---------|
| `CABIO-vision.md` | Why CABIO matters |
| `CABIO-product-roadmap.md` | Customer value, phases, timeline |
| `CABIO-implementation-guide.md` | Technical specs, compliance, UX |
| `architecture/` | PlantUML diagrams (current, SMB, enterprise) |

### Examples (`examples/mcp-server-client/`)

Complete MCP server-client implementation (Python). Generated via legacy FRD
workflow. Has its own `src/`, `tests/`, full GUI.

## What Is Empty / Minimal

- `src/` -- no application logic
- `tests/` -- only hello-world
- `context/PRDs/` -- no generated PRDs checked in
- `context/FRPs/` -- no generated FRDs checked in (dir to be renamed)
- `context/features/` -- no feature descriptions checked in
- `context/logs/` -- no execution logs
- `state/` -- does not exist

## What Must Not Be Touched

1. `.claude/commands/*.md` (5 files)
2. `context/templates/*.md` (5 files)
3. `context/config/paths.md`
4. `AGENTS.md`
5. `.claude/agents/*.md` (3 files)
6. `.claude/settings.json`
7. Existing Makefile recipes (append only)
8. `pyproject.toml` tooling config (ruff, pyright, hypothesis)
