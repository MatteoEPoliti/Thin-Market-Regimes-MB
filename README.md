# Thin-Market Regimes in the Italian Balancing Market

Empirical identification of a structural two-regime behavior in MB prices,
using TERNA, GME and ENTSO-E data (Jan–Aug 2025).

![Price BM vs Volume BM — thin-market regime](docs/price_vs_volume.png)

---

## Key finding

Balancing prices in the Italian MB show a sharp volatility spike in the
**[−50, +50] MWh zone** of net imbalance volume — a *thin-market regime*
where individual players lose price-taker status. Regime switches of
roughly **130 €/MWh** occur at the zero-crossing (from ~200 €/MWh to
~70 €/MWh depending on sign).

A global linear model fits the data with an RMSE/std ratio close to 1 —
no better than the unconditional mean. Regime-segmented evaluation,
by contrast, reveals structure invisible to the global fit, and forms
the empirical core of my Master's thesis in Energy Engineering.

---

## Why it matters

Players whose imbalance brings them into the thin zone face structurally
different settlement prices from those who stay outside it. A forecasting
or strategy framework that ignores this non-linearity is mis-specified
by construction — not by noise. The finding motivates the regime-based
architecture of the full thesis.

---

## Data

| Source   | Data                                   | Format              | Period       |
|----------|----------------------------------------|---------------------|--------------|
| GME      | Day-ahead prices: PUN, CNOR            | XML → XLSX (Jun→Jul)| Jan–Aug 2025 |
| ENTSO-E  | Load forecast and actual, IT-North     | CSV                 | Jan–Aug 2025 |
| ENTSO-E  | Solar and wind forecast and actual, IT | CSV                 | Jan–Aug 2025 |
| TERNA    | Balancing volumes and prices, IT-North | CSV                 | Jan–Aug 2025 |

> GME changed its publication format mid-year (June → July), requiring
> a dual ingestion path for day-ahead prices.

---

## Pipeline

### 1. Data cleaning

- Parsing of XML, CSV and XLSX across four independent sources
- Italian decimal handling (`","` → `"."`)
- Timestamp alignment from hourly (MGP, load) to 15-minute resolution
  (MB, RES) via row expansion
- **Source-aware NaN handling**: solar and wind split before `dropna()`
  to avoid discarding ~7,000 solar records due to missing wind intraday
  data; residual wind gaps filled via forward-fill (<0.8% affected)
- **Geographic scope**: IT-North/Centre macrozone only; "SUD" excluded

### 2. Feature engineering

- **Congestion index**: percentage price divergence between PUN and CNOR
- **Forecast errors**: load, solar and wind (actual − forecast)
- **Lag shift(16)**: exogenous features lagged by 4 hours (16 × 15 min),
  consistent with ENTSO-E publication delay — prevents data leakage
- **Autoregressive terms**: balancing volume and price lagged by 4 hours
- **Time features**: hour of day, weekend flag

### 3. Modelling

Linear regression baseline trained on January–June 2025 and tested on
July–August 2025, targeting balancing volume and price separately.

| Target     | RMSE / Std |
|------------|------------|
| Volume BM  | ~0.93      |
| Price BM   | ~0.94      |

Performance is intentionally limited: the model is a diagnostic baseline,
not a forecasting tool. Its failure to generalize is the motivation for
the regime-based framework, not a flaw to be optimized away.

### 4. Regime detection

The scatter plot of Price BM vs. Volume BM reveals the structural break
described above. The thin-market zone is derived empirically from the
data, not imposed a priori — which is the analytical claim the thesis
builds on.

---

## Methodological notes

- Pearson correlation underestimates feature relevance for non-linear
  targets; scatter plots surfaced structure that the correlation
  heatmap missed
- A global RMSE is methodologically misleading when the data contains
  structurally distinct regimes — regime-specific evaluation is both
  more rigorous and substantively different
- The CNOR day-ahead price is the strongest proxy for fundamental
  market conditions in the MB
- RES forecast errors show weak *linear* correlation with balancing
  prices — consistent with non-linear, threshold-dependent effects

---

## Stack

`Python` · `pandas` · `numpy` · `scikit-learn` · `matplotlib` · `seaborn`

Jupyter Notebook, VS Code, Git.

---

## Relation to the thesis

This repository covers data ingestion, feature engineering, baseline
modelling and empirical regime identification. The full thesis extends
the work with a regime classification framework, a two-layer
ARIMAX + Random Forest forecasting architecture, and backtesting of
two trading strategies on regime-segmented data over an expanded
(2–3 year) dataset.
