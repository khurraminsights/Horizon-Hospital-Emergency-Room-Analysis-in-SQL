# Horizon Hospital — Emergency Room Operations Analysis (SQL Server)

![DALL·E 2025-02-03 05 03 17 - A professional and visually stunning cover image for a hospital emergency room analysis portfolio project  The image should feature a modern hospital ](https://github.com/user-attachments/assets/4c1a0730-77ad-47c2-b798-4042d150e636)

**Role:** Data Analyst · **Tools:** SQL Server (T-SQL, window functions, CTEs, ranking) · **Domain:** Healthcare Operations

---

## Executive Summary

Horizon Hospital's Emergency Room had no structured way to see *where* patient flow was breaking down — which departments were overloaded, which patients were waiting longest, and whether satisfaction was tracking with operational performance. This project turns raw ER admission records into a decision-ready SQL analysis covering patient demographics, wait-time bottlenecks, department-level workload, and satisfaction drivers — with the goal of giving hospital administrators a clear, evidence-based basis for staffing and process decisions.

---

## Business Problem

Hospital leadership faced four blind spots:
- No visibility into **which departments** had the longest wait times or the lowest patient satisfaction.
- No way to identify **peak admission periods** to plan staffing proactively instead of reactively.
- No structured comparison of **patient experience vs. operational load** — so it was unclear whether long waits were actually driving down satisfaction.
- No repeatable reporting layer — every question required a new manual data pull.

## Objectives

- **Improve patient care** — surface where satisfaction is lowest and why.
- **Optimize resource allocation** — quantify departmental workload and peak-hour patterns.
- **Improve operational efficiency** — flag abnormal wait-time and admission spikes.
- **Enable data-driven decisions** — replace ad-hoc requests with a reusable SQL reporting layer stakeholders can query directly.

## Dataset

Patient-level ER admission records including demographics (age, gender, race), admission and discharge timestamps, department referral, wait time, and satisfaction score.

![Schema](https://github.com/user-attachments/assets/2ca2ded4-99b8-40b7-a666-d4ee4148d077)

---

## Analytical Approach

Rather than treating this as a flat query exercise, the analysis was structured around four business questions, using progressively more advanced SQL techniques to answer each:

| Business Question | SQL Technique Used |
|---|---|
| Which departments are overloaded, and by how much? | `GROUP BY`, `HAVING` |
| Which patients/departments are outliers on wait time? | Subqueries, comparison against aggregate benchmarks |
| How does each department rank internally on wait time and satisfaction? | Window functions — `RANK() OVER (PARTITION BY ...)` |
| What's the *real* second-most-common patient demographic, not just the mode? | Ranked CTE — `RANK() OVER (ORDER BY COUNT(*) DESC)` |
| How long is each patient actually in the hospital? | `DATEDIFF`, date/time logic |
| Are there seasonal or weekday admission patterns? | `DATENAME`, `DATEPART`, `MONTH()`, `YEAR()` time-series grouping |

### Representative Queries

**Department-level wait-time ranking (window function):**
```sql
SELECT Patient_Id, Department_Referral, Patient_Waittime,
       RANK() OVER (PARTITION BY Department_Referral ORDER BY Patient_Waittime DESC) AS Wait_Rank
FROM Horizon_Hospital;
```

**Patients waiting longer than the hospital-wide average (subquery benchmark):**
```sql
SELECT * FROM Horizon_Hospital
WHERE Patient_Waittime > (SELECT AVG(Patient_Waittime) FROM Horizon_Hospital);
```

**True second-most-common patient demographic (ranked CTE, not just MODE):**
```sql
SELECT TOP 1 Patient_Race FROM (
    SELECT Patient_Race, RANK() OVER (ORDER BY COUNT(*) DESC) AS rnk
    FROM Horizon_Hospital
    GROUP BY Patient_Race
) AS RankedRaces
WHERE rnk = 2;
```

**Top 3 departments by patient satisfaction:**
```sql
SELECT TOP 3 Department_Referral, AVG(Patient_Satisfaction_Score) AS Avg_Satisfaction
FROM Horizon_Hospital
GROUP BY Department_Referral
ORDER BY Avg_Satisfaction DESC;
```

*(30+ supporting queries — covering admissions trends, discharge time calculations, weekday patterns, and null-data handling — are available in the full SQL script in this repo.)*

---

## Key Insights

- **Wait time is concentrated in a small number of departments** — ranking wait times within each department (rather than hospital-wide) revealed specific bottlenecks that a flat average would have hidden.
- **Satisfaction and wait time move together** — departments with above-average wait times consistently showed lower satisfaction scores, giving a measurable link between operational delay and patient experience.
- **Admissions are not evenly distributed** — grouping by month and weekday surfaced predictable peak periods, meaning current staffing is likely static against a demand curve that isn't.
- **A handful of departments carry disproportionate patient volume**, based on the `HAVING COUNT(*) > 10` threshold — these are the departments where a staffing change would have the largest impact.

## Business Recommendations

- **Reallocate staff to the identified peak windows** (by month/weekday) instead of flat shift scheduling.
- **Prioritize process review in the departments with the highest wait-time rank**, since they disproportionately drag down overall satisfaction.
- **Set an automated wait-time threshold alert** (e.g., flag any patient exceeding the hospital-wide average, as isolated in the analysis) so delays are caught in real time, not after the fact.
- **Build this SQL logic into a recurring report** (or a Power BI layer on top) so administrators can self-serve these answers instead of requesting ad-hoc pulls.

---

## Skills Demonstrated

`SQL Server` · `Window Functions (RANK, PARTITION BY)` · `CTEs` · `Correlated & Scalar Subqueries` · `Date/Time Analysis` · `Aggregation & HAVING Filters` · `Healthcare Operations Analytics` · `Business Recommendation Writing`
