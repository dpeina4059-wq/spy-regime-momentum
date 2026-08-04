# SPY Vol Regime + Momentum Strategy

A systematic long-only trading strategy on SPY combining two uncorrelated signals: 
a rolling volatility regime classifier and an EMA momentum filter. Built in R and 
validated with a train/test split across 15 years of market data (2010–2024).

## Motivation

Developed as part of a personal quant finance research project to apply mathematical 
and statistical foundations from an actuarial science background to live market 
strategy development. The vol regime classifier architecture was directly inspired by 
dynamic threshold monitoring work completed during an actuarial internship at Chubb 
Insurance, where sliding time-window percentile thresholds were used for seasonal 
claims analysis.

## Strategy Logic

| Signal | Condition | Role |
|--------|-----------|------|
| Vol Regime | Rolling 10-day annualized vol < 25th percentile (252-day window) | Entry filter |
| Momentum | EMA(10) > EMA(30) | Entry filter |
| Vol Regime | Rolling vol > 75th percentile | Exit trigger |
| Momentum | EMA(10) < EMA(30) | Exit trigger |

**Entry**: Both signals must confirm (AND logic) — strict entry  
**Exit**: Either signal is sufficient (OR logic) — loose exit

## Results

| Period | Total PnL | Win Rate | Avg PnL/Trade | Trades |
|--------|-----------|----------|----------------|--------|
| Train (2010–2020) | $25.88 | 42.9% | $0.74 | 35 |
| Test (2021–2024) | $135.29 | 63.6% | $12.30 | 11 |

Out-of-sample performance held up and improved relative to training, thus no evidence 
of overfitting given parameters were not tuned to the training data.

## Structure

```
spy-regime-momentum/
├── R/
│   └── SPY-Vol-Regime-Momentum-Strategy.Rmd
├── plots/
├── results/
├── renv.lock
└── README.md
```

## Setup

```r
renv::restore()
```

Then knit `R/SPY-Vol-Regime-Momentum-Strategy.Rmd` to reproduce all results and visualizations.

## Limitations & Next Steps

- Sample size thin: walk-forward testing across rolling windows planned
- Fixed 1-share position sizing: Kelly Criterion sizing to be implemented
- Long-only: short signals via momentum reversal to be explored
- SPY only: generalization to sector ETFs and pairs trading planned

## Tech Stack

R | tidyverse | quantmod | tidyquant | TTR | ggplot2