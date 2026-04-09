<div align="center">

<img src="https://raw.githubusercontent.com/crediqs/.github/main/assets/crediqs-banner.png" width="100%" />

**Institutional credit risk infrastructure · Production ready**

[![pyccr](https://img.shields.io/badge/pyccr-v0.8.0-845ef7)](https://github.com/crediqs/pyccr)
[![pycredit](https://img.shields.io/badge/pycredit-v0.3.0-845ef7)](https://github.com/crediqs/pycredit)
[![pychain](https://img.shields.io/badge/pychain-v0.3.0-845ef7)](https://github.com/crediqs/pychain)
[![crediqs-core](https://img.shields.io/badge/crediqs--core-v0.2.0-845ef7)](https://github.com/crediqs/crediqs-core)
[![tests](https://img.shields.io/badge/tests-1%2C437%2B%20passing-22c55e)](https://github.com/crediqs)
[![license](https://img.shields.io/badge/license-Apache%202.0-06b6d4)](https://github.com/crediqs/pyccr/blob/main/LICENSE)
[![api](https://img.shields.io/badge/API-live-22c55e)](https://crediqs-api-demo.up.railway.app/docs)

</div>

---

Institutional risk intelligence platform covering counterparty credit risk (SA-CCR, xVA, exposure simulation), IFRS 9 expected credit loss, on-chain DeFi risk, and real-world asset (RWA) risk. Every model is calibrated to academic research and regulatory standards, fully documented in the [crediqs-catalogue](https://github.com/crediqs/crediqs-catalogue), and available via a [public REST API](https://crediqs-api-demo.up.railway.app/docs).

---

## Quick Start

```bash
pip install pyccr        # Counterparty credit risk · SA-CCR · xVA
pip install pycredit     # IFRS 9 ECL · PD curves · Scorecards
pip install pychain      # DeFi · Stablecoin · RWA · Smart contract
```

```python
from pyccr import CVA
cva = CVA(pd_1y=0.007, lgd=0.60, notional=10_000_000, maturity=5.0).compute()

from pycredit import compute_ecl
ecl = compute_ecl(ead=500_000, maturity=3.0, pd_1y=0.02, lgd=0.45)

from pychain.risk.smart_contract import assess_known_protocol
sc = assess_known_protocol("aave-v3")
```

Or use the API directly — no installation required:
```bash
curl https://crediqs-api-demo.up.railway.app/health
```
---

## What Each Library Does

**pyccr** — Full counterparty credit risk stack: Hull-White 1F + GBM Monte Carlo simulation, exposure profiles (EPE/PFE/EEPE), CVA/DVA/FVA/MVA/KVA, SA-CCR EAD for all 5 asset classes, ISDA SIMM v2.6, and IRB/CEM. Standards: Basel III CRR3 Art. 274-280, BCBS d279, IFRS 13.

**pycredit** — IFRS 9 expected credit loss: PD term structures, SICR stage classification, LGD models, macro overlays for PIT adjustment, WOE behavioural scorecards, and portfolio-level ECL with scenario weighting. Standard: IFRS 9 §5.5.

**pychain** — On-chain credit risk covering the full PDACS taxonomy: DeFi protocol liquidation risk (Aave V3, Compound V3), stablecoin peg stability, MiCA compliance monitoring, RWA tokenised debt, smart contract exploit risk (4-factor model), governance token risk (RSR/HHI/SMA), and carbon credit risk (reversal/vintage/registry).

**crediqs-core** — The unification layer: a single `CreditRating` object from any source (CDS spread, DeFi liquidation, scorecard, RWA) with S&P TTC transition matrices and migration-based PD curves that capture the credit cliff effect.

---

## REST API

| Group | Endpoints |
|---|---|
| Credit risk | `/v1/ecl` · `/v1/cva` · `/v1/xva` · `/v1/rating` · `/v1/rating/ensemble` |
| On-chain | `/v1/stablecoin` · `/v1/protocol` · `/v1/rwa` · `/v1/smart-contract` · `/v1/governance-token` · `/v1/debt-token` · `/v1/carbon-token` |
| Compliance | `/v1/compliance/check` · `/v1/compliance/alerts` |
| Portfolio | `/v1/portfolio` · `/v1/scenario` |

---

## Compliance Intelligence

Automated regulatory checks with exact citations — built into crediqs-api v0.3.

Regulations covered: **MiCA** (Art. 43-58) · **CRR3** (Art. 274-280) · **IFRS 9** (§5.5)

---

## Citation

```bibtex
@software{crediqs2025,
  title   = {crediqs: Institutional credit risk infrastructure},
  author  = {crediqs},
  year    = {2025},
  url     = {https://github.com/crediqs},
}
```

---

<div align="center">

[Website](https://crediqs.ai) · [API Docs]· [Model Catalogue] · [Contact](mailto:info@crediqs.ai)

</div>
