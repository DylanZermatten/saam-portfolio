# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Academic project for **Sustainability Aware Asset Management (SAAM)**, implementing climate-aware portfolio optimization in Python. The deliverable is a fully reproducible Jupyter notebook (`.ipynb`) with no hard-coded paths, a PDF report (≤30 pages), a 1-page sales pitch, and a 10-minute video.

**Key dates:**
- Preliminary submission (summary stats + monthly returns in template Excel): April 12, 2026
- Final submission: May 29, 2026

##  Data

`Sustainability.xlsx` contains multiple sheets:
- **Static**: 2545 firms with ISIN, names, regions
- **Time series (monthly/annual)**: CO2 Scope 1 & 2 emissions, revenues, market cap, total return indices
- Market data: 1999–2025; carbon data: 2002–2024 (reliable from ~2010)
- Analysis begins end-of-2013

## Development

No build system. Use Python with a Jupyter notebook as the primary deliverable.

Recommended libraries: `pandas`, `numpy`, `scipy.optimize` (or `cvxpy`), `matplotlib`/`seaborn`, `openpyxl`.

Run notebook:
```bash
jupyter notebook
```

## Architecture

The implementation follows two sequential phases:

### Part I — Standard Minimum-Variance Portfolio (due April 12)

1. **Data cleaning**: drop firms with missing prices, prices < $0.50, or > 50% stale returns (consecutive zeros)
2. **Investment set**: filter by region, require sufficient return observations, require carbon data availability
3. **Return/covariance estimation**: 10-year rolling window of monthly returns (out-of-sample)
4. **Optimization**: long-only minimum-variance (quadratic program, non-negative weights, weights sum to 1)
5. **Rebalancing**: annual (Dec 2013 → Dec 2024), performance evaluated monthly
6. **Benchmark**: value-weighted portfolio for comparison

### Part II — Carbon-Aware Portfolios

Carbon intensity metric: **tonnes CO₂ per million USD revenue** (Scope 1 + Scope 2).

Three strategies:
1. **Min-variance + 50% carbon footprint (CF) reduction** vs. benchmark
2. **Tracking error minimization + 50% CF reduction** (passive/index-hugging approach)
3. **Net-zero portfolio**: 10% annual CF reduction compounded from base year

Key metrics: Weighted-Average Carbon Intensity (WACI) and Carbon Footprint (CF) of portfolio.

## Constraints

- **Long-only** portfolios (non-negative weights) — required for carbon footprint interpretation
- **Out-of-sample** construction only (no look-ahead bias)
- Notebook must run end-to-end without hard-coded absolute paths
- Report must include an LLM disclosure section
