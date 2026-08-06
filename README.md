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

## Results

| Period | Total PnL | Win Rate | Avg PnL/Trade | Sharpe | Max Drawdown | Trades |
|--------|-----------|----------|----------------|--------|--------------|--------|
| Train (2010–2020) | $25.88 | 42.9% | $0.74 | 0.09 | -$25.33 | 35 |
| Test (2021–2024) | $135.29 | 63.6% | $12.30 | 0.46 | -$19.58 | 11 |

Out-of-sample performance held up and improved relative to training: no evidence 
of overfitting given parameters were not tuned to the training data. Low Sharpe 
ratios reflect fixed 1-share position sizing rather than percentage returns,  
proper capital-weighted sizing is a planned next iteration.

## Walk-Forward Validation

Rolling walk-forward test across 12 independent 1-year out-of-sample windows (2013–2024) 
using 3-year training windows revealed the strategy's regime dependency:

- **Profitable years (5/12)**: 2013, 2017, 2021, 2023, 2024
- **Losing years (7/12)**: 2014, 2015, 2016, 2018, 2019, 2020, 2022
- **Strongest year**: 2021 — 100% win rate, Sharpe 0.92, +$73.01
- **Worst year**: 2018 — 0% win rate, Sharpe -1.62, -$14.71

The strategy performs well in trending low-vol bull markets and struggles in choppy 
or declining regimes where signals whipsaw. This motivates the next iteration: 
adding an ADX trend strength filter to confirm momentum magnitude before entry.

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