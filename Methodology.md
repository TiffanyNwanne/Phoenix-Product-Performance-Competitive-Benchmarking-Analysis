# Methodology

## **Objective**

The objective of this analysis is to evaluate Product Phoenix’s performance over a five-year period across key metrics including sales, revenue, marketing, and user engagement. This evaluation is compared directly with a competing product (e.g., PS4) to inform the decision on whether to discontinue or reposition Phoenix.

## **Data Sources**

- Internal Sales & Marketing Data: Units sold, revenue, marketing spend, engagement metrics (Phoenix).
- Simulated Competitor Data: Equivalent KPIs for benchmark comparison.
- Period Covered: 2020 – 2024
- Granularity: Quarterly and regional breakdown (NA, EU, APAC)

## **Tools & Environment**

- Database Engine: PostgreSQL
- Visualization: Tableau (for post-analysis dashboarding)
- Languages Used: SQL (PostgreSQL dialect), optionally Python for preprocessing
- Dataset: phoenix_competitor_data.csv loaded into PostgreSQL as table phoenix_data

## **Data Exploration**

Initial queries were used to understand the dataset’s structure, completeness, and distributions.

```sql
-- Preview data
SELECT * FROM phoenix_data LIMIT 10;

-- View schema
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'phoenix_data';

-- Count records per year and region
SELECT year, region, COUNT(*) AS count
FROM phoenix_data
GROUP BY year, region
ORDER BY year, region;
```

```sql
-- Preview data
SELECT * 
FROM phoenix_data 
LIMIT 10;

-- View schema
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_schema = 'public'  -- specify schema if needed
  AND table_name = 'phoenix_data';

-- Count records per year and region
SELECT year, region, COUNT(*) AS record_count
FROM phoenix_data
GROUP BY year, region
ORDER BY year, region;

```

## **Data Cleaning**

Data was checked for anomalies, nulls, and duplicates. Cleaned results were saved in a working table for consistent analysis.

```sql
-- Identify nulls or invalid revenue entries
SELECT *
FROM phoenix_data
WHERE units_sold_phoenix IS NULL OR revenue_phoenix < 100000;

-- Remove or exclude records with null/missing core metrics
CREATE TEMP TABLE phoenix_cleaned AS
SELECT *
FROM phoenix_data
WHERE units_sold_phoenix IS NOT NULL AND revenue_phoenix >= 100000;

-- Check for duplicates
SELECT year, quarter, region, COUNT(*)
FROM phoenix_cleaned
GROUP BY year, quarter, region
HAVING COUNT(*) > 1;
```

```sql
-- Identify records with NULL or potentially invalid revenue entries
SELECT *
FROM phoenix_data
WHERE units_sold_phoenix IS NULL 
   OR revenue_phoenix < 100000;

-- Create a cleaned temporary table excluding invalid/missing core metrics
CREATE TEMP TABLE phoenix_cleaned AS
SELECT *
FROM phoenix_data
WHERE units_sold_phoenix IS NOT NULL 
  AND revenue_phoenix >= 100000;

-- Check for potential duplicate records by key fields
SELECT year, quarter, region, COUNT(*) AS duplicate_count
FROM phoenix_cleaned
GROUP BY year, quarter, region
HAVING COUNT(*) > 1;
```

## Analysis

**Time Series Analysis**

Yearly sales and revenue trends were analyzed to understand growth or decline patterns.

```sql
SELECT
  year,
  SUM(units_sold_phoenix) AS total_sales_phoenix,
  SUM(revenue_phoenix) AS total_revenue_phoenix,
  SUM(units_sold_competitor) AS total_sales_competitor,
  SUM(revenue_competitor) AS total_revenue_competitor
FROM phoenix_cleaned
GROUP BY year
ORDER BY year;
```

```sql
-- Aggregate annual sales and revenue for Phoenix and its competitor
SELECT
  year,
  SUM(units_sold_phoenix)     AS total_units_phoenix,
  SUM(revenue_phoenix)        AS total_revenue_phoenix,
  SUM(units_sold_competitor)  AS total_units_competitor,
  SUM(revenue_competitor)     AS total_revenue_competitor
FROM phoenix_cleaned
GROUP BY year
ORDER BY year;

```

**Marketing Effectiveness**

CTR and marketing spend were evaluated for both products.

```sql
SELECT
  year,
  ROUND(AVG(marketing_spend_phoenix), 2) AS avg_spend_phoenix,
  ROUND(AVG(ctr_phoenix), 2) AS avg_ctr_phoenix,
  ROUND(AVG(marketing_spend_competitor), 2) AS avg_spend_competitor,
  ROUND(AVG(ctr_competitor), 2) AS avg_ctr_competitor
FROM phoenix_cleaned
GROUP BY year
ORDER BY year;
```

```sql
-- Calculate average marketing spend and CTR by year for Phoenix and its competitor
SELECT
  year,
  ROUND(AVG(marketing_spend_phoenix), 2)     AS avg_marketing_spend_phoenix,
  ROUND(AVG(ctr_phoenix), 2)                 AS avg_ctr_phoenix,
  ROUND(AVG(marketing_spend_competitor), 2)  AS avg_marketing_spend_competitor,
  ROUND(AVG(ctr_competitor), 2)              AS avg_ctr_competitor
FROM phoenix_cleaned
GROUP BY year
ORDER BY year;
```

**Customer Engagement Analysis**

Metrics such as monthly active users, play hours, and retention were studied.

```sql
SELECT
  year,
  ROUND(AVG(monthly_active_users_phoenix)) AS avg_mau,
  ROUND(AVG(avg_play_hours_phoenix), 2) AS avg_play_time,
  ROUND(AVG(subscription_rate_phoenix), 2) AS avg_subscription_rate,
  ROUND(AVG(online_retention_phoenix), 2) AS avg_retention_rate
FROM phoenix_cleaned
GROUP BY year
ORDER BY year;
```

```sql
-- Compute yearly averages for Phoenix user engagement and retention metrics
SELECT
  year,
  ROUND(AVG(monthly_active_users_phoenix))        AS avg_monthly_active_users,
  ROUND(AVG(avg_play_hours_phoenix), 2)           AS avg_play_hours,
  ROUND(AVG(subscription_rate_phoenix), 2)        AS avg_subscription_rate,
  ROUND(AVG(online_retention_phoenix), 2)         AS avg_online_retention_rate
FROM phoenix_cleaned
GROUP BY year
ORDER BY year;
```

**User Engagement Quality**

```sql
SELECT 
    AVG(Monthly_Active_Users_Phoenix) AS Avg_Monthly_Active_Users,
    AVG(Avg_Play_Hours_Phoenix) AS Avg_Play_Hours,
    AVG(Subscription_Rate_Phoenix) AS Avg_Subscription_Rate,
    AVG(Online_Retention_Phoenix) AS Avg_Online_Retention
FROM 
    marketing_data;
```

```sql
-- Calculate overall averages for Phoenix user engagement and retention
SELECT 
    ROUND(AVG(monthly_active_users_phoenix))        AS avg_monthly_active_users,
    ROUND(AVG(avg_play_hours_phoenix), 2)           AS avg_play_hours,
    ROUND(AVG(subscription_rate_phoenix), 2)        AS avg_subscription_rate,
    ROUND(AVG(online_retention_phoenix), 2)         AS avg_online_retention
FROM marketing_data;
```

**Marketing Performance** 

```sql
SELECT 
    SUM(Marketing_Spend_Phoenix) / SUM(Ad_Reach_Phoenix) AS CPR,
    SUM(Marketing_Spend_Phoenix) / SUM(Units_Sold_Phoenix) AS CPA,
    (SUM(Units_Sold_Phoenix) / SUM(Ad_Reach_Phoenix)) * 100 AS Conversion_Rate_Percent
FROM 
    marketing_data;
```

```sql
-- Calculate CPR, CPA, and conversion rate for Phoenix
SELECT 
    ROUND(SUM(marketing_spend_phoenix) / NULLIF(SUM(ad_reach_phoenix), 0), 2) AS cpr,
    ROUND(SUM(marketing_spend_phoenix) / NULLIF(SUM(units_sold_phoenix), 0), 2) AS cpa,
    ROUND((SUM(units_sold_phoenix) / NULLIF(SUM(ad_reach_phoenix), 0)) * 100, 2) AS conversion_rate_percent
FROM marketing_data;
```

**Regional Performance**

Sales performance was compared across NA, EU, and APAC.

```sql
SELECT
  year, region,
  SUM(units_sold_phoenix) AS sales_phoenix,
  SUM(units_sold_competitor) AS sales_competitor
FROM phoenix_cleaned
GROUP BY year, region
ORDER BY year, region;
```

```sql
-- Compare Phoenix and competitor sales by year and region
SELECT
  year,
  region,
  SUM(units_sold_phoenix)     AS total_units_phoenix,
  SUM(units_sold_competitor)  AS total_units_competitor
FROM phoenix_cleaned
GROUP BY year, region
ORDER BY year, region;
```

**Competitive Gap Measurement**

To quantify performance gaps:

```sql
SELECT
  year,
  ROUND(AVG(units_sold_competitor - units_sold_phoenix), 0) AS avg_sales_gap,
  ROUND(AVG(ctr_competitor - ctr_phoenix), 2) AS avg_ctr_gap
FROM phoenix_cleaned
GROUP BY year
ORDER BY year;
```

```sql
-- Calculate average annual gap between competitor and Phoenix in sales and CTR
SELECT
  year,
  ROUND(AVG(units_sold_competitor - units_sold_phoenix), 0) AS avg_units_sold_gap,
  ROUND(AVG(ctr_competitor - ctr_phoenix), 2)               AS avg_ctr_gap
FROM phoenix_cleaned
GROUP BY year
ORDER BY year;
```

## **Efficiency & Performance Ratios**

To evaluate the effectiveness of Phoenix’s marketing and user engagement strategies, we incorporated key marketing performance metrics: cost per reach (CPR), cost per acquisition (CPA), conversion rates, and user averages. These help determine whether spend is translating into value.

**a. Cost Per Reach (CPR)**

Measures the average cost to reach one potential customer. A lower CPR indicates efficient marketing reach.

```sql
SELECT 
    SUM(marketing_spend_phoenix) / SUM(ad_reach_phoenix) AS cpr
FROM 
    phoenix_cleaned;
```

**b. Cost Per Acquisition (CPA)**

Represents the marketing cost required to convert one sale. A key ROI indicator.

```sql
SELECT 
    SUM(marketing_spend_phoenix) / SUM(units_sold_phoenix) AS cpa
FROM 
    phoenix_cleaned;
```

**c. Conversion Rate (% Reach → Sale)**

Indicates what percentage of users who saw ads actually purchased the product.

```sql
SELECT 
    (SUM(units_sold_phoenix) / SUM(ad_reach_phoenix)) * 100 AS conversion_rate_percent
FROM 
    phoenix_cleaned;
```

**d. Average Monthly Active Users**

Helps assess platform stickiness and recurring engagement.

```sql
SELECT 
    AVG(monthly_active_users_phoenix) AS avg_monthly_active_users
FROM 
    phoenix_cleaned;
```

**e. Average Play Hours**

Assesses product engagement by tracking average usage time per user.

```sql
SELECT 
    AVG(avg_play_hours_phoenix) AS avg_play_hours
FROM 
    phoenix_cleaned;
```

**f. Average Subscription Rate**

Indicates the monetization efficiency and product stickiness via subscription adoption.

```sql
SELECT 
    AVG(subscription_rate_phoenix) AS avg_subscription_rate
FROM 
    phoenix_cleaned;
```

**g. Average Online Retention**

Measures user satisfaction and platform loyalty.

```sql
SELECT 
    AVG(online_retention_phoenix) AS avg_online_retention
FROM 
    phoenix_cleaned;
```

**h. Consolidated Phoenix vs Competitor Metrics**

This aggregate comparison summarizes the total and efficiency metrics for both products.

```sql
SELECT 
    -- Units Sold
    SUM(units_sold_phoenix) AS total_units_sold_phoenix,
    SUM(units_sold_competitor) AS total_units_sold_competitor,

    -- Marketing Spend
    SUM(marketing_spend_phoenix) AS total_marketing_spend_phoenix,
    SUM(marketing_spend_competitor) AS total_marketing_spend_competitor,

    -- Ad Reach
    SUM(ad_reach_phoenix) AS total_ad_reach_phoenix,
    SUM(ad_reach_competitor) AS total_ad_reach_competitor,

    -- CPA
    SUM(marketing_spend_phoenix) / SUM(units_sold_phoenix) AS cpa_phoenix,
    SUM(marketing_spend_competitor) / SUM(units_sold_competitor) AS cpa_competitor,

    -- CPR
    SUM(marketing_spend_phoenix) / SUM(ad_reach_phoenix) AS cpr_phoenix,
    SUM(marketing_spend_competitor) / SUM(ad_reach_competitor) AS cpr_competitor,

    -- Conversion Rate
    (SUM(units_sold_phoenix) / SUM(ad_reach_phoenix)) * 100 AS conversion_rate_phoenix,
    (SUM(units_sold_competitor) / SUM(ad_reach_competitor)) * 100 AS conversion_rate_competitor

FROM 
    phoenix_cleaned;
```

## **Outcome**

These queries formed the backbone of the analysis presented in the executive summary and clarified key business questions related to Phoenix’s marketing investment, product engagement, and competitive positioning.