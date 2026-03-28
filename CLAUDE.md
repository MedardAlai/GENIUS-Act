# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Research Title:
Estimating the Causal Impact of the GENIUS Act on U.S. Stablecoin Market Dynamics Using
Bayesian Structural Time-Series Modeling.

Research question:
To what extent did the 2025 GENIUS Act alter the relative demand dynamics for U.S. dollar-
pegged stablecoins, after accounting for macroeconomic conditions?

## Environment Setup

```bash
pip install pandas numpy matplotlib seaborn scipy statsmodels scikit-learn causalimpact
```

The `causalimpact` package (`pycausalimpact`) is non-standard and required for all causal inference cells. If import fails, try `pip install pycausalimpact`.

## Running the Analysis

```bash
jupyter notebook Project3.ipynb
```

There is no build system, test suite, or CI/CD. All analysis lives in `Project3.ipynb`.

## Data Files

| File | Source | Contents |
|------|--------|----------|
| `rwa-token-timeseries-export-*.csv` | rwa.xyz | 89 stablecoin tokens with daily market cap; 2017–2026 |
| `Dollar Index.csv` | FRED (DTWEXBGS) | Trade-weighted USD index |
| `CoinGecko-GlobalCryptoMktCap-*.csv` | CoinGecko | Global crypto market cap & volume |
| `Global Lquidity IndexWPC.csv` | FRED (WPC) | Weekly global liquidity index |

## Notebook Architecture

The notebook is structured as a linear research pipeline:

1. **Data Loading & Cleaning** — Imports all four datasets, parses timestamps, ensures numeric types.
2. **Token Categorization** — Splits 89 tokens into two buckets:
   - **Regulated:** USDC, Ripple USD, PayPal USD, Binance USD, Pax Dollar, USD1, Gemini Dollar
   - **Non-Regulated:** all remaining tokens (computed as residual)
3. **Outcome Variable Engineering** — Constructs log-transformed variables. The primary outcome is `relative_log = log(regulated_mcap) − log(non_regulated_mcap)`.
4. **CausalImpact Models** — Bayesian Structural Time-Series models using `causalimpact`:
   - **Policy intervention date:** May 1, 2025 (GENIUS Act)
   - **Pre-period:** 2023-01-01 – 2025-04-30
   - **Post-period:** 2025-05-01 – 2026-02-08
   - Multiple model specs: basic, +DXY, +crypto market cap, +GLI (full controls)
5. **Robustness / Placebo Tests** — `run_causalimpact_placebo()` re-runs models with fake intervention dates.

## Key Helper Functions

- `snap_to_index(date, index)` — Snaps an arbitrary date to the nearest available date in a time-series index.
- `run_causalimpact_placebo(pre_start, fake_intervention, post_end, data, covariates)` — Runs a CausalImpact model with a synthetic intervention date for placebo testing.

## Econometric Notes

- **Multicollinearity warning:** Non-regulated mcap vs. crypto mcap correlation ≈ 0.95; non-regulated mcap vs. DXY ≈ 0.80. This affects coefficient stability but not predictive power of the state-space model.
- **Anticipatory adjustment:** Evidence suggests market reallocation began before formal GENIUS Act enactment, weakening a discrete structural-break interpretation.
- **Pre-period sensitivity:** Models using only 2024+ pre-period data show the effect disappears (posterior ~62%), vs. ~93% with 2023+ data.
