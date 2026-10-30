<div align="center">

# Horizon Hospital | Emergency Room Analytics

<img width="1672" height="941" alt="Horizon Hospital Emergency Room Analytics Banner" src="https://github.com/user-attachments/assets/7875920e-2150-491c-a844-1126bc8db188" />


### SQL Server Portfolio Case Study

**Patient Flow · Waiting Times · Department Referrals · Patient Satisfaction · Admission Trends**

![SQL Server](https://img.shields.io/badge/SQL_Server-T--SQL-CC2927?style=for-the-badge) ![Healthcare](https://img.shields.io/badge/Domain-Healthcare_Analytics-0099A8?style=for-the-badge) ![Approach](https://img.shields.io/badge/Approach-Exploratory_SQL-163D5B?style=for-the-badge) ![Status](https://img.shields.io/badge/Project-Independent_Case_Study-3D6475?style=for-the-badge)

*Using T-SQL to explore emergency-room patient activity and convert operational questions into reproducible analytical queries.*

[Overview](#executive-summary) · [Data](#dataset--field-guide) · [Questions](#analytical-question-framework) · [SQL](#sql-implementation-highlights) · [QA](#data-quality--sql-review) · [Recommendations](#operational-recommendations)

</div>

> **Banner placement:** Drag the Horizon Hospital banner image into GitHub's README editor **above the first heading**. GitHub will create the hosted image link automatically. The banner is conceptual artwork, not a screenshot of a deployed hospital system.
>
> **Project scope:** Independent educational SQL analysis based on the supplied `horizon_hospital.sql` script. The attached script contains analysis queries but **does not contain the underlying patient dataset, `CREATE TABLE` definition, actual query output, or evidence of a deployed clinical dashboard**. Hospital-specific performance results and clinical outcomes therefore cannot be verified from it.

---

## Executive Summary

Emergency departments need visibility into patient arrivals, waiting times, referral demand and the patient experience. SQL can help identify operational patterns that deserve closer investigation, provided metrics are correctly defined and data quality is checked.

This project applies **Microsoft SQL Server (T-SQL)** to patient-level hospital records to explore:

- **Admissions and activity:** patient volume, monthly and weekly patterns, recent visits and day-of-week visits.
- **Waiting times:** average waiting time, longest waits, department-level comparisons and patients above the overall average.
- **Department referrals:** referral volume, larger departments and departmental waiting-time ranks.
- **Patient experience:** satisfaction scores, missing feedback and top-rated departments.
- **Patient characteristics:** gender, age and other fields included in the source queries, considered descriptively rather than as causes of care outcomes.

**Role:** Data Analyst  
**Primary technology:** Microsoft SQL Server / T-SQL  
**Project type:** Exploratory SQL portfolio analysis  
**Deliverable reviewed:** `horizon_hospital.sql`

### What this project demonstrates

Translating healthcare operations questions into SQL filters, aggregations, date analysis, subqueries, `HAVING`, `RANK()` and grouped reporting—while recognizing where query logic needs further validation.

---

## Business Problem

An emergency-room operations team might ask:

1. How many patient records are recorded, and how is activity distributed over time?
2. Which referral departments receive the highest patient volume?
3. How long do patients wait overall and by department?
4. Which patients wait longer than the current dataset average?
5. How do patient satisfaction scores differ across referral departments?
6. Which patient and timestamp records require validation before reporting?

**Analytical goal:** Create a reliable foundation for reporting operational activity and identifying possible service bottlenecks—not diagnosing patients or inferring clinical causation.

---

## Dataset & Field Guide

The supplied SQL uses one main table: **`Horizon_Hospital`**.

| Column referenced in SQL | Analytical purpose |
|---|---|
| `Patient_Id` | Patient identifier used in counts and rankings |
| `Patient_First_Inital`, `Patient_Last_Name` | Displayed patient name fields |
| `Patient_Age` | Age-group and patient filtering |
| `Patient_Gender` | Patient composition |
| `Patient_Race` | Descriptive demographic counts |
| `Patient_Admission_Date` | Admission date, year, month and weekday |
| `Department_Referral` | Department-level referral analysis |
| `Patient_Waittime` | Waiting-time comparisons and rankings |
| `Patient_Satisfaction_Score` | Patient-experience reporting and missing-value checks |

*Field names reflect the supplied SQL, including `Patient_First_Inital` and `Patient_Waittime`. Column types, date coverage, units, uniqueness and the table schema were not supplied.*

### Analytical workflow

```text
Horizon_Hospital (patient-level records)
                 |
       Data-quality review
                 |
            T-SQL queries
                 |
  +--------------+--------------+
  |              |              |
Admissions   Referrals &     Satisfaction
& Trends      Wait Times      & Missingness
  |              |              |
  +--------------+--------------+
                 |
       Operational questions
                 |
     Validate -> Interpret -> Report
```

---

## Analytical Question Framework

The original script is organized into **basic and intermediate** exercises followed by more advanced grouped and window-function analysis. The groupings below preserve those topics but make them easier for hiring managers to scan.

| Analysis theme | Questions addressed | Core SQL concepts |
|---|---|---|
| **Patient exploration** | List records; distinct genders; patients over 50; patient names and admissions; total patients | `SELECT`, `DISTINCT`, `WHERE`, `COUNT`, `CONCAT`, `CAST` |
| **Patient filters** | Physiotherapy referrals; female patients; surname beginning with S; satisfaction filtering | `WHERE`, `LIKE`, `IS NULL` |
| **Time-based activity** | Latest admissions; past six months; Monday admissions; monthly/weekly totals; current and prior year | `ORDER BY`, `DATEADD`, `DATENAME`, `DATEPART`, `YEAR` |
| **Patient wait times** | Overall average; longest wait; above-average waits; department with highest average; within-department patient rank | `AVG`, `TOP`, scalar subquery, `RANK() OVER` |
| **Department performance** | Counts per referral department; departments with over 10 patients; satisfaction by department; top three by satisfaction | `GROUP BY`, `HAVING`, `AVG`, `COUNT` |
| **Patient composition** | Gender counts; age filtering; second-most-common race | `GROUP BY`, `RANK()` |
| **Feedback completeness** | Patients with null satisfaction scores | `IS NULL` |

These are **queries implemented or attempted in the supplied script**, not verified results from patient records.

---

## SQL Implementation Highlights

The following examples are **improved illustrative versions** of the original SQL questions. They are not replacements for the source script; run them only after confirming the actual table schema and data types.

### 01 — Patient volume by month

```sql
SELECT
    DATEFROMPARTS(
        YEAR(Patient_Admission_Date),
        MONTH(Patient_Admission_Date), 1
    ) AS AdmissionMonth,
    COUNT(*) AS PatientRecords
FROM dbo.Horizon_Hospital
WHERE Patient_Admission_Date IS NOT NULL
GROUP BY
    YEAR(Patient_Admission_Date),
    MONTH(Patient_Admission_Date)
ORDER BY AdmissionMonth;
```

**Why this matters:** Grouping by **year and month** avoids accidentally combining the same month across different years. `COUNT(*)` counts rows, not necessarily unique patients or completed admissions.

### 02 — Average wait time by referral department

```sql
SELECT
    Department_Referral,
    COUNT(*) AS PatientRecords,
    AVG(CAST(Patient_Waittime AS DECIMAL(12,2)))
        AS AvgWaitMinutes
FROM dbo.Horizon_Hospital
WHERE Patient_Waittime IS NOT NULL
GROUP BY Department_Referral
ORDER BY AvgWaitMinutes DESC;
```

**Why this matters:** Compare waiting times alongside department-level volumes. The label “minutes” assumes the source field is recorded in minutes, as suggested by the original script; confirm this before publication.

### 03 — Patients waiting above the overall average

```sql
WITH OverallWait AS (
    SELECT AVG(CAST(Patient_Waittime AS DECIMAL(12,2)))
        AS AvgWait
    FROM dbo.Horizon_Hospital
    WHERE Patient_Waittime IS NOT NULL
)
SELECT
    h.Patient_Id,
    h.Department_Referral,
    h.Patient_Waittime,
    w.AvgWait
FROM dbo.Horizon_Hospital AS h
CROSS JOIN OverallWait AS w
WHERE h.Patient_Waittime > w.AvgWait
ORDER BY h.Patient_Waittime DESC;
```

**Why this matters:** Makes the threshold explicit and helps identify unusually long waits for further operational investigation.

### 04 — Ranking waiting times within departments

```sql
SELECT
    Patient_Id,
    Department_Referral,
    Patient_Waittime,
    RANK() OVER (
        PARTITION BY Department_Referral
        ORDER BY Patient_Waittime DESC
    ) AS DepartmentWaitRank
FROM dbo.Horizon_Hospital
WHERE Patient_Waittime IS NOT NULL;
```

**Why this matters:** Window functions compare patients **within** the same department instead of across the whole hospital. Equal waiting times receive the same rank.

### 05 — Patient satisfaction by referral department

```sql
SELECT TOP (3)
    Department_Referral,
    COUNT(Patient_Satisfaction_Score) AS Responses,
    AVG(CAST(Patient_Satisfaction_Score AS DECIMAL(12,2)))
        AS AvgSatisfaction
FROM dbo.Horizon_Hospital
WHERE Patient_Satisfaction_Score IS NOT NULL
GROUP BY Department_Referral
ORDER BY AvgSatisfaction DESC, Responses DESC;
```

**Why this matters:** Ranking departments only on average satisfaction can be misleading if very few patients responded. Display the **response count** with the score and check the rating scale.

### 06 — Admission activity: current and previous year

```sql
SELECT
    YEAR(Patient_Admission_Date) AS AdmissionYear,
    COUNT(*) AS PatientRecords
FROM dbo.Horizon_Hospital
WHERE Patient_Admission_Date >=
    DATEFROMPARTS(YEAR(GETDATE()) - 1, 1, 1)
  AND Patient_Admission_Date <
    DATEFROMPARTS(YEAR(GETDATE()) + 1, 1, 1)
GROUP BY YEAR(Patient_Admission_Date)
ORDER BY AdmissionYear;
```

**Why this matters:** Limits results to the calendar years intended by the question. A fair year-on-year comparison may additionally require **year-to-date vs. year-to-date** normalization.

---

## Data Quality & SQL Review

Reviewing analytical logic is important in healthcare reporting. The original script contains several places where the **question and SQL do not fully match**:

| Source question / query | Issue | Improvement |
|---|---|---|
| “Satisfaction score above 8” | Query uses `< 5` | Use `> 8` if the source score scale supports it |
| “Youngest patients aged 15–25” | Query filters `BETWEEN '25' AND '55'` | Use `BETWEEN 15 AND 25` on a numeric age column |
| “Patients admitted per month” | Query groups by `Patient_Id` and month name | Aggregate by month **and year** without patient ID, unless patient-level detail is intended |
| “Department with longest average wait” | Query orders grouped records but does not return an aggregate or `TOP 1` | Return average wait and select the top department |
| “Second most common race” | `RANK()` may return multiple tied categories | Define tie handling and show supporting counts |
| “Patients admitted per week” | Week number alone mixes years | Group by year and a consistent week definition |
| “Patients in past six months” | Uses `GETDATE()` | Suitable for a live report; for historical datasets use a clear fixed as-of date |
| Initial `CREATE DATABASE` followed by table query | No table schema or data import included | Document prerequisite database/table setup |

### Recommended validation checklist

- [ ] Confirm patient identifier uniqueness and what one record represents.
- [ ] Verify `Patient_Waittime` units and missing/negative values.
- [ ] Confirm the permitted satisfaction-score scale and missing-response rate.
- [ ] Confirm admission timestamps and historical reporting dates.
- [ ] Check spelling and standardization of department names.
- [ ] Separate demographic **description** from explanations of service outcomes.
- [ ] Verify every reported count, average and ranked result from executed SQL output.
- [ ] Publish only aggregate or appropriately de-identified patient data.

> **Privacy:** Patient names, IDs and potentially identifying row-level combinations should not appear in public screenshots or sample exports from real health records. Use synthetic or properly de-identified examples for a public GitHub portfolio.

---

## Operational Recommendations

The script provides methods for investigation; it does **not** include actual results showing that a particular hospital department is underperforming or that interventions improved care.

| Decision area | What to examine | Possible follow-up |
|---|---|---|
| **Long waits** | Department averages, patient counts and distribution of wait times | Review capacity, arrival peaks and triage workflow |
| **Department pressure** | Referral volumes by date and department | Compare workload with available resources |
| **Admissions** | Weekday, month and year trends | Investigate patterns before planning capacity changes |
| **Patient experience** | Satisfaction, response counts and missing scores | Examine whether low scores align with waits and other factors |
| **Reporting quality** | Nulls, data types, date filters and metric definitions | Standardize validated KPI calculations |

These are suggested analytical next steps, not verified clinical recommendations or measured outcomes.

---

## Skills Demonstrated

**SQL fundamentals:** `SELECT`, `WHERE`, `DISTINCT`, `LIKE`, `COUNT`, `CAST`, `CONCAT`.  
**Aggregations:** `AVG`, `GROUP BY`, `HAVING`, `TOP`, sorting.  
**Date analysis:** `DATEADD`, `DATEPART`, `DATENAME`, `YEAR`.  
**Advanced techniques:** scalar subqueries, `RANK() OVER (PARTITION BY ...)`.  
**Analytical practice:** KPI definition, grouped comparisons, missing-value review and error identification.

---

## Repository Files

| File | Description |
|---|---|
| `horizon_hospital.sql` | Original SQL script with basic, intermediate and window-function questions |
| `README.md` | Project context, techniques, SQL examples, technical review and recommended validation |

To reproduce the full results, the source table schema and example data must also be provided. No Power BI report or live dashboard is established by the supplied script.

---

## Next Improvements

1. Add a reproducible schema and synthetic dataset (without real patient information).
2. Correct the mismatched SQL questions and query logic.
3. Run the script in SQL Server and save verified aggregate outputs.
4. Add a short data dictionary explaining wait-time units, timestamps and score ranges.
5. If a reporting dashboard is built later, include actual screenshots and clearly define measures.

---

<div align="center">

### About the Analyst

**Khurram Naveed**  
Data Analyst | SQL Server | Business Intelligence

[![GitHub](https://img.shields.io/badge/GitHub-khurraminsights-181717?style=for-the-badge&logo=github)](https://github.com/khurraminsights)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/khurram-naveed-0083851aa/)

**From healthcare data to clearer operational questions.**

*Independent SQL analytics portfolio project.*

</div>

