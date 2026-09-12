# Time Series Fundamentals

## What is it?

A **time series** is data indexed by time, where **order matters and adjacent points are typically correlated** — unlike the i.i.d. assumption most of this vault's earlier modules implicitly lean on. This single difference is why time series needs its own modeling and validation approach rather than being treated as an ordinary tabular regression problem.

---

## Decomposition: Trend, Seasonality, Residual

A time series is commonly decomposed into three components:

$$y_t = T_t + S_t + R_t \quad \text{(additive)} \qquad \text{or} \qquad y_t = T_t \times S_t \times R_t \quad \text{(multiplicative)}$$

- **Trend ($T_t$)** — the long-run direction (growing, shrinking, flat).
- **Seasonality ($S_t$)** — a pattern that repeats at a fixed, known period (daily, weekly, yearly).
- **Residual ($R_t$)** — whatever's left after removing trend and seasonality — ideally close to unpredictable noise, though a bad decomposition can leave real structure trapped here.

Use additive decomposition when seasonal fluctuations have roughly constant absolute magnitude regardless of the trend's level; multiplicative when seasonal swings grow proportionally with the trend (e.g. holiday sales spikes that are a fixed *percentage* increase over a growing baseline, not a fixed dollar amount).

## Stationarity

A series is **stationary** if its statistical properties (mean, variance, autocorrelation) don't change over time. Many classical time series models (ARIMA in particular) assume stationarity; a non-stationary series (one with trend or changing variance) typically needs **differencing** — modeling $y_t - y_{t-1}$ instead of $y_t$ directly — to remove the trend and produce something closer to stationary before those models apply cleanly.

## Classical Models: ARIMA

**ARIMA (AutoRegressive Integrated Moving Average)** combines three ideas: **AR** (the current value depends on a weighted sum of its own recent past values), **I** (differencing, to handle non-stationarity), and **MA** (the current value depends on recent forecast errors, not just recent actual values). Still a strong, interpretable baseline for many business time series, especially with clear trend/seasonality and a moderate amount of history — before reaching for something heavier.

## Modern Approaches

Gradient-boosted trees ([[Gradient Boosting]]) or neural networks ([[Recurrent Neural Network (RNN)]], or Transformer-based forecasting models) can outperform classical methods when there's substantial history, multiple related series, or useful external features (weather, promotions, holidays) to incorporate — at the cost of needing more data and more careful validation than a classical model requires to avoid overfitting.

---

## Why Standard Train/Test Splitting Is Wrong Here

A random train/test split (as used everywhere else in this vault) can put a future point in the training set and a past point in the test set — the model would then be evaluated on its ability to predict the past *using the future*, which is unrealistic and produces an optimistic performance estimate that won't hold up in real deployment. This is a direct instance of [[Data Leakage]], specific to temporal data. The correct approach is a **temporal split**: train on the past, test on a later period — see [[Cross Validation Strategy]]'s and [[Rolling-Origin Validation]]'s treatment of forward-chaining validation, which is the time-series-correct analog of standard k-fold CV.

---

## Interview Questions

**Why can't you use a standard random train/test split for a time series forecasting problem?** A random split can place future data points in the training set while test points come from an earlier time — the model would then effectively be predicting the past using information from the future, which is unrealistic and produces an evaluation that doesn't reflect real deployment; a temporal (forward-chaining) split, training on the past and testing on a strictly later period, is required instead.

**When would you use a multiplicative rather than additive decomposition?** When seasonal fluctuations scale proportionally with the trend's current level (e.g. a growing business's holiday sales spike is a percentage increase over a rising baseline, not a fixed absolute amount) — additive decomposition assumes the seasonal swing's absolute size stays constant regardless of the trend level, which doesn't hold in that case.

**Why does ARIMA require differencing for many real-world series?** ARIMA's core assumptions rely on stationarity (constant statistical properties over time); a series with a clear trend has a mean that changes over time, violating that assumption directly — differencing (modeling period-over-period changes instead of raw values) removes the trend and typically produces a series closer to stationary that ARIMA's assumptions actually hold for.

## Connections

- [[Data Leakage]], [[Rolling-Origin Validation]], [[Cross Validation Strategy]] — why time series needs its own validation discipline
- [[Recurrent Neural Network (RNN)]] — a neural approach to sequential forecasting, sharing structure with sequence-model-based text tasks
- [[Gradient Boosting]] — a common modern practical choice for forecasting with rich external features
- [[Correlation vs Causation]] — a time series correlated with an external event still needs the same causal scrutiny as any other observational data

## One-line Summary

> Time series data violates the i.i.d. assumption most other models rest on — decompose into trend/seasonality/residual, check stationarity before applying classical models like ARIMA, and always validate with a temporal (not random) split, since a random split is a specific form of data leakage unique to sequential data.
