# Competitive Landscape

Comparison of CABIO against similar projects in the AI orchestration space.

## CABIO vs Paperclip

### What Paperclip Is

[Paperclip](https://github.com/paperclipai/paperclip) (MIT, TypeScript, 21k+
stars, v0.3.1, 2026-03-02): Node.js + React orchestration for "zero-human
companies." PostgreSQL-backed, self-hosted.

Core abstraction: a company is an **org chart of AI agents** with roles, budgets,
heartbeats, and governance. Agents are workers (Claude Code, Codex, Cursor, etc.)
receiving tasks via a ticket system.

### Feature Comparison

| Dimension | CABIO | Paperclip |
|-----------|-------|-----------|
| **Core abstraction** | Business document pipeline (BRD->PRD->FRD) | Company org chart with agent employees |
| **What it orchestrates** | Document generation stages with quality gates | Agent teams with roles and reporting lines |
| **Agent model** | Claude Code only (via `claude -p`) | Any agent (CC, Codex, Cursor, HTTP) |
| **Pipeline concept** | Linear chain with quality gates | No pipeline -- ticket-based delegation |
| **Coding** | Delegated to Ralph WT | Delegated to whatever agent fills "engineer" role |
| **Market research** | cc-utils-plugin (market-research) | No built-in concept |
| **Cost tracking** | Per-stage, per-project, aggregated | Per-agent monthly budgets with auto-pause |
| **Multi-company** | Workspace isolation via filesystem paths | PostgreSQL-level data isolation |
| **State persistence** | JSON file + atomic writes | PostgreSQL |
| **Governance** | Quality gates (8/7/8/7 thresholds) | Board approval model (approve hires, strategy) |
| **UI** | Terminal dashboard (rich) | React web UI + mobile |
| **Scheduling** | On-demand (human or CLI) | Heartbeats (cron-like agent wake cycles) |
| **Audit trail** | State file + git commits | Immutable append-only log in DB |
| **Stack** | Python, Pydantic, minimal deps | Node.js, React, PostgreSQL |
| **Maturity** | Pre-implementation (docs/refactor) | v0.3.1, 21k stars, active |

### Complementary, Not Competing

They solve different problems at different layers:

```
Paperclip (company layer)     "Run a business with AI agents"
  ↓ could use CABIO for product strategy
CABIO (business pipeline)     "Generate requirements from intent"
  ↓ hands FRDs to
Ralph WT (coding layer)       "Implement features via TDD"
```

### What CABIO Can Learn

| Concept | Value | Priority |
|---------|-------|----------|
| Per-agent budgets with auto-pause | Per-project/stage cost caps, not just tracking | HIGH |
| Heartbeats (scheduled wake) | Pipelines on schedule, not just on-demand | MEDIUM (Phase 3) |
| Immutable audit log | Append-only log alongside state file | MEDIUM |
| Agent-agnostic runtime | `AgentRuntime(Protocol)` in Phase 3 | Already planned |
| Portable templates (Clipmart) | Reusable BRD/PRD configs per industry | Future |

### What Paperclip Lacks

| CABIO Capability | Gap |
|-----------------|-----|
| Document quality gates | No staged quality validation |
| Business requirements pipeline | No BRD/PRD/FRD |
| Market intelligence integration | No built-in market research |
| FRD fan-out | No feature extraction from requirements |
| Separation of "what" from "how" | Agents code directly |
| Template-driven generation | No structured business document templates |

### CC Teams as Alternative to Paperclip Org Chart

See [cc-teams-org-simulation.md](cc-teams-org-simulation.md) for analysis of
using Claude Code Teams mode to achieve Paperclip's org chart behavior natively,
without the Node.js/React/PostgreSQL infrastructure.

## Closest Competitor: aidoc-flow-framework

[vladm3105/aidoc-flow-framework](https://github.com/vladm3105/aidoc-flow-framework)
(MIT, Python, 9 stars): "AI-First Specification-Driven Development (SDD)" --
the most directly comparable project.

### Comparison

| Dimension | CABIO | aidoc-flow-framework |
|-----------|-------|---------------------|
| **Pipeline** | BRD->PRD->FRD (3 stages) | REF->BRD->PRD->EARS->BDD->ADR->... (up to 15 layers) |
| **Depth variants** | Single pipeline | SDD-Lite (4 layers), Standard (7), Full (15) |
| **Document quality** | Quality gates with 8/7/8/7 thresholds | Per-layer validator->reviewer->fixer->audit cycle |
| **Multi-project** | Core (multi-company dashboard) | Single project focus |
| **Coding delegation** | Ralph WT (separate engine) | Not separated -- same framework |
| **Market research** | cc-utils-plugin (market-research) | No built-in concept |
| **Cost tracking** | Per-stage, per-project | Not tracked |
| **Agent model** | Claude Code (CC bridge) | CC, Gemini CLI, Copilot |
| **Skill system** | cc-utils-plugin ecosystem | Built-in skills (doc-brd, doc-prd, etc.) |
| **MVP lifecycle** | Not formalized | Core (MVP->PROD->NEW MVP cycle) |
| **Playbooks.com** | No | Published skills on playbooks.com |

### What CABIO Can Learn

- **Validator->Reviewer->Fixer cycle**: More granular than a single quality gate.
  Could split CABIO's gate into validation (structural), review (semantic), fix
  (automated correction).
- **MVP->PROD->NEW MVP lifecycle**: Formalizes iteration. CABIO could track which
  BRD iteration a project is on (BRD-01, BRD-02, ...).
- **Scalable depth (Lite/Standard/Full)**: CABIO could offer depth modes for
  different project sizes.

### Where CABIO Differentiates

- **Multi-project cockpit**: aidoc-flow is single-project.
- **Cost tracking and budgets**: Not in aidoc-flow.
- **Separated coding engine**: aidoc-flow doesn't separate "what to build" from
  implementation.
- **Market intelligence plugin**: No equivalent in aidoc-flow.
- **CC Teams org simulation**: Not in aidoc-flow.

## Also Noted: OrbitOS (artificialoutreach.com)

Commercial "AI Product Manager" -- PRD generation from vision. Not open source,
limited public technical detail. Validates market demand for the capability but
not a direct competitor to CABIO's open-source, multi-project approach.

## Competitive Matrix

| Capability | CABIO | Paperclip | aidoc-flow | CrewAI | AutoGen |
|-----------|-------|-----------|------------|--------|---------|
| **Business requirements pipeline** | Core (BRD->PRD->FRD) | -- | Core (up to 15 layers) | -- | -- |
| **Document quality gates** | Core (8/7/8/7) | -- | Core (validator->reviewer->fixer) | -- | -- |
| **Multi-project / multi-company** | Core | Core | -- | -- | -- |
| **Cost tracking** | Core | Core | -- | -- | -- |
| **Agent-agnostic runtime** | Phase 3 | Core | Partial (CC, Gemini, Copilot) | Core | Core |
| **Org chart / hierarchy** | CC Teams | Core | -- | Role-based | Role-based |
| **Scheduled execution** | Phase 3 | Core (heartbeats) | -- | -- | -- |
| **Web UI / mobile** | Phase 2 (terminal) | Core (React) | -- | -- | -- |
| **Governance / approval** | Core (quality gates) | Core (board model) | Core (audit cycle) | -- | -- |
| **Coding delegation** | Ralph WT | Any agent | Not separated | Built-in | Built-in |
| **Market research** | Plugin | -- | -- | Tool | Tool |
| **MVP lifecycle** | -- | -- | Core (BRD-01->02->03) | -- | -- |
| **Depth variants** | -- | -- | Core (Lite/Standard/Full) | -- | -- |
| **Self-hosted** | Yes | Yes | Yes | Cloud + self | Yes |
| **Stack** | Python | TypeScript | Python | Python | Python |
| **Stars** | -- | 21k+ | 9 | 26k+ | 40k+ |
| **Primary use case** | Business intent -> requirements -> code | Autonomous company ops | Spec-driven development | Task automation | Research agents |

### Key Insight

CABIO's unique position: **multi-project business pipeline with separated coding
engine and market intelligence**. aidoc-flow-framework is closest in document
pipeline depth but lacks multi-project orchestration and cost tracking. Paperclip
is closest in multi-company operations but lacks business requirements pipeline.
No existing tool combines all three.
