# SAAM — Portfolio Allocation with a Carbon Objective

**Sustainability Aware Asset Management** — HEC Lausanne, 2026  
Pacific Region (PAC) | Scope 1 CO₂ Emissions

---

## Overview

This project implements climate-aware portfolio construction strategies for the PAC region, combining minimum-variance optimization with carbon emission constraints.

**Part I — Standard Portfolio Allocation**
- Minimum-variance portfolio $P_{oos}^{(mv)}$ optimized out-of-sample (Dec 2013 → Dec 2024)
- Value-weighted benchmark $P^{(vw)}$
- Annual rebalancing, monthly performance (Jan 2014 – Dec 2025)

**Part II — Carbon-Constrained Portfolios**
- $P_{oos}^{(mv)}(0.5)$ : min-variance with CF ≤ 50% of unconstrained CF
- $P_{oos}^{(vw)}(0.5)$ : tracking-error minimization with CF ≤ 50% of VW CF
- $P_{oos}^{(vw)}(NZ)$ : net-zero trajectory, CF reduced by 10% per year from 2013 base

---

## Key Results

| Portfolio | Ann. Return | Ann. Volatility | Sharpe | Cum. Return |
|-----------|-------------|-----------------|--------|-------------|
| $P_{oos}^{(mv)}$ | 9.81% | 10.70% | 0.917 | 187.4% |
| $P^{(vw)}$ | 7.74% | 13.22% | 0.586 | 120.7% |
| $P_{oos}^{(mv)}(0.5)$ | 9.91% | 10.88% | 0.911 | 190.0% |
| $P_{oos}^{(vw)}(0.5)$ | 7.73% | 13.23% | 0.585 | 120.3% |
| $P_{oos}^{(vw)}(NZ)$ | 7.72% | 13.24% | 0.583 | 120.0% |

---

## Repository Structure

```
SAAM/
├── saam_portfolio.ipynb        # Main notebook (all results reproducible top-to-bottom)
├── SAAM_PartI_Results.xlsx     # Preliminary submission (April 12)
├── audit_notebook.md           # Code audit vs. project specification
├── fig1_cumulative_returns_part1.png
├── fig2_waci_evolution.png
├── fig3_cf_evolution.png
├── fig4_cumret_mv_vs_mv50.png
├── fig5_cumret_vw_vs_vw50.png
├── fig6_final_comparison.png
├── fig7_weight_evolution.png
└── fig8_top10_carbon_intensity.png
```

> **Data file** (`Sustainability.xlsx`) is not included in this repository as it is proprietary course data provided by the instructor.

---

## How to Run

1. Place `Sustainability.xlsx` in the same folder as `saam_portfolio.ipynb`
2. Install dependencies:
   ```bash
   pip install pandas numpy cvxpy matplotlib scikit-learn openpyxl
   ```
3. Run all cells top-to-bottom — no manual intervention required

---

## Methodology Highlights

- **Covariance estimator**: Ledoit-Wolf shrinkage (necessary from Dec 2017 when N > τ = 120)
- **Return observations threshold**: ≥ 60 months (more conservative than the 36-month minimum)
- **Stale price filter**: exclude firms with > 50% zero monthly returns over the 10-year window
- **Delisting**: −100% return injected at delisting month (parsed from firm name)
- **Missing annual data**: forward-filled; beginning-of-sample NaN → firm excluded until data available
- **Risk-free rate**: Rf = 0 (not specified in the brief)
