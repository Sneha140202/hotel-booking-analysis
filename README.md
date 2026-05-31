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
Over **27.85% of bookings cancel**, putting ~£3.7M in revenue at risk per reporting
cycle. This project predicts which bookings are high-risk so hotels can trigger
re-confirmation campaigns, set overbooking buffers, and recover lost revenue.

### 2. Pricing Optimisation
Using ADR, lead time, and seasonal patterns, the analysis identifies pricing gaps —
including a £370K opportunity from under-priced May demand.

---

## Dataset

| Attribute | Value |
|-----------|-------|
| **Source** | Antonio, Almeida & Nunes (2019) — [ScienceDirect](https://doi.org/10.1016/j.dib.2018.11.126) |
| **Raw rows** | 119,390 |
| **Cleaned rows** | 85,598 |
| **Final features** | 34 (after encoding & engineering) |
| **Target** | `is_canceled` — 0 = stayed, 1 = cancelled |
| **Hotels** | City Hotel & Resort Hotel, Portugal (Jul 2015–Aug 2017) |

---

## Week 3 — Model Performance

All models trained on **SMOTE-balanced** data (98,814 samples, 50:50) and evaluated
on the **real test set** (17,120 samples, original 72:28 distribution).

### Model Results

| Model | Accuracy | Precision | Recall | F1-Score | **ROC-AUC** | False Negatives |
|-------|----------|-----------|--------|----------|------------|-----------------|
| [Logistic Regression](notebooks/08_logistic_regression.ipynb) | 73.85% | 52.51% | 63.76% | 57.59% | 0.7852 | 1,728 |
| [Decision Tree (depth=6)](notebooks/09_decision_tree.ipynb) | 71.51% | 49.18% | 68.23% | 57.16% | 0.7862 | 1,515 |
| **[Random Forest (tuned)](notebooks/10_random_forest.ipynb)** | **79.07%** | **62.65%** | 61.49% | **62.07%** | **0.8333 ✅** | 1,836 |

### Selected Model: Random Forest (tuned)

**Best hyperparameters** (RandomizedSearchCV, cv=5):
`n_estimators=50` · `max_depth=20` · `min_samples_split=5` · `max_features='sqrt'`

**Why RF over Decision Tree despite lower recall?**
The DT catches ~320 more cancellations but generates **1,614 more false positives**
(3,362 vs 1,748) — wasting ~£16K/cycle on unnecessary retention actions.
The RF's higher AUC (0.8333) allows threshold tuning: lowering from 0.50 → 0.40
recovers recall for peak-season periods without permanently hurting precision.

### Top 5 Cancellation Predictors (RF Feature Importance)

| Rank | Feature | Importance | Business Meaning |
|------|---------|-----------|-----------------|
| 1 | `lead_time` | 20.87% | Longer advance bookings cancel more |
| 2 | `revenue_per_booking` | 12.67% | High-value bookings carry more risk |
| 3 | `adr` | 12.66% | Pricing tier signal |
| 4 | `total_of_special_requests` | 8.81% | Requests = guest commitment |
| 5 | `arrival_date_month` | 6.58% | Seasonal patterns (summer = higher risk) |

### Why Recall > Accuracy

A dummy model predicting "not cancelled" always achieves **72.15% accuracy** —
useless for revenue management. The real cost:

| Error | Cost |
|-------|------|
| **False Negative** (missed cancellation) | **~£403** per booking — empty room, no overbooking buffer, revenue permanently lost |
| **False Positive** (wrong flag) | **~£5–15** intervention cost |

**27:1 cost ratio** — catching cancellations is overwhelmingly more valuable.
**ROC-AUC** is the primary metric because it evaluates ranking ability across
all thresholds, not just the default 0.5 cut-off.

---

## Week 2 — Key EDA Findings

1. **Non-Refundable deposits → 94.7% cancel rate** — likely OTA artefact, not causal
2. **Lead time is a monotonic risk amplifier** — 16.7% (0–30d) → 40.0% (180+d)
3. **Early Planners (>90d) generate 88% more revenue but cancel 37% of the time**

---

## Week 1 — Cleaning Summary

| Step | Action | Impact |
|------|--------|--------|
| Imputation | `children`, `agent`, `company`, `country` NaN filled | ~129K values |
| ADR removal | `adr ≤ 0` dropped | −1,960 rows |
| Deduplication | Exact duplicates removed | −31,832 rows |
| Outlier capping | ADR capped at Q3+1.5×IQR = £225.75 | 2,538 capped |
| Feature engineering | `arrival_date`, `total_nights`, `total_guests`, `revenue_per_booking` | +4 cols |

---

## Notebooks

| # | Notebook | Purpose |
|---|----------|---------|
| 01 | [data_loading](notebooks/01_data_loading.ipynb) | Load & inspect raw dataset |
| 02a | [data_cleaning](notebooks/02_data_cleaning.ipynb) | Full cleaning pipeline → `hotel_cleaned.csv` |
| 02b | [missing_values](notebooks/02_missing_values.ipynb) | Missing value analysis & strategy |
| 03 | [eda_cancellations](notebooks/03_eda_cancellations.ipynb) | Cancellation EDA by hotel, deposit, segment, lead time |
| 04 | [eda_pricing_seasonality](notebooks/04_eda_pricing_seasonality.ipynb) | Monthly ADR, booking curve, pricing elasticity |
| 05 | [correlation_analysis](notebooks/05_correlation_analysis.ipynb) | Feature correlations & multicollinearity |
| 06 | [customer_segmentation](notebooks/06_customer_segmentation.ipynb) | Corporate/Leisure + Early/Last-Minute KPIs |
| 07 | [feature_engineering](notebooks/07_feature_engineering.ipynb) | Encoding, SMOTE, train/test split |
| 08 | [logistic_regression](notebooks/08_logistic_regression.ipynb) | LR baseline — AUC=0.7852 |
| 09 | [decision_tree](notebooks/09_decision_tree.ipynb) | Decision Tree — AUC=0.7862 |
| 10 | [random_forest](notebooks/10_random_forest.ipynb) | **RF (tuned) — AUC=0.8333 ✅** |
| 11 | [model_comparison](notebooks/11_model_comparison.ipynb) | Final comparison + business justification |

---

## Tech Stack

| Category | Tools |
|----------|-------|
| Language | Python 3.x |
| Data | Pandas, NumPy |
| Visualisation | Matplotlib, Seaborn |
| ML | Scikit-Learn, imbalanced-learn (SMOTE) |
| BI | Tableau / Power BI (Week 4) |
| Notebooks | Jupyter — outputs stripped via `nbstripout` |
| Version Control | Git, GitHub |

---

## How to Run

```bash
git clone https://github.com/Sneha140202/hotel-booking-analysis.git
cd hotel-booking-analysis
python -m venv venv && source venv/bin/activate
pip install pandas numpy matplotlib seaborn scikit-learn imbalanced-learn jupyter nbstripout
# Place hotel_bookings.csv in data/raw/
# Source: https://www.kaggle.com/datasets/jessemostipak/hotel-booking-demand
jupyter notebook   # run in order: 01 → 11
```

> **Note:** Model pickles (`.pkl`) are gitignored. Run `07_feature_engineering.ipynb`
> first to regenerate all train/test/SMOTE arrays before running notebooks 08–11.

---

## Author

**Sneha** — [GitHub: Sneha140202](https://github.com/Sneha140202)

