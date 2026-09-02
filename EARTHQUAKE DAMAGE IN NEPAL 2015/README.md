# 🌫️ Air Quality in Nairobi

A time-series forecasting project predicting hourly PM2.5 air quality readings in Nairobi,
comparing three linear-family forecasting approaches: **AutoReg**, **ARMA**, and a plain
**Linear Regression on engineered lag features**.

![Python](https://img.shields.io/badge/Python-3.11-blue?logo=python&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-data%20wrangling-150458?logo=pandas&logoColor=white)
![statsmodels](https://img.shields.io/badge/statsmodels-time%20series-8CAAE6)
![scikit-learn](https://img.shields.io/badge/scikit--learn-modeling-F7931E?logo=scikitlearn&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)

## 📌 Project Overview

Air quality sensors produce noisy, irregularly-timed readings — this project builds a full
pipeline that:

1. Cleans raw 5-minute PM2.5 sensor readings from Nairobi (timezone localization, outlier
   removal, hourly resampling, gap-filling)
2. Explores the series for autocorrelation structure (ACF/PACF) to understand what a
   forecasting model actually has to work with
3. Trains and compares **three linear-family time series models** on the exact same
   train/test split, each in its own notebook
4. Evaluates every model with **walk-forward validation** (or the closest fair equivalent)
   so the results reflect realistic forecasting conditions, not lookahead bias

## 🗂️ Repo Structure

```
nairobi-air-quality/
├── data/
│   ├── nairobi-city-air-quality.csv       # raw sensor data
│   ├── y_train.csv / y_test.csv           # cleaned hourly series, shared across all models
│   └── *_results.txt                      # test MAE from each model, used by comparison notebook
├── notebooks/
│   ├── 1_eda.ipynb                        # cleaning + exploratory data analysis
│   ├── 2_model_autoreg.ipynb              # Model 1: AutoReg (AR)
│   ├── 3_model_arma.ipynb                 # Model 2: ARMA
│   ├── 4_model_linear_regression.ipynb    # Model 3: Linear Regression on lag features
│   └── 5_model_comparison.ipynb           # side-by-side comparison of all 3 models
├── images/                                # chart exports
├── requirements.txt
├── LICENSE
└── README.md
```

## 🧹 Data Cleaning Summary

| Issue in raw data | How it was handled |
|---|---|
| 5-minute readings, no timezone info | Parsed timestamps, localized to `Africa/Nairobi` |
| A handful of sensor-glitch readings (implausibly high) | Dropped anything above 500 µg/m³ |
| Irregular / high-frequency readings | Resampled to hourly mean |
| Small gaps in the hourly series | Forward-filled |

Final series: **2,928 hourly PM2.5 readings** spanning September–December 2018, split
chronologically 90/10 into train/test (no shuffling — this is time series, order matters).

## 🤖 Why Three Models?

| Model | What it does | Notebook |
|---|---|---|
| **AutoReg (AR)** | Predicts the next value as a linear combination of the last *p* hours | `2_model_autoreg.ipynb` |
| **ARMA** | Extends AR with a moving-average term on past forecast errors | `3_model_arma.ipynb` |
| **Linear Regression on lags** | Manually engineers lag columns and fits plain `sklearn.LinearRegression` | `4_model_linear_regression.ipynb` |

All three are fundamentally linear models — the interesting question is whether a
purpose-built time-series tool (AutoReg, ARMA) beats a general-purpose regression doing the
same underlying math by hand.

Each model notebook uses the same `p` / lag-order search pattern: loop over candidate values,
score each by training MAE, pick the best by `idxmin()` — mirrors the hyperparameter search
approach used in the other projects in this portfolio.

## 📊 Results

| Model | Test MAE (PM2.5 units) | Evaluation method |
|---|---|---|
| **AutoReg** | **~1.24** | Walk-forward (refit every hour) |
| Linear Regression on lags | ~1.24 | Walk-forward (refit every hour) |
| ARMA | ~2.55 | One-shot multi-step forecast |

*(Exact values are in `5_model_comparison.ipynb`.)*

AutoReg and Linear Regression land at essentially the same MAE — expected, since they're
doing the same underlying math (a linear combination of past values) through two different
implementations. ARMA scores worse here, but the comparison isn't fully apples-to-apples:
refitting ARMA at every hour like the other two models would have taken far longer to run, so
it was evaluated with a single multi-step forecast instead, letting errors compound over the
test window rather than get corrected by each new observation.

**Final model: AutoReg** — purpose-built for this task, and matched Linear Regression's
accuracy with a simpler setup.

## 🚀 How to Run This Yourself

```bash
git clone https://github.com/<your-username>/air-quality-nairobi.git
cd air-quality-nairobi
pip install -r requirements.txt
jupyter notebook notebooks/1_eda.ipynb
```

Run notebooks in numeric order — `1_eda.ipynb` generates `data/y_train.csv` and
`data/y_test.csv`, which every model notebook depends on, and each model notebook writes its
own results file that `5_model_comparison.ipynb` reads at the end.

## 📁 Data Source

Sensor data adapted from a public air-quality monitoring dataset for Nairobi. This repo is an
independently built analysis inspired by coursework I completed on time-series forecasting —
rebuilt from scratch here (using a local CSV rather than a live MongoDB instance) so I could
keep a portfolio copy of the workflow and reasoning.

## 🔭 Possible Next Steps

- Re-run ARMA with true walk-forward validation for a fairer comparison, accepting the
  longer runtime
- Try SARIMA to capture daily/weekly seasonality explicitly
- Add exogenous features (weather, traffic patterns) if available

## 📄 License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

---

*Third project in my Data Science / Machine Learning portfolio — see my
[GitHub profile](https://github.com/<your-username>) for the rest.*
