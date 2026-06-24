<div align="center">

<img src="https://raw.githubusercontent.com/crediqs/.github/main/assets/crediqs-banner.png" width="100%" />

# crediqs

**Financial risk intelligence platform. Graph-native. Memory-native. Regulator-ready.**

## What crediqs is

crediqs is the only risk platform where TradFi derivatives and DeFi positions share the same probability-of-default curve, the same CVA engine, and the same regulator-citable explanation.
---

very number is traceable. Every decision is explainable. Every assessment makes the next one smarter.

## Platform architecture

```
 Data Ingestion          OIS · CDS · FX · vol · DeFi · climate
       ▼
 Real-Time Signals       GNN contagion · cashflow velocity · NLP sentiment
       ▼
 Climate Overlay          Physical risk · transition risk · NGFS v5 · OSFI SCSE
       ▼
 Orchestration            LangGraph agents · Decision Engine · Memory Layer (6 layers)
       ▼
 Six Risk Engines         Credit · Liquidity · Third-party · DeFi · ESG · Market
       ▼
 Explainability           SHAP · lineage · trace · counterfactual · fairness
       ▼
 Stress Testing           EBA · CCAR · FRTB · ICAAP · reverse stress · backtest
       ▼
 Outputs                  Regulatory reports · decisions · alerts · explanations
       ▼
 Platform             Projects · environments · DashboardLoader · AI Workspace
```

## Repositories

| Repository | Description | Tests | Status |
|---|---|---|---|
| [`pyccr`](https://github.com/crediqs/pyccr) | Counterparty credit risk and market risk engine. CVA/DVA/FVA/KVA via Hull-White Monte Carlo, SA-CCR, VaR/ES, OIS curve bootstrapping, CDS curve, sensitivity analysis. | 487 | ● Active |
| [`pyccr.vol`](https://github.com/crediqs/pyccr) | Volatility surface module. BSM, Black-76, SABR (Hagan 2002), Heston (1993), neural calibration, Greeks (1st + 2nd order), FRTB SBA vega + curvature charges. | 144 | ◐ Building |
| [`pycredit`](https://github.com/crediqs/pycredit) | IFRS 9 expected credit loss engine. PD/LGD/EAD, WOE logistic scorecard, XGBoost PD, SICR detection, vintage analysis, forward-looking 3-scenario weighting. | 225 | ● Active |
| [`pychain`](https://github.com/crediqs/pychain) | DeFi risk engine. Liquidation bootstrap PD, smart contract exploit scoring, stablecoin/MiCA, governance HHI, oracle risk, bridge risk, GNN contagion, real-time monitor. | 367 | ● Active |
| [`crediqs-core`](https://github.com/crediqs/crediqs-core) | Shared infrastructure. Unified PDCurve, CreditRating, TransitionMatrix, CalibrationRegistry (73 parameters), RiskEngineResult protocol, zero-fallback contracts. | 29 | ● Active |
| [`crediqs-data`](https://github.com/crediqs/crediqs-data) | Data ingestion layer. TradFi plane (FRED, ECB, CME), on-chain plane (DefiLlama, Dune, The Graph), carbon plane (Toucan, Verra). CalibrationRegistry store. GraphStore (NetworkX + DeXposure-FM). | 205 | ● Active |
| [`crediqs-explain`](https://github.com/crediqs/crediqs-explain) | Explainability and model governance. 5 layers: data lineage, feature attribution (SHAP/analytic/transparent), decision trace, counterfactual, fairness testing. 9 registered adapters. | 195 | ● Active |
| [`crediqs-lab`](https://github.com/crediqs/crediqs-lab) | Stress testing and model validation. Scenario engine (EBA, CCAR, historical, climate, DeFi), sensitivity sweep, backtest (55 events, traffic light, Kupiec), reverse stress, ICAAP. | 142 | ◐ Extending |
| [`crediqs-api`](https://github.com/crediqs/crediqs-api) | FastAPI REST API. Risk engine orchestrator, 6 engine routers (v0.7), MiCA compliance endpoint, stress/sensitivity endpoints. Live on Railway. | — | ● Active |
| [`crediqs-catalogue`](https://github.com/crediqs/crediqs-catalogue) | YAML-driven node catalogue. Input/output schemas, governance metadata, regulatory citations. Database-seeded, data-driven — zero hardcoded definitions. | — | ● Active |
| [`agent-service`](https://github.com/crediqs/agent-service) | LangGraph 6-node intelligence agent. Classify → plan → execute → assess → synthesize → persist. Memory Layer (6 layers), graph enrichment, explain integration. | — | ● Active |
| [`platform-backend`](https://github.com/crediqs/platform-backend) | FastAPI backend. Projects, environments, workflows, cases, Decision Engine (22 rules), model governance lifecycle, memory authority, WebSocket streaming. | — | ● Active |
| [`platform-frontend`](https://github.com/crediqs/platform-frontend) | Next.js 15 / React 19 / TypeScript. R3 architecture: DashboardLoader, WIDGETS registry, node-driven rendering, Settings → Models governance, AI Workspace copilot. | — | ● Active |

## Technical foundation

### Unified PDCurve

The single most important architectural decision. Three PD sources — CDS bootstrap (market-implied), WOE/XGBoost scorecard (statistical/ML), and on-chain liquidation bootstrap (DeFi) — produce the same `PDCurve` object that feeds the same CVA engine and the same ECL computation.

```python
# Same interface, three sources
pd_curve = CDSCurve.from_spread(85, lgd=0.45)              # TradFi: market-implied
pd_curve = WOEScorecard.predict(features).to_pd_curve()      # Credit: statistical
pd_curve = LiquidationBootstrap.fit(snapshot).pd_curve        # DeFi: on-chain
# All three feed:
cva = compute_cva(exposure_profile, pd_curve, ois_curve)     # same CVA engine
ecl = compute_ecl(ead, pd_curve, lgd, stage)                  # same ECL engine
```

### Graph-native

The graph is not a visualization layer — it is the primary data structure the system reasons from. Every assessment enriches a graph node. Every dependency is a weighted edge. DebtRank propagates distress. The agent reads graph neighbours during classification, not after.

- **GraphStore** — NetworkX DiGraph in crediqs-data. Node = entity. Edge = financial relationship.
- **DeXposure-FM** — 4,300 protocol dependency edges across 602 chains. Free, from HuggingFace.
- **GNN Contagion** — GAT attention mechanism. Directed weighted financial graphs. Contagion multiplier per entity.
- **DebtRank** — iterative distress propagation for systemic importance scoring.

### Memory-native

Memory is not conversation history replayed into a prompt. Memory is structured, searchable, and accumulative. The classify node reads entity memory, graph neighbours, similar cases, and org patterns **before** the LLM is called.

| Layer | What it stores | Where it lives |
|---|---|---|
| Turn memory | Last 3 conversation turns | AgentState |
| Session memory | Full session state, survives restarts | PostgreSQL checkpointer |
| Entity memory | Last rating, ECL, action, risk summary per entity | memory_entities table |
| Semantic search | 384-dim embeddings, cosine similarity | pgvector |
| Org patterns | Accumulated decision patterns per org | memory_patterns table |
| Sector patterns | Cross-org aggregation (privacy-preserving) | memory_interactions table |

### Zero-fallback principle

Every number shown on the platform must be traceable to a successful real-time market data pull, tool result, model output, database record, or user input. No hardcoded numeric fallbacks anywhere. If a registry key is missing or infrastructure is unreachable, modules raise `ValueError`, not silently use a wrong number.

### Explainability (9 adapters)

Every engine output has a registered explain adapter. Every number carries its attribution.

| Adapter | Engine | Class | Method |
|---|---|---|---|
| `pyccr_cva` | pyccr | Analytic | ∂CVA/∂param sensitivity |
| `pycredit_scorecard` | pycredit | Transparent | points per bin |
| `pycredit_ecl` | pycredit | Analytic | ∂ECL/∂PD, ∂ECL/∂LGD |
| `pychain_sc` | pychain | Transparent | 4-factor weighted |
| `pychain_stablecoin` | pychain | Transparent | peg + reserve |
| `pychain_governance` | pychain | Transparent | RSR + HHI + SMA |
| `pychain_liquidation` | pychain | Analytic | ∂PD/∂λ |
| `pychain_compliance` | pychain | Transparent | rule pass/fail |
| `crediqs_core_rating` | crediqs-core | Transparent | ensemble weights |

### Data-driven architecture (R3)

Zero hardcoded profiles, templates, or dashboard mappings. Everything from the database:

- **Node Catalogue** — 32+ nodes in `catalogue_nodes` table. Each declares `requires[]`, `produces[]`, `input_schema`, `output_schema`, governance fields.
- **DashboardLoader** — reads workflow nodes → matches `produces[]` to WIDGETS registry → lazy-loads the right dashboard. No `if (type === "tradfi")`.
- **Model Registry** — Settings → Models with governance lifecycle: draft → validated → production → deprecated. Test Contract validation before promotion.
- **Agent Discovery** — classify.py fetches effective-nodes from API. collect.py carries explain_adapter from catalogue. No hardcoded routing in agent-service.

## Regulatory coverage

| Regulation | What crediqs covers | Engine |
|---|---|---|
| Basel IV CRR3 | CVA, SA-CCR, IRB capital, FRTB SBA | pyccr |
| IFRS 9 | ECL staging (1/2/3), SICR detection, 3-scenario weighting | pycredit |
| MiCA | EMT/ART classification, CASP readiness, stablecoin monitoring | pychain |
| EU AI Act Art. 13 | Explainability for high-risk AI (credit scoring) | crediqs-explain |
| SR 11-7 | Model documentation, validation, backtest, drift monitoring | crediqs-lab |
| EBA Pillar 3 ESG | Physical + transition risk overlay | pyccr.climate |
| FRTB MAR 21.8 | Vega + curvature risk charges | pyccr.vol |
| DORA | ICT third-party risk (planned) | — |
| EMIR REFIT | Trade reporting (planned) | — |

## Research foundations

Every model cites its academic source. Every regulatory computation cites the specific article.

| Domain | Key references |
|---|---|
| CVA/xVA | Hull-White (1994), Brigo-Morini-Pallavicini (2013), Basel CRE 32.42 |
| Credit risk | Vasicek (1987, 2002), Merton (1974), IFRS 9 §B5.5.17 |
| DeFi risk | DeXposure-FM (arXiv 2602.03981), Beanstalk post-mortem, MiCA Art. 36-96 |
| Volatility | Black-Scholes (1973), Hagan et al. (2002), Heston (1993), Horvath et al. (2021) |
| Climate | BIS WP 1274, NGFS v5, Sopgoui (OSFI SCSE), Le Guenedal/Tankov |
| Contagion | Battiston et al. DebtRank (2012), GAT (Veličković 2018), TAGN (2026) |
| Fairness | EU AI Act Annex III §5(b), ECOA Reg B §1002.9, UK Equality Act |
| Explainability | Lundberg TreeSHAP (2020), Capriotti-Giles AAD (2010), Malliavin calculus |

## Technology stack

| Layer | Technology |
|---|---|
| Risk engines | Python 3.12, NumPy, SciPy, XGBoost, PyTorch (neural calibration) |
| Graph | NetworkX, PyTorch Geometric (GNN), DeXposure-FM |
| API | FastAPI, SQLAlchemy async, Alembic |
| Agent | LangGraph, sentence-transformers, pgvector |
| Database | PostgreSQL 16 + pgvector extension |
| Frontend | Next.js 15, React 19, TypeScript, Tailwind, ReactFlow, Zustand |
| Infrastructure | Docker, Redis, Railway (API), Render (backend), Vercel (frontend) |

<div align="center">

[Website](https://crediqs.ai) · [Contact](mailto:info@crediqs.ai)

---

*Graph-native risk intelligence. Every assessment makes the next one smarter.*

</div>
