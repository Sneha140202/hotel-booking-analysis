# Hotel Booking Analysis

## Project Status

| Week | Focus | Status |
|------|-------|--------|
| **Week 1** | Data acquisition, cleaning & feature engineering | ✅ Complete |
| **Week 2** | Exploratory Data Analysis (EDA) & Segmentation | ✅ Complete |
| Week 3 | Cancellation prediction modelling | 🔄 Up next |
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

| Step | Action | Rows Affected |
|------|--------|--------------|
| Imputation | `children`, `agent`, `company`, `country` NaN filled | ~129K values |
| ADR removal | `adr ≤ 0` rows dropped | −1,960 |
| Deduplication | Exact duplicate rows removed | −31,832 |
| Outlier capping | ADR capped at Q3 + 1.5×IQR = £225.75 | 2,538 capped |
| Feature engineering | `arrival_date`, `total_nights`, `total_guests`, `revenue_per_booking` | +4 columns |

**Final cleaned file:** `data/cleaned/hotel_cleaned.csv` — 85,598 rows × 36 columns, 0 nulls.

---

## Week 2 — EDA Summary & Key Findings

Exploratory analysis across notebooks `03` through `06`.

### 🔍 Key Finding 1 — Non-Refundable Bookings Have a 94.7% Cancellation Rate

The single most counterintuitive result in the dataset: bookings marked as "Non Refund" deposit type cancel at **94.7%** — nearly 3.5× higher than the 27.0% rate for "No Deposit" bookings. The most likely explanation is an OTA (Booking.com) system classification artefact where certain modified or repriced bookings are flagged as cancellations internally while revenue is already settled. This finding means `deposit_type` must be treated with caution in modelling — it may represent data leakage rather than a genuine causal signal. See `notebooks/03_eda_cancellations.ipynb`, Part 3.

### 🔍 Key Finding 2 — Lead Time Is a Monotonic Cancellation Risk Amplifier

Cancellation rate rises steadily from **16.7%** (0–30 day bookings) to **40.0%** (180+ day bookings) — a 2.4× increase. Cancelled guests book an average of **35 days earlier** than non-cancelled guests (106 vs 71 days mean lead time). Combined with the ADR elasticity analysis (bookings made 91–150 days ahead pay the most at £113–115/night, while last-minute pays the least at £96), this creates a compounding risk: the highest-revenue advance bookings are also the most likely to cancel. See `notebooks/04_eda_pricing_seasonality.ipynb`.

### 🔍 Key Finding 3 — Early Planners Generate 88% More Revenue But Cancel at 37%

Customer segmentation reveals a critical trade-off: Early Planners (lead time > 90 days) generate £519 average revenue per booking vs £276 for Last-Minute bookers — but cancel at **37%** vs **17%**. Similarly, Leisure guests generate 165% more revenue than Corporate guests but cancel at 29% vs 13%. The optimal revenue strategy is a tiered deposit and pricing structure that captures Early Planner commitment while preserving Last-Minute premium pricing. See `notebooks/06_customer_segmentation.ipynb`.

---

### EDA Notebooks

| Notebook | Topic | Key Output |
|----------|-------|-----------|
| `03_eda_cancellations.ipynb` | Cancellation rates, deposit type, customer type, lead time | 7 charts |
| `04_eda_pricing_seasonality.ipynb` | Monthly ADR, seasonal patterns, booking curve, pricing elasticity | 6 charts |
| `05_correlation_analysis.ipynb` | Feature correlations, multicollinearity | 3 charts |
| `06_customer_segmentation.ipynb` | Corporate/Leisure + Early/Last-Minute KPI comparison | 2 charts |

### Visuals Gallery

All charts exported to `/visuals/` with `plt.savefig()`:

| File | Description |
|------|-------------|
| `cancellation_overall_pie.png` | Overall 27.85% cancellation rate donut |
| `cancellation_by_hotel_type.png` | City Hotel (30.5%) vs Resort Hotel (23.8%) |
| `cancellation_by_deposit_type.png` | Non Refund 94.7% counterintuitive finding |
| `cancellation_by_customer_type.png` | Transient 30.5% vs Group 7.2% |
| `lead_time_distribution.png` | Lead time histogram: cancelled vs not cancelled |
| `lead_time_bucket_cancellation.png` | Cancellation rate per 0–30 / 31–90 / 91–180 / 180+ buckets |
| `monthly_adr_vs_volume.png` | Dual-axis ADR + booking volume by month |
| `adr_heatmap_hotel_month.png` | ADR by hotel type × month heatmap |
| `monthly_cancellation_rate.png` | Seasonal cancellation rate bar chart |
| `booking_curve_by_hotel.png` | Lead time booking curve: City vs Resort |
| `adr_vs_lead_time.png` | ADR elasticity across lead time buckets |
| `correlation_heatmap_full.png` | Full 20×20 Pearson correlation matrix |
| `segment_kpi_comparison.png` | Corporate/Leisure + Early/Last-Minute KPI bars |

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
│   ├── raw/              # Original dataset (gitignored)
│   └── cleaned/          # hotel_cleaned.csv — 85,598 × 36, 0 nulls (gitignored)
│
├── notebooks/
│   ├── 01_data_loading.ipynb            ✅ Data loading & inspection
│   ├── 02_data_cleaning.ipynb           ✅ Full cleaning pipeline (Parts 1–4)
│   ├── 03_eda_cancellations.ipynb       ✅ Cancellation EDA (Parts 1–3)
│   ├── 04_eda_pricing_seasonality.ipynb ✅ Pricing & seasonality (Parts 1–2)
│   ├── 05_correlation_analysis.ipynb    ✅ Feature correlations & selection
│   └── 06_customer_segmentation.ipynb  ✅ Segment KPI comparison
│
├── reports/              # Final analysis reports
├── visuals/              # 13+ exported charts (plt.savefig)
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

# 5. Run notebooks in order (01 → 06)
jupyter notebook
```

---

## Author

**Sneha** — [GitHub: Sneha140202](https://github.com/Sneha140202)
