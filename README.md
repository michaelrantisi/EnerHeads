# Enerheads Quantitative Challenge

This repository contains solutions for the Enerheads Quantitative Challenge, which covers two core aspects of energy trading:

- **Imbalance and Activations Prediction Model**
- **BESS Schedule Optimization**

Both tasks address key challenges in modern energy markets where increasing renewable energy penetration leads to greater variability and imbalances, and where battery energy storage systems (BESS) can provide flexibility and arbitrage opportunities.

---

## 1. Imbalance and Activations Prediction Model

### Task Description

#### **Objective:**
Forecast system imbalances and mFRR activations using meteorological and market data for 2025 Q1.

#### **Data:**
- **Meteorological Data:**  
  Files (e.g., `openmeteo_location0.csv` to `openmeteo_location3.csv`) provide day-ahead forecasts (columns ending with `_previous_day1`) and intraday values.
- **Market Data:**  
  The file `spot_balancing_2025Q1.csv` contains market indicators including:
  - `LT_mfrr_SA_up_activ` (mFRR upwards activations in MW)
  - `LT_mfrr_SA_down_activ` (mFRR downwards activations in MW)
  - `LT_imb_MW` (system imbalance in MW)
  - `10YLT-1001A0008Q_DA_eurmwh` (Nord Pool day-ahead auction cleared price in EUR/MWh)

#### **Methodology:**
- **Feature Engineering:**  
  Compute forecast deltas, lag features, and rolling window statistics.
- **Model Training:**  
  Use Random Forest and XGBoost to predict both imbalance and mFRR activations.
- **Time Series Forecasting:**  
  Implement a rolling one‑step ahead ARIMA model for mFRR activations.
- **Analysis:**  
  Evaluate prediction errors (RMSE, MAE) and examine the correlation between imbalance and activations.

#### **Key Outputs:**
- **Imbalance Predictions:**
  - Random Forest RMSE: ~11.51, MAE: ~6.79
  - XGBoost RMSE: ~9.78, MAE: ~6.05
- **mFRR Activations Predictions:**
  - Random Forest RMSE: ~2.20, MAE: ~0.42
  - XGBoost RMSE: ~2.47, MAE: ~0.52
- **ARIMA Forecast (Rolling):**
  - RMSE: ~3.42, MAE: ~1.03
- **Correlation:**
  - Approximately -0.49 between imbalance and activations
- **Visualizations:**
  - Time-series plots comparing actual vs. predicted activations
  - Hourly average imbalance trends

## 2. BESS Schedule Optimization

### Task Description

#### **Objective:**
Optimize the schedule of a battery energy storage system (BESS) to maximize profit from arbitrage in the balancing market.

#### **Constraints & Assumptions:**
- **Battery power:** 1 MW, **Capacity:** 2 MWh.
- The battery can complete up to 2 cycles per day (i.e., a full cycle equals charging/discharging 2 MWh).
- Market activations are only available if they occur in that time unit.
- Charge and discharge cannot occur simultaneously.
- Minimal bid is 1 MW (0.25 MWh per 15-minute interval).
- The operator is a price taker with perfect foresight (all prices and activations are known in advance).

#### **Methodology:**

- **Optimization Model:**
  - Implemented using Pyomo in the `SingleMarketSolver` class.
  - The model includes constraints for energy dynamics, power (no simultaneous charge/discharge), activation limits, and daily cycle constraints.
  - The objective is to maximize profit, calculated as the difference between discharge revenue and charging cost.
  
- **Rolling Horizon:**
  - The function `optimize_over_horizon` runs the optimization daily (24-hour horizon) and computes the profit per MWh.
  - A 95% confidence interval for profit is calculated.
  
- **Cycle Depth Calculation:**
  - A helper function computes the average cycle depth from the battery’s state-of-charge (SoC) profile, which is an indicator of battery wear.

#### **Key Outputs:**

- **Profit per MWh:**
  - Mean profit per MWh: ~383.39 EUR/MWh
  - 95% Confidence Interval: (289.44, 477.33) EUR/MWh
  
- **Cycle Depth:**
  - For January 7, 2025, the average cycle depth was 0% (indicating minimal battery cycling under the given market conditions).
  
- **Schedule:**
  - Daily solver outputs include timestamps, charging/discharging decisions, and SoC values.
  - The schedule for the last solved day is saved as `BESS_schedule_output.csv`.

---

### Code Overview

#### **Imbalance & Activations Prediction Code**

- **Data Processing:**
  - Loads meteorological and market data, converts timezones (market data from EET to UTC), and computes forecast deltas.
  
- **Feature Engineering:**
  - Creates lag and rolling window features.
  
- **Modeling:**
  - Trains Random Forest and XGBoost regression models.
  - Implements a rolling ARIMA forecast to update predictions using actual values as they become available.
  
- **Evaluation & Visualization:**
  - Computes RMSE and MAE for both models.
  - Visualizes the actual vs. predicted activations and hourly imbalances.

#### **BESS Optimization Code**

- **Data Processing:**
  - Loads and processes market data with correct timezone conversion.
  
- **Pyomo Optimization Model:**
  - The `SingleMarketSolver` class sets up the model with decision variables for charging, discharging, and state-of-charge (SoC).
  - Constraints enforce initial SoC, energy dynamics, power limits, activation-based decisions, and daily cycle restrictions.
  
- **Rolling Horizon & Profit Analysis:**
  - Daily optimization runs are conducted with the `optimize_over_horizon` function.
  - Profit per MWh is computed along with a 95% confidence interval.
  
- **Cycle Depth Analysis:**
  - A function calculates the average cycle depth from the SoC profile.
  
- **Output:**
  - Schedules and profit metrics are printed, and the schedule is saved to CSV.
 
# How to Run the Code

## Dependencies:
- Python 3.x
- Pyomo
- pandas
- numpy
- scipy
- scikit-learn
- xgboost
- statsmodels
- matplotlib
- GLPK (for solving Pyomo models)

## Data Files:
- Ensure that the meteorological data files (`openmeteo_location0.csv`, etc.) and the market data file (`spot_balancing_2025Q1.csv`) are placed in the specified directory.
- Update file paths if necessary.

## Execution:
1. Open the provided Jupyter Notebook or Python script.
2. Run the cells sequentially to load data, build models, and generate outputs.
3. Check the console for printed outputs and view the generated plots.
4. The optimized schedule for BESS is saved as `BESS_schedule_output.csv`.

## Summary of Outputs

### Imbalance & Activations Prediction:
- **Imbalance Prediction:**
  - RF: RMSE ≈ 11.51, MAE ≈ 6.79
  - XGBoost: RMSE ≈ 9.78, MAE ≈ 6.05
- **mFRR Activations Prediction:**
  - RF: RMSE ≈ 2.20, MAE ≈ 0.42
  - XGBoost: RMSE ≈ 2.47, MAE ≈ 0.52
- **ARIMA (Rolling Forecast):**
  - RMSE ≈ 3.42, MAE ≈ 1.03
- **Correlation:**
  - -0.49 (between imbalance and activations)
- **Visualizations:**
  - Forecast vs. actual plots and hourly average imbalance charts.

### BESS Optimization:
- **Profit Analysis:**
  - Mean profit per MWh: ~383.39 EUR/MWh
  - 95% CI: (289.44, 477.33) EUR/MWh
- **Cycle Depth:**
  - Average cycle depth: 0% for January 7, 2025 (indicating minimal cycling)
- **Daily Schedule:**
  - Detailed schedule output with timestamps, decisions, and SoC values.

## Conclusion
The project successfully addresses the Enerheads Quantitative Challenge by integrating predictive modeling for energy imbalances with an optimization framework for BESS operation. The forecasting models provide insights into imbalance dynamics, while the optimization model identifies profitable strategies under realistic market constraints. The low cycle depth on certain days may reflect either conservative operation or favorable market conditions, offering additional insights for battery health management.

Feel free to explore the code further, adjust parameters, and test on different time horizons to refine the solution.
