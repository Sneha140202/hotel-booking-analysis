# Hotel Booking Analysis

## Project Status

| Week | Focus | Status |
|------|-------|--------|
| **Week 1** | Data acquisition, cleaning & feature engineering | ✅ Complete |
| **Week 2** | Exploratory Data Analysis (EDA) & Segmentation | ✅ Complete |
| **Week 3** | ML Modelling — Cancellation Prediction | ✅ Complete |
| Week 4 | Dashboard & Reporting | 🔄 Up next |

---

## Business Problem Statement

### 1. Cancellation Prediction
Hotel cancellations result in significant revenue loss and operational inefficiencies. This project identifies the key drivers behind booking cancellations and builds a predictive model to flag high-risk bookings early — enabling hotels to implement targeted retention strategies, adjust overbooking policies, and reduce last-minute revenue leakage.

### 2. Pricing Optimisation
Dynamic pricing is critical in the hospitality industry. This project analyses booking patterns, seasonal trends, lead times, and guest segments to uncover pricing opportunities — helping hotels maximise Average Daily Rate (ADR) and Revenue Per Available Room (RevPAR) while staying competitive.

---

## Dataset

| Attribute | Detail |
|-----------|--------|
| **Name** | Hotel Booking Demand |
| **Source** | Antonio, Almeida & Nunes (2019) — [ScienceDirect](https://doi.org/10.1016/j.dib.2018.11.126) |
| **Raw rows** | 119,390 |
| **Cleaned rows** | 85,598 |
| **Final columns** | 36 (32 original + 4 engineered) |
| **Target variable** | `is_canceled` (0 = stayed, 1 = cancelled) |
| **Key metric** | `adr` — Average Daily Rate (£) |

---

## Week 3 — Model Performance Summary

All models trained on SMOTE-balanced data (98,814 samples, 50:50) and evaluated on the real-world test set (17,120 samples, 72:28 split).

### Model Results Table

| Model | Accuracy | Precision (cl.1) | Recall (cl.1) | F1 (cl.1) | **ROC-AUC** | False Negatives |
|-------|----------|-----------------|---------------|-----------|------------|-----------------|
| Logistic Regression | 73.85% | 52.51% | 63.76% | 57.59% | 0.7852 | 1,728 |
| Decision Tree (depth=6) | 71.51% | 49.18% | 68.23% | 57.16% | 0.7862 | 1,515 |
| **Random Forest (tuned)** | **79.07%** | **62.65%** | 61.49% | **62.07%** | **0.8333** ✅ | 1,836 |

### Selected Model: Random Forest (tuned)

**Why Random Forest?**
- **Highest ROC-AUC (0.8333)** — strongest overall ranking ability across all decision thresholds
- **Highest F1-Score (62.07%)** — best balance of precision and recall for production use
- **Fewest false positives (1,748)** — minimises unnecessary retention interventions
- **Best accuracy (79.07%)** — lowest overall error rate
- Fully tunable threshold: lowering from 0.5 → 0.35 increases recall for high-stakes peak-season periods

**Best hyperparameters** (found via `RandomizedSearchCV`, cv=5):
`max_depth=20`, `min_samples_split=5`, `n_estimators=50`, `max_features='sqrt'`

### Top 5 Cancellation Predictors (Random Forest)

| Rank | Feature | Importance | Meaning |
|------|---------|-----------|---------|
| 1 | `lead_time` | 20.87% | Longer advance bookings cancel more |
| 2 | `revenue_per_booking` | 12.67% | Higher-value bookings carry more risk |
| 3 | `adr` | 12.66% | Pricing tier signal |
| 4 | `total_of_special_requests` | 8.81% | Requests = guest commitment |
| 5 | `arrival_date_month` | 6.58% | Seasonal cancellation patterns |

### Why Recall Matters More Than Accuracy

A dummy model predicting "not cancelled" every time achieves **72% accuracy** — completely useless. Each missed cancellation (False Negative) means:
- An empty room with no overbooking buffer
- No retention action triggered
- Revenue permanently lost (~£403 avg per booking)
- At scale: **~£3.7M in unprotected revenue** per 2-year period

ROC-AUC is the primary evaluation metric — it measures ranking ability across all thresholds rather than at a fixed 0.5 cut-off, making it robust to class imbalance.

---

## Week 1 — Data Cleaning Summary

| Step | Action | Impact |
|------|--------|--------|
| Imputation | `children`, `agent`, `company`, `country` NaN filled | ~129K values |
| ADR removal | `adr ≤ 0` rows dropped | −1,960 rows |
| Deduplication | Exact duplicate rows removed | −31,832 rows |
| Outlier capping | ADR capped at Q3 + 1.5×IQR = £225.75 | 2,538 capped |
| Feature engineering | `arrival_date`, `total_nights`, `total_guests`, `revenue_per_booking` | +4 columns |

---

## Week 2 — Key EDA Findings

1. **Non-Refundable bookings have a 94.7% cancellation rate** — likely an OTA system artefact, not genuine causal signal. Treat with caution in modelling.
2. **Lead time is a monotonic cancellation amplifier** — rate rises from 16.7% (0–30 days) to 40.0% (180+ days). Cancelled guests book 35 days earlier on average.
3. **Early Planners generate 88% more revenue but cancel at 37%** — vs 17% for Last-Minute bookers. Tiered deposit strategy recommended.

---

## Tech Stack

| Category | Tools |
|----------|-------|
| Language | Python 3.x |
| Data Manipulation | Pandas, NumPy |
| Visualisation | Matplotlib, Seaborn |
| Machine Learning | Scikit-Learn, imbalanced-learn (SMOTE) |
| BI & Dashboarding | Tableau / Power BI (Week 4) |
| Notebook Environment | Jupyter Notebook |
| Version Control | Git, GitHub |

---

## Project Structure

```
hotel-booking-analysis/
│
├── data/
│   ├── raw/              # Original dataset (gitignored)
│   └── cleaned/          # Cleaned dataset + model pickles (gitignored)
│
├── notebooks/
│   ├── 01_data_loading.ipynb            ✅ Data loading & inspection
│   ├── 02_data_cleaning.ipynb           ✅ Full cleaning pipeline
│   ├── 02_missing_values.ipynb          ✅ Missing value analysis
│   ├── 03_eda_cancellations.ipynb       ✅ Cancellation EDA
│   ├── 04_eda_pricing_seasonality.ipynb ✅ Pricing & seasonality
│   ├── 05_correlation_analysis.ipynb    ✅ Feature correlations
│   ├── 06_customer_segmentation.ipynb  ✅ Segment KPI comparison
│   ├── 07_feature_engineering.ipynb    ✅ Feature prep & SMOTE
│   ├── 08_logistic_regression.ipynb    ✅ LR baseline (AUC=0.7852)
│   ├── 09_decision_tree.ipynb          ✅ Decision Tree (AUC=0.7862)
│   ├── 10_random_forest.ipynb          ✅ Random Forest (AUC=0.8333)
│   └── 11_model_comparison.ipynb       ✅ Final model selection
│
├── reports/
│   └── model_comparison_results.csv    ← Model metrics table
│
├── visuals/              # 20+ exported charts (plt.savefig)
├── .gitignore
└── README.md
```

---

## How to Run

```bash
# 1. Clone the repository
git clone https://github.com/Sneha140202/hotel-booking-analysis.git
cd hotel-booking-analysis

# 2. Create and activate a virtual environment
python -m venv venv
source venv/bin/activate        # On Windows: venv\Scripts\activate

# 3. Install dependencies
pip install pandas numpy matplotlib seaborn scikit-learn imbalanced-learn jupyter nbstripout

# 4. Download the dataset and place in /data/raw/hotel_bookings.csv
#    Source: https://www.kaggle.com/datasets/jessemostipak/hotel-booking-demand

# 5. Run notebooks in order (01 → 11)
jupyter notebook
```

> **Note:** All model pickle files and large CSVs are gitignored. Re-run `07_feature_engineering.ipynb` first to regenerate `X_features.pkl`, `y_target.pkl`, and the train/test split files before running notebooks 08–11.

---

## Author

**Sneha** — [GitHub: Sneha140202](https://github.com/Sneha140202)
