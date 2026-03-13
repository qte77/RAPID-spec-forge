# CABIO Vision

Why Context-Aware Business Intelligence Orchestration matters

## The Problem We've Experienced

**Small teams lack structured business analysis frameworks.**

From our own development experience, we've seen how small teams (5-25 people) struggle with business decisions:

- No dedicated business analysts or market researchers on team
- Ad-hoc business analysis leads to inconsistent, incomplete frameworks
- Templates exist but require extensive manual refinement and context gathering
- Business requirements disconnect from market realities due to resource constraints

**The result?** Small teams make critical product decisions without professional business intelligence frameworks, while larger organizations have dedicated resources for structured analysis.

## Our Vision: The Python Orchestration Cockpit

**A Python cockpit that drives the BRD→PRD→FRD pipeline across multiple projects and companies.**

We envision an evolution from the current make-command workflow toward a Python orchestrator that manages Claude Code agents, tracks costs, enforces quality gates, and provides dashboard visibility.

**Phase 0 (Foundation)**: Data models, state persistence, CC bridge, workspace isolation

- Pydantic models for projects, stages, and state
- JSON persistence with atomic writes
- `claude -p` subprocess wrapper for headless CC execution

**Phase 1 (Single-Project Orchestration)**: Pipeline runner, quality gates, FRD fan-out, CLI

- Auto-chain BRD→PRD→FRD stages with quality gate scoring
- Fan-out: extract features from PRD, generate one FRD per feature
- Execute stage hands off to Ralph WT for TDD implementation
- CLI: `cabio project create`, `cabio run`, `cabio status`

**Phase 2 (Multi-Project Cockpit)**: Dashboard, parallel FRD, multi-company workspace

- Rich terminal dashboard showing all projects, stages, and costs
- Parallel FRD generation via ThreadPoolExecutor
- Multi-company workspace isolation

**Phase 3 (Future — Design Only)**: Intelligence layer

- Automated market intelligence as pre-BRD stage
- Memory systems, cost budgets, agent runtime abstraction
- Not implemented — architecture accommodates future extension

See [transformation-plan.md](refactor/transformation-plan.md) for the authoritative engineering plan.

## Core Value Propositions

### Multi-Project, Multi-Company Management

One cockpit manages pipelines for multiple projects across multiple companies. Each project has isolated workspace, state, and cost tracking.

### Cost Tracking & Visibility

Every CC invocation logs token usage and cost. Dashboard aggregates per-project and per-company. Phase 3 adds budget auto-throttling.

### Quality Gates

Each pipeline stage passes through a quality gate (CC-scored) before advancing. Failed gates block progression with actionable feedback.

## Why This Matters Now

### The AI Agent Revolution

AI agents are sophisticated enough to handle complex business analysis, but they work in isolation. We need orchestration that makes agents work together intelligently — CABIO never codes itself, it orchestrates CC agents that do.

### The Small Team Reality

Small teams (5-25 people) typically operate without dedicated business analysts, market researchers, or strategy consultants due to budget and scale constraints. The cockpit automates the orchestration overhead so teams focus on decisions, not process.

## Our North Star

**Transform business decision-making from ad-hoc to structured.**

Instead of making decisions based on incomplete analysis, small teams should have access to professional business intelligence frameworks that scale with their growth.

We're evolving from make-command invocations toward a **Python orchestration cockpit** — starting with foundation and single-project automation, building toward multi-project dashboards and intelligence layers.

## Personal Mission

This comes from our own experience building products:

- **Frustration** with manual business analysis refinement and context gathering
- **Recognition** that structured frameworks improve product decisions
- **Belief** that automated orchestration can reduce analysis overhead
- **Vision** that small teams deserve professional business intelligence frameworks

CABIO evolves business intelligence from manual template refinement toward intelligent agent orchestration for teams of any size.

---

*This vision drives everything we build — from foundation models to cockpit dashboards to intelligence layers.*
