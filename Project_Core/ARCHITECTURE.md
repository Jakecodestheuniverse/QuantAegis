# Architecture

> **[PROJECT_NAME] — System Design & Technical Decisions**

This document describes the high-level architecture, design philosophy, and key technical decisions behind [PROJECT_NAME]. It is intended for contributors, enterprise evaluators, and anyone doing a technical assessment of the system.

---

## Design Philosophy

Five principles guide every architectural decision:

1. **Auditability First** — Every decision made by any agent must be traceable to its inputs, reasoning, and model version. No black boxes.
2. **Modularity** — Components are composable. Use only what you need.
3. **Regulatory Alignment** — Data flows, storage, and access patterns are designed with MiFID II, SEC, and Basel III in mind from day one — not bolted on.
4. **No Vendor Lock-In** — Model-agnostic, cloud-agnostic. Run on OpenAI, Anthropic, local LLaMA, or any combination.
5. **Production-Grade Defaults** — Sensible security defaults, structured logging, and health checks out of the box.

---

## System Layers

```
┌──────────────────────────────────────────────────────────────┐
│                      API / Interface Layer                    │
│      REST API  |  Python SDK  |  CLI  |  Webhook Receivers   │
├──────────────────────────────────────────────────────────────┤
│                    Orchestration Layer                        │
│         Agent Router  |  Task Planner  |  Memory Manager     │
├─────────────────┬──────────────────┬─────────────────────────┤
│  Research &     │  Risk & Quant    │   Compliance &           │
│  Analysis       │  Agents          │   Reporting Agents       │
│  Agents         │                  │                          │
│  ─────────      │  ─────────────   │   ─────────────────      │
│  • Earnings     │  • Portfolio     │   • Regulatory           │
│    Analysis     │    Risk (VaR,    │     Report Gen           │
│  • News/NLP     │    CVaR)         │   • Trade Surveillance   │
│  • Filing       │  • Factor        │   • AML Flagging         │
│    Parser       │    Attribution   │   • Audit Trail          │
│  • Alt Data     │  • Stress Test   │     Export               │
│    Signals      │  • Backtest      │                          │
├─────────────────┴──────────────────┴─────────────────────────┤
│                  Data & Model Layer                           │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  Data Connectors                                      │    │
│  │  Market: Bloomberg / Refinitiv / yFinance / Polygon  │    │
│  │  Alt: News APIs / SEC EDGAR / Social Sentiment       │    │
│  │  Internal: Existing firm data via connector plugins  │    │
│  └──────────────────────────────────────────────────────┘    │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  Model Registry                                       │    │
│  │  LLMs: OpenAI, Anthropic, LLaMA, Mistral, custom     │    │
│  │  Quant: FinGPT adapters, custom fine-tuned models    │    │
│  │  Traditional ML: sklearn, XGBoost pipelines          │    │
│  └──────────────────────────────────────────────────────┘    │
├──────────────────────────────────────────────────────────────┤
│              Audit & Explainability Layer                     │
│   Decision Logger | XAI Engine | Lineage Tracker | Diff Log  │
├──────────────────────────────────────────────────────────────┤
│                   Infrastructure Layer                        │
│   PostgreSQL | Redis | S3-compatible | Vector DB (pgvector)   │
└──────────────────────────────────────────────────────────────┘
```

---

## Key Components

### 1. Orchestration Layer

The **Agent Router** receives a task (e.g., "generate risk report for portfolio X") and decomposes it into sub-tasks, routing each to the appropriate specialized agent. It maintains task state and handles retries, timeouts, and partial failures gracefully.

The **Memory Manager** provides:
- **Short-term memory** — in-context state within a single agent run (Redis-backed)
- **Long-term memory** — vector-search over historical analyses and firm-specific knowledge (pgvector)
- **Episodic memory** — structured log of past agent runs for reproducibility

### 2. Agent Framework

All agents inherit from a base `FinancialAgent` class that enforces:
- Structured input/output schemas (Pydantic models)
- Mandatory logging of every external call
- Output confidence scoring
- Automatic audit trail emission

Agents are stateless by design — all state is externalized to the memory and data layers.

### 3. Audit & Explainability Layer

This is [PROJECT_NAME]'s primary differentiator. Every agent action emits a structured **AuditEvent**:

```python
class AuditEvent(BaseModel):
    event_id: UUID
    timestamp: datetime
    agent_id: str
    agent_version: str
    action: str
    inputs: dict          # sanitized inputs
    outputs: dict         # model outputs
    model_used: str
    model_version: str
    reasoning_trace: list[str]    # chain-of-thought steps
    confidence: float
    regulatory_tags: list[str]    # e.g., ["MiFID_II_Art_17", "SEC_15a-6"]
    human_readable_summary: str
```

Events are written to an append-only audit log (immutable, tamper-evident). Exporters are available for common SIEM and GRC platforms.

### 4. Compliance Rule Engine

Compliance agents evaluate outputs against a **rule registry** — a versioned, human-readable set of rules expressed in a domain-specific language:

```yaml
rule_id: MIFID_BEST_EXEC_001
description: "Best execution obligation check"
regulation: MiFID II Article 27
severity: HIGH
condition: |
  trade.venue_price <= market.best_bid_ask_spread_midpoint * 1.001
action_on_breach: ALERT | BLOCK | LOG
```

Rules are versioned, auditable, and can be extended by firms without forking the core project.

### 5. Data Layer

All data connectors implement a standard `DataConnector` interface, making it trivial to swap or add sources. Data is validated against Pydantic schemas at ingestion. PII and sensitive firm data is handled via a **Data Masking Middleware** that can be configured per-deployment.

---

## Security Architecture

- **Zero-trust internal networking** — agents authenticate with each other via short-lived JWT tokens
- **Secrets management** — integrates with HashiCorp Vault, AWS Secrets Manager, or Azure Key Vault; never reads secrets from environment variables in production mode
- **Data encryption** — at-rest (AES-256) and in-transit (TLS 1.3) by default
- **Role-based access control (RBAC)** — fine-grained permissions per agent type and data source
- **Dependency scanning** — `pip-audit` runs in CI on every PR

See [SECURITY.md](SECURITY.md) for the full security posture and vulnerability disclosure policy.

---

## Deployment Topologies

| Topology | Use Case |
|---|---|
| **Local / Laptop** | Development, research, prototyping |
| **Docker Compose** | Small team deployment, demos |
| **Kubernetes** | Enterprise production deployment |
| **Air-gapped** | Highly regulated environments (no internet egress) |

Helm charts and Terraform modules are provided in `deploy/`.

---

## Technology Choices

| Component | Choice | Rationale |
|---|---|---|
| Language | Python 3.10+ | Ecosystem fit; type safety via mypy |
| LLM Abstraction | LiteLLM | Provider-agnostic, battle-tested |
| Agent Framework | Custom (thin) | Avoids vendor lock-in; full auditability |
| Vector DB | pgvector (PostgreSQL extension) | Minimizes operational complexity for enterprises already running Postgres |
| Task Queue | Celery + Redis | Mature, well-understood in enterprise environments |
| Schema Validation | Pydantic v2 | Performance + strict type safety |
| Financial Math | `decimal.Decimal` + `numpy` | Precision-critical financial calculations use Decimal |
| API | FastAPI | Async, OpenAPI spec auto-generation |
| Serialization | Protobuf (internal) + JSON (external) | Efficiency + interoperability |

---

## What [PROJECT_NAME] Is NOT

- **Not a trading system** — outputs are analytical; execution is your responsibility
- **Not financial advice** — all outputs are labeled as analytical tools and must be reviewed by qualified humans
- **Not a data vendor** — you bring your own data; we provide the connectors and the intelligence layer

---

## RFC Process

Significant architectural changes go through an RFC (Request for Comments) process. See `docs/rfcs/README.md` for the template and process.
