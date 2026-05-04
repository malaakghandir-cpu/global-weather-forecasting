# global-weather-forecasting

# Global Weather Repository — Analysis & Forecasting

**Author:** Malaak Ghandir  

---

## PM Accelerator Mission

> *"By making industry-leading tools and education available to individuals from all backgrounds, we level the playing field for future PM leaders. This is the PM Accelerator motto, as we grant aspiring and experienced PMs what they need most – Access. We introduce you to industry leaders, surround you with the right PM ecosystem, and discover the new world of AI product management skills."*
>


---

## Project Overview

This project presents a full-stack analysis of the Global Weather Repository — a dataset of 137,998 daily weather observations spanning 257 cities, 211 countries, and two years of coverage (May 2024 to April 2026). The analysis covers the complete data science pipeline: diagnostics, preprocessing, feature engineering, exploratory data analysis, time series investigation, and predictive modelling.

The primary forecasting target is global daily mean temperature. Six models are built and compared — SARIMA, Holt-Winters, Linear Regression, Random Forest, XGBoost, and a selective ensemble — alongside a full suite of advanced analyses covering anomaly detection, air quality, spatial patterns, climate zones, and feature importance.

---

## Repository Structure

```
├── weather_analysis_final.ipynb   # Main Jupyter notebook — full pipeline
├── Final report.pdf               # Written technical report
├── GlobalWeatherRepository.csv    # Raw dataset
├── requirements.txt               # Python dependencies
├── spatial_comfort_map.html       # Interactive city comfort index map
├── spatial_aq_heatmap.html        # Interactive air quality heatmap
└── README.md                      # This file
```

---

## Dataset

The dataset is sourced from Kaggle:  
**[Global Weather Repository — Kaggle](https://www.kaggle.com/datasets/nelgiriyewithana/global-weather-repository/code)**

Download `GlobalWeatherRepository.csv` and place it in the root of this repository before running the notebook.

---

## How to Run

### 1. Clone the repository

```bash
git clone https://github.com/malaakghandir-cpu/global-weather-forecasting.git
cd global-weather-forecasting
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Launch the notebook

```bash
jupyter notebook weather_analysis_final.ipynb
```

Run all cells in order from top to bottom. The notebook is self-contained — all preprocessing, feature engineering, modelling, and visualisation steps are sequential and documented with markdown explanations.

---

## Requirements

Key dependencies:

```
pandas
numpy
scipy
matplotlib
seaborn
folium
statsmodels
scikit-learn
xgboost
shap
pycountry
fuzzywuzzy
python-Levenshtein
jupyter
```

Install all at once:

```bash
pip install -r requirements.txt
```

---

## Methodology Summary

### Data Cleaning & Preprocessing
- Detected and standardised 21 non-standard country names across 6 languages using a canonical English country list
- Verified unit conversion formulas empirically before dropping 8 redundant imperial columns
- Replaced 11 physically impossible values using bounds grounded in physical reality
- Detected anomalies using z-score > 4σ on calendar-date seasonal differences (1,256 flagged, 0.82%)
- Imputed remaining NaN via forward-fill within each city to preserve temporal continuity

### Feature Engineering
- **Cyclical encoding** — hour, day-of-year, month, wind direction (sin/cos pairs)
- **Astronomical features** — daylight hours, solar noon deviation
- **Log/sqrt transforms** — 6 heavily skewed air quality columns (log1p) + Ozone (sqrt)
- **Lag features** — temperature, pressure , PM2.5, humidity
- **Rolling statistics** — 7-day mean and standard deviation for temperature, humidity, pressure
- **Meteorological composites** — feels-like delta, atmospheric instability index, heat-humidity stress, dew point
- **Composite indices** — Human Comfort Index (5-component weighted), WHO-weighted AQ Health Risk Score
- **Geographic grouping** — continent (7 regions), simplified Koppen climate zone (5 classes)
- **Multicollinearity** — iterative VIF removal until all features < 10 (14 removed, 17 retained for LR)

### Exploratory Data Analysis
- Global temperature and precipitation trends with 30-day rolling average
- Temperature distribution by climate zone
- Countries with highest and lowest mean temperatures
- Geographic extremes (northernmost, southernmost, easternmost, westernmost cities)
- Categorical frequency analysis — condition text, moon phase, wind direction
- Correlation heatmap across raw and engineered features
- Moon phase vs precipitation analysis

### Advanced EDA
- Anomaly visualisation
- Regional temperature trends by Koppen climate zone
- Meteorological drivers of air quality (atmospheric inversion hypothesis)
- Interactive spatial maps — comfort index and air quality heatmap
- Continent radar chart across 6 weather dimensions

### Time Series Analysis
- ADF stationarity test — confirmed d=1 (raw p=0.385, differenced p=0.0001)
- ACF and PACF plots — confirmed SARIMA(1,1,1) specification
- Rolling statistics and volatility of global daily series

### Models Built
| Model | Type | Training Data |
|---|---|---|
| SARIMA (1,1,1)(1,0,1,7) | Time series | 560 global daily observations |
| Holt-Winters (s=30) | Exponential smoothing | 560 global daily observations |
| Linear Regression | Parametric ML | 122,068 city-level observations |
| Random Forest | Tree ensemble | 122,068 city-level observations |
| XGBoost | Gradient boosting | 122,068 city-level observations |
| Artificial Neural Network | Neural network | 122,068 city-level observations |
| Ensemble (selective) | Inverse-MAE weighted | Top 3 ML models combined |

### Evaluation Metrics
MAE, RMSE, MAPE, R², MASE, Directional Accuracy

### Feature Importance
- Linear regression coefficients
- Permutation importance (XGBoost, 20 repeats)
- SHAP values and dependence plots

---

## Key Results

| Model | MAE (°C) | R² | MASE | DirAcc (%) |
|---|---|---|---|---|
| Ensemble (selective) | 0.024 | 1.000 | 0.033 | 98.0 |
| Random Forest | 0.025 | 0.999 | 0.034 | 96.0 |
| ANN | 0.037 | 1.000 | 0.051 | 98.0 |
| XGBoost | 0.061 | 0.998 | 0.083 | 94.7 |
| Linear Regression | 1.209 | 0.389 | 1.646 | 74.0 |
| SARIMA | 1.573 | -0.059 | 2.141 | 43.3 |
| Holt-Winters | 2.659 | -1.940 | 3.620 | 43.3 |

**Note:** ML model results are evaluated on the global daily mean after averaging predictions across 257 cities. Individual city-level accuracy would be lower. Results should be interpreted with this in mind — see Limitations section of the report.

---

## Notable Findings

- 21 non-standard country names were found spanning 6 languages, including entries that passed standard ASCII character filters — highlighting a common data quality issue in API-sourced global datasets
- Kuwait and Ar Riyadh ranked as the least comfortable cities globally by the Human Comfort Index
- Anomaly density is significantly higher in 2025-2026 versus 2024-2025
- Engineered features (dew point, heat-humidity stress, feels-like delta) consistently outranked most of the original 41 columns in feature importance, validating the feature engineering effort
- Linear Regression achieved 74% directional accuracy with no lag features — suggesting the meteorological features alone carry strong predictive signal for temperature direction

---

## Limitations

- Global-level ML MAEs are inflated by the averaging effect across 257 cities
- Models rely heavily on yesterday's temperature (lag1), raising questions about genuine meteorological learning
- SARIMA's weekly seasonality (s=7) is a sample size concession, not a meteorological argument
- Two years of data is insufficient for meaningful climate trend analysis
- Country standardisation requires manual verification and would need to be repeated on dataset updates

---

## Demo Video

[Link to demo video](https://drive.google.com/file/d/1Q7oo1eEvK3BRiJTIIlXMM9JJqGcVh8Y9/view?usp=drive_link)

---

## Report

The full written report (`Final report.pdf`) covers methodology, results, interpretation, and limitations in detail.
