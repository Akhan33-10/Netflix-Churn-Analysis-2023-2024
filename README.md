# Netflix Churn Analysis 2023-2024

## Dashboard Preview
![Dashboard Preview](Dashboard.png)

---

## Project Overview

This project analyzes why customers leave Netflix using a dataset of 1.2 million customers across 2023-2024. Built to answer one core business question:

"Why are customers leaving and what can we do about it?"

---

## Tools & Technologies

- Python (Pandas, NumPy) — Dataset generation
- PostgreSQL 18 — Data storage and SQL analysis
- pgAdmin 4 — Query execution
- Power BI Desktop — Interactive dashboard
- DAX — Custom measures and KPIs

---

## Dataset Overview

- Total Rows: 1,200,000
- Columns: 17
- Years: 2023–2024
- Overall Churn Rate: 5.35%

---

## Key KPIs

- Total Customers: 1.2M
- Churned Customers: 64.2K
- Retained Customers: 1.14M
- Overall Churn Rate: 5.35%
- 2023 Churn Rate: 7.49%
- 2024 Churn Rate: 3.21%
- YOY Improvement: -57%

---

## SQL Analysis

### Q1. Total Churned vs Retained
SELECT 
    SUM(CASE WHEN churn_status = 'Yes' THEN 1 ELSE 0 END) AS churn_customer,
    SUM(CASE WHEN churn_status = 'No' THEN 1 ELSE 0 END) AS retained_customer,
    ROUND(SUM(CASE WHEN churn_status = 'Yes' THEN 1 ELSE 0 END)*100.0/COUNT(*),2) AS churn_pct
FROM netflix_dataset;

Result: 64,199 churned | 1,135,801 retained | 5.35%

---

### Q2. Churn by Subscription Plan
SELECT subscription_plan,
    SUM(CASE WHEN churn_status = 'Yes' THEN 1 ELSE 0 END) AS churn_customer,
    ROUND(SUM(CASE WHEN churn_status = 'Yes' THEN 1 ELSE 0 END)*100.0/COUNT(*),2) AS churn_pct
FROM netflix_dataset
GROUP BY subscription_plan
ORDER BY churn_pct DESC;

Result: Basic 9.15% > Standard 6.16% > Premium 3.27%
Insight: Basic plan customers are 3x more likely to churn than Premium

---

### Q3. Avg Watch Time by Device
SELECT device_used_most_often,
    ROUND(AVG(daily_watch_time_hours), 2) AS avg_time
FROM netflix_dataset
GROUP BY device_used_most_often
ORDER BY avg_time DESC;

Result: Smart TV 4.19hrs > Desktop 3.43hrs > Laptop 2.83hrs > Tablet 2.23hrs > Mobile 1.53hrs
Insight: Smart TV users watch 2.7x more than Mobile users

---

### Q4. Churn by Region
SELECT region,
    ROUND(SUM(CASE WHEN churn_status = 'Yes' THEN 1 ELSE 0 END)*100.0/COUNT(*),2) AS churn_pct
FROM netflix_dataset
GROUP BY region
ORDER BY churn_pct DESC;

Result: Africa 8.72% > South America 7.03% > Europe 6.04% > North America 5.20% > Asia 4.36%
Insight: Africa has 2x higher churn than Asia

---

### Q5. Genre vs Churn Rate
SELECT genre_preference,
    ROUND(AVG(customer_satisfaction_score), 2) AS avg_score,
    ROUND(SUM(CASE WHEN churn_status = 'Yes' THEN 1 ELSE 0 END)*100.0/COUNT(*), 2) AS churn_pct
FROM netflix_dataset
GROUP BY genre_preference
ORDER BY churn_pct DESC;

Result: Action highest churn (6.31%), Sci-Fi most loyal (6.15%)

---

### Q6. Income: Churned vs Retained
SELECT churn_status,
    ROUND(AVG(monthly_income), 2) AS avg_income,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY monthly_income) AS median_income
FROM netflix_dataset
GROUP BY churn_status
ORDER BY avg_income DESC;

Result: Retained median $5,223 vs Churned median $4,013
Insight: Churned customers earn $1,210 less — price sensitivity drives churn

---

### Q7. Delayed Payment and Churn
SELECT COUNT(*) AS delayed_and_churned
FROM netflix_dataset
WHERE payment_history = 'Delayed'
AND churn_status = 'Yes';

Result: 22,305 customers with delayed payment also churned

---

### Q8. Monthly Trend with Growth %
WITH A AS(
    SELECT EXTRACT(MONTH FROM join_date) AS "month",
           EXTRACT(YEAR FROM join_date) AS "year",
           COUNT(*) AS total_customer
    FROM netflix_dataset
    GROUP BY EXTRACT(YEAR FROM join_date), EXTRACT(MONTH FROM join_date)
)
SELECT *,
    LAG(total_customer,1) OVER(ORDER BY "year","month") AS prev_customer,
    ROUND((total_customer - LAG(total_customer,1) OVER(ORDER BY "year","month"))
    / LAG(total_customer,1) OVER(ORDER BY "year","month")::DECIMAL * 100.0, 2) AS growth_pct
FROM A
ORDER BY "year","month";

Insight: Month-over-month growth tracked using LAG window function

---

### Q9. Churn by Age Group and Plan
SELECT 
    CASE WHEN age BETWEEN 18 AND 25 THEN '18-25'
         WHEN age BETWEEN 26 AND 35 THEN '26-35'
         WHEN age BETWEEN 36 AND 50 THEN '36-50'
         WHEN age >= 50 THEN '50+' 
    END AS age_group,
    subscription_plan,
    ROUND(SUM(CASE WHEN churn_status = 'Yes' THEN 1 ELSE 0 END)*100.0/COUNT(*),2) AS churn_pct
FROM netflix_dataset
GROUP BY age_group, subscription_plan
ORDER BY churn_pct DESC;

Result: 36-50 Basic = highest churn (9.19%), Premium lowest across all ages
Insight: Plan type matters more than age for predicting churn

---

### Q10. Support Queries vs Churn
SELECT churn_status,
    ROUND(AVG(support_queries_logged), 2) AS avg_support_queries
FROM netflix_dataset
GROUP BY churn_status;

Result: Churned avg 7.99 queries vs Retained avg 2.00
Insight: Churned customers log 4x more queries — strongest churn predictor

---

## DAX Measures

Total_customer = COUNTROWS(netflix_dataset)

Churn_customer = 
COUNTROWS(FILTER(netflix_dataset, netflix_dataset[Churn Status (Yes/No)] = "Yes"))

Retained_customer = 
COUNTROWS(FILTER(netflix_dataset, netflix_dataset[Churn Status (Yes/No)] = "No"))

Churn% = 
DIVIDE(
    COUNTROWS(FILTER(netflix_dataset, netflix_dataset[Churn Status (Yes/No)] = "Yes")),
    COUNTROWS(netflix_dataset)
) * 100

YTD_Churn_Rate =
VAR SelectedYear = SELECTEDVALUE(netflix_dataset[Year])
RETURN
IF(
    ISBLANK(SelectedYear),
    DIVIDE(
        COUNTROWS(FILTER(netflix_dataset, netflix_dataset[Churn Status (Yes/No)] = "Yes")),
        COUNTROWS(netflix_dataset)
    ) * 100,
    CALCULATE(
        DIVIDE(
            COUNTROWS(FILTER(netflix_dataset, netflix_dataset[Churn Status (Yes/No)] = "Yes")),
            COUNTROWS(netflix_dataset)
        ) * 100,
        netflix_dataset[Year] = SelectedYear
    )
)

Year = YEAR(netflix_dataset[Join Date])

---

## Dashboard Story

WHERE  — Africa 2x higher churn than Asia
WHO    — Basic plan 3x riskier than Premium
WHEN   — Churn fell 57% from 2023 to 2024
WHY    — High support queries + low satisfaction = churn

---

## Key Business Insights

1. Plan is the #1 churn predictor — Basic 9.15% vs Premium 3.27%
2. Support queries strongly predict churn — 6+ queries = always churned
3. Africa and South America need retention strategy
4. Churn improved 57% from 2023 to 2024
5. Churned customers earn $1,210 less — flexible pricing could help

---

## Note

This project was built as part of my data analytics learning journey.
Dataset was synthetically generated using Python with realistic correlations.
