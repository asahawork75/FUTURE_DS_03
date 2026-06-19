# Bank Marketing Campaign - Funnel Analysis Dashboard

> **Internship Track:** Data Science & Analytics (`DS`)
> **Repository Name:** `FUTURE_DS_03`
> **Submitted By:** Arindam Saha
> **Submission Date:** 19-06-2026

---

## Table of Contents

1. [Project Overview](#-project-overview)
2. [Dataset Description](#-dataset-description)
3. [Funnel Design & Logic](#-funnel-design--logic)
4. [Tools & Technologies](#tools--technologies)
5. [DAX Measures & Calculated Columns](#-dax-measures--calculated-columns)
6. [Dashboard Pages](#-dashboard-pages)
7. [Key Insights & Findings](#-key-insights--findings)
8. [Business Recommendations](#-business-recommendations)
9. [Repository Structure](#-repository-structure)
10. [How to Use This Project](#-how-to-use-this-project)
11. [Author](#-author)

---

## Project Overview

This project performs a **Marketing Funnel Analysis** on a bank's telemarketing campaign dataset using **Power BI Desktop**. The analysis maps customer journey stages from initial contact through to final conversion (term deposit subscription), identifying drop-off points, high-performing channels, and actionable optimization opportunities.

### Business Questions Answered

| # | Question |
|---|---|
| 1 | Where are users dropping off in the funnel? |
| 2 | Which channels bring high-quality leads? |
| 3 | How can conversion rates be improved? |
| 4 | Which stages need the most optimization? |

---

## Dataset Description

| Attribute | Details |
|---|---|
| **Source** | Bank Marketing Campaign Dataset (UCI Machine Learning Repository) |
| **File** | `Bank_Marketing_Campaign_Dataset_-_bank.csv` |
| **Total Records** | 49,732 customer contacts |
| **Columns** | 17 attributes |
| **Target Variable** | `y` - whether the client subscribed to a term deposit (yes/no) |

### Column Reference

| Column | Description |
|---|---|
| `age` | Age of the customer |
| `job` | Type of job (management, technician, blue-collar, etc.) |
| `marital` | Marital status |
| `education` | Education level |
| `default` | Has credit in default? (yes/no) |
| `balance` | Average yearly balance (EUR) |
| `housing` | Has housing loan? (yes/no) |
| `loan` | Has personal loan? (yes/no) |
| `contact` | Contact channel: cellular / telephone / unknown |
| `day` | Last contact day of month |
| `month` | Last contact month of year |
| `duration` | Last contact duration in seconds |
| `campaign` | Number of contacts during this campaign |
| `pdays` | Days since last contact from previous campaign (-1 = not contacted) |
| `previous` | Number of contacts before this campaign |
| `poutcome` | Outcome of previous campaign (success / failure / other / unknown) |
| `y` | Subscribed to term deposit? - **target variable** |

---

## Funnel Design & Logic

Since the raw dataset does not have explicit funnel stage timestamps, the following funnel mapping was constructed from available fields:

```
Stage 1 — Total Contacted    → All 49,732 rows
         ↓
Stage 2 — Engaged            → duration > 0 (call was answered and lasted > 0 seconds)
         ↓
Stage 3 — Qualified          → poutcome ≠ unknown (has prior campaign history = known lead)
         ↓
Stage 4 — Converted          → y = "yes" (subscribed to term deposit)
```

| Stage | Count | Drop-off |
|---|---|---|
| Total Contacted | 49,732 | - |
| Engaged | ~47,000+ | Calls with duration > 0 |
| Qualified | 9,068 | poutcome known |
| Converted | 5,810 | y = yes |
| **Overall conversion rate** | **11.68%** | |

---

## Tools & Technologies

| Tool | Purpose |
|---|---|
| **Power BI Desktop** | Funnel visualization, DAX modeling, dashboard design |
| **DAX** | Calculated columns, funnel measures, conversion rates |
| **CSV / Excel** | Raw data source |
| **GitHub** | Documentation & submission |

---

## DAX Measures & Calculated Columns

### Calculated Columns (created in Data view - New Column)

```dax
-- Flag whether the contact engaged (call was answered)
EngagedFlag =
IF('bank'[duration] > 0, 1, 0)

-- Flag whether the contact is a qualified lead (known prior history)
QualifiedFlag =
IF('bank'[poutcome] <> "unknown", 1, 0)

-- Flag whether the contact converted to customer
ConvertedFlag =
IF('bank'[y] = "yes", 1, 0)

-- Bucket call durations for deeper analysis
DurationBucket =
SWITCH(TRUE(),
  'bank'[duration] = 0,         "No Contact (0s)",
  'bank'[duration] <= 60,       "Very Short (≤1 min)",
  'bank'[duration] <= 180,      "Short (1-3 min)",
  'bank'[duration] <= 600,      "Medium (3-10 min)",
  "Long (>10 min)"
)


### Measures (created via New Measure)

```dax
Total Contacted = COUNTROWS('bank')

Total Engaged = SUM('bank'[EngagedFlag])

Total Qualified = SUM('bank'[QualifiedFlag])

Total Converted = SUM('bank'[ConvertedFlag])

Traffic-to-Engaged Rate = DIVIDE([Total Engaged], [Total Contacted], 0)

Engaged-to-Qualified Rate = DIVIDE([Total Qualified], [Total Engaged], 0)

Qualified-to-Customer Rate = DIVIDE([Total Converted], [Total Qualified], 0)

Overall Conversion Rate = DIVIDE([Total Converted], [Total Contacted], 0)

Funnel Value =
SWITCH(
  SELECTEDVALUE('FunnelStages'[Stage]),
  "1. Total Contacted", [Total Contacted],
  "2. Engaged",         [Total Engaged],
  "3. Qualified",       [Total Qualified],
  "4. Converted",       [Total Converted]
)
```

---

## Dashboard Pages

The Power BI dashboard (`Bank_Data_Analysis.pbix`) contains **4 pages**:

---

### 1 Funnel Overview

**Visuals:** Funnel chart . 2 KPI cards . Navigation button

Shows the complete customer journey from total contacts down to conversions. The funnel chart makes drop-off visible at each stage. KPI cards highlight the overall conversion rate and total customers converted.

[Funnel Overview](01_funnel_overview)

---

### 2 Channel Comparison

**Visuals:** Clustered column chart . Data table . Navigation button

Compares conversion rates across the three contact channels (cellular, telephone, unknown). The supporting table provides exact counts alongside the visual for easy reporting.

| Channel | Total Contacts | Converted | Conversion Rate |
|---|---|---|---|
| Cellular | 32,181 | 4,785 | **14.87%** |
| Telephone | 3,207 | 434 | 13.53% |
| Unknown | 14,344 | 591 | 4.12% |

---

### 3 Campaign & Time Trends

**Visuals:** Line chart (monthly trend) . Clustered column chart (prior outcome) . Navigation button

Reveals strong seasonality in conversion - some months convert at 5* the rate of others. Also shows the dramatic impact of previous campaign success on current conversion rates.

| Previous Outcome | Conversion Rate |
|---|---|
| Success | **64.70%** |
| Other | 16.94% |
| Failure | 12.63% |
| Unknown | 9.16% |

---

### 4 Segment Drill-down

**Visuals:** Horizontal bar chart (by job) . 3 slicers (contact / month / poutcome)

Compares conversion rates across all job segments. Interactive slicers allow filtering by channel, month, and prior outcome to isolate the highest-value combinations.

| Job Segment | Conversion Rate |
|---|---|
| Student | **28.18%** |
| Retired | 22.85% |
| Unemployed | 15.02% |
| Management | 13.73% |
| Blue-collar | 7.28% |

---

## Key Insights & Findings

### 1. Where are users dropping off?
The largest single drop happens at the **Qualified → Converted** stage. Of 49,732 people contacted, **88.3% do not convert**. The biggest dead weight is the `poutcome = unknown` segment - 40,664 contacts (82% of the dataset) who have no prior campaign history, converting at just **9.16%**.

### 2. Which channels bring high-quality leads?
**Cellular is the strongest channel** - 14.87% conversion vs just 4.12% for unknown contact methods. The "unknown" channel is nearly 4× less effective, meaning a significant portion of campaign budget is being spent on poorly targeted outreach with no channel verification.

### 3. How can conversion rates be improved?
The single strongest signal: customers with `poutcome = success` (previously converted) convert at **64.70%** — nearly 7× the overall average. Re-engaging past customers should be the top campaign priority. Additionally, focusing campaigns on **March, October, September, and December** (40–51% conversion rates) instead of **May** (only 6.71%) would dramatically improve campaign ROI.

### 4. Which stages need the most optimization?
- **Channel verification** - reduce the 14,344 "unknown" contacts; identify and classify them
- **Timing** - shift campaign resources from May (lowest performing) to Q1 and Q4 months
- **Targeting** - prioritize students and retirees who convert at 2-3× the rate of blue-collar workers

---

## Business Recommendations

| Priority | Action | Expected Impact |
|---|---|---|
| 🔴 High | Re-target previously successful customers first | 64.7% vs 9.2% conversion |
| 🔴 High | Replace "unknown" channel contacts with verified cellular | +10 percentage points |
| 🟡 Medium | Shift campaign calendar to Mar, Oct, Sep, Dec | 4-5× higher monthly conversion |
| 🟡 Medium | Focus segments on students and retirees | 2-3× higher segment conversion |
| 🟢 Low | Reduce contacts to >5 per person (campaign fatigue) | Reduce wasted outreach |

---

## Repository Structure

```
FUTURE_DS_03/
│
├── README.md                              ← Project documentation (this file)
├── Bank_Marketing_Campaign_Dataset_-_bank.csv   ← Raw dataset
├── Bank_Data_Analysis.pbix                ← Power BI dashboard file
└── Screenshot/
    └── 01_funnel_overview.png
    └── 02_channel_Comparison.png
    └── 03_campaign_ time Trends.png
    └── 04_segment_drill_down_job_channel.png    ← Dashboard screenshot
```

---

## How to Use This Project

1. **Download or clone** this repository
2. Open `Bank_Data_Analysis.pbix` in **Power BI Desktop** (free download from Microsoft)
3. If prompted to refresh data source, point it to `Bank_Marketing_Campaign_Dataset_-_bank.csv` in the same folder
4. Use the **4 page tabs** at the bottom to navigate between Funnel Overview, Channel Comparison, Campaign Trends, and Segment Drill-down
5. Use the **slicers** on Page 4 to filter by contact channel, month, or prior outcome

---

## Author

| Field | Details |
|---|---|
| **Name** | Arindam Saha |
| **Track** | Data Science & Analytics (DS) |
| **Task** | Task 03 - Bank Marketing Funnel Analysis |
| **LinkedIn** |linkedin.com/in/arindam-saha-data-analyst|
| **GitHub** | github.com/asahawork75 |

---

> *This project was completed as part of the Future Interns Data Science & Analytics internship programme.*
