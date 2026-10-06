# Oil & Gas Production Intelligence & Well Performance Analytics

An end-to-end **Data Analytics project** for analyzing oil & gas field production, well performance, water injection, operational availability, WOR, GOR, and production decline indicators.

The project transforms raw daily well-production data into **clean analytical datasets, business KPIs, SQL-based analysis, and Power BI-ready outputs** that can support operational investigation and management reporting.

---

## Project Objective

Oil & gas production systems generate large volumes of daily well-level operational data. Raw production records are useful for storage and monitoring, but they do not directly answer important business questions such as:

- Which wells contribute the most oil and gas?
- How is field production changing over time?
- Which wells show declining production?
- Which wells have increasing Water-Oil Ratio (WOR)?
- Which wells show significant changes in Gas-Oil Ratio (GOR)?
- How does water injection change relative to oil production?
- Which wells have stronger operational availability?
- Which wells should be prioritized for further investigation?

This project addresses these questions by building a reproducible analytics pipeline using **Python, Pandas, SQL, SQLite, and Power BI-ready datasets**.

---

## Business Problem

The core problem is to convert raw production measurements into **actionable operational intelligence**.

Instead of manually inspecting thousands of daily records, the pipeline:

**Cleans → Aggregates → Calculates KPIs → Identifies trends → Screens wells → Generates analytical outputs**

The final outputs help identify wells or production periods that deserve further operational investigation.

> **Analytical Disclaimer:** Relationships between injection, production, WOR, GOR, and other variables are treated as associations for monitoring and hypothesis generation. The project does not claim causal relationships or replace reservoir-engineering analysis.

---

## Project Workflow

```text
Raw Excel Data
      │
      ▼
Data Ingestion
      │
      ▼
Data Cleaning & Quality Checks
      │
      ▼
Analytical Dataset
      │
      ├───────────────┐
      ▼               ▼
Field KPIs       Well KPIs
      │               │
      └───────┬───────┘
              ▼
      Monthly Aggregation
              │
              ▼
      WOR / GOR Analysis
              │
              ▼
      Production Decline
              │
              ▼
         SQL Analysis
              │
              ▼
      Power BI-ready CSVs
              │
              ▼
       Business Insights
```

---

## Key Analytical Questions

The project is structured around practical Data Analyst questions:

1. What are the total field oil, gas, and water production volumes?
2. How does production change over time?
3. Which wells contribute the most oil and gas?
4. Which wells show production decline?
5. How does water injection move relative to oil production?
6. Which wells have increasing WOR?
7. Which wells have significant GOR changes?
8. Which wells have stronger operational availability?
9. Which wells should be prioritized for further investigation?

---

# Data Pipeline

## 1. Data Ingestion

The primary source is an Excel workbook containing a `Daily Production Data` sheet.

Important fields include:

- `DATEPRD`
- `WELL_NAME`
- `BORE_OIL_VOL`
- `BORE_GAS_VOL`
- `BORE_WAT_VOL`
- `BORE_WI_VOL`
- `ON_STREAM_HRS`
- `WELL_TYPE`

The notebook loads the workbook using Pandas and validates that the required analytical columns exist.

---

## 2. Data Cleaning

The pipeline performs reproducible data-quality processing:

- Standardizes column names
- Converts dates to datetime format
- Converts production fields to numeric values
- Handles missing numeric values
- Removes exact duplicate records
- Removes records without valid dates or well names
- Handles negative physical production/injection values
- Creates a consistent `WELL_NAME`
- Performs schema and quality checks

This ensures that downstream KPIs are generated from a consistent analytical dataset.

---

# KPI Development

## Field-Level KPIs

The project calculates:

- Total Oil Production
- Total Gas Production
- Total Water Production
- Total Water Injection
- Unique Wells
- Operational Records

These KPIs provide a high-level overview of field performance.

---

## Monthly Production KPIs

Daily records are aggregated into monthly production tables.

The monthly dataset contains:

- Oil production
- Gas production
- Water production
- Water injection
- GOR
- WOR

Monthly aggregation reduces daily noise and makes long-term production trends easier to analyze.

---

# Well Performance Analysis

A well-level analytical table is generated containing:

| Metric | Description |
|---|---|
| Total Oil | Cumulative oil production |
| Total Gas | Cumulative gas production |
| Total Water | Cumulative water production |
| Total Injection | Cumulative water injection |
| Recorded Days | Number of days present in the dataset |
| Operational Days | Days classified as operational |
| Average On-Stream Hours | Average daily operating hours |
| Operational Time % | Operational days as a percentage of recorded days |
| WOR | Water production / oil production |
| GOR | Gas production / oil production |

This allows wells to be compared using both **production and operational metrics**.

---

# Production Decline Screening

The project uses a simple screening metric to identify wells with potential production decline.

For each well:

```text
Oil Change % =
(Last Positive Monthly Oil - First Positive Monthly Oil)
/
First Positive Monthly Oil × 100
```

This provides a straightforward first-to-last production comparison.

For example:

```text
Well A → -12%
Well B → -37%
Well C → +8%
```

Wells with large negative changes can be prioritized for further investigation.

### Important

This is a **screening indicator**, not a formal reservoir-engineering decline-curve analysis.

---

# WOR Analysis

### Water-Oil Ratio

```text
WOR = Water Production / Oil Production
```

WOR is calculated from aggregated production volumes rather than averaging daily ratios.

The project compares:

```text
Recent Average WOR
        vs
Historical Median WOR
```

A large increase is used to flag wells for further investigation.

This helps answer:

> Which wells are producing significantly more water relative to oil than their historical behavior?

---

# GOR Analysis

### Gas-Oil Ratio

```text
GOR = Gas Production / Oil Production
```

The project calculates monthly GOR and tracks its movement over time.

Significant changes in GOR can be identified as screening signals for further investigation.

---

# Water Injection Analysis

The project compares:

```text
Monthly Water Injection
              vs
Monthly Oil Production
```

This helps monitor whether production and injection trends are moving together or diverging over time.

The analysis is intentionally treated as **trend analysis and hypothesis generation**, rather than causal inference.

---

# Operational Performance

A simple availability-style KPI is calculated:

```text
Operational Time % =
Operational Days / Recorded Days × 100
```

This provides a comparative view of well operational performance.

The metric can help identify wells that have:

- High production but lower operational availability
- Strong production and strong availability
- Low production and low availability

These combinations can be useful when prioritizing operational investigations.

> The metric should not be interpreted as an official uptime metric without validating the organization's operational definition.

---

# SQL Analytics

The project stores analytical tables in a local SQLite database:

```text
well_summary
monthly_well
monthly_field
```

SQL is then used to answer analytical questions independently of Pandas.

### Example: Top Producing Wells

```sql
SELECT
    WELL_NAME,
    Total_Oil_Sm3,
    Total_Gas_Sm3,
    Operational_Time_Pct
FROM well_summary
ORDER BY Total_Oil_Sm3 DESC
LIMIT 10;
```

### Example: Ranking Wells

```sql
SELECT
    WELL_NAME,
    Total_Oil_Sm3,
    RANK() OVER (
        ORDER BY Total_Oil_Sm3 DESC
    ) AS Oil_Production_Rank
FROM well_summary
ORDER BY Oil_Production_Rank;
```

The SQL layer demonstrates the ability to move beyond Python-only analysis and apply relational analytical techniques.

---

# Power BI Integration

The project generates three Power BI-ready datasets:

```text
outputs/
├── powerbi_field_monthly.csv
├── powerbi_well_summary.csv
└── powerbi_monthly_well.csv
```

These datasets can be imported into Power BI to build an interactive production-performance dashboard.

### Suggested Dashboard Sections

#### Executive Overview

- Total Oil Production
- Total Gas Production
- Total Water Production
- Total Water Injection
- Number of Wells

#### Production Trends

- Monthly Oil Production
- Monthly Gas Production
- Monthly Water Injection

#### Well Performance

- Top Oil-Producing Wells
- Top Gas-Producing Wells
- Operational Availability

#### Investigation / Screening

- Declining Wells
- High WOR Wells
- High GOR Wells
- Low Operational Availability Wells

---

# Technology Stack

### Programming & Analytics

- Python
- Pandas
- NumPy
- Matplotlib

### Database & SQL

- SQLite
- SQL
- Window Functions

### Business Intelligence

- Power BI-ready CSV outputs

### Data Source

- Excel (`.xlsx`)

---

# Project Structure

```text
volvo-field-production-analysis/
│
├── volve_field_oil_gas_analytics_rystad_ready.ipynb
├── oilwell_production_data.xlsx
│
├── outputs/
│   ├── oil_gas_analytics.db
│   ├── powerbi_field_monthly.csv
│   ├── powerbi_well_summary.csv
│   └── powerbi_monthly_well.csv
│
└── README.md
```

---

# How to Run

## 1. Clone the repository

```bash
git clone https://github.com/VivekRangu45/volvo-field-production-analysis.git
cd volvo-field-production-analysis
```

## 2. Install dependencies

```bash
pip install pandas numpy matplotlib openpyxl
```

## 3. Open the notebook

```bash
jupyter notebook
```

Or open the notebook directly using Google Colab.

## 4. Place the source Excel file in the project directory

The notebook expects:

```text
oilwell_production_data.xlsx
```

## 5. Run all notebook cells

The notebook will generate:

```text
outputs/oil_gas_analytics.db
outputs/powerbi_field_monthly.csv
outputs/powerbi_well_summary.csv
outputs/powerbi_monthly_well.csv
```

---

# Key Business Outputs

The project produces several decision-support outputs:

### Field Performance

Provides a consolidated view of total oil, gas, water production and water injection.

### Well Ranking

Identifies the wells contributing the largest volumes of oil and gas.

### Production Decline Screening

Highlights wells with large first-to-last monthly production changes.

### WOR Screening

Identifies wells whose recent water-oil ratio is substantially above their historical behavior.

### GOR Monitoring

Tracks changes in gas relative to oil production.

### Operational Performance

Compares operational availability across wells.

These outputs can be used to create a shortlist of wells for deeper engineering or operational investigation.

---

# Key Takeaways

This project demonstrates an end-to-end analytics workflow:

```text
Business Problem
      ↓
Raw Operational Data
      ↓
Data Cleaning
      ↓
KPI Engineering
      ↓
Exploratory Analysis
      ↓
Well-Level Analytics
      ↓
SQL Analysis
      ↓
BI-Ready Data
      ↓
Business Insights
```

The main focus is not simply producing charts, but **turning operational data into structured analytical questions and investigation priorities**.

---

# Skills Demonstrated

- Data Cleaning & Validation
- Exploratory Data Analysis
- KPI Development
- Time-Series Analysis
- Well-Level Performance Analysis
- Ratio Analysis
- Production Trend Analysis
- SQL Aggregations
- SQL Window Functions
- SQLite
- Python/Pandas
- NumPy
- Data Visualization
- Power BI Data Preparation
- Business Problem Framing
- Analytical Reasoning

---

# Future Enhancements

Potential extensions include:

- Interactive Power BI dashboard
- Automated data refresh pipeline
- Statistical anomaly detection
- More robust production decline models
- Well clustering based on production behavior
- Forecasting future production
- Correlation and lag analysis between injection and production
- Automated alerting for abnormal WOR/GOR changes
- Integration with additional operational or maintenance data
- Reservoir/engineering-domain validation of analytical indicators

---

## Disclaimer

This project is intended as a **data analytics and decision-support demonstration**.

The identified trends and screening metrics should be validated using operational records, reservoir-engineering knowledge, and domain-specific context before being used for production or field-management decisions.
