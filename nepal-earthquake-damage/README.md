# 🏚️ Earthquake Damage in Nepal

A binary classification project predicting which buildings suffered severe structural damage
in the 2015 Nepal earthquake — data is pulled and joined from a **SQLite database using SQL**,
then compared across **Logistic Regression**, **Decision Tree**, and **Random Forest**
classifiers.

![Python](https://img.shields.io/badge/Python-3.11-blue?logo=python&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-SQLite-003B57?logo=sqlite&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-data%20wrangling-150458?logo=pandas&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-modeling-F7931E?logo=scikitlearn&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)

## 📌 Project Overview

The 2015 Gorkha earthquake damaged hundreds of thousands of buildings across Nepal. This
project builds a full pipeline that:

1. **Loads the raw survey data into a normalized SQLite database** (two tables —
   `building_structure` and `building_damage` — linked by `building_id`), then uses actual
   SQL (`SELECT`, `WHERE`, `JOIN`, `GROUP BY`, `DISTINCT`, `COUNT`) to explore and extract
   the data — no shortcuts through pandas for the data-access layer
2. Cleans the SQL output and engineers a binary `severe_damage` target from the original
   5-level damage grade
3. Explores which building characteristics (roof type, foundation type, footprint size)
   visibly track with damage
4. Trains and compares **three classification models** — Logistic Regression, Decision Tree,
   and Random Forest — each in its own notebook
5. Surfaces feature importance / odds ratios from each model to see which building attributes
   actually drive the predictions

## 🗂️ Repo Structure

```
nepal-earthquake-damage/
├── data/
│   ├── nepal.sqlite                       # SQLite database (building_structure + building_damage tables)
│   ├── nepal-wrangled.csv                 # output of the SQL notebook's wrangle() function
│   ├── X_train.csv / X_test.csv / y_train.csv / y_test.csv  # shared split across all models
│   └── *_results.txt                      # test accuracy from each model, used by comparison notebook
├── notebooks/
│   ├── 1_sql_exploration.ipynb            # SQL: connect, explore schema, join tables, wrangle
│   ├── 2_eda.ipynb                        # exploratory data analysis + train/test split
│   ├── 3_model_logistic_regression.ipynb  # Model 1: Logistic Regression
│   ├── 4_model_decision_tree.ipynb        # Model 2: Decision Tree
│   ├── 5_model_random_forest.ipynb        # Model 3: Random Forest
│   └── 6_model_comparison.ipynb           # side-by-side comparison of all 3 models
├── images/                                # chart exports
├── requirements.txt
├── LICENSE
└── README.md
```

**Note on the database:** the original nationwide survey covers ~762K buildings across 11
districts — turning all of it into a SQLite file lands north of GitHub's 100MB limit. The
database shipped here keeps the target district (24, the largest) plus 3 smaller ones (12,
21, 29), which is enough to genuinely demonstrate multi-district SQL exploration
(`DISTINCT district_id`, `GROUP BY` counts) without bloating the repo.

## 🗃️ The SQL Layer

`1_sql_exploration.ipynb` is where the actual database work happens — this isn't pandas
dressed up to look like SQL, it's real queries doing real filtering and joining:

```sql
-- how many buildings per district?
SELECT district_id, COUNT(*) AS n_buildings
FROM building_structure
GROUP BY district_id
ORDER BY n_buildings DESC

-- join structure + damage records for the target district
SELECT s.*, d.damage_grade, d.count_floors_post_eq, d.height_ft_post_eq, d.condition_post_eq
FROM building_structure AS s
JOIN building_damage AS d ON s.building_id = d.building_id
WHERE s.district_id = 24
```

The two-table schema (`building_structure` holds pre-earthquake building attributes,
`building_damage` holds the post-earthquake outcome) mirrors how a real damage-assessment
database would actually be organized, so the join isn't just there to justify using SQL —
it's a natural consequence of the data actually living in two related tables.

## 🧹 Data Cleaning Summary

| Issue in raw data | How it was handled |
|---|---|
| Nationwide data spanning many districts, split across two logical tables (structure vs. damage) | Loaded into SQLite, joined via SQL `WHERE`/`JOIN`, filtered to the largest district |
| Target (`damage_grade`) has 5 levels, not binary | Engineered `severe_damage` = 1 if Grade 4 or 5, else 0 |
| Several columns describe the building *after* the earthquake (`count_floors_post_eq`, `height_ft_post_eq`, `condition_post_eq`) | Dropped — using these to predict damage would be leakage, since they wouldn't be known in advance |
| `count_floors_pre_eq` and `height_ft_pre_eq` are correlated at 0.75 | Dropped `count_floors_pre_eq` to avoid multicollinearity in the linear model |
| A handful of rows missing `position` / `plan_configuration` | Dropped |

Final dataset: **~98,000 rows** for the chosen district, with 7 categorical building-structure
features (roof type, foundation type, etc.) plus building age, footprint area, and
pre-earthquake height.

## 🤖 Why Three Models?

| Model | Encoding used | Notebook |
|---|---|---|
| **Logistic Regression** | One-hot encoding | `2_model_logistic_regression.ipynb` |
| **Decision Tree** | Ordinal encoding, `max_depth` tuned via validation curve | `3_model_decision_tree.ipynb` |
| **Random Forest** | Ordinal encoding, `n_estimators` tuned via validation curve | `4_model_random_forest.ipynb` |

Logistic Regression gives an interpretable linear baseline (odds ratios per feature); Decision
Tree adds non-linear splits and interaction effects; Random Forest checks whether averaging
over many trees actually buys anything over a single well-tuned one.

## 📊 Results

| Model | Test Accuracy |
|---|---|
| Baseline (majority class) | 0.54 |
| Logistic Regression | ~0.650 |
| **Decision Tree** | **~0.663** |
| Random Forest | ~0.651 |

*(Exact values are in `5_model_comparison.ipynb`.)*

All three models clear the baseline comfortably but land in a tight band — none dramatically
outperforms the others. Decision Tree edges out the other two slightly. This is a fairly
common outcome for tabular data like this: once a model captures the handful of features that
actually carry signal (roof type, foundation type, footprint size), extra model complexity
doesn't buy much more. The likely ceiling is the data itself — two buildings with identical
recorded features can still end up with different damage grades due to factors this dataset
doesn't capture (soil conditions, exact distance to the epicenter, undocumented construction
quality).

**Final model: Decision Tree** — matched or beat the more complex Random Forest while staying
easy to visualize and explain to a non-technical audience.

## 🚀 How to Run This Yourself

```bash
git clone https://github.com/<your-username>/nepal-earthquake-damage.git
cd nepal-earthquake-damage
pip install -r requirements.txt
jupyter notebook notebooks/1_sql_exploration.ipynb
```

Run notebooks in numeric order — `1_sql_exploration.ipynb` queries `nepal.sqlite` and writes
`data/nepal-wrangled.csv`, which `2_eda.ipynb` picks up to generate the train/test split every
model notebook depends on, and each model notebook writes its own results file that
`6_model_comparison.ipynb` reads at the end.

## 📁 Data Source

Building-survey data adapted from the 2015 Nepal earthquake open dataset. This repo is an
independently built analysis inspired by coursework I completed on SQL, classification, and
data ethics — rebuilt from scratch here (own database schema, own queries, own wrangle logic)
so I could keep a portfolio copy of the workflow and reasoning.

## ⚖️ A Note on Data Ethics

The original coursework this project is inspired by specifically explored how features like
caste and income — present in the fuller household-level version of this dataset — can
introduce discriminatory bias into a damage-prediction model, even when they aren't the
"intended" predictors. This repo's feature set is limited to physical building
characteristics for that reason, but it's worth explicitly noting: any model trained on
disaster-response data should be audited for whether it's encoding proxies for protected
characteristics (e.g., neighborhood or building-material choices that correlate with income
or ethnicity) before being used to prioritize real-world aid or resources.

## 🔭 Possible Next Steps

- Bring back household demographic data and explicitly test the model for proxy
  discrimination before treating it as production-ready
- Try gradient boosting (XGBoost / LightGBM) as a fourth comparison point
- Extend the analysis across multiple districts instead of just one

## 📄 License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

---

*Fourth project in my Data Science / Machine Learning portfolio — see my
[GitHub profile](https://github.com/<your-username>) for the rest.*
