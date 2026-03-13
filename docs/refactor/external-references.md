# External References & Validation

Industry research that validates CABIO's existing design decisions. These
concepts were already present in CABIO's scaffold and architecture before
encountering these sources -- listed here as supporting evidence, not origins.

## Validated Design Decisions

### 1. Multi-Layer Verification Gates

Industry pattern: Replace line-by-line review with automated verification gates
(security, compliance, standards). Humans review only architecture and business
logic.

**Already in CABIO**: Quality gates between pipeline stages (AGENTS.md 8/7/8/7
thresholds). Reinforces the multi-layer approach:
- Automated: template completeness, section validation
- Agent-evaluated: `claude -p` scoring against thresholds
- Human sense-check: optional `--require-human` for high-stakes transitions

### 2. Upstream Quality as Amplifier

Industry finding: AI amplifies both good and bad decisions. Increasing
throughput with unclear thinking makes problems more expensive, not cheaper.

**Already in CABIO**: BRD quality determines everything downstream. The
BRD->PRD->FRD chain is inherently an amplifier -- a bad BRD produces many bad
FRDs fast. Reinforces strict gating at BRD->PRD as the highest-leverage point.

### 3. Cognitive Load as the Real Bottleneck

Industry finding: Parallelism increases throughput but human working memory
is the bottleneck. Better tooling must externalize context.

**Already in CABIO**: The dashboard (Phase 2) exists precisely for this reason.
Reinforces adding **attention signals** (which project needs input now?) not
just status display.

### 4. Product-Minded Architect as Target User

Industry trend: Engineer role evolving from coder to architect to "founder" --
value in knowing **what** to build, not how.

**Already in CABIO**: The cockpit IS the tool for this role. Human owns intent,
coherence, risk. Agents own execution across BRD->PRD->FRD->Ralph WT.

### 5. Modular Prompts Over Mega-Context

Industry failure: Shared rules repositories break under their own weight --
manual distribution, context windows can't hold everything, compaction loses
nuance.

**Already in CABIO**: `.claude/commands/*.md` structure is modular by design.
CC's `/project:` command routing keeps prompts focused. Reinforces: don't
concatenate all context into a single prompt.

### 6. Encoded Best Practices for Teams Without Champions

Industry finding: Tools are commoditized. Differentiation is in how much
potential gets materialized. AI champions unlock potential for others.

**Already in CABIO**: The cockpit encodes best practices into the pipeline
itself -- the tool IS the champion for teams that lack one.

## CABIO Differentiators (Not Found in Industry)

| Capability | Status |
|-----------|--------|
| Per-project cost tracking across all stages | Core (Phase 0) |
| Multi-project, multi-company orchestration | Core (Phase 2) |
| Persistent pipeline state with atomic writes | Core (Phase 0) |
| Business requirements pipeline upstream of code | Core (Phase 1) |
| Market intelligence plugin feeding BRD | Plugin (cc-utils-plugin) |
| Delegation to separate coding engine (Ralph WT) | Core architecture |
| Remote/async execution | Phase 3 (`claude --remote`) |

## Sources

- [Parloa: The engineer, reimagined](https://www.parloa.com/labs/insights/ai-driven-development-parloa-engineer-reimagined/) (2026-03-12) -- 500+ tools CC kitchen, >95% AI code, verification gates, cognitive load findings
