# Hotel Booking Analysis

## Project Status

| Week | Focus | Status |
|------|-------|--------|
| **Week 1** | Data acquisition, cleaning & feature engineering | ✅ Complete |
| **Week 2** | Exploratory Data Analysis (EDA) & Segmentation | ✅ Complete |
| **Week 3** | ML Modelling — Cancellation Prediction | ✅ Complete |
| Week 4 | Dashboard & Reporting | 🔄 Up next |

---

## Business Problem

### 1. Cancellation Prediction
Over **27.85% of bookings cancel**, representing ~£3.7M in unprotected revenue per reporting cycle. This project predicts which bookings are at risk so hotels can trigger re-confirmation campaigns, set overbooking buffers, and reduce last-minute revenue loss.

### 2. Pricing Optimisation
Using ADR, lead time, and seasonal patterns, the analysis surfaces pricing gaps — including an identified £370K opportunity in May from under-priced peak-demand periods.

---

## Dataset

| Attribute | Value |
|-----------|-------|
| **Source** | Antonio, Almeida & Nunes (2019) — [ScienceDirect](https://doi.org/10.1016/j.dib.2018.11.126) |
| **Raw rows** | 119,390 |
| **Cleaned rows** | 85,598 |
| **Features (final)** | 34 (after encoding & engineering) |
| **Target** | `is_canceled` — 0 = stayed, 1 = cancelled |
| **Hotels** | City Hotel & Resort Hotel, Portugal (Jul 2015 – Aug 2017) |

---

## Week 3 — Model Performance

All models trained on **SMOTE-balanced** data (98,814 samples, 50:50) and evaluated on the **original test set** (17,120 samples, real 72:28 distribution).

### Results Table

| Model | Accuracy | Precision | Recall | F1-Score | **ROC-AUC** | False Negatives |
|-------|----------|-----------|--------|----------|------------|-----------------|
| [Logistic Regression](notebooks/08_logistic_regression.ipynb) | 73.85% | 52.51% | 63.76% | 57.59% | 0.7852 | 1,728 |
| [Decision Tree (depth=6)](notebooks/09_decision_tree.ipynb) | 71.51% | 49.18% | 68.23% | 57.16% | 0.7862 | 1,515 |
| **[Random Forest (tuned)](notebooks/10_random_forest.ipynb)** | **79.07%** | **62.65%** | 61.49% | **62.07%** | **0.8333 ✅** | 1,836 |

### Selected Model: Random Forest (tuned)

- **Highest ROC-AUC (0.8333)** — best ranking ability across all operating thresholds
- **Best accuracy (79.07%)** and **F1-Score (62.07%)**
- **Fewest false positives (1,748)** — saves ~£16K/cycle vs Decision Tree on wasted retention actions
- **Tunable threshold** — lowering from 0.50 → 0.40 recovers high recall for peak season

**Best hyperparameters** (RandomizedSearchCV, cv=5): `n_estimators=50`, `max_depth=20`, `min_samples_split=5`, `max_features='sqrt'`

### Top 5 Cancellation Predictors

| Rank | Feature | Importance | Meaning |
|------|---------|-----------|---------|
| 1 | `lead_time` | 20.87% | Long advance bookings cancel more |
| 2 | `revenue_per_booking` | 12.67% | High-value bookings carry more risk |
| 3 | `adr` | 12.66% | Pricing tier signal |
| 4 | `total_of_special_requests` | 8.81% | Requests = commitment signal |
| 5 | `arrival_date_month` | 6.58% | Seasonal cancellation patterns |

### Why Recall > Accuracy

A dummy model predicting "not cancelled" always achieves **72% accuracy** — useless.

| Error | Cost |
|-------|------|
| **False Negative** (missed cancel) | ~£403 avg revenue lost permanently |
| **False Positive** (wrong flag) | ~£5–15 intervention cost |

Cost ratio **27:1** — catching cancellations is overwhelmingly more valuable than avoiding false alarms. **ROC-AUC** is the primary metric because it evaluates ranking ability across all thresholds, not just the default 0.5 cut-off.

---

## Week 2 — Key EDA Findings

1. **Non-Refundable deposits → 94.7% cancellation rate** — OTA system artefact, not causal; treat with caution in modelling
2. **Lead time amplifies risk monotonically** — 16.7% (0–30 days) → 40.0% (180+ days)
3. **Early Planners generate 88% more revenue but cancel 37% of the time** — vs 17% for last-minute bookers; tiered deposit policy recommended

---

## Week 1 — Cleaning Summary

| Step | Action | Impact |
|------|--------|--------|
| Imputation | `children`, `agent`, `company`, `country` NaN → filled | ~129K values |
| ADR removal | `adr ≤ 0` dropped | −1,960 rows |
| Deduplication | Exact duplicates removed | −31,832 rows |
| Outlier capping | ADR capped at Q3 + 1.5×IQR = £225.75 | 2,538 capped |
| Engineering | `arrival_date`, `total_nights`, `total_guests`, `revenue_per_booking` | +4 columns |

---

## Notebooks

| Notebook | Purpose | Key Output |
|----------|---------|-----------|
| [01_data_loading](notebooks/01_data_loading.ipynb) | Load & inspect raw data | Shape, dtypes, column definitions |
| [02_data_cleaning](notebooks/02_data_cleaning.ipynb) | Full cleaning pipeline | `hotel_cleaned.csv` — 85,598 × 36 |
| [02_missing_values](notebooks/02_missing_values.ipynb) | Missing value analysis | Imputation strategy |
| [03_eda_cancellations](notebooks/03_eda_cancellations.ipynb) | Cancellation EDA | Rate by hotel, deposit, customer type, lead time |
| [04_eda_pricing_seasonality](notebooks/04_eda_pricing_seasonality.ipynb) | Pricing & seasonality | Monthly ADR, booking curve, elasticity |
| [05_correlation_analysis](notebooks/05_correlation_analysis.ipynb) | Feature correlations | Multicollinearity flags, feature rankings |
| [06_customer_segmentation](notebooks/06_customer_segmentation.ipynb) | Segment KPIs | Corporate/Leisure + Early/Last-Minute |
| [07_feature_engineering](notebooks/07_feature_engineering.ipynb) | Encoding & SMOTE | `X_features.pkl`, train/test split |
| [08_logistic_regression](notebooks/08_logistic_regression.ipynb) | LR baseline | AUC=0.7852, Recall=63.76% |
| [09_decision_tree](notebooks/09_decision_tree.ipynb) | Decision Tree | AUC=0.7862, Recall=68.23% |
| [10_random_forest](notebooks/10_random_forest.ipynb) | **Random Forest ✅** | **AUC=0.8333**, F1=62.07% |
| [11_model_comparison](notebooks/11_model_comparison.ipynb) | Final comparison | Model selection + business justification |

---

## Tech Stack

| Category | Tools |
|----------|-------|
| Language | Python 3.x |
| Data Manipulation | Pandas, NumPy |
| Visualisation | Matplotlib, Seaborn |
| Machine Learning | Scikit-Learn, imbalanced-learn (SMOTE) |
| BI / Dashboarding | Tableau / Power BI (Week 4) |
| Notebooks | Jupyter (outputs stripped via nbstripout) |
| Version Control | Git, GitHub |

---

## How to Run

```bash
# 1. Clone
git clone https://github.com/Sneha140202/hotel-booking-analysis.git
cd hotel-booking-analysis

# 2. Virtual environment
python -m venv venv && source venv/bin/activate

# 3. Install dependencies
pip install pandas numpy matplotlib seaborn scikit-learn imbalanced-learn jupyter nbstripout

# 4. Place raw data at data/raw/hotel_bookings.csv
#    Download: https://www.kaggle.com/datasets/jessemostipak/hotel-booking-demand

# 5. Run in order (01 → 11)
jupyter notebook
```

> **Note:** Model pickles and large CSVs are gitignored. Run `07_feature_engineering.ipynb`
> first to regenerate `X_features.pkl` and all train/test split files before running 08–11.

---

## Author

**Sneha** — [GitHub: Sneha140202](https://github.com/Sneha140202)
