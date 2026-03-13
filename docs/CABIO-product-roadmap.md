# CABIO Product Roadmap

What we're building and the value it delivers

## Value Proposition

**Python orchestration cockpit for the BRD→PRD→FRD pipeline, managing multiple projects across multiple companies with cost tracking and quality gates.**

CABIO transforms the current make-command workflow into a Python orchestrator that drives Claude Code agents, tracks costs, enforces quality gates, and provides dashboard visibility. See [transformation-plan.md](refactor/transformation-plan.md) for the authoritative engineering plan.

## Customer Pain Points & Gains

### Current Pain Points (Small Teams Focus)

- **Lack of Structure**: Business decisions made without systematic analysis framework
- **Manual Template Refinement**: Current BRD generation requires extensive manual editing and context gathering
- **Time Pressure**: Need structured business requirements quickly for product decisions
- **Documentation Gap**: Informal business thinking not suitable for investors or team alignment
- **Resource Constraints**: Small teams lack dedicated business analysts or market researchers

### Customer Gains with CABIO

- **Automated Pipeline**: Cockpit auto-chains BRD→PRD→FRD stages with quality gate scoring
- **Multi-Project Visibility**: Dashboard shows all projects, stages, costs across companies
- **Reduced Manual Work**: Quality gates and fan-out replace manual orchestration
- **Professional Output**: Business documentation ready for stakeholders and implementation

## Target Customers

### Primary Markets

1. **Small Teams & Startups (5-25 people)** — *Primary Focus*

   - Need structured business analysis but lack dedicated roles
   - Making critical product decisions with limited resources
   - Want professional business intelligence without complexity

2. **Solo Entrepreneurs & Founders** — *Core Market*

   - Making business decisions without analysis resources
   - Need validation for product ideas and market strategies
   - Want professional business documentation for investors

3. **Small Consulting Firms** — *Early Adopters*

   - Need to deliver business analysis quickly for clients
   - Want consistent, high-quality frameworks at small team scale
   - Require cost-effective strategic services delivery

### Secondary Markets (Future Phases)

1. **SMB Companies (25-100 people)** — *Phase 2 Expansion*

   - Growing teams needing scalable business intelligence
   - Multiple product lines requiring coordinated analysis

2. **Enterprise Teams** — *Phase 3 / Future*

   - Complex market dynamics and real-time intelligence needs
   - Advanced workflow integration and compliance requirements

## Implementation Phases

### Phase 0: Foundation (Week 1)

- [ ] **Data Models**: Pydantic models for PipelineStage, StageStatus, StageRecord, Project, StateFile
- [ ] **State Persistence**: JSON persistence with atomic writes
- [ ] **CC Bridge**: `claude -p` subprocess wrapper for headless execution
- [ ] **Workspace Isolation**: Project-level path resolvers

### Phase 1: Single-Project Orchestration (Weeks 2-3)

- [ ] **Pipeline Runner**: Auto-chain BRD→PRD→FRD stages in order
- [ ] **Quality Gates**: CC-scored quality assessment between stages
- [ ] **FRD Fan-Out**: Extract features from PRD, generate one FRD per feature
- [ ] **Execute Handoff**: FRD→Ralph WT for TDD implementation
- [ ] **CLI**: `cabio project create`, `cabio run`, `cabio status`

### Phase 2: Multi-Project Cockpit (Weeks 3-4)

- [ ] **Dashboard**: Rich terminal table showing projects, stages, costs
- [ ] **Parallel FRD**: ThreadPoolExecutor for concurrent FRD generation
- [ ] **Multi-Company**: Workspace isolation per company, cost aggregation
- [ ] **Vibe Kanban Integration**: Optional REST push for project tracking

### Phase 3: Intelligence Layer (Future — Design Only)

Not implemented. Architecture accommodates:

- [ ] **Automated Market Intelligence**: Pre-BRD stage using market-research plugin
- [ ] **Memory Systems**: Long-term, working, episodic per project
- [ ] **Cost Budgets**: Auto-throttle at 80% of budget
- [ ] **Agent Runtime Abstraction**: Protocol for CC and Gemini backends
- [ ] **MCP / Real-Time Data**: Live connections to business intelligence sources

## Core Technology Stack

- **Python**: Orchestration cockpit, Pydantic models, CLI
- **Claude Code CLI**: `claude -p` headless execution for pipeline stages
- **Ralph WT**: TDD implementation via git worktrees (execute stage handoff)
- **Rich** (optional): Terminal dashboard rendering

## Expected Benefits

### Pipeline Automation

- **Auto-Chaining**: Stages run in sequence without manual intervention
- **Quality Gates**: Failed stages block progression with actionable feedback
- **Fan-Out**: Multiple FRDs generated in parallel from a single PRD

### Cost Visibility

- **Per-Stage Tracking**: Token usage and cost logged per CC invocation
- **Per-Project Aggregation**: Dashboard shows total cost per project
- **Per-Company Rollup**: Multi-company cost reporting

### Multi-Project Management

- **Workspace Isolation**: Each project has independent state and artifacts
- **Dashboard**: Single view across all projects and companies
- **Parallel Execution**: Independent projects and features run concurrently

## Risks & Mitigation

### Technical Risks

- **CC Bridge Reliability**: Implement retries and timeout handling for `claude -p`
- **State Persistence**: Atomic writes prevent corruption on interruption
- **Quality Gate Accuracy**: Iterative calibration of scoring prompts

### Process Risks

- **Learning Curve**: CLI mirrors existing `make` command patterns
- **Backward Compatibility**: Existing make recipes remain functional
- **Data Privacy**: Workspace isolation prevents cross-project data leakage

## Success Metrics

### Phase 0-1 Success Criteria

- [ ] **Pipeline Completion**: Single project runs BRD→PRD→FRD→Execute end-to-end
- [ ] **Quality Gates**: Stages blocked on low-quality output, re-run succeeds
- [ ] **State Persistence**: Pipeline resumes after interruption from last completed stage
- [ ] **Tests Pass**: `make ruff`, `make check_types`, `make test_all` all green

### Phase 2 Success Criteria

- [ ] **Dashboard Renders**: 3+ projects across 2+ companies display correctly
- [ ] **Parallel FRD**: 5 features complete with correct state records
- [ ] **Cost Accuracy**: Dashboard sums match individual stage costs

## Next Steps

1. **Phase 0**: Data models, state store, CC bridge, workspace isolation
2. **Phase 1**: Pipeline runner, quality gates, FRD fan-out, CLI
3. **Phase 2**: Dashboard, parallel FRD, multi-company
4. **Phase 3**: Intelligence layer (design only, implemented when validated)

---

*This document will be updated as implementation progresses. See [transformation-plan.md](refactor/transformation-plan.md) for detailed file lists and verification steps.*
