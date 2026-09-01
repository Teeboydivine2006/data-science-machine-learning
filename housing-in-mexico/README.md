# 🏠 Housing in Mexico City — Does Size or Location Drive Price?

An end-to-end exploratory data analysis and linear regression project investigating what
actually predicts apartment prices in Mexico City: property **size**, **location** (borough),
or both.

![Python](https://img.shields.io/badge/Python-3.11-blue?logo=python&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-data%20wrangling-150458?logo=pandas&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-modeling-F7931E?logo=scikitlearn&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)

## 📌 Project Overview

Real-estate pricing is influenced by many factors, but two of the most intuitive are how big a
property is and where it's located. This project uses ~4,700 cleaned apartment listings from
Mexico City to answer a concrete question:

> **If you could only know one thing about an apartment — its square footage or its borough —
> which one gets you closer to guessing its price?**

The analysis:
1. Cleans and merges three raw CSV exports of real-estate listings
2. Explores price distribution, geographic spread, and price-by-borough patterns
3. Measures the correlation between price and covered surface area
4. Trains and compares three linear regression models (**size-only**, **location-only**,
   **size + location**) against a mean-prediction baseline

**Result:** Size is the stronger single predictor of price, but combining size and location
consistently beats either feature alone — they capture different, complementary information.

## 📊 Key Visuals

| Price Distribution | Price by Borough |
|---|---|
| ![price distribution](images/price_distribution.png) | ![price by borough](images/price_by_borough.png) |

| Price vs. Size | Model Comparison |
|---|---|
| ![price vs size](images/price_vs_size.png) | ![model comparison](images/model_comparison.png) |

## 🗂️ Repo Structure

```
housing-mexico/
├── data/                                # Raw CSV files (Properati Mexico City exports)
│   ├── mexico-city-real-estate-1.csv
│   ├── mexico-city-real-estate-2.csv
│   └── mexico-city-real-estate-3.csv
├── notebooks/
│   └── housing_in_mexico_city.ipynb     # Main analysis notebook (fully executed, outputs included)
├── images/                              # Saved chart exports (used in this README)
├── requirements.txt                     # Python dependencies
├── LICENSE
└── README.md
```

## 🧹 Data Cleaning Summary

| Issue in raw data | How it was handled |
|---|---|
| Mixed property types (apartments, houses, stores, land) | Filtered to `apartment` only |
| Listings from across Mexico, not just Mexico City | Filtered to `Distrito Federal` |
| Prices in MXN, USD, and ARS | Kept MXN listings only |
| Extreme price outliers | Trimmed top/bottom 10% |
| `lat-lon` combined into one string field | Split into numeric `lat`, `lon` |
| Borough buried inside a pipe-delimited location string | Extracted into a clean `borough` column |
| Extreme surface-area outliers | Trimmed top/bottom 10% |
| Mostly-empty or target-leaking columns (`floor`, `rooms`, `price_per_m2`, etc.) | Dropped |

Final dataset: **4,690 rows**, 5 usable columns (`price_aprox_usd`, `surface_covered_in_m2`,
`lat`, `lon`, `borough`), no missing values in the modeling features.

## 🤖 Modeling Results

Three simple `LinearRegression` models were trained on an 80/20 train/test split and compared
against a baseline that always predicts the mean training price:

| Model | R² (higher = better) | MAE (lower = better) |
|---|---|---|
| Baseline (mean price) | 0.000 | ~$54,200 |
| Size only | ~0.25 | ~$45,800 |
| Location only | ~0.13 | ~$49,100 |
| **Size + Location** | **~0.33** | **~$43,200** |

*(Exact values are in the notebook — they'll shift slightly if you re-run with a different
random seed or after re-downloading fresh data.)*

**Takeaway:** size explains roughly twice as much price variance as location does on its own,
but the combined model outperforms both — confirming that price is genuinely driven by more
than one factor, and a real pricing model shouldn't rely on just one.

## 🚀 How to Run This Yourself

```bash
git clone https://github.com/<your-username>/housing-in-mexico-city.git
cd housing-in-mexico-city
pip install -r requirements.txt
jupyter notebook notebooks/housing_in_mexico_city.ipynb
```

## 📁 Data Source

Listings data adapted from the [Properati](https://www.properati.com.mx/) Mexico real-estate
export, as used in WorldQuant University's Applied Data Science Lab coursework. This repo
contains an independently rebuilt version of that first project so I could keep a portfolio
copy — WQU's own platform doesn't allow exporting course notebooks.

## 🔭 Possible Next Steps

- Engineer a `price_per_m2` feature to compare boroughs on a normalized basis
- Try non-linear models (Random Forest, Gradient Boosting) to capture size × location interactions
- Add amenity-level features (transit access, schools, crime rates) for a richer model


*This is the first project in my Data Science / Machine Learning portfolio. More coming soon —
see my [GitHub profile](https://github.com/<your-username>) for the rest.*
