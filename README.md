# PM2.5 Research Project

## Predicting Site-Level PM2.5 Concentrations in Santa Cruz, California During Wildfire Season Using Machine Learning

## Overview
This project investigates how accurately different machine learning and statistical models can predict daily, site-level PM2.5 (fine particulate matter) concentrations at a single EPA monitoring site in Santa Cruz, CA, one day ahead. Thirteen predictive approaches — a persistence baseline, three regression-based methods (Linear Regression, Polynomial Regression with Lasso/Ridge regularization, Elastic Net), two ensemble methods (Random Forest, Gradient Boosting), three neural networks (a multi-layer perceptron, a convolutional neural network, and a hybrid MLP + Linear Regression model), two classical time-series methods (ARIMA, SARIMAX), an exploratory LLM few-shot prompting approach, and a Zero-Shot Rule-Based heuristic baseline — were trained and evaluated on the same leakage-audited dataset to see which techniques actually deliver the most reliable forecasts.

**This repository originally targeted San Jose and reported inflated results due to a temporal-leakage bug (same-day weather, fire, and vegetation data were being used to predict same-day rather than next-day PM2.5). Both issues are corrected in the current version — see [Correction Notice](#correction-notice) below.**

## Correction Notice
An earlier version of this project reported R² = 0.7995 for the top model, using a dataset framed around San Jose. Two independent issues inflated that number and mislabeled the study area:

1. **Temporal leakage.** Weather, fire, and vegetation predictors were not lagged to a forecast origin, so the model had access to same-day information when predicting same-day PM2.5. This is closer to same-day estimation than genuine next-day forecasting. All predictors are now explicitly shifted to day *t*-1 before being used to predict day *t*'s PM2.5.
2. **Study area.** San Jose's nearest AQS monitors did not have a complete daily PM2.5 record for 2025. The actual data used throughout this project's analysis, then and now, came from the nearest complete-record site: Santa Cruz, about 40 km (25 miles) southwest of San Jose. This README previously described the study as "neighborhood-level" San Jose forecasting; the analysis has only ever used a single monitoring site, so "site-level Santa Cruz" is the accurate description.

The corrected notebook in this repository fixes both issues, adds several predictors that were previously missing or scale-incorrect, and reruns the full model comparison. **The corrected top-model R² is 0.478, not 0.7995.** See Results below for the full corrected table.

## Motivation
Wildfires have grown more frequent and severe across California, and their smoke can travel far beyond the burn area. PM2.5 is the most harmful pollutant in that smoke, small enough to penetrate deep into the lungs and bloodstream, and is linked to asthma attacks, COPD, cardiovascular disease, and higher mortality. Santa Cruz doesn't sit near a fire's origin, but its air quality still degrades from smoke and pollutants carried in from elsewhere. A reliable next-day forecast, delivered with enough lead time, could help residents limit exposure, help schools plan activities, and give healthcare providers and emergency planners advance notice.

This project does not attempt to determine whether measured PM2.5 came from wildfire smoke specifically. Regional fire activity (from satellite hotspot detections) is used as one candidate predictor among several, not as proof that smoke reached the monitor — see Limitations.

## Research Question
How accurately can machine learning models forecast next-day, site-level PM2.5 concentrations in Santa Cruz, California, once the dataset is built in a way that avoids temporal information leakage?

## Data Sources
All data comes from public sources:
- **PM2.5 measurements (target variable):** EPA Air Quality System (AQS), site 60870007, Santa Cruz, CA — 360 daily observations, calendar year 2025
- **Weather variables:** NOAA High-Resolution Rapid Refresh (HRRR) model, 0-hour analysis fields, pulled via the Herbie Python package — temperature, dewpoint, wind speed and direction (derived from u/v components), relative humidity (derived from temperature and dewpoint), boundary layer height, surface pressure, and precipitation (from the following hour's forecast field, since analysis fields have zero accumulated precipitation by definition)
- **Wildfire activity:** NASA Fire Information for Resource Management System (FIRMS) — Suomi NPP VIIRS and NOAA-20 VIIRS satellite detections, confidence-filtered, aggregated within 100 km into detection count, mean/max/total fire radiative power (FRP), an inverse-distance-weighted FRP sum, and a 3-day cumulative detection count
- **Vegetation conditions (NDVI):** MODIS MOD13Q1 via the Google Earth Engine Python API, correctly scaled (raw values ÷ 10,000) and interpolated to a daily time series
- **Study area:** Both the target PM2.5 data and this analysis have always used the Santa Cruz AQS site specifically (not Santa Clara County / San Jose). Predictor variables are drawn from a wider Northern/Central California region, since fires affecting Santa Cruz's air typically originate elsewhere.

## Methodology
1. Defined a single operational forecast origin (00:00 local time on day *t*) and collected weather, wildfire, vegetation, and PM2.5 data onto a common daily time scale
2. Shifted every non-PM2.5 predictor forward one day, so day *t*'s feature row only reflects information available before day *t* — the fix for the original temporal-leakage bug
3. Fixed PM2.5 rolling-average features to exclude the current day's own value (previously a direct target leak)
4. Filled HRRR gaps causally (forward-fill only, never using a later pull to fill an earlier gap)
5. Audited missing data before dropping any rows (see notebook for the full table); 7 of 360 days were dropped, leaving 353 usable days
6. Selected the polynomial-regression degree and all hyperparameters using nested, time-series cross-validation on the training data only — never the test set
7. Split data chronologically into training (282 days) and testing (71 days)
8. Trained and evaluated 12 models plus a persistence baseline
9. Compared all models using MAE, MSE, RMSE, and R², plus a paired bootstrap significance test
10. Ran a temporally valid ablation study (PM2.5 history only / weather only / fire only / combinations) to test whether fire and weather predictors actually add value
11. Checked coefficient stability across 5 time-series cross-validation folds
12. Compared tuned vs. untuned versions of Random Forest, Gradient Boosting, MLP, and CNN

### Models Evaluated
- Persistence Baseline (previous day's PM2.5 value)
- Linear Regression
- Polynomial Regression with Ridge/Lasso regularization (degree and penalty selected via nested CV)
- Elastic Net
- Random Forest (tuned)
- Gradient Boosting (tuned)
- Multi-Layer Perceptron (MLP, tuned)
- Convolutional Neural Network (CNN, tuned) — included as an exploratory nonlinear comparator; a 1D convolution over an unordered feature vector has no clear physical justification here
- Hybrid MLP + Linear Regression (linear regression on MLP hidden-layer features)
- ARIMA
- SARIMAX (with the full predictor set as exogenous regressors)
- LLM few-shot prompting (distilgpt2, exploratory; see notebook for full prompt template)
- Zero-Shot Rule-Based (fixed, hand-set heuristic weights, not trained on data)

## Results

| Model | MAE | MSE | RMSE | R² |
|---|---|---|---|---|
| Polynomial Regression + Lasso (Degree 1) | 1.0204 | 1.8141 | 1.3484 | 0.4779 |
| Baseline (persistence) | 1.2352 | 2.2345 | 1.4948 | 0.3583 |
| Random Forest (tuned) | 1.3084 | 2.6673 | 1.6332 | 0.2340 |
| Gradient Boosting (tuned) | 1.3195 | 2.6952 | 1.6417 | 0.2260 |
| Zero-Shot Rule-Based | 1.3687 | 2.7608 | 1.6616 | 0.2072 |
| ARIMA | 1.5613 | 3.4956 | 1.8696 | -0.0038 |
| Elastic Net | 1.3717 | 3.5558 | 1.8857 | -0.0211 |
| MLP (tuned) | 1.6811 | 4.5811 | 2.1404 | -0.3155 |
| Linear Regression | 1.6117 | 6.2507 | 2.5001 | -0.7950 |
| CNN (tuned) | 1.9918 | 6.2534 | 2.5007 | -0.7958 |
| SARIMAX | 2.0191 | 6.4864 | 2.5468 | -0.8627 |
| LLM Few-Shot Prompting | 2.7873 | 11.8410 | 3.4411 | -2.4003 |
| MLP + Linear Regression | 2.8913 | 21.1757 | 4.6017 | -5.0809 |

**The polynomial degree was selected honestly this time** (via nested cross-validation on training data only, not by comparing final test-set scores across degrees, which is itself a leakage issue). Both Ridge and Lasso consistently selected **degree 1** — no actual polynomial curve. The top model is, in practice, a regularized linear (Lasso) model.

A paired bootstrap comparison (10,000 resamples) found the top model statistically **significantly better** than Linear Regression, Elastic Net, and the persistence baseline (all 95% confidence intervals for the RMSE difference exclude zero, favoring the top model in 97.6–99.9% of resamples).

**Ablation study** (does fire/weather actually help, tested directly): PM2.5 history alone gets R² = 0.42. Adding weather nudges it to R² = 0.47 (not statistically significant). Fire alone performs badly (R² = −1.58), and **adding fire predictors to a weather-only model made it significantly worse, not better** (95% CI for the RMSE change is entirely negative: −0.174 to −0.090). The fire and NDVI predictors used here do not show a statistically detectable positive contribution beyond PM2.5 history and weather.

**Coefficient stability** (Lasso refit across 5 time-series CV folds): relative humidity and wind direction (sine/cosine) were the most stable, reliably-signed predictors (100% sign-consistency across all folds). Several fire-intensity features (max FRP, distance-weighted FRP) were reduced to exactly zero by Lasso in every single fold — independent evidence, from a different analysis than the ablation, that these particular features carry little linear signal here.

**Tuning effect:** hyperparameter tuning substantially improved Random Forest (R² 0.124 → 0.234) and Gradient Boosting (R² −0.194 → 0.226); the MLP and CNN improved by smaller margins and stayed negative either way.

## Key Takeaway
Model complexity did not translate into better predictions on this dataset, and this held up under a corrected, leakage-audited pipeline. A simple, regularized linear model — cheap to run and easy to interpret — outperformed ensemble methods, neural networks, time-series models, and an LLM-based approach for next-day, site-level PM2.5 forecasting. Unlike the original (leaky) version of this analysis, this result should **not** be read as evidence that the underlying predictor–PM2.5 relationship is fundamentally linear: the coefficient-stability check shows the top model's own coefficients are not fully stable across folds, and a ranking like this is also consistent with the study's modest sample size and the effect of regularization itself. The most defensible statement is narrower: under this dataset size, predictor set, and evaluation design, regularized linear models generalized as well as or better than every nonlinear alternative tested, and PM2.5's own recent history, not the fire or weather predictors, accounts for most of the achievable accuracy.

## Limitations
- Single calendar year (2025) and a single AQS monitoring site; findings may not generalize to other years, other Santa Cruz-area conditions, or to San Jose, the project's originally intended target
- This project does not attempt to attribute measured PM2.5 to wildfire smoke specifically; FIRMS hotspot detections indicate regional fire activity, not confirmed smoke impact at the monitor
- HRRR weather data were sampled for only 72 of 360 days and forward-filled for the rest, since pulling the full daily record was not computationally practical in this environment
- NDVI's 16-day composite interpolation still draws on composites from both before and after a given day, which is not fully causal; fixing this would require the raw sparse composite export, which wasn't available
- The neural-network and SARIMAX models were trained on a comparatively small sample (282 training days), which likely limits their ability to reach their full potential
- The final test partition contains only 71 days, which limits the precision of every reported metric and confidence interval
- This is a retrospective evaluation using historical data, not a validated live/operational forecasting system
- Predictors don't include land cover, traffic emissions, or atmospheric chemistry data

## Future Work
- Incorporate multiple years of data and additional nearby AQS monitors to increase sample size
- Add a genuine smoke-attribution product (e.g., the NOAA Hazard Mapping System smoke product) to test whether wildfire smoke specifically, not just regional fire activity, affects PM2.5 here
- Retrieve a complete daily HRRR record and the raw sparse NDVI export, to close the remaining causality gaps in weather and vegetation data
- Test probabilistic/interval forecasts and explicit AQI-threshold classification, which would be more directly actionable than a point PM2.5 estimate
- Apply explainable-AI methods (e.g., SHAP) to extend the coefficient-stability check to the nonlinear models
- Test whether "simple beats complex" holds at other single-site locations, using the same leakage-audit and ablation procedures

## Repository Structure
```
PM2.5-Research-Project/
├── data/
│   ├── raw/                 # Raw wildfire archive downloads (VIIRS C2 fire archives)
│   ├── processed/            # Cleaned, daily-resolution datasets per source (wildfire, NDVI, HRRR)
│   ├── merged/                # Progressive merges of PM2.5, NDVI, wildfire, and HRRR data
│   ├── predictions/           # Model output predictions (raw and scaled)
│   └── Daily_Data.csv         # Final combined daily dataset used for modeling
├── notebook/                  # Corrected Colab notebook: data pipeline, leakage fixes, model training,
│                               # ablation study, coefficient stability, tuned-vs-untuned comparison
└── README.md
```

## Code
The full analysis, data collection, preprocessing, model training, and evaluation was developed in a Google Colab notebook. The corrected version fixes the temporal-leakage bugs described above, adds a missing-data audit, adds four new/corrected predictors (relative humidity, wind direction, boundary layer height, surface pressure, and precipitation), and adds the ablation, coefficient-stability, and tuned-vs-untuned analyses.

## Tools & Libraries
Python, pandas, scikit-learn, TensorFlow/Keras + Keras Tuner (CNN), statsmodels (ARIMA/SARIMAX), HuggingFace transformers (LLM few-shot), Herbie (NOAA HRRR access), Google Earth Engine Python API, matplotlib

## Author
Pranav Ganesh
