# quant-portfolio-risk

A quantitative portfolio construction and risk analysis system using two complementary approaches: PCA-based Eigenportfolio optimization with Random Matrix Theory noise filtering, and Monte Carlo simulation for forward-looking risk assessment.

> **Ticker is configurable.** The eigenportfolio was built on 20 large-cap US stocks and the Monte Carlo simulator was validated on HDFCBANK (NSE). Both work with any ticker or stock universe supported by `yfinance`.

---

## What this does

Traditional portfolio optimization (Markowitz) relies on a raw sample covariance matrix which is notoriously noisy — especially with many assets and limited history. This system takes a more rigorous approach: it uses PCA and Random Matrix Theory to separate genuine market signals from statistical noise before constructing portfolios. The Monte Carlo engine then stress-tests those portfolios across 10,000 simulated futures.

```
Historical price data (20 stocks)
         │
         ▼
┌─────────────────────┐
│  Returns & Corr     │  ← log returns, correlation matrix
│  Matrix             │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  PCA + RMT Filter   │  ← separates signal from noise
│  Eigenportfolios    │     using Marchenko-Pastur threshold
└──────────┬──────────┘
           │  cleaned weights
           ▼
┌─────────────────────┐
│  Factor Momentum    │  ← trades Market + Style factors
│  Strategy           │     with rolling momentum signals
└──────────┬──────────┘
           │  strategy returns
           ▼
┌─────────────────────┐
│  Monte Carlo        │  ← 10,000 simulated price paths
│  Risk Engine        │     VaR, loss probability, scenarios
└─────────────────────┘
```

---

## Project structure

```
quant-portfolio-risk/
│
├── notebooks/
│   ├── 01_Eigen_Portfolio.ipynb         # PCA + RMT portfolio construction
│   └── 02_MonteCarlo_Stock_Simulator.ipynb  # Forward risk simulation
│
├── research/
│   └── Portfolio_Optimization_Indian_Market.pdf
│   └── Monte_Carlo_Risk_Modeling.pdf
│
├── requirements.txt
└── README.md
```

---

## Notebooks

### 1. Eigenportfolio Optimizer (`01_Eigen_Portfolio.ipynb`)

Applies Principal Component Analysis to the return covariance matrix, then uses Random Matrix Theory (Marchenko-Pastur law) to identify which eigenvalues represent genuine market factors vs pure noise.

**The core idea — why raw covariance fails:**
A sample covariance matrix estimated from T days and N stocks has estimation error. When T/N is not very large, many eigenvalues are statistically indistinguishable from noise. Trading on these fake factors destroys performance.

**RMT noise threshold (Marchenko-Pastur):**
```
λ_max = (1 + √(N/T))²
```
Eigenvalues below `λ_max` are discarded. Only signal factors are kept.

**What the PCA finds:**
| Factor | Interpretation | Character |
|--------|---------------|-----------|
| PC1 | Market factor | Moves with broad market |
| PC2 | Style factor | Growth vs Value tilt |
| PC3+ | Sector/noise | Filtered out by RMT |

**Portfolio construction:**
- Cleaned correlation matrix rebuilt from signal eigenvalues only
- Inverse-variance weights applied on cleaned matrix
- Factor momentum strategy: 252-day rolling signal on PC1 + PC2

**Performance metrics computed:** Sharpe ratio, max drawdown, total return, cumulative equity curve.

---

### 2. Monte Carlo Risk Engine (`02_MonteCarlo_Stock_Simulator.ipynb`)

Simulates 10,000 independent price paths over a 252-day horizon using Geometric Brownian Motion calibrated to historical log returns.

**Simulation model:**
```
S(t) = S(t-1) × exp((μ - ½σ²) + σZ)
where Z ~ N(0,1)
```

**Risk metrics produced:**
| Metric | Description |
|--------|-------------|
| Expected price | Mean of 10,000 terminal prices |
| Best case (95th pct) | Upper tail outcome |
| Worst case (5th pct) | Lower tail outcome |
| Loss probability | P(final price < entry price) |

**Output:** Full fan chart of all 10,000 paths + terminal price distribution histogram.

---

## Quickstart

```bash
git clone https://github.com/yourusername/quant-portfolio-risk
cd quant-portfolio-risk
pip install -r requirements.txt
```

Open notebooks in order: `01` → `02`.  
All data downloads automatically via `yfinance`. No CSV files needed.

To change the stock universe in `01`, edit the `stocks` list at the top.  
To change the ticker in `02`, edit the `stock` variable at the top.

---

## Requirements

```
yfinance
numpy
pandas
matplotlib
scikit-learn
```

---

## Key concepts

| Concept | Used in |
|---------|---------|
| PCA (Principal Component Analysis) | Eigenportfolio construction |
| Random Matrix Theory | Noise filtering via Marchenko-Pastur |
| Eigenportfolios | Market + Style factor extraction |
| Factor momentum | Rolling signal on PC1, PC2 |
| Geometric Brownian Motion | Monte Carlo path simulation |
| Value at Risk (VaR) | 5th percentile terminal wealth |
| Loss probability | Fraction of paths below entry |

---

## Limitations & honest notes

- GBM assumes constant volatility — real markets have volatility clustering (GARCH effects).
- RMT filtering assumes returns are approximately Gaussian — fat tails are not captured.
- Factor momentum uses a fixed 252-day lookback — not optimised or walk-forward validated.
- No transaction costs modelled.
- This is a research and learning project, not a production system.

---

## What's next

- [ ] GARCH-driven volatility in Monte Carlo paths
- [ ] Walk-forward validation of factor momentum strategy
- [ ] Extend to Indian market universe (NSE stocks)
- [ ] Combine with `adaptive-regime-ai` — regime-conditional portfolio weights

---

## Author

**Piyush Patil** — AI & Data Science / Quantitative Finance  
Built as part of a personal quantitative research portfolio covering Indian and US equity markets.
