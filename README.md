# Context Engineering Template

> ## ⚠️ Archived 2026-04-26 — superseded by https://github.com/qte77/qte77
>
> [`qte77/qte77`](https://github.com/qte77/qte77) is taking over. Active follow-up tracking has moved. See:
>
> | What | Where it landed |
> | --- | --- |
> | Spec methodology (BRD → PRD → FRP) | To be extracted as `spec-forge` plugin in [`qte77/claude-code-plugins`](https://github.com/qte77/claude-code-plugins) |
> | `goal_id` frontmatter for templates (was [#5](https://github.com/qte77/RAPID-spec-forge/issues/5)) | [`qte77/qte77#50`](https://github.com/qte77/qte77/issues/50) |
> | Spec-lifecycle cockpit (was [#9](https://github.com/qte77/RAPID-spec-forge/issues/9) / [PR #1](https://github.com/qte77/RAPID-spec-forge/pull/1)) | [`qte77/qte77#51`](https://github.com/qte77/qte77/issues/51) |
> | Doc-search skill + memory store (was [PR #10](https://github.com/qte77/RAPID-spec-forge/pull/10)) | [`qte77/qte77#52`](https://github.com/qte77/qte77/issues/52) — code preserved in branch [`chore/license-apache-2.0`](https://github.com/qte77/RAPID-spec-forge/tree/chore/license-apache-2.0) |
>
> The content below is preserved as historical context.

Business workflow system with AI agent support. Evolving toward intelligent orchestration: from business analysis to requirements to implementation.

[![License](https://img.shields.io/badge/license-Apache--2.0-green.svg)](LICENSE)
![Version](https://img.shields.io/badge/version-1.0.0-58f4c2)
[![CodeQL](https://github.com/qte77/context-engineering-template/actions/workflows/codeql.yaml/badge.svg)](https://github.com/qte77/context-engineering-template/actions/workflows/codeql.yaml)
[![CodeFactor](https://www.codefactor.io/repository/github/qte77/context-engineering-template/badge)](https://www.codefactor.io/repository/github/qte77/context-engineering-template)
[![ruff](https://github.com/qte77/context-engineering-template/actions/workflows/ruff.yaml/badge.svg)](https://github.com/qte77/context-engineering-template/actions/workflows/ruff.yaml)
[![pytest](https://github.com/qte77/context-engineering-template/actions/workflows/pytest.yaml/badge.svg)](https://github.com/qte77/context-engineering-template/actions/workflows/pytest.yaml)

**DevEx**  [![vscode.dev](https://img.shields.io/static/v1?logo=visualstudiocode&label=&message=vscode.dev&labelColor=2c2c32&color=007acc&logoColor=007acc)](https://vscode.dev/github/qte77/context-engineering-template)
[![Codespace](https://img.shields.io/static/v1?logo=visualstudiocode&label=&message=Codespace%20Dev&labelColor=2c2c32&color=007acc&logoColor=007acc)](https://github.com/codespaces/new?repo=qte77/context-engineering-template&devcontainer_path=.devcontainer/devcontainer.json)

## Product State & Vision

### Current State

- **Template-based business workflow system** for small teams (5-25 people) using `BRD → PRD → FRP` generation via make commands and Claude Code.
- Business Requirements Definition to Product Requirements Definition to Feature Request Prompt to Implementation
- Business Requierements can be provided via complete requirements template, description or iterative process
- Cline and Gemini configurations are present as fall-back solutions.

### RAPID Vision

> **Formerly CABIO** (Context-Aware Business Intelligence Orchestration).
> Renamed to **RAPID-spec-forge** (Requirements-to-Agent Pipeline &
> Implementation Driver: Spec Forge) to better reflect the tool's function
> as a requirements pipeline orchestrator rather than a business analytics
> platform.
> Legacy repo: [qte77/CABIO-test](https://github.com/qte77/CABIO-test)

- **RAPID-spec-forge** evolving from enhanced templates with agent orchestration (8-12 weeks) to real-time market data integration and enterprise scalability (12+ months)

### Business Analytics Outlook

RAPID focuses on requirements orchestration (BRD→PRD→FRD), but analytics
capabilities can be layered on top via existing ecosystem repos:

| Capability | Source | Integration |
|-----------|--------|-------------|
| Market research & GTM analysis | `pmf-gtm` / `agentic-market-research-to-gtm` | Feed market data into BRDs |
| Coding agent evaluation metrics | `ai-agents-research` | Inform agent selection per FRD |
| Product lifecycle tracking | `sdlc-lcm-manager` | Phase-aware dashboards |
| Cost/token tracking | OTel metrics from CC runs | Budget per BRD→FRD cycle |
| Quality trend analysis | `make validate` + test coverage history | Trend per sprint/product |

Future: if analytics become a first-class concern, extract to a dedicated
analytics module or integrate with Tuleap CE (see
[oss-alm-landscape.md](https://github.com/qte77/ai-agents-research/blob/main/docs/sdlc-lcm/oss-alm-landscape.md)).

## Quick Start

```bash
# 1. Setup environment
make setup_dev

# 2. business-driven product development approach wih example files
make brd_gen_claude "ARGS=example_ai_assistant.md"        # Business Requierements Definition
make prd_gen_claude "ARGS=example_ai_assistant.md"        # Product Requierements Definition
make frp_gen_claude "ARGS=example_ai_assistant.md task_automation"    # Feature Request Prompt generation
make frp_exe_claude "ARGS=example_ai_assistant_task_automation.md"    # Feature Request Prompt execution
```

## The Product

**Template-based business workflow system for small teams (5-25 people).**

Structured `BRD → PRD → FRP` workflow with AI agent configurations. Currently uses templates and make commands, evolving toward automated business intelligence orchestration.

**Problem**: Small teams need structured business analysis but lack dedicated resources for professional frameworks.

**Current Solution**: Template-driven workflow with Claude Code, Cline, and Gemini agent support

**Evolution Path**: Enhanced templates → context management → intelligent agent orchestration (RAPID)

### Current Capabilities (Template System)

- **Business-Driven Development**: Complete BRD → PRD → FRP workflow via templates
- **AI Agent Configurations**: Ready-to-use prompts for Claude Code, Cline, and Gemini
- **Quality Automation**: Integrated testing, linting, and validation workflows
- **Professional Output**: Business documentation ready for stakeholders

## Evolution: RAPID (Requirements-to-Agent Pipeline & Implementation Driver)

### Enhanced Templates (8-12 weeks)

- **Agent Orchestration**: Automated handoffs between specialized AI agents
- **Context Compression**: Smart information filtering and allocation
- **Market Research Integration**: Templates with competitive analysis frameworks
- **Context Preservation**: Better information flow between analysis phases
- **Streamlined Workflow**: Reduced manual editing and refinement time

### RAPID Vision (12+ months)

- **Real-time Intelligence**: Live market data and competitive monitoring
- **Enterprise Scalability**: Advanced workflows for larger organizations
- **Accessible Intelligence**: Professional business analysis without dedicated specialists
- **Faster Iterations**: Complete business requirements in hours rather than days

**Goal**: Transform template-based workflow into intelligent agent orchestration.

Building on proven BRD→PRD→FRP foundation, RAPID represents our evolution path: enhanced templates (8-12 weeks) → agent orchestration → comprehensive requirements pipeline for small teams scaling to larger organizations.

### Learn More

- **[RAPID Vision](docs/RAPID-vision.md)** - Vision for automated requirements orchestration
- **[RAPID Product Roadmap](docs/RAPID-product-roadmap.md)** - Customer value, target markets, and feature development
- **[RAPID Implementation Guide](docs/RAPID-implementation-guide.md)** - Technical architecture and implementation roadmap

**Development**: Iterative approach starting with enhanced templates for small teams (8-12 weeks), then agent orchestration, expanding based on validation and user feedback. Current BRD→PRD→FRP workflow remains the foundation.

## Workflow Visualizations

<details>
    <summary>📊 Current Workflow (Template-based) - Visualization</summary>
    <img src="assets/images/Business-Driven-Development-Current-Concise-light.png#gh-light-mode-only" alt="Current Business-Driven Development Workflow Diagram" title="Visualization: Current template-based BRD→PRD→FRP workflow with AI agent support" width="80%" />
    <img src="assets/images/Business-Driven-Development-Current-Concise-dark.png#gh-dark-mode-only" alt="Current Business-Driven Development Workflow Diagram" title="Visualization: Current template-based BRD→PRD→FRP workflow with AI agent support" width="80%" />
</details>
    <details>
    <summary>🔄 Enhanced Templates (8-12 weeks) - Agent Orchestration Visualization</summary>
    <img src="assets/images/Business-Driven-Development-RAPID-SMB-Concise-light.png#gh-light-mode-only" alt="Enhanced Templates Agent Orchestration Diagram" title="Visualization: Agent orchestration with context compression and 50% less manual editing" width="80%" />
    <img src="assets/images/Business-Driven-Development-RAPID-SMB-Concise-dark.png#gh-dark-mode-only" alt="Enhanced Templates Agent Orchestration Diagram" title="Visualization: Agent orchestration with context compression and 50% less manual editing" width="80%" />
</details>
<details>
    <summary>🚀 Enterprise RAPID (12+ months) - Real-time Intelligence Visualization</summary>
    <img src="assets/images/Business-Driven-Development-RAPID-Enterprise-Concise-light.png#gh-light-mode-only" alt="Enterprise RAPID Requirements Pipeline Diagram" title="Visualization: Real-time market data with enterprise scalability and multi-model optimization" width="80%" />
    <img src="assets/images/Business-Driven-Development-RAPID-Enterprise-Concise-dark.png#gh-dark-mode-only" alt="Enterprise RAPID Requirements Pipeline Diagram" title="Visualization: Real-time market data with enterprise scalability and multi-model optimization" width="80%" />
</details>

## Comprehensive Usage

### Business-Driven Development (Recommended)

1. **Create business input**: `cp context/templates/business_input_base.md context/business_inputs/my_project.md`
2. **Generate BRD**: `make brd_gen_claude "ARGS=my_project.md"`
3. **Generate PRD**: `make prd_gen_claude "ARGS=my_project.md"`
4. **Generate FRPs**: `make frp_gen_claude "ARGS=my_project.md feature_name"`
5. **Implement**: `make frp_exe_claude "ARGS=my_project_feature_name.md"`

### Legacy Development

1. **Create feature**: `cp context/templates/feature_base.md context/features/my_feature_description.md`
2. **Generate FRP**: `make frp_gen_legacy_claude "ARGS=my_feature_description.md"`
3. **Implement**: `make frp_exe_legacy_claude "ARGS=my_feature_frp.md"`

## Status

**Current**: Template-based BRD→PRD→FRP workflow functional for small teams. Enhanced templates in development (8-12 weeks), RAPID orchestration planned (12+ months).

See [CHANGELOG.md](CHANGELOG.md) for version history.

## Documentation

- **[Usage Guide](docs/usage-guide.md)** - Detailed workflow instructions
- **[Examples](docs/examples.md)** - Complete examples and demonstrations
- **[AGENTS.md](AGENTS.md)** - Agent configuration and behavior
- **[examples/mcp-server-client/](examples/mcp-server-client/)** - Working MCP implementation, implemented with legacy custom commands workflow

## Lineage

This repository continues [`context-engineering-template-legacy`](https://github.com/qte77/context-engineering-template-legacy) (created 2025-07-06; CABIO vision introduced 2025-08-11), where the BRD → PRD → FRP pipeline originated. The legacy repo is reference-only.
