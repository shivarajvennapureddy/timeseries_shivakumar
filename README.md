German Electricity Demand Forecasting — Time Series Case Study
Overview
This project models and forecasts German electricity demand at hourly, daily and weekly resolution, comparing statistical, feature-based and neural forecasting approaches. The central question is how temporal aggregation changes volatility, seasonality and predictability, and which model is appropriate for each decision horizon. The analysis covers January 2015 to October 2020 and evaluates every model on a common two-year test period (October 2018 to September 2020).
Data

Electricity load: German actual load (country code DE) from the Open Power System Data platform, 60-minute resolution, 2020-10-06 release. The raw hourly series is cleaned, converted to a regular UTC index, and aggregated to daily and weekly means.
Temperature: Historical hourly temperature for Berlin from the Open-Meteo archive API, used as a representative location for Germany and aggregated to weekly features (mean, minimum, maximum, heating and cooling degree days).
Calendar: German public holiday counts, working days, a Christmas–New Year indicator and annual sine/cosine terms, constructed programmatically.

Both data sources are publicly available and downloaded automatically by the pipeline; no manual data placement is required.
Methods

Exploratory analysis — multi-resolution plots, weekday–hour structure, STL decomposition, ACF/PACF at each resolution.
Stationarity testing — ADF and KPSS tests on the original, first-differenced, seasonally differenced and doubly differenced weekly series.
Benchmarks — Mean, Naive, Seasonal Naive and Drift forecasts over the two-year horizon.
SARIMA — exhaustive AIC grid search over p ∈ [0,6], d ∈ [0,2], q ∈ [0,6] with seasonal orders at period 52, followed by residual diagnostics (Ljung–Box, Jarque–Bera) and forecasting with 95% prediction intervals.
SARIMAX — covariate ablation adding temperature features, calendar features, and both combined, with explicit distinction between conditional (observed future temperature) and operational (calendar-only) forecasts.
Gradient Boosting — leakage-safe supervised design where every feature is built strictly from data before the target week, recursive multi-step forecasting, and hyperparameter tuning with time-series cross-validation.
LSTM — compact hourly network with a 168-hour input window, chronological train/validation/test split, architecture selection on the validation set, and a rolling one-hour-ahead forecast protocol, evaluated at hourly level and aggregated to weekly for the common comparison.

Key results
ModelRMSE (MW)NoteLSTM aggregated805One-step protocol; not comparable with multi-step forecastsGradient Boosting2,952Best fair multi-step model (+1.2% vs Seasonal Naive)Seasonal Naive2,988Strongest benchmarkSARIMAX Combined5,647Best SARIMAX variant; conditional forecastSARIMA8,873Over-differenced (d = 2); underperforms
The Gradient Boosting model is recommended for operational medium-term use based on accuracy, covariate availability at the forecast origin, interpretability and maintenance cost. The LSTM is the right tool for near-term hourly balancing. Full discussion, figures and evaluation tables are in the accompanying report.
How to run

Clone the repository.
Install dependencies: pandas, numpy, matplotlib, statsmodels, scikit-learn, tensorflow, holidays, requests.
Open the notebook in Jupyter or Google Colab and run all cells top to bottom. Cells are ordered and annotated; data download, modelling, evaluation and figure export happen automatically.
All figures and result tables are saved to an outputs directory created at runtime.

An internet connection is required on first run to download the load and temperature data.
Repository practices

Each analysis stage is implemented as documented functions with a single responsibility.
No data leakage: all features are constructed strictly from information available before each forecast target, and all splits are chronological.
Random seeds are fixed where applicable for reproducibility.
Figures are exported with readable axis labels for direct use in the report.
