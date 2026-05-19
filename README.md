![Excel](https://img.shields.io/badge/Excel-217346?style=flat&logo=microsoft-excel&logoColor=white)
![NHS Data](https://img.shields.io/badge/NHS%20England-005EB8?style=flat&logo=data:image/png;base64,&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
# NHS England A&E Performance Dashboard
### April 2025 – March 2026 | Portfolio Case Study

---

## Project Overview

This project delivers a full year analytical dashboard of NHS England Accident & Emergency performance, covering **April 2025 to March 2026**. The dataset spans **209 NHS trusts across 7 regions**, comprising **2,385 rows × 17 columns** of monthly operational data. The focus is on **Type 1 (major emergency) departments** and **other attendance types**, with particular attention to waiting time breaches the NHS's most visible and politically sensitive performance metric.

---

## The Real Business Problem

The NHS 4-hour A&E target that 76% of patients should be seen, treated, and either admitted or discharged within 4 hours has not been consistently met nationally for years. The consequences are real: prolonged waits increase patient risk, create unsafe crowding, and signal systemic breakdown in hospital flow.

This dashboard was built to answer questions that raw data alone cannot: **Why** are waits happening? **Where** is the system under most strain? **When** does it get worse? And critically are there trusts managing to beat the odds that others can learn from?

The analysis moves through three analytical layers: **descriptive** (what happened), **diagnostic** (why it happened), and **predictive** (what comes next).

---

## Dataset Scope

| Dimension | Detail |
|---|---|
| Time Period | April 2025 – March 2026 (12 months) |
| Total Records | ~2,385 trust-month rows |
| Trusts Covered | 209 NHS trusts |
| Type 1 Active Trusts | 124 trusts with emergency department activity |
| Regions | 7 NHS England regions |
| Key Metrics | T1 attendances, breach rates, 12-hour waits, seasonal/quarterly splits |

---

## 1. Data Collection

### Source
- **Primary Source:** [NHS England Official Statistics — A&E Attendances and Emergency Admissions](https://www.england.nhs.uk/statistics/statistical-work-areas/ae-waiting-times-and-activity/)
- Data is published monthly by NHS England as open government data in `.xlsx` format, covering all NHS Trusts in England.

### What Was Downloaded
- 12 monthly statistical releases (April 2025 – March 2026)
- Each file contains trust-level breakdowns of:
  - Type 1 (major A&E) attendances
  - Type 2 and Type 3 (minor/other) attendances
  - Patients waiting over 4 hours
  - Patients waiting over 12 hours from arrival (decision to admit)

### Structure of Raw Data
Each monthly file contained the following key columns (before renaming):

| Raw Column | Meaning |
|---|---|
| `Org Code` | NHS Organisation code |
| `Org Name` | Trust name |
| `Region` | NHS England region |
| `Type 1 Attendances` | Major emergency department visits |
| `Type 2 Attendances` | Single specialty A&E |
| `Type 3 Attendances` | Minor injury / urgent treatment |
| `Total Attendances` | Sum of all types |
| `Type 1 > 4 hrs` | Patients breaching 4-hour standard (Type 1) |
| `12+ hours from arrival` | Patients waiting over 12 hours (all types) |

### Consolidation
All 12 monthly files were consolidated into a single flat table with a `Period` column added (e.g., `April-2025`, `May-2025`) to enable time-series analysis. This produced the master dataset of **2,385 rows × 17 columns**.

---

## 2. Data Cleaning

### Steps Performed

#### 2.1 Standardising Column Names
Raw NHS column headers contain spaces, symbols, and inconsistent capitalisation. These were renamed to snake_case for compatibility with pivot tables and calculated columns:

```
"Type 1 Attendances"     → T1_Attendances
"Type 1 > 4 hrs"         → T1_Over4hrs
"12+ hours from arrival" → Wait_12hr_plus
"Org Name"               → Trust_Name
```

#### 2.2 Handling Zero and Null Records
Some trusts report zero Type 1 attendances in certain months (e.g., trusts without a major A&E, or those temporarily closed). These were:
- **Retained in full dataset** for total attendance calculations
- **Filtered out for trust-level breach rate analysis** to avoid division-by-zero errors and misleading outliers (e.g., a trust with 0 attendances and 0 breaches would show 0% breach — technically correct but analytically useless)

#### 2.3 Data Type Correction
- `Period` field formatted as text categories with consistent `Month-YYYY` labelling
- Attendance and wait columns cast to integers (some cells contained text-formatted numbers from raw export)
- Percentage columns stored as decimals (e.g., 0.394 for 39.4%) not as Excel percentage-formatted strings

#### 2.4 Calculated Column Creation
Four new columns were added to enable analysis:

| New Column | Formula |
|---|---|
| `T1_Breach_Rate` | `= T1_Over4hrs / T1_Attendances` |
| `Wait_12hr_Rate` | `= Wait_12hr_plus / T1_Attendances` |
| `Total_Attendance` | `= T1_Attendances + Other_Attendances` |
| `Month_Index` | `= 1 to 12` (for trend charting) |

#### 2.5 Season and Quarter Classification
Each monthly period was categorised:

| Label | Months |
|---|---|
| Spring | March, April, May |
| Summer | June, July, August |
| Autumn | September, October, November |
| Winter | December, January, February |
| Q1 | April – June |
| Q2 | July – September |
| Q3 | October – December |
| Q4 | January – March |

#### 2.6 Duplicate and Consistency Checks
- Cross-checked trust counts per month (expected: ~124 Type 1 active trusts per period)
- Verified regional totals summed correctly to national figures
- No duplicate rows detected after consolidation

---

## 3. Exploratory Data Analysis (EDA)

### 3.1 Univariate Analysis — Understanding Each Variable

**T1 Attendances (monthly, per trust):**
- Range: 0 to ~35,000 per trust per month
- Highly right-skewed a small number of large trusts (e.g., UHB, Manchester) drive a disproportionate share of volume
- National monthly T1 volume ranged from **1,271,830** (February) to **1,450,980** (January)

**T1 Breach Rate (per trust, full year):**
- Range: 14.8% (Alder Hey Children's) to 56.4% (Mid Cheshire)
- National weighted mean: **39.4%** — well above the 24% threshold
- Distribution: roughly normal but with a long right tail of very poor performers

**12-Hour Wait Rate:**
- Range: near 0% to over 8% for some trusts in winter months
- National annual average: **3.41%** (approximately 1 in every 29 emergency patients)

### 3.2 Bivariate Analysis — Relationships Between Variables

**Volume vs. Breach Rate:**
- No meaningful positive correlation found between trust size and breach rate
- Calderdale (192k T1 attendances, 16.6% breach) disproves the assumption that high volume causes poor performance
- Scatter plot confirms: operational management, not volume, is the primary performance driver

**Month vs. Breach Rate:**
- Pearson correlation between Month_Index and breach rate: slight U-shape best in summer (Q2), worst in winter (Q3–Q4)
- January consistently represents the annual peak across all years of NHS data

**Season vs. 12-Hour Wait Rate:**
- T1 demand variation between winter and summer: **only 2.3% lower in winter**
- 12-hour wait rate variation: **60.7% higher in winter** than summer
- This divergence is the statistical signature of **exit block** — ward-level congestion rather than front-door demand

### 3.3 Regional Distribution Analysis

Box plot analysis by region revealed:
- North West and Midlands: both high median breach rates AND high variance (trusts performing both very badly and moderately)
- North East and Yorkshire: lowest median breach rate with tightest distribution most consistent region in England
- London: mid-range breach rate but highest 12-hour wait rate among better-performing regions (3.77%), suggesting bed flow issues despite better front-door throughput

### 3.4 Outlier Detection

**Statistical method:** Trusts with breach rates more than 1.5× the interquartile range above Q3 were flagged as underperformers. Five trusts exceeded this threshold:
- Mid Cheshire (56.4%), Hull (55.5%), Shrewsbury and Telford (55.5%), Nottingham (55.5%), Plymouth (54.7%)

**High-performing outliers:** Trusts below Q1 − 1.5×IQR for breach rate:
- Calderdale and Huddersfield (16.6%), Alder Hey Children's (14.8%), Maidstone and Tunbridge Wells (22.3%)

### 3.5 Seasonal Decomposition

Separating demand-side from flow-side pressure:

| Factor | Summer | Winter | Change |
|---|---|---|---|
| T1 Attendances | 4,180,141 | 4,085,008 | −2.3% |
| 12-Hour Waits | 110,059 | 176,941 | +60.7% |
| 12-Hour Wait Rate | 2.63% | 4.33% | +64.6% |

**EDA conclusion:** Winter crisis is a **throughput and discharge problem**, not an attendance problem. This directly shapes the recommendations.

---

## Key Findings by the Numbers

### 1. Scale of Demand

- **26,360,540** total A&E attendances recorded across the full year
- **16,743,700** were Type 1 (major emergency) — representing **63.5%** of all attendances
- **9,616,840** were other attendance types (urgent treatment centres, walk-ins, minor injury units)
- Peak demand month: **January 2026** with **1,450,980** Type 1 attendances
- Trough month: **February 2026** with **1,271,830**  a **12.3% drop** from peak

### 2. The 4-Hour Breach Crisis

- **National weighted breach rate: 39.4%** — more than **1.6× the maximum acceptable threshold**
- Every single month exceeded the 24% breach ceiling
- **Best month: March 2026** at **36.1%** — still 12 percentage points above target
- **Worst month: January 2026** at **42.9%** — nearly **79% above target**
- Over the full year, **6,604,228 patients** waited longer than 4 hours

> **Methodology note:** Breach rates were calculated as `SUM(T1_Over4hrs) / SUM(T1_Attendances)` not a simple average of rates. This prevents the "average of averages" distortion that artificially inflates performance in low-volume months.

### 3. The 12-Hour Wait Crisis

- **570,931 patients** waited over 12 hours approximately **1 in every 29 emergency patients**
- Winter quarter recorded **176,941** 12-hour waits  **60.7% more than Summer** (110,059)

| Season | T1 Attendances | 12-Hour Waits | 12-Hour Wait Rate |
|---|---|---|---|
| Summer | 4,180,141 | 110,059 | 2.63% |
| Autumn | 4,253,306 | 149,727 | 3.52% |
| Spring | 4,225,245 | 134,204 | 3.18% |
| **Winter** | **4,085,008** | **176,941** | **4.33%** |

### 4. Regional Variation

| Region | T1 Attendances | 4-Hour Breach Rate | 12-Hour Wait Rate |
|---|---|---|---|
| NHS England North West | 2,380,539 | **42.6%** | 5.01% |
| NHS England South West | 1,493,885 | 42.5% | 2.71% |
| NHS England Midlands | 3,160,282 | 41.6% | 4.64% |
| NHS England North East & Yorkshire | 2,647,310 | 37.95% | 1.63% |
| NHS England East of England | 1,815,322 | 37.6% | 2.49% |
| NHS England London | 2,769,899 | 37.5% | 3.77% |
| **NHS England South East** | **2,476,463** | **36.9%** | 2.90% |

### 5. Quarterly Trend

| Quarter | T1 Attendances | 4-Hour Breach Rate | 12-Hour Wait Rate |
|---|---|---|---|
| Q1 (Apr–Jun 2025) | 4,169,595 | 39.3% | 3.03% |
| Q2 (Jul–Sep 2025) | 4,170,240 | 38.3% | 2.78% |
| Q3 (Oct–Dec 2025) | 4,275,757 | **40.2%** | 3.64% |
| Q4 (Jan–Mar 2026) | 4,128,108 | 39.9% | **4.19%** |

### 6. Trust-Level Outliers

**Best Performing:** Calderdale and Huddersfield NHS Foundation Trust — **16.6%** breach rate across 192,172 Type 1 attendances

**Worst Performing:** Mid Cheshire (56.4%), Hull (55.5%), Shrewsbury & Telford (55.5%), Nottingham (55.5%), Plymouth (54.7%)

---

## Business Problems Solved

| # | Problem | Finding | Recommendation |
|---|---|---|---|
| 1 | What is overall A&E demand? | 26.4M total; T1 = 63%; Peak January 2026 | Monitor Jan–Feb capacity annually |
| 2 | Are we meeting the 76% standard? | No. 39.4% breach  1.6× the 24% max | NHS-wide investment in patient flow |
| 3 | Does winter cause more demand or more waits? | T1 demand fell 2.3%; 12-hr waits rose 64.5%  exit block | Fund social care discharge, not A&E expansion |
| 4 | Which regions underperform? | North West worst (42.6%); South East best (36.9%) | Replicate North East & Yorkshire flow models |
| 5 | Can high-volume trusts still perform? | Yes. Calderdale: 192k T1, 16.6% breach | Replicate Gold Standard protocols nationally |
| 6 | Who are the worst trusts? | Mid Cheshire (56.4%), Hull (55.5%), Shrewsbury (55.5%) | NHS England improvement notices |
| 7 | Will performance worsen? | Breach peaks Jan–Feb; consistent seasonal pattern | Pre-position winter surge plans by October |

---

## Analytical Methodology

### Weighting Approach — Avoiding the Average of Averages Trap

All breach rates at regional, seasonal, quarterly, and national level were calculated as:
```
Breach Rate = SUM(T1_Over4hrs) / SUM(T1_Attendances)
```
Not as `AVERAGE(T1_Breach_Rate)`. A simple average treats a trust seeing 5,000 patients the same as one seeing 50,000 — producing a misleading national figure.

### Analytical Layers
- **Descriptive**: Monthly KPI cards, demand volume, T1 vs Other split, trust-level league tables
- **Diagnostic**: Seasonal decomposition isolating exit block from demand; regional benchmarking; scatter plot of volume vs breach rate
- **Predictive**: Monthly trend analysis; seasonal pattern identification for winter surge forecasting

### Dashboard Design
- NHS England colour palette and branding conventions
- Zero formula errors across all pivot tables and chart data
- Six charts answering six distinct analytical questions
- KPI card layer surfacing the five most critical metrics immediately

---

## Tools Used

- **Microsoft Excel** — data cleaning, pivot tables, chart construction, dashboard design
- **Power Query / Pivot Tables** — weighted aggregation across regional and seasonal dimensions
- **NHS England Official Statistics** — primary data source

