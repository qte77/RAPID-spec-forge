# Data Models & Core Components (Phase 0)

## Models (`src/models/project.py`)

```python
from enum import StrEnum
from datetime import datetime
from pydantic import BaseModel, Field


class PipelineStage(StrEnum):
    """Pipeline stages in execution order.

    BRD/PRD/FRP are CABIO's domain (claude -p commands).
    Execute hands off to Ralph WT (make ralph_run).
    """
    brd = "brd"
    prd = "prd"
    frp = "frp"
    execute = "execute"  # delegates to Ralph WT, not claude -p


class StageStatus(StrEnum):
    """Stage execution status."""
    pending = "pending"
    in_progress = "in_progress"
    passed = "passed"
    failed = "failed"
    skipped = "skipped"


class StageRecord(BaseModel):
    """Record of a single pipeline stage execution."""
    stage: PipelineStage
    status: StageStatus = StageStatus.pending
    started_at: datetime | None = None
    completed_at: datetime | None = None
    output_path: str | None = None
    cost_usd: float = 0.0
    error: str | None = None
    retry_count: int = 0
    feature_slug: str | None = None  # for FRP fan-out


class Project(BaseModel):
    """A project tracked by the cockpit."""
    id: str  # slug
    name: str
    company: str = ""
    description: str = ""
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)
    pipeline: dict[PipelineStage, StageRecord] = Field(default_factory=dict)
    workspace_path: str = ""
    tags: list[str] = Field(default_factory=list)
    metadata: dict[str, str] = Field(default_factory=dict)


class StateFile(BaseModel):
    """Root state file persisted to JSON."""
    version: str = "1"
    generated: datetime = Field(default_factory=datetime.utcnow)
    projects: dict[str, Project] = Field(default_factory=dict)
```

## Pipeline Config (`src/models/pipeline.py`)

```python
from src.models.project import PipelineStage

# Registry dict mapping stage -> CC command name (BRD/PRD/FRP)
# Execute stage is NOT a CC command -- it delegates to Ralph WT
STAGE_COMMANDS: dict[PipelineStage, str] = {
    PipelineStage.brd: "generate-brd",
    PipelineStage.prd: "generate-prd-from-brd",
    PipelineStage.frp: "generate-frp-from-prd",
    # execute: handled by Ralph WT (make ralph_run), not CC command
}

# Ralph WT configuration for execute stage
RALPH_COMMAND = "make ralph_run"
RALPH_INIT_COMMAND = "make ralph_init_loop"

STAGE_ORDER: list[PipelineStage] = [
    PipelineStage.brd,
    PipelineStage.prd,
    PipelineStage.frp,
    PipelineStage.execute,
]
```

## State Store (`src/store/state_store.py`)

```python
"""JSON file persistence for project state."""

import json
import os
import tempfile
from pathlib import Path
from src.models.project import StateFile, Project, PipelineStage, StageRecord

DEFAULT_STATE_PATH = Path("state/projects.json")


class StateStore:
    """Atomic JSON state persistence.

    Safety rules:
    - Atomic write via tmp + os.replace()
    - Passed stages are immutable (from Ralph pattern)
    """

    def __init__(self, path: Path = DEFAULT_STATE_PATH) -> None:
        self.path = path

    def load(self) -> StateFile:
        if not self.path.exists():
            return StateFile()
        data = json.loads(self.path.read_text())
        return StateFile.model_validate(data)

    def save(self, state: StateFile) -> None:
        self.path.parent.mkdir(parents=True, exist_ok=True)
        data = state.model_dump_json(indent=2)
        # Atomic write: write to tmp, then os.replace()
        fd, tmp = tempfile.mkstemp(
            dir=self.path.parent, suffix=".tmp"
        )
        try:
            os.write(fd, data.encode())
            os.close(fd)
            os.replace(tmp, self.path)
        except Exception:
            os.close(fd) if not os.get_inheritable(fd) else None
            if os.path.exists(tmp):
                os.unlink(tmp)
            raise

    def get_project(self, project_id: str) -> Project | None:
        state = self.load()
        return state.projects.get(project_id)

    def upsert_project(self, project: Project) -> None:
        state = self.load()
        state.projects[project.id] = project
        self.save(state)

    def update_stage(
        self, project_id: str, record: StageRecord
    ) -> None:
        state = self.load()
        project = state.projects.get(project_id)
        if project is None:
            raise KeyError(f"Project {project_id!r} not found")
        # Immutability: passed stages cannot be overwritten
        existing = project.pipeline.get(record.stage)
        if existing and existing.status.value == "passed":
            raise ValueError(
                f"Stage {record.stage.value!r} already passed, immutable"
            )
        project.pipeline[record.stage] = record
        self.save(state)
```

## CC Bridge (`src/runner/cc_bridge.py`)

```python
"""Subprocess wrapper for claude -p invocations."""

import json
import re
import subprocess
import time
from pydantic import BaseModel


class BridgeResult(BaseModel):
    """Result from a Claude Code command invocation."""
    returncode: int
    stdout: str = ""
    cost_usd: float = 0.0
    duration_ms: int = 0
    error: str | None = None


# CWE-78 input sanitization
_SAFE_ARG = re.compile(r"^[a-zA-Z0-9_./ -]+$")
_MAX_ARG_LEN = 500


def _sanitize(arg: str) -> str:
    if not arg or not arg.strip():
        raise ValueError("Empty argument")
    if arg.startswith("-"):
        raise ValueError(f"Argument must not start with dash: {arg!r}")
    if len(arg) > _MAX_ARG_LEN:
        raise ValueError(f"Argument too long ({len(arg)} > {_MAX_ARG_LEN})")
    if not _SAFE_ARG.match(arg):
        raise ValueError(f"Unsafe characters in argument: {arg!r}")
    return arg


def run_command(
    command: str,
    args: str,
    workspace: str,
    timeout: int = 600,
) -> BridgeResult:
    """Run a Claude Code command via subprocess.

    Args:
        command: CC command name (e.g. "generate-brd")
        args: Arguments to pass
        workspace: Working directory for claude
        timeout: Timeout in seconds

    Returns:
        BridgeResult with returncode, stdout, cost, duration
    """
    command = _sanitize(command)
    args = _sanitize(args)

    prompt = f"/project:{command} {args}"
    cmd = [
        "claude", "-p", prompt,
        "--output-format", "json",
    ]

    start = time.monotonic()
    try:
        result = subprocess.run(
            cmd, capture_output=True, text=True,
            timeout=timeout, cwd=workspace,
        )
    except subprocess.TimeoutExpired:
        return BridgeResult(
            returncode=-1,
            error=f"Timeout after {timeout}s",
            duration_ms=int((time.monotonic() - start) * 1000),
        )

    duration_ms = int((time.monotonic() - start) * 1000)
    cost = 0.0

    # Parse cost from JSON output
    try:
        data = json.loads(result.stdout)
        cost = float(data.get("total_cost_usd", 0.0))
    except (json.JSONDecodeError, TypeError, ValueError):
        pass

    return BridgeResult(
        returncode=result.returncode,
        stdout=result.stdout,
        cost_usd=cost,
        duration_ms=duration_ms,
        error=result.stderr if result.returncode != 0 else None,
    )
```

## Workspace Router (`src/runner/workspace.py`)

```python
"""Project workspace isolation and path resolution."""

from pathlib import Path

BASE_PROJECTS_DIR = Path("context/projects")


def _slug(text: str) -> str:
    """Convert text to filesystem-safe slug."""
    return text.lower().replace(" ", "-").strip("-")


def workspace_root(project_id: str, company: str = "") -> Path:
    """Resolve project workspace root.

    Multi-company: context/projects/<company>/<project>/
    No company:    context/projects/<project>/
    """
    if company:
        return BASE_PROJECTS_DIR / _slug(company) / _slug(project_id)
    return BASE_PROJECTS_DIR / _slug(project_id)


def ensure_workspace(project_id: str, company: str = "") -> Path:
    """Create project workspace with all required subdirs."""
    root = workspace_root(project_id, company)
    for subdir in ["BRDs", "PRDs", "FRPs", "features", "logs"]:
        (root / subdir).mkdir(parents=True, exist_ok=True)
    return root


def brd_path(project_id: str, company: str = "") -> Path:
    return workspace_root(project_id, company) / "BRDs"


def prd_path(project_id: str, company: str = "") -> Path:
    return workspace_root(project_id, company) / "PRDs"


def frp_path(project_id: str, company: str = "") -> Path:
    return workspace_root(project_id, company) / "FRPs"
```
