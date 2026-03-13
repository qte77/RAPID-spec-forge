# CABIO Orchestration Cockpit -- Refactor Plan

Transformation plan: from manual `make` commands to Python orchestrator driving
Claude Code agents per project.

## Documents

| File | Purpose |
|------|---------|
| [current-state.md](current-state.md) | Inventory of existing assets, what to preserve, what is empty |
| [ecosystem.md](ecosystem.md) | How CABIO relates to cc-research, cc-utils-plugin, Ralph WT, and market-research-to-gtm |
| [transformation-plan.md](transformation-plan.md) | Phased implementation plan (Phase 0-3) with file lists and verification |
| [data-models.md](data-models.md) | Phase 0 Pydantic models, state store, CC bridge specs |
| [patterns-to-port.md](patterns-to-port.md) | Reusable patterns from Agents-eval, Ralph, cc-utils-plugin |
| [cc-integration.md](cc-integration.md) | Claude Code bridge design, settings, hooks, gotchas |
| [competitive-landscape.md](competitive-landscape.md) | CABIO vs Paperclip, PentaGI, CrewAI, AutoGen -- feature matrix |
| [cc-teams-org-simulation.md](cc-teams-org-simulation.md) | CC Teams mode as native org chart alternative to Paperclip |
| [external-references.md](external-references.md) | Industry research validating CABIO's existing design decisions |

## Key Architectural Decision

**Inversion**: From "human runs `make brd_gen_claude` manually per stage" to
"Python orchestrator calls `claude -p` automatically per project, tracking state
and cost."

**Scope boundary**: CABIO is the **business pipeline** (BRD->PRD->FRD). It does
NOT do coding. Coding is delegated to Ralph WT. Market research is delegated to
agentic-market-research-to-gtm.

## Ecosystem

```
cc-research          cc-utils-plugin                          CABIO            Ralph WT
(knowledge)          (tooling + market-research plugin)       (business)       (coding)
┌──────────┐        ┌──────────────────────────────┐        ┌──────────┐    ┌──────────┐
│ Documents │─→      │ Skills, Plugins, Scaffolds   │─→      │ BRD      │    │ PRD.md   │
│ CC caps   │informs │ Hooks, Rules, Statusline     │into    │ PRD      │    │ prd.json │
│ Hooks     │        │                              │both    │ FRD      │──→ │ TDD loop │
│ Teams     │        │ market-research plugin:      │        │ Dashboard│FRD │ Worktrees│
│ Settings  │        │   Source→Landscape→PMF→GTM   │        │ Cost     │as  │ Judge    │
│ OTel      │        │   (from agentic-mkt-to-gtm)  │        │ Multi-co │PRD │ Validate │
└──────────┘        └──────────────────────────────┘        └──────────┘    └──────────┘
  reference           consumed by all projects                 hands off to
  only                (plugins installed per project)          Ralph WT
```

## Constraints

- All 5 `.claude/commands/*.md` are preserved (proven prompt engineering)
- All `context/templates/*.md` are preserved
- `AGENTS.md` quality thresholds (8/7/8/7) are preserved
- Existing Makefile recipes are preserved (new ones appended)
- Zero new runtime deps in Phase 0-1 (Pydantic needs adding to pyproject.toml)
- Optional `rich` dep for Phase 2 dashboard only
- CABIO never touches `src/` code -- that's Ralph WT's domain
