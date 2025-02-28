## Numerical Data Analysis using Ga4 & Bigquery

## Introduction

In today’s competitive eCommerce landscape, understanding customer behavior is key to improving conversion rates. This project, GA4 Data Analysis Using BigQuery for CRO, leverages Google Analytics 4 (GA4) and BigQuery to analyze user data, uncover insights, and optimize conversion rates.

### Objectives

i) Analyze user behavior and engagement metrics.

ii) Identify conversion bottlenecks and drop-offs.

iii) Provide actionable, data-driven recommendations.

iv) Build dashboards to monitor CRO progress.


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

-- Create a temporary dataset to calculate key eCommerce metrics
with ecommerce_data as (
  select
    -- Count unique users
    count(distinct user_pseudo_id) as total_users,
    -- Count total purchases
    countif(event_name = 'purchase') as total_purchases,
    -- Sum the total number of items sold
    sum(if(event_name = 'purchase', ecommerce.total_item_quantity, null)) as total_item_quantity_sold,
    -- Sum the total revenue from purchases
    sum(if(event_name = 'purchase', ecommerce.purchase_revenue, null)) as total_purchase_revenue,
    -- Count total 'add to cart' events
    countif(event_name = 'add_to_cart') as total_add_to_cart,
    -- Count total 'begin checkout' events
    countif(event_name = 'begin_checkout') as total_checkouts
  from `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
  where parse_date('%Y%m%d', event_date) between start_date and end_date
)

-- Calculate and display key performance indicators
select
  total_users, -- Total number of unique users
  total_purchases, -- Total number of completed purchases
  total_item_quantity_sold, -- Total quantity of items sold
  total_purchase_revenue, -- Total revenue generated from purchases
  round(safe_divide(total_purchases, total_users) * 100, 2) as conversion_rate, -- Percentage of users who made a purchase
  round(safe_divide(total_purchase_revenue, total_purchases), 2) as avg_order_value, -- Average revenue per order
  round(safe_divide(total_purchase_revenue, total_users), 2) as revenue_per_visitor, -- Revenue generated per unique visitor
  round(safe_divide((total_add_to_cart - total_purchases), total_add_to_cart) * 100, 2) as total_cart_abandonment, -- Percentage of users who added to cart but did not purchase
  round(safe_divide(total_checkouts - total_purchases, total_checkouts) * 100, 2) as checkout_abandonment_rate -- Percentage of users who started checkout but did not complete purchase
from ecommerce_data;

```

#### Query Result: 

![image](https://github.com/user-attachments/assets/2b00020c-6d5f-49c3-84cc-9e1cf99c93c4)


#### Summary & Insight:

I) The store recorded 5,692 total purchases, with 22,720 items sold, generating a total purchase revenue of $362,165. Despite strong sales volume, optimizing conversion rates, reducing cart abandonment, and increasing average order value could further enhance revenue growth.

II) High cart abandonment rate (90.28%) and checkout abandonment rate (85.31%) indicate significant friction in the purchase process, leading to lost sales opportunities. Streamlining the checkout and cart process or offering incentives might help.

III) The conversion rate is low (2.11%), pointing to potential optimization opportunities in areas such as mobile usability, site speed, and the checkout flow. Improving these factors could directly enhance the conversion rate and increase the number of purchases.

IV) The Average Order Value (AOV) is $63.63, suggesting that there is room to boost revenue through strategies like cross-selling and bundling. Increasing AOV can help maximize the value of each customer and improve revenue without needing to increase traffic.

V) The revenue per visitor is relatively low (1.34), suggesting that there may be untapped opportunities to improve the monetization of traffic. Enhancing site engagement, upselling, or offering better incentives could lead to higher revenue per visitor.



### SQL Query 2: ✅ Identifying Website Performance Metrics

#### SQL Code:
```
WITH date_range AS (
  SELECT DATE '2020-11-01' AS start_date, DATE '2021-01-31' AS end_date
),

revenue_data AS (
  SELECT  
     user_pseudo_id,
     SUM(ecommerce.purchase_revenue) AS total_revenue
  FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
  WHERE event_name = 'purchase'
    AND PARSE_DATE('%Y%m%d', event_date) BETWEEN (SELECT start_date FROM date_range) 
                                             AND (SELECT end_date FROM date_range)
  GROUP BY user_pseudo_id
),

visitor_data AS (
  SELECT
     user_pseudo_id,
     COUNT(DISTINCT (SELECT value.int_value FROM UNNEST(event_params) WHERE key = 'ga_session_id')) AS sessions,
     COUNT(DISTINCT CASE 
        WHEN (SELECT value.string_value FROM UNNEST(event_params) WHERE key = 'session_engaged') = '1' 
        THEN (SELECT value.int_value FROM UNNEST(event_params) WHERE key = 'ga_session_id') 
     END) AS engaged_sessions
  FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
  WHERE PARSE_DATE('%Y%m%d', event_date) BETWEEN (SELECT start_date FROM date_range) 
                                             AND (SELECT end_date FROM date_range)
  GROUP BY user_pseudo_id
),

session_time AS (
  SELECT
     user_pseudo_id,
     (SELECT value.int_value FROM UNNEST(event_params) WHERE key = 'ga_session_id') AS ga_session_id,
     MIN(event_timestamp) / 1000000 AS session_start_time,
     MAX(event_timestamp) / 1000000 AS session_end_time
  FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
  WHERE PARSE_DATE('%Y%m%d', event_date) BETWEEN (SELECT start_date FROM date_range) 
                                             AND (SELECT end_date FROM date_range)
  GROUP BY user_pseudo_id, ga_session_id
),

session_duration AS (
  SELECT
     user_pseudo_id,
     AVG(session_end_time - session_start_time) AS avg_session_duration_seconds
  FROM session_time
  GROUP BY user_pseudo_id
),

combined_data AS (
  SELECT
     COUNT(DISTINCT v.user_pseudo_id) AS total_users,
     SUM(v.sessions) AS total_sessions,
     GREATEST(SUM(v.sessions) - SUM(v.engaged_sessions), 0) AS bounced,  -- Ensure bounced is not negative
     SUM(IFNULL(r.total_revenue, 0)) AS total_revenue,
     AVG(s.avg_session_duration_seconds) AS avg_session_duration_seconds
  FROM visitor_data v
  LEFT JOIN revenue_data r
    ON v.user_pseudo_id = r.user_pseudo_id
  LEFT JOIN session_duration s
    ON v.user_pseudo_id = s.user_pseudo_id
)

SELECT
   total_users,
   total_revenue,
   total_sessions,
   bounced,
   avg_session_duration_seconds,
   ROUND(SAFE_DIVIDE(total_revenue, total_users), 2) AS revenue_per_visitor,
   ROUND(SAFE_DIVIDE(bounced, NULLIF(total_sessions, 0)) * 100, 2) AS bounce_rate  -- Fixed bounce rate calculation
FROM combined_data;

```

#### Query Result: 

![image](https://github.com/user-attachments/assets/dd76f7ce-787a-49d6-bad3-998f6aa99651)


#### Summary & Insight:

i) Optimize Landing Pages & RPV: Reduce bounce rate (20.68%) and increase revenue per visitor ($1.34) by improving landing page relevance, visuals, speed, and offering upsells/cross-sells.

ii) Focus on CRO: Leverage high traffic (360,129 sessions) and revenue ($362,165) with targeted promotions and discounts to improve conversion rates.

iii) Boost Engagement: Increase average session duration (160.30 seconds) with personalized content and product recommendations to enhance user interaction and reduce bounce rates.



### SQL Query 3:  ✅ Analyzing Conversion Funnel Data for eCommerce Optimization

This analysis focuses on evaluating key stages in the conversion funnel of an eCommerce store, helping to identify where users drop off and why. By tracking user behavior from arrival to checkout, businesses can uncover areas for improvement in the user journey.

#### SQL Code:
```
DECLARE start_date DATE DEFAULT '2020-11-01';
DECLARE end_date DATE DEFAULT '2021-01-31';

WITH funnel_steps AS (
    SELECT 
        user_pseudo_id,
        event_name,
        event_timestamp
    FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
    WHERE event_name IN ('view_item', 'add_to_cart', 'begin_checkout', 'purchase')
      AND PARSE_DATE('%Y%m%d', _TABLE_SUFFIX) BETWEEN start_date AND end_date
),

total_visitors AS (
    SELECT DISTINCT user_pseudo_id 
    FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
    WHERE PARSE_DATE('%Y%m%d', _TABLE_SUFFIX) BETWEEN start_date AND end_date
),

step_counts AS (
    SELECT 'Total Visitors' AS step, COUNT(DISTINCT user_pseudo_id) AS users FROM total_visitors
    UNION ALL
    SELECT 'View Item', COUNT(DISTINCT user_pseudo_id) FROM funnel_steps WHERE event_name = 'view_item'
    UNION ALL
    SELECT 'Add to Cart', COUNT(DISTINCT user_pseudo_id) FROM funnel_steps WHERE event_name = 'add_to_cart'
    UNION ALL
    SELECT 'Checkout', COUNT(DISTINCT user_pseudo_id) FROM funnel_steps WHERE event_name = 'begin_checkout'
    UNION ALL
    SELECT 'Purchase', COUNT(DISTINCT user_pseudo_id) FROM funnel_steps WHERE event_name = 'purchase'
)

SELECT 
    step,
    users,
    ROUND(SAFE_DIVIDE(users, LAG(users) OVER (ORDER BY 
        CASE 
            WHEN step = 'Total Visitors' THEN 1
            WHEN step = 'View Item' THEN 2
            WHEN step = 'Add to Cart' THEN 3
            WHEN step = 'Checkout' THEN 4
            WHEN step = 'Purchase' THEN 5
        END
    )), 2) AS conversion_rate,
    ROUND(1 - SAFE_DIVIDE(users, LAG(users) OVER (ORDER BY 
        CASE 
            WHEN step = 'Total Visitors' THEN 1
            WHEN step = 'View Item' THEN 2
            WHEN step = 'Add to Cart' THEN 3
            WHEN step = 'Checkout' THEN 4
            WHEN step = 'Purchase' THEN 5
        END
    )), 2) AS abandonment_rate
FROM step_counts
ORDER BY 
    CASE 
        WHEN step = 'Total Visitors' THEN 1
        WHEN step = 'View Item' THEN 2
        WHEN step = 'Add to Cart' THEN 3
        WHEN step = 'Checkout' THEN 4
        WHEN step = 'Purchase' THEN 5
    END;

```

#### Query Result: 

![image](https://github.com/user-attachments/assets/ef495a0f-e4dd-4089-b474-fcd862dafb1d)



#### Summary Insight:

I) Total Visitors (270,154 users) show significant drop-offs across each step, with no conversion rate or abandonment rate at this stage. This highlights the need for engaging landing pages and entry point optimizations to retain users.

II) View Item (61,252 users) have a conversion rate of 0.23% and a high abandonment rate of 0.77. Improving product page content, images, and clear call-to-action buttons could enhance interest and reduce abandonment.

III) Add to Cart (12,545 users) see a slight drop in conversion rate (0.2%) with a higher abandonment rate (0.8%). Consider streamlining the add-to-cart experience, showing quick product details, and offering incentives like free shipping to encourage progression.

IV) Checkout (9,715 users) have a conversion rate of 0.77% and an abandonment rate of 0.23%, showing potential in the checkout stage. Focus on simplifying the checkout process, reducing friction points, and offering clear payment options.

V) Purchase (4,419 users) demonstrate a conversion rate of 0.45% and a relatively high abandonment rate of 0.55%. Offering post-purchase incentives, such as related product recommendations or loyalty points, could boost final conversions.


### SQL Query 4: ✅ Device Category-Based Conversion Analysis

This SQL query analyzes user engagement and conversion metrics based on device categories for a specified date range. It calculates key performance indicators (KPIs) like total users, purchases, item quantities, purchase revenue, and conversion rate for different device categories.

#### SQL Code:
```
-- declare start date and end date for the date range filter
declare start_date date default '2020-11-01';  
declare end_date date default '2021-01-31';

-- select device category and aggregate data for user, purchase, quantity, and revenue
select  
     -- category of the device used by the users
     device.category as device_category,  
     
     -- count of distinct users in the specified date range
     count(distinct user_pseudo_id) as total_user,  
     
     -- count of purchases that occurred during the period
     countif(event_name='purchase') as total_purchase,  
     
     -- sum of the total number of items purchased in the transaction, only for 'purchase' events
     sum(if (event_name='purchase', ecommerce.total_item_quantity, null)) as total_item_quantity,  
     
     -- sum of revenue generated from all purchases, only for 'purchase' events
     sum(if (event_name='purchase', ecommerce.purchase_revenue, null)) as total_purchase_revenue,   
     
     -- calculate conversion rate by dividing purchases by unique users, and multiplying by 100 to get percentage
     round(safe_divide(countif(event_name='purchase'), count(distinct user_pseudo_id)) * 100, 2) as conversion_rate  
     
-- specify the dataset being queried from Google BigQuery sample eCommerce events
from `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`  

-- filter data between the start date and end date
where parse_date('%Y%m%d', event_date) between start_date and end_date  

-- group the results by the device category
group by device_category;

```

#### Query Result: 

![image](https://github.com/user-attachments/assets/534daa4a-043e-479e-9730-59176ac91ce7)

#### Summary Insight:

Summary Insights:
I) Mobile users have the highest conversion rate (2.16%) and account for significant revenue ($146,768), indicating a strong preference for mobile shopping. Optimizing mobile experiences, such as faster load times and simplified navigation, could further enhance conversions.

II) Desktop users generate the highest revenue ($208,815) but have a slightly lower conversion rate (2.03%) compared to mobile. Improving engagement through personalized offers or a smoother checkout process could boost desktop performance.

III) Tablet users have the lowest conversion rate (1.78%) and revenue ($6,582), suggesting potential challenges in usability or relevance. Focused improvements for tablet interfaces could help unlock untapped potential in this segment.


### SQL Query 5: ✅  Device Category Based: Average Order Value , Cart and Checkout Abandonment Analysis

This query helps to analyze key ecommerce metrics, such as cart abandonment rate, checkout abandonment rate, and average order value by device category. It aggregates data from the GA4 eCommerce dataset to provide insights on how users engage with the shopping funnel from page view to purchase.

#### SQL Code:
```
-- Declare start and end date variables for the desired analysis period
declare start_date date default '2020-11-01';  
declare end_date date default '2021-01-31';

-- Creating a common table expression (CTE) to aggregate data
with flat_data as (
  SELECT 
    -- Device category: distinguishes the device type (e.g., mobile, tablet, desktop)
    device.category as device_category,
    
    -- Count distinct users who made a purchase event
    count(distinct if(event_name='purchase', user_pseudo_id, null)) as purchase_users,
    
    -- Sum of purchase revenue for the 'purchase' event
    sum(if(event_name='purchase', ecommerce.purchase_revenue, null)) as purchase_revenue,
    
    -- Count the number of page view events
    countif(event_name='page_view') as page_view,
    
    -- Count the number of add to cart events
    countif(event_name='add_to_cart') as add_to_cart,
    
    -- Count the number of begin checkout events
    countif(event_name='begin_checkout') as begin_checkout,
    
    -- Count the number of purchase events
    countif(event_name='purchase') as purchase
  -- From the GA4 eCommerce dataset in BigQuery
  FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*` 
  -- Filter data between the specified start and end dates
  where parse_date('%Y%m%d', event_date) between start_date and end_date
  -- Group the data by device category for breakdown
  group by device_category
)

-- Select relevant KPIs and metrics for analysis
select
    -- Device category for further segmentation
    device_category,
    
    -- Total number of 'add to cart' actions
    add_to_cart,
    
    -- Total number of 'begin checkout' actions
    begin_checkout,
     
   -- Total number of 'purchase' actions
    purchase,
    
    -- Calculate average order value (purchase revenue per user)
    round(safe_divide(purchase_revenue, purchase_users), 2) as avg_order_value,
    
    -- Calculate cart abandonment rate
    round(safe_divide((add_to_cart - purchase), add_to_cart) * 100, 2) as cart_abandonment_rate,
    
    -- Calculate checkout abandonment rate
    round(safe_divide((begin_checkout - purchase), begin_checkout) * 100, 2) as checkout_abandonment_rate
from flat_data;
```

#### Query Result: 

![image](https://github.com/user-attachments/assets/9b65df47-a606-4e2b-a395-c8188ac0497a)


#### Summary Insight:

I) Mobile users have the highest add-to-cart (23,223) and begin-checkout (15,696) counts, but their cart abandonment rate (89.86%) and checkout abandonment rate (85.0%) are also high. Optimizing the mobile checkout process and offering incentives could help reduce abandonment and increase conversions.

II) Desktop users show strong engagement with 34,047 add-to-carts and 22,272 begins-checkout, but their cart abandonment rate (90.52%) and checkout abandonment rate (85.52%) remain concerning. Focusing on improving desktop checkout usability could help retain more users through the purchase stage.

III) Tablet users have lower engagement with 1,273 add-to-carts and 789 begins-checkout, but their abandonment rates are slightly higher than mobile (cart abandonment: 91.28%, checkout abandonment: 85.93%). Improving the tablet experience and addressing potential usability issues could enhance conversions.


### SQL Query 6: ✅ Analyzing Conversion and Abandonment Rates by Traffic Medium 

This SQL query calculates key conversion and abandonment metrics for different traffic sources in an e-commerce context, helping businesses understand how users interact with the website across different stages of the sales funnel (from page views to purchases).

#### SQL Code:
```
-- declare start and end dates for the analysis period
DECLARE start_date DATE DEFAULT '2020-11-01';
DECLARE end_date DATE DEFAULT '2021-01-31';

-- create a dataset to select relevant event data for the analysis
WITH dataset AS (
  SELECT 
      user_pseudo_id,  -- user identifier for each event
      event_name,  -- the name of the event (e.g., page view, add to cart)
      PARSE_DATE('%Y%m%d', event_date) AS event_date,  -- converts the event date into DATE format
      traffic_source.medium AS traffic_medium  -- the medium of the traffic (e.g., organic, paid)
  FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
  WHERE event_name IN ('page_view', 'view_item', 'add_to_cart', 'begin_checkout', 'purchase')  -- filter for relevant events
    AND _TABLE_SUFFIX BETWEEN FORMAT_DATE('%Y%m%d', start_date) AND FORMAT_DATE('%Y%m%d', end_date)  -- filter by the date range
),

-- create a funnel with counts for each event for each user and traffic medium
funnel AS (
  SELECT
      user_pseudo_id,  -- user identifier
      traffic_medium,  -- traffic medium for each user
      countif(event_name='page_view') AS page_view,  -- count of page views
      countif(event_name='view_item') AS view_item,  -- count of view item events
      countif(event_name='add_to_cart') AS add_to_cart,  -- count of add to cart events
      countif(event_name='begin_checkout') AS begin_checkout,  -- count of begin checkout events
      countif(event_name='purchase') AS purchase  -- count of purchase events
  FROM dataset
  GROUP BY user_pseudo_id, traffic_medium  -- group by user and traffic medium
),

-- aggregate the funnel data by traffic medium
aggregated_funnel AS (
  SELECT
      traffic_medium,  -- traffic medium
      COUNT(DISTINCT user_pseudo_id) AS total_user_count,  -- total number of unique users
      SUM(page_view) AS total_page_view_count,  -- total number of page views
      SUM(view_item) AS total_view_item_count,  -- total number of view item events
      SUM(add_to_cart) AS total_add_to_cart_count,  -- total number of add to cart events
      SUM(begin_checkout) AS total_begin_checkout_count,  -- total number of begin checkout events
      SUM(purchase) AS total_purchase_count  -- total number of purchase events
  FROM funnel
  GROUP BY traffic_medium  -- group by traffic medium
)

-- select and calculate the relevant metrics for the final output
SELECT
    traffic_medium,  -- traffic medium for the aggregated data
    total_user_count,  -- total number of users
    total_view_item_count,  -- total number of view item events
    total_add_to_cart_count,  -- total number of add to cart events
    total_begin_checkout_count,  -- total number of begin checkout events
    total_purchase_count,  -- total number of purchases
    ROUND(SAFE_DIVIDE((total_add_to_cart_count - total_purchase_count), total_add_to_cart_count) * 100, 2) AS cart_abandonment_rate,  -- calculate cart abandonment rate
    ROUND(SAFE_DIVIDE((total_begin_checkout_count - total_purchase_count), total_begin_checkout_count) * 100, 2) AS checkout_abandonment_rate,  -- calculate checkout abandonment rate
    ROUND(SAFE_DIVIDE(total_purchase_count, total_user_count) * 100, 2) AS conversion_rate  -- calculate conversion rate
FROM aggregated_funnel
ORDER BY total_user_count DESC, conversion_rate ASC;  -- order by the number of users and conversion rate;
```

#### Query Result: 

![image](https://github.com/user-attachments/assets/1b423d11-ce45-4cdb-a669-02d40d9696bf)

#### Summary Insight:

I) Organic traffic has the highest user count (109,368) but also the highest cart abandonment rate (91.07%) and checkout abandonment rate (86.27%). Focusing on streamlining the checkout process and offering incentives could help reduce abandonment and improve conversions.

II) (none) traffic, likely direct or session-based, has a slightly better conversion rate (1.72%) than organic (1.44%) but still shows high abandonment (cart: 90.64%, checkout: 85.45%). Improving the user experience and incentivizing purchases could lead to higher conversion rates for this channel.

III) Referral traffic has the highest conversion rate (2.46%) and relatively low abandonment rates (cart: 89.41%, checkout: 84.6%). Leveraging this traffic source by improving the landing pages and ensuring a seamless checkout could further capitalize on this strong performance.

IV) CPC (paid) traffic has the lowest conversion rate (1.09%) and the highest cart abandonment rate (91.87%). Optimizing ad targeting and landing page relevance could improve engagement and reduce abandonment in this channel.

V) Data deleted traffic shows the highest conversion rate (6.05%) with a relatively lower checkout abandonment rate (82.97%). This traffic is performing well, and further analysis could help replicate its success in other channels.




### SQL Query 6: ✅ Analyzing New User and Returning Users Report

#### SQL Code:
```
with first_session as (
  select 
    user_pseudo_id,
    min(
      case 
        when (select value.int_value from unnest(event_params) where key = 'ga_session_number') = 1 
        then 'new_visitors' 
        else 'return_visitors' 
      end
    ) as user_type
  from `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
  where parse_date('%Y%m%d', event_date) between date '2020-11-01' and date '2021-01-31'
  group by user_pseudo_id
),

user_session as (
  select 
    fs.user_pseudo_id,
    fs.user_type,
    countif(e.event_name = 'add_to_cart') as add_to_carts,
    countif(e.event_name = 'begin_checkout') as begin_checkout,
    countif(e.event_name = 'purchase') as purchase
  from `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*` e
  join first_session fs on e.user_pseudo_id = fs.user_pseudo_id
  where parse_date('%Y%m%d', e.event_date) between date '2020-11-01' and date '2021-01-31'
  group by fs.user_pseudo_id, fs.user_type
)

select
  user_type,
  count(distinct user_pseudo_id) as total_users,
  round(safe_divide((sum(add_to_carts) - sum(purchase)), sum(add_to_carts)) * 100, 2) as cart_abandonment_rate,
  round(safe_divide((sum(begin_checkout) - sum(purchase)), sum(begin_checkout)) * 100, 2) as checkout_abandonment_rate,
  round(safe_divide(sum(purchase), count(distinct user_pseudo_id)) * 100, 2) as conversion_rate
from user_session
group by user_type;

```

#### Query Result: 

![image](https://github.com/user-attachments/assets/2d747122-c3e4-41ab-84e7-7fdc822b09dd)


#### Summary Insight:

I) New Visitors (261,148 users) have a high cart abandonment rate (91.26%) and checkout abandonment rate (86.61%), resulting in a low conversion rate (1.72%). Since they are unfamiliar with the store, improving trust signals (reviews, guarantees) and simplifying the checkout process could help boost conversions.

II) Returning Visitors (9,006 users) show lower abandonment rates (cart: 83.19%, checkout: 76.86%) and a significantly higher conversion rate (13.26%). Engaging them with personalized offers, loyalty programs, and remarketing strategies could further improve their conversion potential.





