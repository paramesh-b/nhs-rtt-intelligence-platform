# NHS RTT Intelligence Platform

**Tools:** Python | MySQL | Excel | Power BI | Streamlit  
**Data:** NHS England RTT Waiting Times — April 2025 to January 2026  
**Trusts Covered:** 130+ NHS acute trusts across England  
**Rows Analysed:** 36,120  

---

## Project Overview

This platform analyses NHS England Referral to Treatment (RTT) waiting times data 
across all acute trusts in England. The goal was not to build a dashboard — it was 
to give operational teams enough lead time to intervene before a breach happens, 
because in the NHS a breach is not just a number, it is a patient who has waited 
too long.

The project covers the full analytical pipeline from raw data ingestion to 
executive-level reporting, with a specific focus on Gloucestershire Hospitals NHS 
Foundation Trust benchmarked against South West regional peers and national 
performance.

---

## Key Findings — Gloucestershire Hospitals NHS Foundation Trust

- Overall 18-week compliance: **68.5%** against a national target of **92%**
- Total patients waiting: **62,862** across all specialties
- Trust ranks **11th out of 13** acute trusts in the South West region
- **Trauma and Orthopaedics** is the largest and worst performing surgical specialty — 
  8,999 patients waiting, only 55.1% within 18 weeks
- **Gynaecology** showed the steepest decline — from 71.3% in April 2025 to 62.7% 
  in January 2026, a drop of 8.5 percentage points
- **Neurology and Rheumatology** are CRITICAL — both deteriorating consistently 
  over 9 months
- **52-week waiters reduced by 77%** — from 93 in April 2025 to 21 in January 2026, 
  demonstrating successful targeted long-waiter intervention
- **Ophthalmology improved 8.8 percentage points** — the strongest recovery in the trust

---

## Project Architecture

### Layer 1 — Python Data Pipeline
- Automated ingestion of 10 months of NHS England RTT Excel files
- Auto-detection of header rows across file variants
- Column selection by name to handle weekly breakdown columns
- Output: 36,120 rows of clean data saved to CSV

### Layer 2 — MySQL Database
- Schema: `nhs_rtt` database with `rtt_data` table
- 36,120 rows loaded via PyMySQL
- 6 analytical SQL queries covering:
  - GHFT overall performance trend (10 months)
  - GHFT specialty breakdown ranked by compliance
  - Trusts with 3+ consecutive months of deterioration (48 trusts identified)
  - South West regional benchmarking (GHFT ranked 11th of 13)
  - National top 5 specialties by waiting list size
  - Specialty breach risk ranking (2 CRITICAL, 7 HIGH identified)

### Layer 3 — Excel RTT Performance Scorecard
4-sheet professional workbook:
- **Sheet 1 — RAG Scorecard:** 19 specialties RAG rated against 92% target with 
  conditional formatting, summary box and bar chart
- **Sheet 2 — Trend Analysis:** 10 specialties tracked over 10 months with 
  sparklines and direction indicators
- **Sheet 3 — Month on Month:** December 2025 vs January 2026 comparison with 
  automatic Better/Worse status
- **Sheet 4 — Executive Summary:** Written briefing with findings and recommendations 
  formatted for board level

### Layer 4 — Power BI Dashboard
2-page interactive dashboard:
- **Page 1 — National Overview:** 4 KPI cards, 20 worst performing trusts bar chart, 
  92% reference line, region slicer
- **Page 2 — Trust Deep Dive:** Trust and month slicers, 4 dynamic KPI cards showing 
  real-time performance for any selected trust

---

## Screenshots

### Excel — RAG Scorecard
![RAG Scorecard](screenshots/RAG_Scorecard.png)

### Excel — Trend Analysis
![Trend Analysis](screenshots/Trend_Analysis.png)

### Excel — Month on Month
![Month on Month](screenshots/Month_on_Month.png)

### Power BI — National Overview
![National Overview](screenshots/National_overview_powerbi_Dashboard.png)

### Power BI — Trust Deep Dive
![Trust Deep Dive](screenshots/Trust_Deep_Dive_Power_Bi_Dashboard.png)

---

## SQL Queries

### Query 1 — GHFT Overall Performance Trend
```sql
SELECT 
    month,
    total_incomplete_pathways,
    ROUND(pct_within_18_weeks * 100, 1) AS pct_within_18_weeks,
    total_52_plus_weeks
FROM rtt_data
WHERE provider_code = 'RTE'
AND treatment_function_code = 'C_999'
ORDER BY month;
```

### Query 2 — GHFT Specialty Performance
```sql
SELECT 
    treatment_function,
    total_incomplete_pathways,
    ROUND(pct_within_18_weeks * 100, 1) AS pct_within_18_weeks,
    total_52_plus_weeks
FROM rtt_data
WHERE provider_code = 'RTE'
AND treatment_function_code != 'C_999'
AND month = '2026-01'
AND total_incomplete_pathways > 0
ORDER BY pct_within_18_weeks ASC;
```

### Query 6 — Breach Risk Ranking
```sql
SELECT 
    a.treatment_function,
    ROUND(a.pct_within_18_weeks * 100, 1) AS current_pct,
    ROUND(b.pct_within_18_weeks * 100, 1) AS start_pct,
    ROUND((a.pct_within_18_weeks - b.pct_within_18_weeks) * 100, 1) AS pct_change,
    CASE 
        WHEN (0.92 - a.pct_within_18_weeks) > 0.30 
             AND (a.pct_within_18_weeks - b.pct_within_18_weeks) < 0
             THEN 'CRITICAL'
        WHEN (0.92 - a.pct_within_18_weeks) > 0.20
             THEN 'HIGH'
        WHEN (0.92 - a.pct_within_18_weeks) > 0.10
             THEN 'MEDIUM'
        ELSE 'LOW'
    END AS breach_risk
FROM rtt_data a
JOIN rtt_data b
    ON a.provider_code = b.provider_code
    AND a.treatment_function_code = b.treatment_function_code
WHERE a.provider_code = 'RTE'
AND a.month = '2026-01'
AND b.month = '2025-04'
AND a.treatment_function_code != 'C_999'
AND a.total_incomplete_pathways > 100
ORDER BY (0.92 - a.pct_within_18_weeks) DESC;
```

---

## Business Impact

| Deliverable | Business Value |
|---|---|
| Python pipeline | Eliminates manual monthly data collection — saves analyst hours |
| SQL breach risk ranking | Identifies critical specialties before they hit crisis point |
| Excel RAG scorecard | Divisional managers identify problems in 30 seconds |
| Power BI dashboard | Replaces hours of manual extraction with one-click answers |
| Executive summary | Board-ready briefing requiring no further formatting |

---

## Data Source

All data sourced from NHS England published statistics — publicly available, 
no data governance issues.

[NHS England RTT Waiting Times](https://www.england.nhs.uk/statistics/statistical-work-areas/rtt-waiting-times/)

---

## Author

**Paramesh Kumar Bukya**  
MSc Data Science — Manchester Metropolitan University  
[LinkedIn](https://linkedin.com/in/paramesh-kumar-bukya-296368238) | 
[GitHub](https://github.com/paramesh-b)
