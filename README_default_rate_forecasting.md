# LTM Default Rate Forecasting — Institutional Loan Market

Individual assignment (70% of module grade) for Financial Econometrics, MSc Finance, Dublin City University.

## Objective

Forecast the last-12-month (LTM) default rate in the institutional loan market one year ahead, using macro and credit-market data as of September 2020 to predict the rate at September 2021.

## Data

206 monthly observations, August 2003 – September 2020, anonymised institutional loan market data. Columns were interpreted from their behaviour around the 2008 crisis and COVID-19 (e.g. a credit spread proxy, unemployment proxy, VIX, distressed loan ratio, downgrade/migration rate, bank lending standards).

## Feature engineering

Six features selected on economic logic, empirical correlation, and lead/lag structure (lagging inputs so they're observable before the default event, avoiding look-ahead bias):

| Feature | Lag | Rationale |
|---|---|---|
| Credit spread | 0 | Best real-time default signal |
| Distressed loan ratio | 6 months | Distressed loans typically default 6–12 months later |
| Bank lending tightening | 6 months | Tighter standards restrict refinancing |
| Unemployment | 6 months | Rising unemployment impairs debt serviceability |
| Downgrade/migration rate | 3 months | Downgrades closely precede defaults |
| VIX | 0 | Market stress is coincident with default spikes |

The target (default rate) is log-transformed and back-transformed via `exp()` at prediction time, since it's I(1) — non-stationary in levels, stationary after first-differencing.

## Methodology

1. **Stationarity testing** — ADF tests on levels and first differences to confirm d=1 for the target
2. **Order selection** — ACF/PACF on the differenced series to identify AR/MA structure
3. **Chronological train/test split** — 24-month holdout (post-September 2018), never randomised
4. **Three models compared**: OLS baseline, ARIMAX(1,1,1), and SARIMAX(1,1,1)(1,0,1,12)
5. **Model selection by out-of-sample RMSE** — chosen over AIC/BIC since in-sample fit criteria don't penalise overfitting the way held-out accuracy does
6. **Final refit** on the full dataset before generating the live forecast

## Results

| Model | AIC | BIC | OOS RMSE | OOS MAE |
|---|---|---|---|---|
| OLS Baseline | 287.76 | 309.95 | 1.36pp | 0.99pp |
| ARIMAX(1,1,1) | −72.39 | −43.90 | **1.12pp** | 0.62pp |
| SARIMAX (+seasonal) | −79.26 | −44.45 | 1.17pp | 0.65pp |

**ARIMAX(1,1,1)** was selected as the final model on out-of-sample RMSE.

**Forecast**: actual default rate as of September 2020 was 4.64%; the model forecasts 5.65% for September 2021, an implied increase of 1.01 percentage points.

## Note on in-sample fit

The final refit's in-sample RMSE (6.04pp) comes out notably higher than the held-out OOS RMSE (1.12pp), which is the reverse of what's normally expected. This is worth double-checking before presenting the result further — it may be a cold-start artifact in the fitted values (similar to an issue found in a related group project's ARIMAX residuals), rather than a genuine sign the model fits worse in-sample than out-of-sample.

## Tools

Python (`pandas`, `numpy`, `statsmodels`, `scikit-learn`, `matplotlib`, `scipy`)

## Files

- `Aarya_Dantara_FE_Individual_Assignment.ipynb` — full analysis notebook
- `data.xlsx` — anonymised institutional loan market dataset (as provided for the assignment)

## Author

Aarya Dantara
