# Data Models & Core Components (Phase 0)

Contracts and safety rules for Phase 0 implementation. Describes **what**
each component does and its invariants — not how to code it.

## Domain Enums

### PipelineStage

Ordered stages: `brd`, `prd`, `frd`, `execute`.

- BRD/PRD/FRD are CABIO's domain (invoked via `claude -p` commands)
- `execute` delegates to Ralph WT (`make ralph_run`), NOT `claude -p`

### StageStatus

Lifecycle: `pending` -> `in_progress` -> `passed` | `failed` | `skipped`.

## Data Models (Pydantic)

### StageRecord

One per pipeline stage execution.

| Field | Type | Purpose |
|-------|------|---------|
| stage | PipelineStage | Which stage |
| status | StageStatus | Current lifecycle state |
| started_at, completed_at | datetime? | Timing |
| output_path | str? | Where the artifact landed |
| cost_usd | float | Accumulated cost for this stage |
| error | str? | Failure reason if failed |
| retry_count | int | How many retries so far |
| feature_slug | str? | For FRD fan-out (one record per feature) |

### Project

One per tracked project.

| Field | Type | Purpose |
|-------|------|---------|
| id | str | Slug identifier |
| name | str | Display name |
| company | str | Company grouping (empty = ungrouped) |
| pipeline | dict[PipelineStage, StageRecord] | Stage execution state |
| workspace_path | str | Resolved filesystem path |
| tags, metadata | list, dict | Extensible user data |

### StateFile

Root persistence object.

| Field | Type | Purpose |
|-------|------|---------|
| version | str | Schema version for migration |
| generated | datetime | Last write timestamp |
| projects | dict[str, Project] | All tracked projects by id |

## Pipeline Config

### Stage-to-Command Registry

Map each stage to its CC command name. Avoids repeated dispatch chains.

| Stage | CC Command |
|-------|-----------|
| brd | `generate-brd` |
| prd | `generate-prd-from-brd` |
| frd | `generate-frd-from-prd` |
| execute | N/A — delegates to Ralph WT |

Stage order: brd -> prd -> frd -> execute.

## Component Contracts

### State Store

Persists `StateFile` to `state/projects.json`.

**Operations**: load, save, get_project, upsert_project, update_stage.

**Safety rules**:
- Atomic writes (tmp file + rename) — no partial state on disk
- Passed stages are immutable — cannot overwrite a stage with status `passed`

### CC Bridge

Wraps `claude -p "/project:{command} {args}" --output-format json` via subprocess.

**Returns**: returncode, stdout, cost_usd (from JSON `total_cost_usd`), duration_ms, error.

**Safety rules**:
- CWE-78 input sanitization (reject empty, dash-prefix, unsafe chars, length cap)
- Timeout handling

### Workspace Router

Resolves project workspace paths under `context/projects/`.

**Path scheme**:
- With company: `context/projects/<company-slug>/<project-slug>/`
- Without: `context/projects/<project-slug>/`

**Subdirs**: BRDs/, PRDs/, FRDs/, features/, logs/.

**Safety rule**: Idempotent creation (mkdir if not exists).
