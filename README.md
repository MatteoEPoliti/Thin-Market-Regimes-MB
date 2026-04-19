# Italian Electricity Balancing Market — Exploratory Analysis
**BM_light** · Master's Thesis in Energy Engineering · First standalone Python project

---

## Overview

This project is a self-contained exploratory analysis of the Italian electricity 
balancing market (Mercato del Bilanciamento, MB), 
built as part of my Master's thesis in Energy Engineering.

It covers the full pipeline from raw data ingestion to modelling and visualization, 
working directly with real market data from TERNA, GME, and ENTSO-E.

The core finding — a structural two-regime behavior in balancing prices driven 
by a thin market zone around zero imbalance volume — emerged empirically from the 
data and forms the analytical foundation of the thesis.

---

## Data Sources

| Source | Data | Format | Period |
|--------|------|--------|--------|
| GME | Day-ahead prices: PUN, CNOR | XML (Jan–Jun), XLSX (Jul–Aug) | Jan–Aug 2025 |
| ENTSO-E | Load forecast and actual, IT-North | CSV | Jan–Aug 2025 |
| ENTSO-E | Solar and wind forecast and actual, Italy | CSV | Jan–Aug 2025 |
| TERNA | Balancing volumes and prices, IT-North | CSV | Jan–Aug 2025 |

> Note: GME changed its publication format mid-year (June → July), requiring 
> a dual ingestion approach for day-ahead prices.

---

## Pipeline

### 1. Data Cleaning
- Parsing of XML, CSV, and XLSX files across four independent sources
- Italian decimal formatting (`","` → `"."`)
- Timestamp alignment from hourly resolution (MGP, load) 
  to 15-minute intervals (MB, RES) via row expansion
- **Source-aware NaN handling**: solar and wind split before `dropna()` 
  to avoid discarding ~7,000 solar records due to missing wind intraday data; 
  residual wind gaps filled with forward fill (<0.8% of data affected)
- **Geographic scope**: IT-North/Centre macrozone only; 
  "SUD" entries explicitly excluded

### 2. Feature Engineering
- **Congestion index**: percentage price divergence between PUN and CNOR macrozone
- **Forecast errors**: load, solar, and wind (actual − forecast)
- **Lag shift(16)**: all exogenous features lagged by 4 hours (16 × 15 min), 
  consistent with ENTSO-E publication delay to prevent data leakage
- **Autoregressive terms**: balancing volume and price lagged by 4 hours
- **Time features**: hour of day, weekend flag

### 3. Modelling
Linear regression baseline trained on January–June 2025, 
tested on July–August 2025, targeting balancing volume and price separately.

| Target | RMSE / Std |
|--------|-----------|
| Volume BM | ~0.93 |
| Price BM | ~0.94 |

Performance is intentionally limited: the model serves as a diagnostic 
baseline, not a forecasting tool. Its failure to generalize motivates 
the regime-based framework developed in the thesis.

### 4. Key Visualization: Thin Market Detection
The scatter plot of Price BM vs. Volume BM reveals a clear structural break: 
price volatility spikes sharply in the [-50, +50] MWh zone, 
identifying a thin market regime where individual players lose 
price-taker status. This was the central empirical finding of the project.

A regime switch in this zone can cause balancing prices to move 
from ~200 €/MWh to ~70 €/MWh depending on whether net imbalance 
crosses zero — a non-linearity invisible to a global linear model.

---

## Key Takeaways

- Pearson correlation underestimates feature relevance for non-linear targets; 
  scatter plots revealed structure that the heatmap missed
- A global RMSE metric is methodologically misleading when data 
  contains structurally distinct regimes
- The CNOR day-ahead price is the strongest proxy for fundamental 
  market conditions in the MB
- RES forecast errors show weak linear correlation with balancing prices — 
  consistent with non-linear, threshold-dependent effects

---

## Tools

`Python` · `pandas` · `numpy` · `scikit-learn` · `matplotlib` · `seaborn`

---

## Status

This is the first version (BM_light), focused on EDA and baseline modelling.  
The full thesis extends this work with:
- Regime classification framework (thin vs. normal market)
- ARIMAX + Random Forest two-layer forecasting architecture
- Backtesting of two trading strategies on regime-segmented data
- Expanded dataset (2–3 years) to capture annual seasonality
