## GA4 Data Analysis Using Bigquery Project For CRO

## Introduction

In today’s competitive eCommerce landscape, understanding customer behavior is key to improving conversion rates. This project, GA4 Data Analysis Using BigQuery for CRO, leverages Google Analytics 4 (GA4) and BigQuery to analyze user data, uncover insights, and optimize conversion rates.

### Objectives

i) Analyze user behavior and engagement metrics.

ii) Identify conversion bottlenecks and drop-offs.

iii) Provide actionable, data-driven recommendations.

iv) Build dashboards to monitor CRO progress.

By integrating GA4 with BigQuery, this project transforms raw data into strategic insights, empowering eCommerce businesses to enhance performance and boost conversions.

## Data Collection & Setup

### Integrate GA4 & BigQuery

i) Enable BigQuery Linking in GA4: Link your GA4 property to BigQuery to export raw event data automatically.

ii) Set Up a BigQuery Project: Create a project in BigQuery and define a dataset to store GA4 data.

iii) Verify Data Export: Ensure that data is streaming correctly from GA4 to BigQuery by checking tables for events, user properties, and session details.

This integration forms the foundation for advanced data analysis and actionable insights.

## Let's Start SQL Query With Bigquery to get Better Actionable Insights:

### SQL Query 1: ✅ E-commerce Metrics Analysis for CRO Optimization

E-commerce Metrics Analysis for CRO Optimization focuses on evaluating key performance indicators such as conversion rates, cart abandonment, and customer lifetime value to uncover growth opportunities. By leveraging data-driven insights, it helps optimize the user journey, improve sales, and drive sustainable revenue growth.

#### SQL Code:
```
-- Declare the start and end dates for the analysis period
declare start_date date default '2020-11-01';  
declare end_date date default '2021-01-31';

-- Create a temporary view (CTE) named 'flat_data' for aggregating the data
with flat_data as (
  -- Count distinct users to calculate the total unique users
  SELECT 
    count(distinct user_pseudo_id) as total_user,

    -- Count purchases (event_name='purchase') to calculate the total number of purchases
    countif(event_name='purchase') total_purchase,

    -- Sum the purchase revenue for 'purchase' events to calculate the total revenue
    sum(if(event_name='purchase', ecommerce.purchase_revenue, null)) as total_purchase_revenue,

    -- Sum the total quantity of items sold for 'purchase' events to track the number of items sold
    sum(if (event_name='purchase', ecommerce.total_item_quantity, null)) as total_item_quantity_sold
  -- Define the dataset to pull data from
  FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
  
  -- Filter the data based on the date range between start_date and end_date
  where parse_date('%Y%m%d', event_date) between start_date and end_date
)

-- Select the calculated metrics from the 'flat_data' CTE
select
    -- Total number of unique users
    total_user,

    -- Total number of purchases
    total_purchase,

    -- Total number of items sold
    total_item_quantity_sold,

    -- Total purchase revenue
    total_purchase_revenue,

    -- Calculate the conversion rate as the percentage of users who made a purchase
    round(safe_divide(total_purchase, total_user) * 100, 2) as conversion_rate,

    -- Calculate the average order value (AOV) as total revenue per purchase
    round(safe_divide(total_purchase_revenue, total_purchase), 2) as average_order_value

-- Retrieve the results from 'flat_data'
from flat_data;
```

#### Query Result: 

![image](https://github.com/user-attachments/assets/0e1cb404-6bd2-4f20-9e00-e773f032d06f)

#### Summary & Insight:

I) High cart (90.28%) and checkout (85.31%) abandonment rates highlight significant friction in the purchase journey;
streamlining the process or offering incentives could reduce drop-offs.

II) A low conversion rate (2.11%) suggests optimization opportunities like improving mobile usability, site speed, or checkout flow.

III) With an Average Order Value of $63.63, strategies such as cross-selling and bundling can help boost revenue further.









