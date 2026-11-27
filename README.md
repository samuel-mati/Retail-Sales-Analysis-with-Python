# Kakuma Fresh Mart: Retail Sales Analysis

**Cleaning and analysing three weeks of messy point-of-sale data with Python and pandas to find where revenue comes from, what drives it, and when the business sells most.**

![Python](https://img.shields.io/badge/Python-3-blue) ![pandas](https://img.shields.io/badge/pandas-data%20analysis-150458) ![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange)

---

## Executive Summary

Kakuma Fresh Mart is a retailer with three branches in north-west Kenya: **Kakuma Town, Lodwar and Kitale**. I took its raw sales export for **1–21 October 2026** (2,509 rows, with inconsistent labels, mixed date formats, missing and impossible values), cleaned it into a dataset ready for analysis, and answered four business questions.

**Bottom line:** no branch is underperforming. Revenue is concentrated in a few products and at the weekend, so staffing and stock should follow those.

| KPI | Value |
|---|---|
| Net revenue | **KES 949,214** |
| Sales lines analysed | **2,464** |
| Average revenue per sales line | **~KES 385** |
| Weekend vs weekday revenue per day | **+41%** |
| Revenue share from top 5 of 19 products | **54%** |

---

## Business Problem

Management had sales data but no clear picture of performance. The analysis set out to answer:

1. **Where does revenue come from?** Which branch brings in the most revenue, and is that because customers spend more or because there are more sales?
2. **What drives revenue?** Which categories and products carry the business, measured by share of revenue?
3. **Do customers behave differently?** Do returning (loyalty) customers spend more than other shoppers, and how do customers pay?
4. **When do we sell most?** Are there day-of-week patterns, and do they hold in every branch?

---

## Dataset

| Attribute | Detail |
|---|---|
| Source file | `kakuma_fresh_mart_sales.csv` (raw POS export) |
| Period | 1–21 October 2026 (3 weeks) |
| Raw size | 2,509 rows × 9 columns |
| Columns | `receipt`, `date`, `branch`, `product`, `category`, `qty`, `price`, `payment`, `customer` |
| Coverage | 3 branches, 6 categories, 19 products |

Each row is a **sales line** (one product on a receipt), not a customer visit.

---

## Methodology

The project follows three stages, each in its own notebook.

### 1. Explore: `Exploring_Data_with_Pandas.ipynb`
I profiled the raw data to find quality issues before changing anything:

- **Missing values:** 12 quantities, 44 prices, 61 payment methods, 1,394 customer IDs
- **Wrong data types:** `date`, `qty` and `price` were not stored as dates and numbers
- **30 exact duplicate rows**
- **Inconsistent labels:** 12 spellings of 3 branch names, 24 variants of 6 categories, 4 spellings of M-Pesa
- **Mixed date formats:** ISO (`2026-10-01`) and 199 day-first slash dates (`01/10/2026`)
- **Invalid values:** price recorded as `"TBC"` (4 rows) or `0` (5 rows), negative quantities (returns), and extreme quantities of 999, 500 and 250

### 2. Clean: `Cleaning_Data_with_Pandas.ipynb`
I worked on a copy of the raw data and **flagged** questionable records rather than deleting them:

| Issue | Treatment |
|---|---|
| Duplicates | Removed 30 rows (2,509 → 2,479) |
| Branch / category labels | Stripped whitespace and standardised case |
| Payment labels | Mapped `mpesa`, `Mpesa`, `M-PESA` → `M-Pesa`; missing → `Unknown` |
| Mixed dates | Parsed ISO first, then filled the remainder as day-first (`%d/%m/%Y`) |
| Missing quantity | Dropped 12 rows (no way to recover units sold) → 2,467 rows |
| `"TBC"` and zero prices | Converted to missing, then imputed with the **product's median price**; flagged in `imputed_price` |
| Negative quantities | Kept and flagged as `is_return` |
| Quantities over 100 | Kept and flagged as `unusual_qty` |
| Missing customer ID | Flagged as `new_customer` (not in the loyalty scheme) |

Output: `kakuma_fresh_mart_sales_clean.csv` (2,467 rows × 13 columns).

### 3. Analyse: `Analysing_Data_with_Pandas.ipynb`
- Created `revenue = qty × price`
- Excluded the 3 `unusual_qty` lines from the analysis; returns were kept, so revenue is **net**
- Used `groupby`/`agg`, `pivot_table` and `crosstab` to compare branches, categories, products, customer groups and payment methods
- Compared weekend and weekday revenue **per trading day** so the different number of days is accounted for

---

## Findings

### 1. Kakuma Town leads through volume, not bigger baskets
Kakuma Town earns **~40% of revenue** (KES 375,795 of 949,214). It has **1,000 sales lines** against about 730 at each other branch, but its average per line (**KES 376**) is the **lowest** of the three.

| Branch | Revenue (KES) | Sales lines | Avg per line (KES) |
|---|---|---|---|
| Kakuma Town | 375,795 | 1,000 | 376 |
| Lodwar | 287,493 | 727 | 395 |
| Kitale | 285,926 | 737 | 388 |

### 2. Revenue is concentrated in a few categories and products
- **Grains (25.2%), Cooking (24.5%) and Dairy (22.6%)** together make up **~72%** of revenue; Produce is the smallest at 6.5%.
- **Five of 19 products earn 54%** of revenue: Eggs (15.0%), Cooking oil (13.4%), Rice (9.9%), Wheat flour (8.5%), Detergent (7.7%).
- **Eggs lead revenue but rank only 10th by units sold**, while Water is the top seller by units.

### 3. Loyalty members don't spend more, and payment habits are alike everywhere
- Returning customers average **KES 386** per line vs **KES 384** for new customers (medians 250 vs 260), with over 1,000 lines in each group.
- **Cash and M-Pesa** each account for roughly **40–46%** of sales lines in every branch.

### 4. Weekends earn ~41% more than weekdays, in every branch
- Weekend days average **KES 57,141** vs **KES 40,425** on weekdays.
- The lift holds in each branch: **Lodwar +45%, Kitale +42%, Kakuma Town +38%**.

### Recommendations
- **Schedule more staff** for Saturday and Sunday.
- **Make sure stock is full** for weekend trading.
- **Let stock priorities follow the top products** that carry most of the revenue.

### Caveats and open questions
- **Confirm with the business:** the three extreme quantities (999, 500, 250) and that slash dates are day-first.
- **Short window:** only three weeks of data, so each weekday is seen just three times. Differences between individual weekdays are weak evidence; the weekend-vs-weekday gap is the robust signal.
- **Not covered:** customers and visits, because the file records items only.

---

## Skills

| Area | Demonstrated through |
|---|---|
| **Python & pandas** | `read_csv`, `drop_duplicates`, `str` methods, `replace`, `to_datetime`, `to_numeric`, `groupby` / `agg` / `transform`, `pivot_table`, `crosstab` |
| **Data profiling** | Auditing shape, types, missing values, duplicates, unique values and outliers before cleaning |
| **Data cleaning** | Standardising labels, reconciling mixed date formats, handling placeholder and impossible values |
| **Imputation & flagging** | Product-median price imputation; audit flags instead of silent deletion |
| **Business analysis** | KPI design (net revenue, average per line, revenue share, per-day weekend lift); separating volume from spend |
| **Analytical judgement** | Normalising for unequal day counts, stating sample-size limits, separating findings from open questions |
| **Communication** | Turning results into an executive slide deck with a clear bottom line and recommendations |

---



Run the notebooks in order: **Exploring → Cleaning → Analysing**. The cleaning notebook writes the clean CSV that the analysis notebook reads.
