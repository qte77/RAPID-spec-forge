# Ecosystem: How the Repos Fit Together

## Six Repos, Six Roles

| Repo | Local Path | Role | Depends On |
|------|-----------|------|------------|
| **polyforge** | `/workspaces/qte77/polyforge` | Polyrepo dev forge. Parallel AI agent orchestration, credential management, status dashboard, VS Code multi-root workspace. NOT a business tool. | Nothing |
| **cc-research** | `/workspaces/qte77/claude-code-research` | Knowledge base. Documents CC capabilities, hooks, teams, skills, settings. Read-only reference. | Nothing |
| **cc-utils-plugin** | `/workspaces/qte77/claude-code-utils-plugin` | Tooling layer. Provides installable skills, plugins, scaffolds, hooks, rules consumed by all projects. Includes market-research plugin. | cc-research (informs design) |
| **market-research-to-gtm** | `github.com/qte77/agentic-market-research-to-gtm` | Source repo for the market research plugin. Packaged and distributed via cc-utils-plugin. | cc-utils-plugin (distribution) |
| **CABIO-test** | `/workspaces/qte77/CABIO-test` | Business pipeline orchestrator. BRD->PRD->FRD generation. Multi-project, multi-company dashboard and cost tracking. | cc-utils-plugin (all plugins including market-research) |
| **Ralph WT template** | `/workspaces/qte77/ralph-loop-cc-tdd-wt-vibe-kanban-template` | Coding engine. Takes PRD/FRD, runs TDD loops in git worktrees, produces working `src/` + `tests/`. | cc-utils-plugin (python-dev, ralph plugins) |

## Data Flow

```
cc-utils-plugin (installed into CABIO)
  market-research plugin (skills/commands)
  ↓ invoked via claude -p or CC skill
  Source Analysis → Landscape → PMF → GTM → Synthesis
  ↓ outputs feed into
CABIO (business pipeline)
  Business Input → BRD → [gate] → PRD → [gate] → FRD fan-out
  │                                                    │
  │ each FRD becomes a PRD for Ralph                   │
  ▼                                                    ▼
Ralph WT (coding pipeline)                    Ralph WT (coding pipeline)
  FRD-as-PRD → prd.json → TDD loop             FRD-as-PRD → prd.json → TDD loop
  → working code in src/ + tests/               → working code in src/ + tests/
```

## What Each Repo Does NOT Do

| Repo | Does NOT |
|------|----------|
| **polyforge** | Run business logic. Only dev infrastructure (scripts, workspace file, settings templates). |
| **cc-research** | Produce code, plugins, or runtime artifacts. Never imported. |
| **cc-utils-plugin** | Execute business logic or coding. Provides tools others use. |
| **market-research-to-gtm** | Run standalone. Distributed as cc-utils-plugin. Source repo for plugin development only. |
| **CABIO** | Write application code. Never touches `src/` implementation. |
| **Ralph WT** | Make business decisions. Executes whatever PRD/FRD it receives. |

## Integration Points

### Market Research Plugin → CABIO

The market research capability is **installed as a cc-utils-plugin** into CABIO,
not invoked as a separate repo. Source repo (`agentic-market-research-to-gtm`)
is for plugin development only.

**Install**: `claude plugin install market-research@qte77-claude-code-utils`

The plugin provides skills/commands that produce:
- Source analysis (technical capabilities assessment)
- Competitive landscape intelligence
- PMF analysis (product-market fit validation)
- GTM strategy (go-to-market plan)
- Contradiction analysis and synthesis

These outputs **feed into CABIO's BRD generation stage** as data-driven context.

**7-phase pipeline** (parallel where possible):
1. Phase 0 + 1A (parallel): Source analysis + Industry landscape
2. Phase 1B: Market research (depends on 0 + 1A)
3. Phase 2: PMF analysis
4. Phase 3: GTM strategy
5. Phase 4: Contradiction analysis
6. Phase 5: Synthesis
7. Phase 6: Slide decks

**Validation**: Each phase has validation loops with a results-validator subagent
(default 1 correction cycle).

**Modes**: Content depth (concise/detailed) x Strategic approach
(conservative/ambitious).

### CABIO → Ralph WT

CABIO's FRD output becomes Ralph's PRD input:
1. CABIO generates `context/FRDs/project_feature.md`
2. Cockpit copies/transforms FRD into Ralph's `docs/PRD.md` format
3. Cockpit runs `make ralph_init_loop && make ralph_run` in a project workspace
4. Ralph does TDD (red/green/refactor) in git worktrees
5. Ralph reports back: cost, story status, test results
6. Cockpit aggregates into project dashboard

### cc-utils-plugin → All Projects

Installed via `claude plugin install <plugin>@qte77-claude-code-utils`:

| Plugin | Used By | Purpose |
|--------|---------|---------|
| `workspace-setup` | CABIO, Ralph | SessionStart hook, statusline |
| `codebase-tools` | CABIO, Ralph | `researching-codebase`, `compacting-context` |
| `market-research` | CABIO | Source analysis, landscape, PMF, GTM strategy |
| `mas-design` | CABIO | Stateless reducer pattern, MAESTRO security |
| `docs-generator` | CABIO | `generating-tech-spec`, `generating-report` |
| `python-dev` | Ralph | `implementing-python`, `testing-python`, `reviewing-code` |
| `ralph` | Ralph | `generating-prd-json-from-prd-md` |

### cc-research → Design Decisions

Not a runtime dependency. Referenced during architecture and design:
- Hook event catalog → cockpit observability design
- Teams JSON structures → Phase 3 parallel agent design
- Settings.json reference → headless optimization env vars
- Skills frontmatter reference → custom skill creation
- Ralph enhancement research → Ralph WT improvements

## Cost Tracking Across Ecosystem

CABIO aggregates cost from multiple sources:

| Source | Cost Origin | Tracking Method |
|--------|------------|-----------------|
| Market research | `claude -p` via market-research plugin | Parse JSON output |
| BRD generation | `claude -p /project:generate-brd` | CC bridge `total_cost_usd` |
| PRD generation | `claude -p /project:generate-prd-from-brd` | CC bridge `total_cost_usd` |
| FRD generation | `claude -p /project:generate-frd-from-prd` | CC bridge `total_cost_usd` |
| Implementation | Ralph WT `claude -p` calls per story | Ralph progress.txt parsing |

Dashboard shows per-project total = sum of all stages.
