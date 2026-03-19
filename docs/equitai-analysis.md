# How GSD Can Help equitAI

An analysis of how the Get Shit Done (GSD) framework can accelerate and improve development of [equitAI](https://github.com/timmy16744/equitAI) — a multi-agent AI system for equity market analysis.

## Overview

**equitAI** is a multi-agent equity market analysis platform built with FastAPI, LangGraph, PostgreSQL (pgvector), Redis, React, and Claude Sonnet 4.5. It uses 8 specialized AI agents (Macro Economics, Geopolitical, Commodities, Sentiment, Fundamentals, Technical Analysis, Risk Management, and Aggregation Engine) orchestrated with LangGraph to produce actionable market insights.

**GSD** is a meta-prompting, context engineering, and spec-driven development system that manages complex AI-assisted software projects through structured workflows, fresh-context agents, and file-based state. It eliminates context rot — the quality degradation that occurs as AI fills its context window.

This document analyzes how GSD's patterns, workflows, and tooling directly map to equitAI's development challenges.

---

## 1. Multi-Agent Orchestration Patterns

### The Challenge

equitAI coordinates 8 specialized agents that must run in the right order with clean hand-offs. Its architecture has clear dependency chains:

```
Macro ──┐
Geo ────┤
Commod. ─┼──→ Aggregation Engine ──→ Risk Management ──→ Final Insights
Sentim. ─┤
Fundam. ─┤
Tech ────┘
```

Building and maintaining this orchestration layer is complex, with risks of context pollution between agents and cascading failures.

### How GSD Helps

GSD's **wave-based execution model** (see [Architecture](ARCHITECTURE.md)) maps directly to equitAI's agent dependency graph:

| GSD Concept | equitAI Application |
|-------------|---------------------|
| **Wave 1** (independent, parallel) | Macro, Geo, Commodities, Sentiment, Fundamentals, Technical agents run concurrently |
| **Wave 2** (depends on Wave 1) | Aggregation Engine synthesizes all Wave 1 outputs |
| **Wave 3** (depends on Wave 2) | Risk Management applies veto logic on aggregated insights |
| **Fresh context per agent** | Each market agent gets a clean 200K token window — no analysis cross-contamination |

**Configuration mapping** (see [Configuration](CONFIGURATION.md)):

```json
{
  "parallelization": {
    "enabled": true,
    "plan_level": true,
    "max_concurrent_agents": 6
  }
}
```

This pattern ensures the Macro agent's GDP analysis never leaks into the Technical agent's chart pattern recognition, preserving analysis independence.

---

## 2. Context Engineering for Financial Analysis

### The Challenge

Financial analysis involves large context windows: SEC filings, earnings reports, price histories, economic indicators, and news feeds. As analysis chains lengthen, AI quality degrades (context rot).

### How GSD Helps

GSD's core innovation — **fresh context per agent** — is critical for equitAI:

- **Each equitAI agent gets a clean context window** loaded only with relevant artifacts (e.g., the Commodities agent receives only commodity price data and supply chain reports, not tech stock earnings)
- **File-based state** (`.planning/` directory) makes all intermediate analysis human-readable and inspectable
- **Thin orchestrators** coordinate agents without consuming analysis context, reserving token budget for actual market data processing

**Practical benefit**: The Fundamentals agent analyzing Apple earnings won't be polluted by the Geopolitical agent's analysis of trade sanctions. Each analysis is independent and high-quality.

---

## 3. Spec-Driven Development Workflow

### The Challenge

equitAI is a complex full-stack system spanning backend (FastAPI + LangGraph), database (PostgreSQL + pgvector), cache (Redis + Celery), and frontend (React + TypeScript). Building it requires coordinating many components with clear requirements.

### How GSD Helps

GSD's **phased development lifecycle** (see [User Guide](USER-GUIDE.md)) provides a structured path:

| Phase | equitAI Development Activity |
|-------|------------------------------|
| **Phase 1**: Data Pipeline | Market data ingestion (Alpha Vantage, FRED, News API, Yahoo Finance) |
| **Phase 2**: Agent Implementation | 8 specialized agents with LangGraph orchestration |
| **Phase 3**: Aggregation & Risk | Aggregation Engine + Risk Management with veto logic |
| **Phase 4**: Frontend Dashboard | React UI with Recharts visualizations + WebSocket real-time updates |
| **Phase 5**: Deployment | Docker Compose production stack, monitoring, alerting |

For each phase, GSD provides:

1. **Research** — 4 parallel researchers investigate implementation approaches, API capabilities, pitfalls
2. **Planning** — Atomic task plans sized for single context windows, verified across 8 dimensions
3. **Execution** — Wave-based parallel execution with atomic git commits per task
4. **Verification** — Automated checks that all planned work actually shipped

---

## 4. Verification Layers for Financial Systems

### The Challenge

Financial software demands correctness. Bugs in risk calculations, trade execution, or data pipelines can have real monetary consequences. AI-generated code may contain stubs, hardcoded values, or disconnected logic.

### How GSD Helps

GSD's **4-layer verification model** (see [Architecture](ARCHITECTURE.md)) catches issues systematically:

| Layer | Check | equitAI Example |
|-------|-------|-----------------|
| **Exists** | File/function present | Risk agent module exists at `backend/agents/risk_management.py` |
| **Substantive** | Real implementation, not stubs | Volatility calculation uses actual price history, not `return 0.15` |
| **Wired** | Connected to system | Risk agent registered in LangGraph workflow graph |
| **Functional** | Produces correct output | Portfolio risk matches manual calculation within tolerance |

**Nyquist validation** ensures every component has corresponding test coverage during the planning phase — before any code is written. For equitAI, this means verifying that all 8 agents have analysis validation checks, all API integrations have error handling tests, and all risk calculations have boundary condition tests.

---

## 5. Test-Driven Development for Financial Algorithms

### The Challenge

equitAI's core value is in its analysis algorithms: volatility calculations, sentiment scoring, correlation analysis, and risk metrics. These must be provably correct.

### How GSD Helps

GSD's **TDD patterns** (see references) enforce test-first development for high-value logic:

**Example TDD plan for equitAI risk calculation:**

```
Phase: risk-management
Plan: portfolio-volatility

RED:   Write test for portfolio_volatility(holdings, price_history) → float
GREEN: Implement using actual correlation-weighted calculation
REFACTOR: Extract shared matrix operations for reuse

Test cases:
  - Single stock → historical standard deviation
  - Multi-stock → correlation-weighted portfolio volatility
  - Empty portfolio → 0.0
  - Missing price data → raises DataError
```

Each TDD feature produces 2–3 atomic commits (test → implementation → refactor), creating a bisectable history for debugging.

---

## 6. Git as Compliance Audit Trail

### The Challenge

Financial systems need audit trails. Regulators and stakeholders need to trace every decision: what analysis was performed, what data was used, what recommendations were made, and when.

### How GSD Helps

GSD's **git integration strategy** (see references) treats the commit log as a structured decision journal:

```
feat(01-01): implement macro economics agent with FRED integration
feat(01-02): implement geopolitical agent with news API pipeline
feat(02-01): implement aggregation engine with weighted synthesis
feat(02-02): implement risk management agent with veto logic
feat(03-01): implement real-time WebSocket dashboard
```

**Compliance benefits:**
- `git log --grep="risk"` shows all risk-related decisions
- `git diff <hash>^..<hash>` shows exact changes per analysis component
- `git blame` traces each line to the specific analysis task that produced it
- Each component is independently revertable without affecting others

---

## 7. Configuration-Driven Behavior

### The Challenge

equitAI needs different configurations for different contexts: backtesting (fast, relaxed validation) vs. live analysis (careful, strict risk checks) vs. development (verbose logging, mock data).

### How GSD Helps

GSD's configuration system (see [Configuration](CONFIGURATION.md)) provides environment-specific tuning:

| Setting | Backtest Mode | Live Analysis Mode |
|---------|---------------|-------------------|
| `model_profile` | `"budget"` — fast, cost-efficient | `"quality"` — highest accuracy |
| `parallelization.max_concurrent_agents` | `8` — all agents parallel | `3` — conservative resource use |
| `workflow.research` | `false` — skip, use cached data | `true` — fresh market research |
| `workflow.verifier` | `false` — speed over safety | `true` — verify every output |
| `workflow.node_repair` | `false` — fail fast | `true` — auto-retry failures |
| `workflow.node_repair_budget` | `0` | `2` — max 2 retries |

**Model overrides** allow allocating compute by criticality:

```json
{
  "model_overrides": {
    "risk-management-agent": "quality",
    "macro-economics-agent": "balanced",
    "technical-analysis-agent": "balanced",
    "aggregation-engine": "quality"
  }
}
```

---

## 8. Session Continuity and Failure Recovery

### The Challenge

Long-running market analysis may be interrupted (API rate limits, service restarts, context window exhaustion). The system must resume without losing progress.

### How GSD Helps

GSD's **session management** provides:

- **Pause/Resume** — Save analysis state to files, resume with full context reconstruction
- **State tracking** — `STATE.md` tracks which agents have completed, which are pending, and what data has been collected
- **Node repair** — Failed agents automatically retry up to a configurable budget before escalating
- **Atomic commits** — Each completed agent's output is committed independently, so partial progress is preserved

**Example recovery scenario:**
1. Agents 1–5 complete successfully (each committed)
2. Agent 6 (Technical Analysis) fails due to API rate limit
3. GSD detects failure, retries agent 6 with fresh context
4. If retry succeeds → continues to Wave 2
5. If retry budget exhausted → pauses with clear state for manual intervention

---

## 9. Codebase Mapping for Brownfield Integration

### The Challenge

equitAI already has existing code (backend, frontend, database schemas, Docker configuration). New development must integrate with what exists, not overwrite it.

### How GSD Helps

GSD's **codebase mapping** feature (see [Commands](COMMANDS.md)) provides:

- 4 parallel analysis agents map the existing codebase simultaneously
- **Technology stack discovery** — Identifies FastAPI routes, LangGraph workflows, SQLAlchemy models, React components
- **Architecture pattern identification** — Maps agent interaction patterns, data flow, API structure
- **Integration point analysis** — Identifies where new features connect to existing code

This is invoked with `/gsd:map-codebase` before any development phase, ensuring AI agents understand equitAI's existing structure before making changes.

---

## 10. Developer Profiling and Adaptive Behavior

### The Challenge

Different team members working on equitAI have different expertise levels (financial domain knowledge, Python proficiency, React experience) and different working styles.

### How GSD Helps

GSD's **user profiling** system (see [Agents](AGENTS.md)) adapts its behavior across 8 dimensions:

| Dimension | Adaptation for equitAI |
|-----------|----------------------|
| **Expertise** | More guidance for team members new to financial systems |
| **Autonomy** | Less confirmation prompting for experienced quant developers |
| **Communication** | Detailed explanations for complex financial algorithms |
| **Risk tolerance** | Stricter verification for risk-averse development phases |

---

## Summary: Key Benefits for equitAI

| GSD Capability | equitAI Benefit | Impact |
|----------------|-----------------|--------|
| Wave-based parallel execution | 6 market agents run concurrently, dependencies respected | **Performance** |
| Fresh context per agent | No analysis cross-contamination between market segments | **Quality** |
| 4-layer verification | Catch stubs, hardcoded values, disconnected logic in financial code | **Correctness** |
| TDD enforcement | Provably correct risk calculations, volatility models | **Reliability** |
| Git audit trail | Regulatory compliance, traceable decisions | **Compliance** |
| Configuration profiles | Switch between backtest/live/dev modes instantly | **Flexibility** |
| Session continuity | Resume interrupted analysis without losing progress | **Resilience** |
| Codebase mapping | New agents integrate correctly with existing FastAPI/LangGraph code | **Integration** |
| Nyquist validation | Every market segment and risk factor has test coverage | **Coverage** |
| Node repair | Auto-retry failed API calls and agent errors | **Availability** |

## Getting Started

To use GSD for equitAI development:

```bash
# Install GSD
npx get-shit-done-cc@latest

# Initialize equitAI as a GSD project
/gsd:new-project

# Provide equitAI requirements, market segments, risk constraints
# GSD will research, plan, and execute in structured phases
```

See the [User Guide](USER-GUIDE.md) for complete workflow walkthrough.
