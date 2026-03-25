# BM_light — Italian Balancing Market Analysis

## Overview
Preliminary project developed as part of my Master's thesis in Energy Engineering.
The objective was to demonstrate data extraction and cleaning capabilities on real
Italian electricity market data, as requested to develop the thesis. 
The modelling section (6-month train, 2-month test) serves
as a baseline and proof of concept, with known limitations documented below.

This analyzed sources are the Italian electricity balancing market (Mercato del Bilanciamento)
using data from TERNA, GME (Gestore Mercati Energetici) and ENTSO-E.

## Data Sources
- **Day-ahead prices (MGP)**: hourly PUN and CNOR zone prices, January–August 2025
  - January–June: XML files from GME
  - July–August: Excel files from GME (website format changed)
- **Load**: day-ahead forecast and actual total load for IT-North bidding zone (ENTSO-E)
- **RES generation**: solar and wind onshore forecast and actual generation for Italy (ENTSO-E)
  - Note: no offshore wind data available; no onshore wind actual data (intraday only)
- **Balancing Market (MB)**: zonal imbalance volumes and prices for IT-North, January–August 2025 (TERNA)

## Pipeline
1. **Data Cleaning**: parsing of XML, CSV and Excel files from four different sources,
   timestamp alignment from hourly (MGP, load) to 15-minute intervals (MB, RES)
2. **Feature Engineering**:
   - Congestion index: percentage price difference between PUN and CNOR macrozone
   - Forecast errors for load, solar and wind
   - Lagged variables (16 periods = 4 hours) to avoid leakage, consistent with
     ENTSO-E data publication delay
   - Autoregressive terms: balancing volume and price lagged by 4 hours
   - Time features: hour of day, weekend flag
3. **Modelling**: linear regression and Random Forest regressor trained on
   January–June 2025 and tested on July–August 2025,
   targeting balancing volume and balancing price separately

## Results
Both models show limited predictive performance (RMSE/std ≈ 0.94–0.96), attributed to:
- insufficient data length (8 months, no annual seasonality captured)
- missing key drivers (gas price, thermoelectric generation)
- high volatility and asymmetric distribution of balancing volumes and prices

## Future Work
- Extend dataset to 2–3 years to capture annual seasonality
- Add gas price (TTF) and thermoelectric generation as features
- Include additional autoregressive lags (t-1d, t-1w)

## Tools
Python, pandas, scikit-learn, matplotlib, numpy
