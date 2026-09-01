# 🏙️ Apartment Sales in Buenos Aires

A linear regression project predicting apartment prices in Buenos Aires (Capital Federal),
comparing three regression approaches — plain Linear Regression, Ridge, and Lasso — to see
which handles the data best.

![Python](https://img.shields.io/badge/Python-3.11-blue?logo=python&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-data%20wrangling-150458?logo=pandas&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-modeling-F7931E?logo=scikitlearn&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)

## 📌 Project Overview

Real-estate listings are messy — missing coordinates, missing surface area, prices in
different currencies, extreme outliers. This project builds a full pipeline that:

1. Cleans raw Properati listing data for Buenos Aires apartments
2. Explores price, size, and location patterns visually
3. Builds a `scikit-learn` pipeline that **one-hot encodes** 57 neighborhoods and **imputes**
   missing coordinates
4. Trains and compares **three linear models** — Linear Regression, Ridge, and Lasso — to see
   which generalizes best to unseen listings

## 🗂️ Repo Structure

Each stage of the workflow lives in its own notebook, run in order:

```
buenos-aires-apartments/
├── data/
│   ├── buenos-aires-real-estate.csv       # raw data
│   ├── buenos-aires-cleaned.csv           # output of 1_eda.ipynb, used by all model notebooks
│   └── *_results.txt                      # test MAE from each model, used by the comparison notebook
├── notebooks/
│   ├── 1_eda.ipynb                        # cleaning + exploratory data analysis
│   ├── 2_model_linear_regression.ipynb    # Model 1: plain Linear Regression
│   ├── 3_model_ridge_regression.ipynb     # Model 2: Ridge (L2)
│   ├── 4_model_lasso_regression.ipynb     # Model 3: Lasso (L1)
│   └── 5_model_comparison.ipynb           # side-by-side comparison of all 3 models
├── images/                                # chart exports
├── requirements.txt
├── LICENSE
└── README.md
```

**Why separate notebooks per model?** Each one is a clean, standalone experiment — same
cleaned data, same train/test split (`random_state=42`), same pipeline shape, different
estimator. Makes it easy to open just one and see exactly what that model does, without
scrolling past the other two.

## 🧹 Data Cleaning Summary

| Issue in raw data | How it was handled |
|---|---|
| Mixed property types (apartment, house, PH, store) | Filtered to `apartment` only |
| Listings across all of Greater Buenos Aires | Filtered to `Capital Federal` (the city proper) |
| Prices in USD and ARS | Kept USD listings only |
| A small number of ultra-luxury outliers | Capped at `price_aprox_usd < $400,000` |
| Extreme surface-area outliers | Trimmed top/bottom 10% |
| `lat-lon` combined into one string | Split into numeric `lat`, `lon` — **nulls kept on purpose**, imputed later in the pipeline rather than dropped |
| Neighborhood buried in a pipe-delimited location string | Extracted into a clean `neighborhood` column (57 unique values) |
| Mostly-empty or price-leaking columns (`floor`, `rooms`, `price_per_m2`, etc.) | Dropped |

Final dataset: **~6,100 rows**, features = `surface_covered_in_m2`, `lat`, `lon`,
`neighborhood`. Target = `price_aprox_usd`.

## 🤖 Modeling Pipeline

Every model notebook uses the same `ColumnTransformer`:

- **`neighborhood`** → `OneHotEncoder` (57 categories)
- **`surface_covered_in_m2`, `lat`, `lon`** → `SimpleImputer` (mean), since `lat`/`lon` have a
  small number of missing values

...feeding into a `Pipeline` with the estimator swapped per notebook.

## 📊 Results

| Model | Test MAE (USD) |
|---|---|
| Linear Regression | ~$25,206 |
| **Ridge (α=1.0)** | **~$25,110** |
| Lasso (α=1.0) | ~$25,184 |

*(Exact values are in `5_model_comparison.ipynb` — see that notebook for the bar chart.)*

Ridge came out slightly ahead of the other two. With 57 one-hot-encoded neighborhood columns
relative to a dataset of ~6,100 rows, some regularization helps — Ridge shrinks the
neighborhood coefficients toward each other without zeroing any out entirely, which fits this
data better than an unregularized model or Lasso's more aggressive feature-elimination.

## 🚀 How to Run This Yourself

```bash
git clone https://github.com/<your-username>/apartment-sales-buenos-aires.git
cd apartment-sales-buenos-aires
pip install -r requirements.txt
jupyter notebook notebooks/1_eda.ipynb
```

Run the notebooks in numeric order — `1_eda.ipynb` generates `data/buenos-aires-cleaned.csv`,
which the model notebooks depend on, and each model notebook writes its own results file that
`5_model_comparison.ipynb` reads at the end.

## 📁 Data Source

Listings data adapted from the [Properati](https://www.properati.com.ar/) Buenos Aires
real-estate export. This repo is an independently built analysis inspired by coursework I
completed on apartment price prediction — rebuilt from scratch here so I could keep a
portfolio copy of the workflow and reasoning.

## 🔭 Possible Next Steps

- Try a non-linear model (Random Forest, Gradient Boosting) to capture neighborhood × size
  interactions that a linear model can't
- Engineer a `price_per_m2` feature normalized by neighborhood
- Add rooms/floor back in with proper imputation instead of dropping them outright


*Second project in my Data Science / Machine Learning portfolio — see my
[GitHub profile](https://github.com/<your-username>) for the rest.*
