# PM2.5 Research Project

## Predicting Neighborhood-Level Air Quality Concentrations in San Jose During Wildfire Season Using Machine Learning

## Overview
This project investigates how accurately different machine learning and statistical models can predict daily, neighborhood-level PM2.5 (fine particulate matter) concentrations in San Jose, CA during wildfire season. Twelve predictive approaches — spanning regression-based methods (Linear Regression, Polynomial Regression with Lasso regularization, Elastic Net), ensemble methods (Random Forest, Gradient Boosting), neural networks (a multi-layer perceptron, a convolutional neural network, and a hybrid MLP plus Linear Regression model), time-series methods (ARIMA), an exploratory LLM few-shot prompting approach, a persistence baseline, and a Zero-Shot Rule-Based heuristic baseline — were trained and evaluated on the same dataset to see which techniques actually deliver the most reliable forecasts.

## Motivation
Wildfires have grown more frequent and severe across California, and their smoke travels far beyond the burn area, affecting millions of people who never see a flame. PM2.5 is the most harmful pollutant in that smoke, small enough to penetrate deep into the lungs and bloodstream, and is linked to asthma attacks, COPD, cardiovascular disease, and higher mortality. San Jose doesn't sit near a fire's origin, but its air quality still degrades from smoke carried in from elsewhere, and conditions can vary between neighborhoods within the same short window of time. Reliable neighborhood-level forecasts, delivered with enough lead time, could help residents limit exposure, help schools plan activities, and give healthcare providers and emergency planners advance notice.

## Research Question
How accurately can machine learning models predict neighborhood-level air quality concentrations in San Jose during wildfire season?

## Data Sources
All data comes from public sources:
- **PM2.5 measurements (target variable):** EPA Air Quality System (AQS)
- **Weather variables:** NOAA High Resolution Rapid Refresh (HRRR) model, pulled via the Herbie Python package — temperature, wind speed, wind direction, relative humidity, boundary layer height, surface pressure, precipitation
- **Wildfire activity:** NASA Fire Information for Resource Management System (FIRMS) — Suomi NPP VIIRS and NOAA-20 VIIRS satellite detections (fire hotspot location and radiative power)
- **Vegetation conditions (NDVI):** Google Earth Engine Python API, interpolated to a daily time series
- **Study area:** Target PM2.5 data from Santa Clara County; predictor variables drawn from a wider Northern/Central California region, since the fires affecting San Jose's air typically originate elsewhere

## Methodology
1. Collected and merged weather, wildfire, vegetation, and PM2.5 data onto a common daily time scale, keyed by observation date
2. Added previous-day PM2.5 concentration as a predictor
3. Built two dataset versions (raw and standardized) to accommodate models that need scaled features
4. Split data into training and testing sets
5. Trained and evaluated 12 models plus a persistence baseline
6. Compared all models using MAE, MSE, RMSE, and R²

### Models Evaluated
- Persistence Baseline (previous day's PM2.5 value)
- Linear Regression
- Polynomial Regression with Lasso regularization
- Elastic Net
- Random Forest
- Gradient Boosting
- Multi-Layer Perceptron (MLP)
- Convolutional Neural Network (CNN)
- Hybrid MLP + Linear Regression (linear regression on MLP hidden-layer features)
- ARIMA
- LLM few-shot prompting (exploratory)
- Zero-Shot Rule-Based (heuristic baseline)

## Results

| Model | MAE | MSE | RMSE | R² |
|---|---|---|---|---|
| Polynomial Regression + Lasso (Degree 3) | 0.6766 | 0.7059 | 0.8402 | 0.7995 |
| Linear Regression | 0.6734 | 0.7233 | 0.8504 | 0.7946 |
| Elastic Net | 0.6961 | 0.7591 | 0.8713 | 0.7844 |
| Random Forest (tuned) | 0.7984 | 1.0106 | 1.0053 | 0.7130 |
| Gradient Boosting (tuned) | 0.7757 | 1.0124 | 1.0062 | 0.7125 |
| CNN (tuned) | 0.8715 | 1.2148 | 1.1022 | 0.6550 |
| MLP (tuned) | 0.9259 | 1.3363 | 1.1560 | 0.6205 |
| MLP + Linear Regression | 1.2028 | 2.1830 | 1.4775 | 0.3801 |
| Baseline | 1.2319 | 2.2174 | 1.4891 | 0.3703 |
| ARIMA | 1.5929 | 3.6177 | 1.9020 | -0.0273 |
| LLM Few-Shot Prompting | 3.0708 | 13.0135 | 3.6074 | -2.6955 |
| Zero-Shot Rule-Based | 6.0069 | 39.6049 | 6.2932 | -10.2467 |

**Polynomial Regression with Lasso regularization (degree 3) was the top performer**, explaining roughly 80.0% of the variance in PM2.5 concentrations, narrowly ahead of Linear Regression and Elastic Net, which were close behind. Ensemble methods (Random Forest, Gradient Boosting) and the neural networks (CNN, MLP) trailed the regression models. Time-series methods (ARIMA, R² = -0.0273) and the exploratory LLM approach also underperformed, with Zero-Shot Rule-Based performing the worst of all twelve models tested, in several cases failing to beat the naive persistence baseline.

## Key Takeaway
Model complexity did not translate into better predictions on this dataset. Simple regression-based models — cheap to run and easy to interpret — outperformed ensemble methods, neural networks, time-series models, and an LLM-based approach for neighborhood-level wildfire smoke PM2.5 forecasting. This suggests the chosen predictor variables already captured the dominant physical drivers of smoke transport well enough that a linear fit was sufficient.

## Limitations
- Single wildfire season with a limited number of daily observations
- Predictors don't include land cover, traffic emissions, or atmospheric chemistry data
- Models built specifically for San Jose; would need validation elsewhere
- Evaluated on historical data rather than live forecasting conditions

## Future Work
- Incorporate multiple wildfire seasons and expand to other regions of California
- Test spatiotemporal architectures (graph neural networks, transformer-based forecasting, physics-informed ML) given a larger dataset
- Forecast probability of unhealthy AQI events several days in advance

## Repository Structure
## Repository Structure
```
PM2.5-Research-Project/
├── data/
│   ├── raw/                 # Raw wildfire archive downloads (VIIRS C2 fire archives)
│   ├── processed/           # Cleaned, daily-resolution datasets per source (wildfire, NDVI, HRRR)
│   ├── merged/               # Progressive merges of PM2.5, NDVI, wildfire, and HRRR data
│   ├── predictions/          # Model output predictions (raw and scaled)
│   └── Daily_Data.csv        # Final combined daily dataset used for modeling
├── notebook/                 # Google Colab notebook with data pipeline, model training, and evaluation
└── README.md
```

## Code
The full analysis, data collection, preprocessing, model training, and evaluation, was developed in a Google Colab notebook.

## Tools & Libraries
Python, pandas, scikit-learn, statsmodels (ARIMA/SARIMAX), Herbie (NOAA HRRR access), Google Earth Engine Python API

## Author
Pranav Ganesh
