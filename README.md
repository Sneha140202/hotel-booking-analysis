# Hotel Booking Analysis

## Project Status

| Week | Focus | Status |
|------|-------|--------|
| **Week 1** | Data acquisition, cleaning & feature engineering | ✅ Complete |
| Week 2 | Exploratory Data Analysis (EDA) | 🔄 Up next |
| Week 3 | Cancellation prediction modelling | ⏳ Planned |
| Week 4 | Pricing optimisation & reporting | ⏳ Planned |

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
| **Source** | Antonio, Almeida & Nunes (2019) — [ScienceDirect](https://doi.org/10.1016/j.dib.2018.11.126) / TidyTuesday |
| **Raw rows** | 119,390 |
| **Raw columns** | 32 |
| **Cleaned rows** | 85,598 |
| **Final columns** | 36 (32 original + 4 engineered) |
| **Date range** | July 2015 – August 2017 |
| **Hotels** | Resort Hotel & City Hotel (Portugal) |
| **Target variable** | `is_canceled` (0 = stayed, 1 = cancelled) |
| **Key metric** | `adr` — Average Daily Rate (£) |

---

## Week 1 — Data Cleaning Summary

All cleaning was performed in `notebooks/02_data_cleaning.ipynb`.

### Missing Value Imputation

| Column | Missing Count | Strategy | Rationale |
|--------|-------------|----------|-----------|
| `children` | 4 | Fill → `0` | Absence of entry = no children |
| `agent` | 16,340 | Fill → `0` | `0` = direct booking (no agent) |
| `company` | 112,593 | Fill → `0` | `0` = non-corporate booking |
| `country` | 488 | Fill → mode (`PRT`) | Low missingness; mode imputation safe |

### Row Removal

| Reason | Rows Removed |
|--------|-------------|
| Invalid ADR (`adr ≤ 0`) | 1,960 |
| Exact duplicate rows | 31,832 |
| **Total removed** | **33,792 (28.3%)** |

### Outlier Treatment

| Column | Method | Threshold | Rows Capped |
|--------|--------|-----------|-------------|
| `adr` | IQR Winsorisation | Q3 + 1.5×IQR = £225.75 | 2,538 |

### Engineered Features

| Column | Formula | Purpose |
|--------|---------|---------|
| `arrival_date` | `year + month + day` → `datetime` | Time-series analysis, seasonality |
| `total_nights` | `weekend_nights + week_nights` | Stay duration; revenue multiplier |
| `total_guests` | `adults + children + babies` | Party size segmentation |
| `revenue_per_booking` | `adr × total_nights` | Booking value proxy |

**Final cleaned file:** `data/cleaned/hotel_cleaned.csv` — 85,598 rows × 36 columns, 0 nulls.

---

## Tech Stack

| Category | Tools / Libraries |
|----------|------------------|
| Language | Python 3.x |
| Data Manipulation | Pandas, NumPy |
| Visualisation | Matplotlib, Seaborn |
| Machine Learning | Scikit-Learn |
| BI & Dashboarding | Tableau / Power BI |
| Notebook Environment | Jupyter Notebook |
| Version Control | Git, GitHub |

---

## Project Structure

```
hotel-booking-analysis/
│
├── data/
│   ├── raw/              # Original, unprocessed datasets (gitignored)
│   └── cleaned/          # Cleaned & engineered datasets (gitignored — large files)
│
├── notebooks/
│   ├── 01_data_loading.ipynb       # ✅ Data loading & inspection
│   └── 02_data_cleaning.ipynb      # ✅ Full cleaning pipeline (Parts 1–4)
│
├── reports/              # Final analysis reports and summaries
├── visuals/              # Exported charts and plots
│
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
pip install pandas numpy matplotlib seaborn scikit-learn jupyter

# 4. Download the dataset and place in /data/raw/hotel_bookings.csv
#    Source: https://www.kaggle.com/datasets/jessemostipak/hotel-booking-demand

# 5. Launch Jupyter and run notebooks in order
jupyter notebook
```

---

## Author

**Sneha** — [GitHub: Sneha140202](https://github.com/Sneha140202)
