# Executive Summary — ASX Blue-Chip Stock Forecasting

*Full model code: [`stock_forecasting_analysis.ipynb`](stock_forecasting_analysis.ipynb).*

## Business Problem

Risk-averse investors — people saving for retirement, education, or a home deposit — want stable, predictable returns rather than high-risk speculation. Blue-chip companies listed on the Australian Securities Exchange (ASX) are typically considered a lower-risk core holding because they are large, financially established and well covered by analysts.

This analysis asks a specific, testable question: **can time-series forecasting (ARIMA/SARIMA) applied to five years of daily price history reliably model near-term price behaviour for four ASX blue-chips across different sectors — and does that forecast actually add value over the simplest possible alternative, assuming tomorrow's price equals today's?**

## Companies Analysed

| Ticker | Company | Sector |
|---|---|---|
| CBA.AX | Commonwealth Bank of Australia | Banking & financial services |
| BHP.AX | BHP Group | Mining & natural resources |
| CSL.AX | CSL Limited | Healthcare & biotechnology |
| WES.AX | Wesfarmers | Diversified retail & industrial |

Data: daily closing price (AUD), 2020-01-20 to 2025-01-17 (source: Yahoo Finance), no missing values.

## Methodology

1. **Stationarity testing** (Augmented Dickey-Fuller test) on each price series; first-order differencing applied where the raw series was non-stationary.
2. **Model order selection** via ACF/PACF inspection. ARIMA(1,0,1) was used for three stocks; **SARIMA(1,0,1)(1,0,1,5)** was used for Wesfarmers, whose retail-driven business shows a structural weekly seasonal component (`m=5` for the 5-day trading week).
3. **A genuine 30-trading-day holdout test set** for every stock — the model is trained only on data before this window and forecasts it blind, rather than being evaluated on the same data it was fitted on.
4. **A naive baseline** ("tomorrow's price = today's price," held flat for 30 days) computed for every stock as the standard sanity check in price forecasting.

This differs from a typical academic ARIMA exercise in one important respect: **all reported accuracy is out-of-sample.** AIC/BIC (in-sample fit statistics, used only for model selection) are reported separately from RMSE/MAE (calculated only on the unseen 30-day test window).

## Results

| Ticker | Model | Model RMSE (AUD) | Naive RMSE (AUD) | Beats naive baseline? |
|---|---|---|---|---|
| CBA | ARIMA(1,1,1) | 3.085 | 3.077 | No |
| BHP | ARIMA(1,1,1) | 0.948 | 0.948 | Marginal (effectively tied) |
| CSL | ARIMA(1,0,1) | 3.723 | 5.015 | **Yes — ~26% lower RMSE** |
| WES | SARIMA(1,1,1)(1,0,1,5) | 1.958 | 1.936 | No |

**Absolute RMSE is not comparable across stocks** — it scales with price level (CSL trades near $280 AUD vs. $25-160 AUD for the others). The correct comparison is each stock's model against its *own* naive baseline, which is what the "beats naive baseline" column shows.

## Honest Interpretation

For three of the four stocks (CBA, BHP, WES), the ARIMA/SARIMA forecast is statistically indistinguishable from simply assuming no price change over the next month. **This is not a failure of the modelling — it is itself the finding.** It is consistent with the weak-form Efficient Market Hypothesis: for large, liquid, heavily-analysed blue-chips, recent public price history alone should not be expected to give a durable predictive edge, and a model that doesn't beat "no change" is honest evidence that these stocks are efficiently priced rather than systematically predictable from price history alone.

**CSL is the exception.** Its ARIMA model beat the naive baseline by a clear margin over the test window — a genuinely useful, non-obvious result, and notable given CSL also had the largest *absolute* forecast error of the four (an artefact of its higher price level, not a sign the model performed worse).

## Business Recommendations

1. **Treat ARIMA/SARIMA output here as a monitoring tool, not a trading edge**, for CBA, BHP and WES specifically — the data doesn't support claiming predictive power beyond a flat-price assumption for these three over a monthly horizon.
2. **CSL's result deserves a closer look** rather than being dismissed for its higher raw error — of the four, it is the one where price history carried a real, testable signal in this window.
3. **Don't use this in isolation for investment decisions.** Macroeconomic factors (interest rates, commodity prices for BHP, health-sector policy for CSL, consumer spending for Wesfarmers) are not in this model at all. A real low-risk portfolio decision should combine this analysis with fundamentals (e.g. Beta, dividend stability, sector diversification).

## Limitations

- **Single 30-day out-of-sample window per stock.** A more robust evaluation would use rolling-window backtesting across many historical windows, not one holdout period — a single window can be lucky or unlucky.
- **No macroeconomic or company-fundamental variables** — the model uses price history only.
- **Model orders were chosen once via ACF/PACF inspection**, not an automated search (e.g. `auto_arima`); a marginally better-fitting order may exist for any given stock.
- **The five-year window spans the COVID-19 shock and recovery** — an unusually volatile period that may not represent "normal" market behaviour going forward.

## References

- Box, G. E. P., Jenkins, G. M., Reinsel, G. C., & Ljung, G. M. 2015, *Time Series Analysis: Forecasting and Control*, John Wiley & Sons.
- Seabold, S. & Perktold, J. 2010, 'Statsmodels: Econometric and statistical modeling with Python', *Proceedings of the 9th Python in Science Conference*, pp. 57-61.
- Hunter, J. D. 2007, 'Matplotlib: A 2D graphics environment', *Computing in Science & Engineering*, vol. 9, no. 3, pp. 90-95.
- Investopedia (n.d.), *Forecasting: What It Is, How It's Used in Business and Investing*. https://www.investopedia.com/terms/f/forecasting.asp
- Commonwealth Bank of Australia, BHP Group, CSL Limited, Wesfarmers Limited — company overviews via Yahoo Finance and respective annual reports.

---
*Coursework project — Master of Business Analytics, Kaplan Business School (2025).*
