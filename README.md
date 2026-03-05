# Bond Index Replication & Rebalancing Engine

A fixed income portfolio construction system that replicates an investment grade bond index using a constrained optimisation algorithm, automated monthly rebalancing, and a full analytics suite. Built to mirror the kind of portfolio management automation used by index fixed income teams at asset managers like BlackRock.

---

## Links

| | |
|--|--|
| Live Dashboard | https://sara-iqbal.github.io/Bond-Index-Replication-Rebalancing-Engine/ |
| Google Colab Notebook | https://drive.google.com/file/d/11QFcRNmMv9k_OAoPXtLklHGbazWAvzQ2/view?usp=sharing |
| LinkedIn | https://www.linkedin.com/in/saraiqbaldata0602/ |

---

## What this project does

Index fixed income funds have to track a benchmark — usually something like the iBoxx USD Investment Grade index — while staying within strict risk limits on duration, sector exposure, position size, and trading turnover. You cannot simply hold every bond in the index (there are 2,800+ in LQD alone), so you need an algorithm that selects a smaller, representative portfolio and keeps it aligned to the benchmark as markets move each month.

This engine does exactly that. It takes the real published sector, rating, and maturity weights from the iShares LQD ETF, builds a universe of 500 synthetic investment grade bonds with realistic parameters calibrated to 2024 market conditions, then runs a SciPy SLSQP constrained optimiser to select 150 bonds and assign weights that minimise tracking error against the benchmark. Every month a rebalancing event fires, the market drifts, and the optimiser reruns — outputting a complete buy, sell, and hold order list with dollar amounts, just as a portfolio management system would for a trading desk.

The whole pipeline runs end to end in a single Google Colab notebook and the results feed into an interactive dashboard deployed on GitHub Pages.

---

## Pipeline

```
iShares LQD Benchmark Weights (real published data)
        |
500 Synthetic Bonds — calibrated to 2024 IG market
        |
Eligibility Filter — liquidity score >= 0.30, face value >= $100M
        |
Stratified Sample — 150 bonds selected proportionally by sector
        |
SciPy SLSQP Optimiser — minimise tracking error, 7 hard constraints
        |
Monthly Rebalancing Event x12 — market drift, re-optimise, order list
        |
Analytics — Tracking Error, DV01, Duration, Sharpe, IR, VaR, Drawdown
        |
Interactive Dashboard — GitHub Pages
```

---

## Optimiser constraints

The optimiser enforces seven hard constraints on every run. These are not soft penalties — any solution that violates them is rejected.

| Constraint | Limit | Reason |
|------------|-------|--------|
| Fully invested | weights sum to 1.0 | No cash drag |
| Long only | weight >= 0 | Long-only mandate |
| Max single position | 2.0% | Concentration limit |
| Duration mismatch | within +/- 0.5 years of benchmark | Interest rate risk control |
| Sector deviation | within +/- 5% of benchmark per sector | Sector concentration limit |
| Monthly turnover | <= 15% | Transaction cost management |
| Minimum liquidity score | >= 0.30 | Ensures bonds are tradable |

---

## Results

| Metric | Value |
|--------|-------|
| Portfolio size | 150 bonds replicating 2,800-bond benchmark |
| Average tracking error | 8.4 bps per annum |
| Modified duration | 8.39y (benchmark 8.42y, mismatch -0.03y) |
| Portfolio YTM | 5.28% (benchmark 5.31%) |
| Portfolio DV01 | $84,200 per $100M NAV |
| Sharpe ratio | 1.24 |
| Information ratio | 0.87 |
| Annualised return | 5.72% |
| Annualised volatility | 2.14% |
| VaR (95%, 1 month) | -0.48% |
| Maximum drawdown | -1.23% |
| Average monthly turnover | 8.2% (limit 15%) |
| Constraints breached | 0 of 7 across all 12 months |

---

## Technical stack

| Component | Technology |
|-----------|-----------|
| Optimisation | SciPy SLSQP (Sequential Least Squares Programming) |
| Bond universe | NumPy — distributions calibrated to 2024 IG market data |
| Data manipulation | Pandas |
| Visualisations | Plotly |
| Dashboard | HTML, CSS, JavaScript, Chart.js |
| Deployment | GitHub Pages |
| Notebook | Google Colab |

---

## Bond universe — how the synthetic data is generated

The 500 bonds are not random. Each parameter is drawn from a distribution calibrated to match real 2024 investment grade corporate bond market characteristics.

Ratings follow the iBoxx LQD distribution: 5% AAA, 12% AA, 38% A, 45% BBB. Sector weights match the same benchmark. OAS spreads are drawn from rating-specific normal distributions — AAA bonds average around 20 basis points, BBB bonds around 145 basis points, matching levels observed in the 2024 US IG market. Maturities are weighted to replicate the shape of the LQD maturity profile, with the most bonds in the 3–5 year and 7–10 year buckets. Coupon rates are derived from the risk free rate plus spread plus noise, clipped between 1.5% and 9.5%. Liquidity scores are rating-dependent — AAA bonds average 0.90, BBB bonds average 0.50 — reflecting real differences in bid-ask spreads and trading volume across the credit spectrum.

---

## Repository structure

```
bond-index-rebalancer/
|
|-- index.html                     Live dashboard (GitHub Pages entry point)
|-- bond_index_rebalancer.ipynb    Full notebook — run this in Google Colab
|
|-- outputs/                       Generated when you run the notebook
|   |-- dashboard_data.json        Data feed for the dashboard
|   |-- portfolio_holdings.csv     Current 150-bond positions with weights
|   |-- monthly_rebalancing.csv    12-month analytics history
|   |-- order_lists.csv            All buy and sell orders across all months
|   |-- bond_universe.csv          Full 500-bond synthetic universe
|
|-- README.md
```

---

## How to run

**Google Colab — recommended, no setup required**

1. Open the Colab link above
2. Go to Runtime and select Run all
3. The notebook takes around 2 minutes on a T4 instance
4. All output files are exported at the end of Step 11

**Local**

```bash
git clone https://github.com/sara-iqbal/bond-index-rebalancer
cd bond-index-rebalancer
pip install scipy numpy pandas plotly openpyxl
jupyter notebook bond_index_rebalancer.ipynb
```

---

## Dashboard

The dashboard has five tabs. Overview shows the full pipeline, headline metrics, NAV versus benchmark over 12 months, and sector and rating allocation charts. Optimiser shows the status of each constraint, the weight distribution across the top 30 positions, and a duration-spread scatter of the holdings. Rebalancing lets you click any month from January to December and see the order list animate in with each bond's action, size, and new weight. Risk and Analytics shows the Sharpe ratio, information ratio, VaR, drawdown, and a full fixed income KPI comparison table against the benchmark. Holdings shows the top 15 positions with duration, YTM, OAS, and market value.

---

## Benchmark source

Sector, rating, and maturity weights are taken from the iShares iBoxx USD Liquid Investment Grade Corporate Bond ETF (ticker: LQD), which BlackRock publishes as public holdings data. The benchmark-level analytics — modified duration of 8.42 years, YTM of 5.31%, approximately 2,800 constituents — are sourced from the same published dataset as of 2024.

---

## Author

Sara Iqbal  
MSc Data Science · B.Tech Artificial Intelligence and Machine Learning  



---
