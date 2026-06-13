Netflix Churn Analysis 2023-2024

Dashboard Preview

Show Image


🔹 Project Overview

This project analyzes why customers leave Netflix using a dataset of 1.2 million customers across 2023-2024. The analysis covers churn patterns by region, subscription plan, age group, payment history, and customer behavior — built to answer one core business question:


"Why are customers leaving and what can we do about it?"



The dataset was synthetically generated with realistic correlations for educational and portfolio purposes.


🔹 Tools & Technologies

ToolPurposePython (Pandas, NumPy)Dataset generation with realistic correlationsPostgreSQL 18Data storage and SQL analysispgAdmin 4Query execution and database managementPower BI DesktopInteractive dashboardDAXCustom measures and KPI calculations


🔹 Dataset Overview

PropertyValueTotal Rows1,200,000Columns17Years Covered2023-2024Overall Churn Rate5.35%

Columns

customer_id, join_date, subscription_plan, customer_satisfaction_score, daily_watch_time_hours, engagement_rate, device_used_most_often, genre_preference, region, payment_history, churn_status, support_queries_logged, age, monthly_income, promotional_offers_used, number_of_profiles_created

Realistic Correlations Built Into Dataset


Churned customers → satisfaction score 1-4, retained → 6-10
Churned customers → 6-10 support queries, retained → 0-4
Smart TV users → 4.19 hrs watch time, Mobile → 1.53 hrs
Premium plan income → avg $8,500, Basic → avg $2,800
Africa highest churn (8.72%), Asia lowest (4.36%)
Basic plan highest churn (9.15%), Premium lowest (3.27%)



🔹 Key KPIs

MetricValueTotal Customers1.2MChurned Customers64.2KRetained Customers1.14MOverall Churn Rate5.35%2023 Churn Rate7.49%2024 Churn Rate3.21%YOY Churn Improvement-57%


🔹 SQL Analysis — 10 Business Questions

Q1. Total Churned vs Retained Customers

sqlSELECT 
    SUM(CASE WHEN churn_status = 'Yes' THEN 1 ELSE 0 END) AS churn_customer,
    SUM(CASE WHEN churn_status = 'No' THEN 1 ELSE 0 END) AS retained_customer,
    ROUND(SUM(CASE WHEN churn_status = 'Yes' THEN 1 ELSE 0 END)*100.0/COUNT(*),2) AS churn_pct
FROM netflix_dataset;

Result: 64,199 churned | 1,135,801 retained | 5.35% churn rate


Q2. Churn Rate by Subscription Plan

sqlSELECT subscription_plan,
    SUM(CASE WHEN churn_status = 'Yes' THEN 1 ELSE 0 END) AS churn_customer,
    SUM(CASE WHEN churn_status = 'No' THEN 1 ELSE 0 END) AS retained_customer,
    ROUND(SUM(CASE WHEN churn_status = 'Yes' THEN 1 ELSE 0 END)*100.0/COUNT(*),2) AS churn_pct
FROM netflix_dataset
GROUP BY subscription_plan
ORDER BY churn_pct DESC;

Result: Basic 9.15% > Standard 6.16% > Premium 3.27%
Insight: Basic plan customers are 3x more likely to churn than Premium


Q3. Average Daily Watch Time by Device

sqlSELECT device_used_most_often,
    ROUND(AVG(daily_watch_time_hours), 2) AS avg_time
FROM netflix_dataset
GROUP BY device_used_most_often
ORDER BY avg_time DESC;

Result: Smart TV 4.19hrs > Desktop 3.43hrs > Laptop 2.83hrs > Tablet 2.23hrs > Mobile 1.53hrs
Insight: Smart TV users watch 2.7x more than Mobile users


Q4. Churn Rate by Region

sqlSELECT region,
    ROUND(SUM(CASE WHEN churn_status = 'Yes' THEN 1 ELSE 0 END)*100.0/COUNT(*),2) AS churn_pct
FROM netflix_dataset
GROUP BY region
ORDER BY churn_pct DESC;

Result: Africa 8.72% > South America 7.03% > Europe 6.04% > North America 5.20% > Asia 4.36%
Insight: Africa has 2x higher churn than Asia


Q5. Genre Preference vs Churn Rate

sqlSELECT genre_preference,
    ROUND(AVG(customer_satisfaction_score), 2) AS avg_score,
    ROUND(SUM(CASE WHEN churn_status = 'Yes' THEN 1 ELSE 0 END)*100.0/COUNT(*), 2) AS churn_pct
FROM netflix_dataset
GROUP BY genre_preference
ORDER BY churn_pct DESC;

Result: Action fans highest churn (6.31%), Sci-Fi fans most loyal (6.15%)


Q6. Average & Median Income: Churned vs Retained

sqlSELECT churn_status,
    ROUND(AVG(monthly_income), 2) AS avg_income,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY monthly_income) AS median_income
FROM netflix_dataset
GROUP BY churn_status
ORDER BY avg_income DESC;

Result: Retained median $5,223 vs Churned median $4,013
Insight: Churned customers earn $1,210 less — price sensitivity is a key churn driver


Q7. Delayed Payment and Churn

sqlSELECT COUNT(*) AS delayed_and_churned
FROM netflix_dataset
WHERE payment_history = 'Delayed'
AND churn_status = 'Yes';

Result: 22,305 customers with delayed payment also churned


Q8. Monthly Customer Joining Trend with Growth %

sqlWITH A AS(
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

Insight: Month-over-month growth tracked across 24 months using LAG window function


Q9. Churn Rate by Age Group and Subscription Plan

sqlSELECT 
    CASE WHEN age BETWEEN 18 AND 25 THEN '18-25'
         WHEN age BETWEEN 26 AND 35 THEN '26-35'
         WHEN age BETWEEN 36 AND 50 THEN '36-50'
         WHEN age >= 50 THEN '50+' 
    END AS age_group,
    subscription_plan,
    SUM(CASE WHEN churn_status = 'Yes' THEN 1 ELSE 0 END) AS churn_customer,
    ROUND(SUM(CASE WHEN churn_status = 'Yes' THEN 1 ELSE 0 END)*100.0/COUNT(*),2) AS churn_pct
FROM netflix_dataset
GROUP BY age_group, subscription_plan
ORDER BY churn_pct DESC;

Result: 36-50 Basic plan = highest churn (9.19%), Premium users low churn across all ages (3.18-3.33%)
Insight: Plan type matters more than age for predicting churn


Q10. Support Queries vs Churn (Strongest Finding)

sqlSELECT churn_status,
    ROUND(AVG(support_queries_logged), 2) AS avg_support_queries
FROM netflix_dataset
GROUP BY churn_status;

Result: Churned customers avg 7.99 queries vs Retained avg 2.00
Insight: Churned customers log 4x more support queries — strongest churn predictor in dataset


🔹 DAX Measures

dax-- Total Customers
Total_customer = COUNTROWS(netflix_dataset)

-- Churned Customers
Churn_customer = 
COUNTROWS(FILTER(netflix_dataset, netflix_dataset[Churn Status (Yes/No)] = "Yes"))

-- Retained Customers
Retained_customer = 
COUNTROWS(FILTER(netflix_dataset, netflix_dataset[Churn Status (Yes/No)] = "No"))

-- Churn Rate %
Churn% = 
DIVIDE(
    COUNTROWS(FILTER(netflix_dataset, netflix_dataset[Churn Status (Yes/No)] = "Yes")),
    COUNTROWS(netflix_dataset)
) * 100

-- YTD Churn Rate (changes with Year slicer)
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

-- Average Support Queries
Avg_Support_Queries = AVERAGE(netflix_dataset[Support Queries Logged])

-- Year Column
Year = YEAR(netflix_dataset[Join Date])


🔹 Dashboard Story

The dashboard answers 4 key business questions:

QuestionFinding🌍 WHERE are they leaving from?Africa 2x higher churn than Asia💳 WHO is most at risk?Basic plan customers — 3x riskier than Premium📅 WHEN did churn change?Churn fell 57% from 2023 to 2024❓ WHY are they leaving?High support queries + low satisfaction = churn


🔹 Key Business Insights


Plan is the #1 churn predictor — Basic plan customers churn at 9.15% vs Premium at 3.27%. Upselling Basic users could significantly reduce churn.
Support queries strongly predict churn — Customers logging 6+ support queries always churned. Proactive customer support intervention could retain these customers.
Africa and South America need attention — Highest churn regions. Localized pricing or content strategy could improve retention.
Churn improved 57% from 2023 to 2024 — From 7.49% to 3.21%, suggesting retention strategies are working.
Income gap confirms price sensitivity — Churned customers have $1,210 lower median income. Flexible pricing plans could help retain lower-income segments.



🔹 Dashboard Features


Interactive Year slicer (2023/2024)
Device filter slicer
KPI cards with YTD churn rate
Conditional formatting (darker red = higher churn)
2023 vs 2024 monthly trend comparison
Churn by Region, Plan, Age Group, Payment History



📌 Note

This project was built as part of my data analytics learning journey. The dataset was synthetically generated using Python with realistic correlations to simulate real-world Netflix churn patterns.
