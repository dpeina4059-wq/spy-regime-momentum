# SPY Vol Regime + Momentum Strategy

A systematic long-only trading strategy on SPY combining a rolling volatility 
regime classifier, EMA momentum filter, and mean reversion signal. Built in R 
and validated with a train/test split and rolling walk-forward test across 15 
years of market data (2010–2024).

## Motivation

Developed as part of a personal quant finance research project to apply mathematical 
and statistical foundations from an actuarial science background to live market 
strategy development. The vol regime classifier architecture was directly inspired 
by dynamic threshold monitoring work completed during an actuarial internship at 
Chubb Insurance, where sliding time-window percentile thresholds were used for 
seasonal claims analysis.

## Strategy Logic

The strategy deploys different logic depending on the current vol regime:

| Regime | Strategy | Entry Signal | Exit Signal |
|--------|----------|--------------|-------------|
| Low vol | Momentum | EMA(10) > EMA(30) | EMA(10) < EMA(30) |
| Neutral | Mean Reversion | Z-score < -2 (price unusually low) | Z-score > 0 (reversion complete) |
| High vol | Flat | No trades | Cash — put spreads in live trading |

**Entry**: Strict conditions per regime — only enter when signal confirms  
**Exit**: Regime-specific exit logic — each leg has its own exit condition

## Results

| Period | Total PnL | Win Rate | Avg PnL/Trade | Sharpe | Max Drawdown | Trades |
|--------|-----------|----------|----------------|--------|--------------|--------|
| Train (2010–2020) | $52.15 | 56.1% | $1.27 | 0.18 | -$30.11 | 41 |
| Test (2021–2024) | $204.45 | 81.2% | $12.78 | 0.62 | -$19.58 | 16 |

Out-of-sample performance significantly improved relative to training across every 
metric, no evidence of overfitting given parameters were not tuned to training data.

## Walk-Forward Validation

Rolling walk-forward test across 12 independent 1-year out-of-sample windows 
(2013–2024) using 3-year training windows:

- **Profitable years (7/12)**: 2013, 2017, 2019, 2021, 2022, 2023, 2024
- **Losing years (5/12)**: 2014, 2015, 2016, 2018, 2020
- **Strongest year**: 2024 — 100% win rate, Sharpe 1.30, +$57.37
- **Worst year**: 2018 — 0% win rate, Sharpe -1.62, -$14.71

Significant improvement over the single-leg momentum strategy (5/12 profitable) 
achieved by adding a mean reversion leg for neutral vol regimes. Previously losing 
years like 2019 and 2022 turned profitable under the two-leg framework.

Remaining losing years (2014, 2015, 2016, 2018, 2020) share a common characteristic: 
prolonged choppy markets where neither momentum nor mean reversion signals fire 
cleanly. Further iteration planned.

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

Then knit `R/SPY-Vol-Regime-Momentum-Strategy.Rmd` to reproduce all results 
and visualizations.

## Limitations & Next Steps

- **Remaining whipsaw weakness**: 5/12 losing years still driven by choppy 
  markets where neither leg fires cleanly; regime-dependent position sizing planned
- **Position sizing**: fixed 1-share sizing understates real returns; 
  Kelly Criterion sizing to be implemented  
- **High vol leg**: currently flat; live trading would deploy bear put spreads 
  during high vol regimes; options backtester planned as future infrastructure
- **Single asset**: generalization to sector ETFs and pairs trading planned

## Tech Stack

R | tidyverse | quantmod | tidyquant | TTR | ggplot2