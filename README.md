# SPY Vol Regime + Momentum Strategy

A systematic long-only trading strategy on SPY combining a rolling volatility 
regime classifier, EMA momentum filter, and mean reversion signal. Built in R 
and validated with a train/test split and rolling walk-forward test across 15 
years of market data (2010–2024). Position sizing via Kelly Criterion.

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
**Position Sizing**: Quarter Kelly Criterion calibrated on rolling training window

## Results

| Period | Total PnL | Win Rate | Avg PnL/Trade | Sharpe | Max Drawdown | Kelly F | Trades |
|--------|-----------|----------|----------------|--------|--------------|---------|--------|
| Train (2010–2020) | $498.85 | 56.1% | $12.17 | 0.26 | -$208.21 | 0.286 | 41 |
| Test (2021–2024) | $204.45 | 81.2% | $12.78 | 0.62 | -$19.57 | 0.719 | 16 |

Simulated on $5k account with quarter Kelly position sizing. Out-of-sample 
win rate (81.2%) significantly exceeded training (56.1%) with no parameter 
tuning — no evidence of overfitting.

## Walk-Forward Validation

Rolling walk-forward test across 12 independent 1-year out-of-sample windows 
(2013–2024) using 3-year training windows with Kelly fraction passed forward 
from each training window to size the corresponding test window:

- **Profitable years (7/12)**: 2013, 2017, 2019, 2021, 2022, 2023, 2024
- **Losing years (5/12)**: 2014, 2015, 2016, 2018, 2020
- **Strongest year**: 2013 — $54.48 with Kelly sizing, Sharpe 0.44
- **Worst year**: 2014 — -$34.09, driven by choppy false momentum signals

Kelly sizing amplifies both wins and losses — 2013 improved from $8.74 to 
$54.48 while 2014 worsened from -$8.53 to -$34.09. This is expected behavior 
and reflects the strategy's genuine edge profile rather than a flaw.

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

- **Remaining whipsaw weakness** — 5/12 losing years concentrated in choppy 
  markets (2014-2016, 2018, 2020) where neither leg fires cleanly; 
  regime-dependent position sizing and signal filters planned
- **High vol leg** — currently flat; live trading would deploy bear put spreads; 
  options backtester planned as future infrastructure  
- **Capital constraint** — Kelly expression limited at small account sizes; 
  full benefits require $5k+ account or cheaper underlying assets
- **Single asset** — generalization to sector ETFs and pairs trading planned

## Tech Stack

R | tidyverse | quantmod | tidyquant | TTR | ggplot2