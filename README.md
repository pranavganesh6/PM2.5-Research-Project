# PM2.5 Research Project

## Predicting Neighborhood-Level Air Quality Spikes in San Jose During Wildfire Season Using Machine Learning

## Overview
This project investigates how accurately different machine learning and statistical models can predict daily, neighborhood-level PM2.5 (fine particulate matter) concentrations in San Jose, CA during wildfire season. Ten predictive approaches — spanning regression, ensemble learning, neural networks, time-series methods, and an exploratory LLM few-shot prompting approach — were trained and evaluated on the same dataset to see which techniques actually deliver the most reliable forecasts.

## Motivation
Wildfires have grown more frequent and severe across California, and their smoke travels far beyond the burn area, affecting millions of people who never see a flame. PM2.5 is the most harmful pollutant in that smoke, small enough to penetrate deep into the lungs and bloodstream, and is linked to asthma attacks, COPD, cardiovascular disease, and higher mortality. San Jose doesn't sit near a fire's origin, but its air quality still degrades from smoke carried in from elsewhere, and conditions can vary between neighborhoods within the same short window of time. Reliable neighborhood-level forecasts, delivered with enough lead time, could help residents limit exposure, help schools plan activities, and give healthcare providers and emergency planners advance notice.

## Research Question
How accurately can machine learning models predict neighborhood-level air quality spikes in San Jose during wildfire season?

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
5. Trained and evaluated 10 models plus a persistence baseline
6. Compared all models using MAE, MSE, RMSE, and R²

### Models Evaluated
- Persistence Baseline (previous day's PM2.5 value)
- Linear Regression
- Polynomial Regression with Lasso regularization
- Elastic Net
- Random Forest
- Gradient Boosting
- Multi-Layer Perceptron (MLP)
- Hybrid MLP + Linear Regression (linear regression on MLP hidden-layer features)
- ARIMA
- SARIMAX
- LLM few-shot prompting (exploratory)

## Results

| Model | MAE | MSE | RMSE | R² |
|-------|-----|-----|------|-----|
| **Linear Regression** | **0.673** | **0.723** | **0.850** | **0.795** |
| Polynomial Regression + Lasso (Degree 1) | 0.682 | 0.728 | 0.853 | 0.793 |
| Elastic Net | 0.696 | 0.759 | 0.871 | 0.784 |
| Random Forest | 0.831 | 1.064 | 1.032 | 0.698 |
| Gradient Boosting | 0.962 | 1.323 | 1.150 | 0.624 |
| MLP | 1.023 | 1.750 | 1.323 | 0.503 |
| Persistence Baseline | 1.232 | 2.217 | 1.489 | 0.370 |
| MLP + Linear Regression | 1.218 | 2.390 | 1.546 | 0.321 |
| ARIMA | 1.590 | 3.568 | 1.889 | -0.013 |
| LLM Few-Shot Prompting | 2.790 | 11.683 | 3.418 | -2.318 |
| SARIMAX | 6.237 | 48.614 | 6.972 | -12.805 |

**Linear Regression was the top performer**, explaining roughly 79.5% of the variance in PM2.5 concentrations, with Polynomial Regression (Lasso, degree 1) and Elastic Net close behind. Ensemble methods (Random Forest, Gradient Boosting) and the MLP trailed the regression models. Time-series methods (ARIMA, SARIMAX) and the exploratory LLM approach performed worst, in several cases failing to beat the naive persistence baseline.

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
