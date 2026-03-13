# Architecture Decisions

Architectural Decision Records (ADRs) capturing key design choices made during
the refactor planning phase. Each decision documents context, options considered,
and rationale.

## ADR-001: FRD over FRP Terminology

**Status**: Accepted

**Context**: CABIO v1 used "FRP" (Feature Requirements Prompt) for the third
pipeline stage. This conflated the prompt mechanism with the output artifact.

**Decision**: Rename to FRD (Feature Requirements Document) throughout. Legacy
file names in the codebase are preserved with "to be renamed during refactor"
annotations.

**Rationale**: FRD accurately describes the output (a document), not the
mechanism (a prompt). Aligns with industry terminology (BRD, PRD, FRD).

## ADR-002: Execute Stage Delegates to Ralph WT

**Status**: Accepted

**Context**: The execute stage could invoke `claude -p` directly for coding,
or delegate to an external engine.

**Decision**: The execute stage spawns Ralph WT (`make ralph_init_loop` +
`make ralph_run`), not `claude -p`. CABIO never generates application code.

**Rationale**: Separation of concerns. CABIO is the business pipeline
(BRD->PRD->FRD). Ralph WT is the coding engine (TDD in worktrees, judge
validation). Each does one thing well.

## ADR-003: Market Research as cc-utils-plugin

**Status**: Accepted

**Context**: `agentic-market-research-to-gtm` could be used as a standalone
repo, a git submodule, or a cc-utils-plugin.

**Decision**: Distributed as a plugin via cc-utils-plugin. Installed with
`claude plugin install market-research@qte77-claude-code-utils`.

**Rationale**: Plugin distribution means zero code porting, automatic
updates, and consistent installation across CABIO projects. The plugin runs
within CABIO's CC session -- no separate repo invocation needed.

## ADR-004: CC Teams over Paperclip for Org Simulation

**Status**: Accepted (Phase 3)

**Context**: Paperclip provides org chart simulation with Node.js + React +
PostgreSQL. CC Teams (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`) provides
similar capability natively.

**Decision**: Use CC Teams as the org chart runtime. Each role (Product
Strategist, Requirements Architect, Quality Reviewer, SWE Lead) is a
teammate with role-specific cc-utils-plugins.

**Rationale**: Zero infrastructure overhead. CC Teams runs in the same CC
session. Tradeoff: less persistent state (JSON vs PostgreSQL), no web UI.
For CABIO's use case (quality-gated document generation, not 24/7 autonomous
company), this tradeoff is favorable.

See [cc-teams-org-simulation.md](cc-teams-org-simulation.md).

## ADR-005: Python over Rust/Go for Orchestrator

**Status**: Accepted

**Context**: The orchestrator could be written in Python, Rust, or Go.

**Decision**: Python with Pydantic.

**Rationale**: Ecosystem alignment (cc-utils-plugin is Python-oriented),
Pydantic for data validation, Hypothesis for property testing, existing
`pyproject.toml` tooling (ruff, pyright). The orchestrator is I/O-bound
(subprocess calls to `claude -p`), not CPU-bound -- Python is sufficient.

## ADR-006: JSON State File over Database

**Status**: Accepted

**Context**: State could be stored in PostgreSQL (like Paperclip), SQLite,
or a JSON file.

**Decision**: JSON file at `state/projects.json` with atomic writes
(tmp + `os.replace()`). Immutable passed stages.

**Rationale**: Zero runtime deps. Portable (state travels with workspace).
Git-diffable. Sufficient for the expected scale (tens of projects, not
thousands). Atomic writes prevent corruption. Pattern proven in Ralph's
`prd.json`.

## ADR-007: cc-research as Read-Only Reference

**Status**: Accepted

**Context**: cc-research contains extensive CC capability documentation,
hook analysis, teams orchestration research, and settings references.

**Decision**: cc-research is read-only reference material. It is never
imported, installed, or executed. Design decisions reference it but don't
depend on it at runtime.

**Rationale**: cc-research is a living research repo that may change
independently. CABIO should be informed by it, not coupled to it.

## ADR-008: Plugin Split Between CABIO and Ralph WT

**Status**: Accepted

**Context**: cc-utils-plugin provides many plugins. Both CABIO and Ralph WT
could install all of them.

**Decision**: Each project installs only the plugins it needs:
- **CABIO**: workspace-setup, codebase-tools, market-research, mas-design,
  docs-generator
- **Ralph WT**: python-dev, ralph

**Rationale**: Minimal context pollution. Each CC session loads only the
skills relevant to its role. Prevents accidental scope creep (e.g., CABIO
should never have `implementing-python` skills).

## ADR-009: Quality Gates Preserve AGENTS.md Thresholds

**Status**: Accepted

**Context**: Quality gate scoring could use arbitrary thresholds or
match existing project conventions.

**Decision**: Preserve the existing AGENTS.md 8/7/8/7 thresholds
(Context: 8/10, Clarity: 7/10, Alignment: 8/10, Success: 7/10) for
document quality gates between pipeline stages.

**Rationale**: These thresholds are proven in practice. Consistency
across CABIO and its agent infrastructure reduces cognitive load.
