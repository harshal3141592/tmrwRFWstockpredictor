# tmrwRFWstockpredictor
This is basic Random Forest Algorithm project. Here, I get the entire OHLCV data for Nifty50 and try to predict what would be the share point for the next day

## Overview

Stock price movement is notoriously hard to predict, but this project explores how far a simple, interpretable model can get using only price/volume history and engineered technical features — with an emphasis on realistic backtesting rather than a single train/test split.

## Approach
**Data collection**: Pulled _~18 years_ of historical Nifty 50 OHLCV data using the _yfinance_ API.

**Target definition**: A binary label was created to differentiate between the high or low closing position. 1 if tomorrow's close is higher than todays, or 0 if that's not the case.

**Baseline model**: We have trained a RandomForestClassifier on raw OHLCV features (Open, High, Low, Close, Volume).

**Backtesting framework**: Built an expanding-window backtest (train on all data up to day i, test on the next 250 days, repeat) to simulate realistic, sequential trading conditions and avoid lookahead bias.

**Feature engineering**: A rolling-window features across five horizons (2, 5, 60, 250, and 1000 days) was introduced. This helps in capturing the different speed in market behaviour. The capturing from short day span to years gives more context.

**Close ratio**: current close relative to the rolling average close

**Trend**: The sum of positive-movement days over the window

**Threshold tuning**: Used predicted probabilities (rather than raw class predictions) with a 0.6 threshold to favor precision over recall, better suited to a risk-conscious trading use case.

## Results
| Model version | Precision | Base rate (naive "always up") |
|---|---|---|
| Baseline (raw OHLCV features) | ~0.53 | ~0.54 |
| With multi-horizon rolling features + tuned threshold | ~0.55 | ~0.54 |

Precision alone doesn't mean much without a baseline to compare against: Nifty50 closes higher than the previous day roughly 54% of the time historically, so a model that did nothing but always guess "up" would already score ~0.54 precision. The baseline OHLCV model in this project actually performs *slightly below* that naive rate — indicating it isn't picking up any real directional signal from raw price/volume levels alone. The multi-horizon feature model narrowly edges past the baseline, suggesting a small but real signal from relative momentum.

Precision was prioritized over accuracy: in a trading context, a false "buy" signal is more costly than a missed opportunity, so the model is tuned to be more confident before predicting an upward move.

> **Note:** Because this notebook pulls live Nifty50 data via `yfinance`, exact figures shift slightly on each run as the training/testing window moves forward in time. The pattern above the baseline near or below the naive rate, engineered features narrowly beating it has held consistently across recent runs.
> 
## Limitations
- No transaction costs, slippage, or brokerage fees are modeled a real strategy would need each "buy" to clear brokerage and impact costs, not just beat the baseline.
- No position sizing or risk management: the model outputs a direction, not how much capital to risk.
- Single-index scope: trained only on Nifty50; performance may not generalize to individual stocks or other indices.
- No macro or news features: The model only sees price/volume history, ignoring earnings, budget announcements, or global market moves.
- The 0.6 probability threshold was chosen manually, not tuned via cross-validation.

## Tech Stack
1. Python
2. pandas
3. scikit-learn (RandomForestClassifier)
4. yfinance

### Future Improvements
1. I might add a comparison based against gradient-boosted models (XGBoost, LightGBM)
2. Incorporate additional features (macro indicators, sentiment data from twitter/Reddit, sector indices)
3. Walk-forward hyperparameter tuning instead of fixed model parameters

### Disclaimer

This project is for educational purposes only and does not constitute financial advice. Past performance and backtested results are not indicative of future returns.
