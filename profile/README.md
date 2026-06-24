<div align="center">

<img src="https://raw.githubusercontent.com/crediqs/.github/main/assets/crediqs-banner.png" width="100%" />

**Financial risk intelligence platform. Graph-native. Memory-native. Regulator-ready.**

## What crediqs is

crediqs is the only risk platform where TradFi derivatives and DeFi positions share the same probability-of-default curve, the same CVA engine, and the same regulator-citable explanation.
---

very number is traceable. Every decision is explainable. Every assessment makes the next one smarter.

## Platform architecture

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

<div align="center">

[Website](https://crediqs.ai) · [Contact](mailto:info@crediqs.ai)

</div>
