# Artificial Trading Agent

Proprietary **AI trading research platform** for building, evaluating, and evolving decision systems on market data.

Model line: **ATA-v1 → ATA-v2 → ATA-v3**

This is **not** a ChatGPT wrapper. It is a full AI system you own end-to-end: data, models, agents, risk, paper execution, and learning loops.

---

## What This Project Is

Most people treat “AI” as a single model file.

In production reality:

```text
              Artificial Trading Agent
                         │
        ┌────────────────┼────────────────┐
        │                │                │
  Data Pipeline      AI Model       Trading Engine
        │                │                │
        └────────────────┼────────────────┘
                         │
                   Learning System
```

| Pillar | Responsibility |
|--------|----------------|
| **Data Pipeline** | Stream and store market truth (candles, trades, gaps, backfill) |
| **AI Model** | Map state → `BUY` / `SELL` / `WAIT` / `EXIT` (+ size/risk later) |
| **Trading Engine** | Risk checks + paper execution with fees/slippage |
| **Learning System** | Train, compare, promote, monitor, retrain |

The model is the heart. The **system** is the product.

---

## Goals

### Primary product goal

Build an **Artificial Trading Agent** that can:

1. Ingest live and historical market data
2. Engineer features into versioned datasets
3. Produce trading decisions with a proprietary model (`ATA-*`)
4. Orchestrate Observe → Think → Decide → Act via an agent graph
5. Execute only through a risk-gated paper broker (live later, if ever earned)
6. Track experiments scientifically and improve over time
7. Act as a **research platform** where LSTM, Transformer, PPO, or future architectures can be plugged in and compared fairly

### Secondary learning goal

Use this build to develop production AI engineering skills:

Python → Data Engineering → ML → Deep Learning → Time Series → RL → Agentic AI → MLOps → Serving → Deployment → Monitoring

---

## Model Generations

| Version | Focus | Outputs |
|---------|--------|---------|
| **ATA-v1** | Supervised / sequence models (MLP, LSTM, small Transformer) | `action`, `confidence` |
| **ATA-v2** | Reinforcement learning policies (Gymnasium + Stable-Baselines3) | + `position_size`, `risk`, `expected_reward` |
| **ATA-v3** | Agentic orchestration (LangGraph tools, memory, planning) | Same + auditable plan/tool traces |

### Action space (v1)

```text
BUY   → open / increase long exposure (per engine rules)
SELL  → short / reduce long (semantics fixed in the engine)
WAIT  → no new action
EXIT  → flatten current position
```

Action meaning must be identical in the model, RL environment, agent, and paper broker.

---

## High-Level Architecture

```text
Exchange (e.g. Binance)
        │
        ▼
WebSocket + AsyncIO Ingest (+ REST backfill)
        │
        ├──────────► MySQL (history / audit)
        └──────────► Redis (hot state, later)
        │
        ▼
Pandas / NumPy Feature Pipeline
        │
        ▼
ATA Model (PyTorch / Stable-Baselines3)
        │
        ▼
FastAPI  POST /predict
        │
        ▼
LangGraph Agent
  Observe → Think → Risk Check → Decide → Execute
        │
        ▼
Risk Engine ──► Paper Trading Engine
        │
        ▼
Results → MySQL
        │
        ├──► MLflow (train / compare / promote)
        └──► Prometheus + Grafana (monitor)
```

### Hot path (decision tick)

```text
Bar close
  → build features / load window
  → agent graph starts
  → risk pre-check (fresh data, kill switch)
  → model inference
  → risk post-check (size, limits)
  → paper execute or skip
  → persist prediction + order + equity
  → export metrics
```

**Hard rule:** the model proposes; the **risk engine disposes**. Nothing bypasses risk.

---

## Design Principles

1. **Model ≠ system** — architecture first, model second
2. **Paper first** — live trading is a privilege, not a default
3. **Version everything** — data, features, model, agent graph, configs
4. **Baselines before deep learning** — Scikit-Learn sanity floor always
5. **No time leakage** — walk-forward evaluation only
6. **Fail closed** — stale data, errors, or limit breaches → no trade
7. **Swappable brains** — same body, different `ATA` versions behind one interface
8. **Observable by default** — every decision must be auditable
9. **Simple before clever** — add Kafka/Redis/Polars only when measured need appears

---

## Technology Stack

| Layer | Tools | Why |
|-------|-------|-----|
| Language | Python, asyncio | AI-native stack + concurrent IO |
| Market data | websockets, httpx | Live stream + historical backfill |
| Storage | MySQL, Redis (later) | Durable history + hot cache |
| Processing | Pandas, NumPy | Cleaning, features, arrays |
| Classical ML | Scikit-Learn | Baselines and metrics |
| Deep Learning | **PyTorch** (MPS on Mac) | Proprietary neural models |
| RL | **Gymnasium** + **Stable-Baselines3** | Env + PPO/DQN-style policies |
| Agent | **LangGraph** | Tool-using decision workflow |
| Serving | **FastAPI**, Pydantic | Stable `/predict` contracts |
| MLOps | **MLflow** | Experiment tracking + registry |
| Deploy | Docker, GitHub Actions | Repeatable packaging/CI |
| Monitor | Prometheus, Grafana | System + strategy health |
| Future | Kafka, Polars | Scale when bottlenecks appear |

### Example inference contract

```http
POST /predict
```

```json
{
  "action": "BUY",
  "confidence": 0.91,
  "position_size": 0.25,
  "risk": 0.12,
  "expected_reward": 0.04,
  "model_version": "ATA-v1.3",
  "features_version": "dataset_v1.4"
}
```

---

## Target Repository Structure

Spine-first growth (do not create empty cathedrals on day one):

```text
aiCandlePattern/
├── README.md
├── pyproject.toml
├── configs/                     # app, model, risk, train
├── apps/
│   ├── ingest/                  # websocket + backfill
│   ├── features/                # dataset jobs
│   ├── train/                   # sklearn / torch / rl
│   ├── serve/                   # FastAPI inference
│   ├── agent/                   # LangGraph nodes/tools
│   ├── engine/                  # risk + paper broker
│   └── monitor/                 # metrics exporters
├── packages/
│   ├── ata_core/                # shared types/contracts
│   ├── ata_data/                # DB / cache access
│   ├── ata_models/              # swappable model adapters
│   └── ata_env/                 # Gymnasium trading env
├── migrations/
├── docker/
├── scripts/
├── tests/
└── .github/workflows/
```

---

## Delivery Roadmap (Spine)

| Stage | Deliverable |
|-------|-------------|
| 0 | Python package skeleton + configs |
| 1 | Candle ingest → MySQL (REST backfill + WS) |
| 2 | Feature job + versioned dataset |
| 3 | Scikit-Learn baseline + MLflow tracking |
| 4 | ATA-v1 (PyTorch) + FastAPI `/predict` |
| 5 | Risk engine + paper broker closed loop |
| 6 | LangGraph agent on bar-close events |
| 7 | ATA-v2 RL candidate (Gymnasium + SB3) |
| 8 | Docker Compose + Grafana MVP |
| 9 | Multi-model research harness (fair compare) |

---

## Evaluation Philosophy

A model is not “good” because win-rate looks nice in a notebook.

We care about:

- Walk-forward / purged validation (no shuffled time series)
- Cost-aware metrics (fees, slippage, turnover)
- Drawdown and path risk, not only average return
- Stability across regimes
- Reproducible runs tied to data + code + config versions
- Promotion gates before any paper (or future live) activation

Abort examples: stale feed, max daily loss, elevated error rate, broken model distribution.

---

## Hardware Notes

Primary development target: **MacBook Pro (Apple Silicon)** with PyTorch **MPS**.

Suitable for: ingest, features, API, paper trading, agents, small/medium training, RL research at project scale.

Not the target for: multi-billion-parameter training or massive multi-GPU jobs — use cloud GPU later if needed.

---

## Current Status

- [x] Product vision and architecture defined
- [ ] Implementation spine (ingest → features → ATA-v1 → paper loop)

---

## Safety

- Research / educational system first
- Paper trading until evaluation is boringly consistent
- Secrets never committed (`.env` is ignored)
- Live trading, if ever enabled, requires separate config, tiny limits, and a kill switch

Markets are non-stationary. Past paper results do not guarantee future performance.
