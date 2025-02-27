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


### SQL Query 2: ✅ Identifying Website Performance Metrics

#### SQL Code:
```
with date_range as (
  select date '2020-11-01' as start_date, date '2021-01-31' as end_date
),

revenue_data as (
  select  
     user_pseudo_id,
     sum(ecommerce.purchase_revenue) as total_revenue
  from `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`, date_range
  where event_name = 'purchase'
    and parse_date('%Y%m%d', event_date) between date_range.start_date and date_range.end_date
  group by user_pseudo_id
),

visitor_data as (
  select
     user_pseudo_id,
     count(distinct (select value.int_value from unnest(event_params) where key = 'ga_session_id')) as sessions,
     max(coalesce((select value.string_value from unnest(event_params) where key = 'session_engaged'), '0')) as is_engaged
  from `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`, date_range
  where parse_date('%Y%m%d', event_date) between date_range.start_date and date_range.end_date
  group by user_pseudo_id
),

session_time as (
  select
     user_pseudo_id,
     (select value.int_value from unnest(event_params) where key = 'ga_session_id') as ga_session_id,
     min(event_timestamp) / 1000000 as session_start_time,
     max(event_timestamp) / 1000000 as session_end_time
  from `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`, date_range
  where parse_date('%Y%m%d', event_date) between date_range.start_date and date_range.end_date
  group by user_pseudo_id, ga_session_id
),

session_duration as (
  select
     user_pseudo_id,
     avg(session_end_time - session_start_time) as avg_session_duration_seconds
  from session_time
  group by user_pseudo_id
),

combined_data as (
  select
     count(distinct v.user_pseudo_id) as total_users,
     sum(v.sessions) as total_sessions,
     countif(v.is_engaged = '0') as bounced,
     sum(ifnull(r.total_revenue, 0)) as total_revenue,
     avg(s.avg_session_duration_seconds) as avg_session_duration_seconds
  from visitor_data v
  left join revenue_data r
    on v.user_pseudo_id = r.user_pseudo_id
  left join session_duration s
    on v.user_pseudo_id = s.user_pseudo_id
)

select
   total_users,
   total_revenue,
   total_sessions,
   bounced,
   avg_session_duration_seconds,
   round(safe_divide(total_revenue, total_users), 2) as revenue_per_visitor,
   round(safe_divide(bounced, total_sessions) * 100, 2) as bounce_rate
from combined_data;

```

#### Query Result: 

![image](https://github.com/user-attachments/assets/06276bf8-330a-486f-9c47-5e4f5fd1fadd)

#### Summary & Insight:

i) Optimize Landing Pages & RPV: Reduce bounce rate (20.68%) and increase revenue per visitor ($1.34) by improving landing page relevance, visuals, speed, and offering upsells/cross-sells.

ii) Focus on CRO: Leverage high traffic (360,129 sessions) and revenue ($362,165) with targeted promotions and discounts to improve conversion rates.

iii) Boost Engagement: Increase average session duration (160.30 seconds) with personalized content and product recommendations to enhance user interaction and reduce bounce rates.



### SQL Query 3:  ✅ Analyzing Conversion Funnel Data for eCommerce Optimization

This analysis focuses on evaluating key stages in the conversion funnel of an eCommerce store, helping to identify where users drop off and why. By tracking user behavior from arrival to checkout, businesses can uncover areas for improvement in the user journey.

#### SQL Code:
```
-- step 1: create a base dataset containing events filtered by event name and date range
with dataset as (
  select 
      user_pseudo_id,
      event_name,
      parse_date('%Y%m%d', event_date) as event_date
  from `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*` 
  where event_name in ('view_item', 'add_to_cart', 'begin_checkout', 'purchase')  -- filtering relevant events
    and event_date between '2020-11-01' and '2021-12-31'  -- filtering events within the date range
),

-- step 2: extract data for 'view_item' event
view_item as (
  select
      user_pseudo_id,
      event_name,
      event_date
  from dataset
  where event_name = 'view_item'  -- filter to only include 'view_item' events
),

-- step 3: extract data for 'add_to_cart' event
add_to_cart as (
  select
      user_pseudo_id,
      event_name,
      event_date
  from dataset
  where event_name = 'add_to_cart'  -- filter to only include 'add_to_cart' events
),

-- step 4: extract data for 'begin_checkout' event
begin_checkout as (
  select
      user_pseudo_id,
      event_name,
      event_date
  from dataset
  where event_name = 'begin_checkout'  -- filter to only include 'begin_checkout' events
),

-- step 5: extract data for 'purchase' event
purchase as (
  select
      user_pseudo_id,
      event_name,
      event_date
  from dataset
  where event_name = 'purchase'  -- filter to only include 'purchase' events
),

-- step 6: create a funnel of events by joining the data from each stage (view_item -> add_to_cart -> begin_checkout -> purchase)
funnel as (
  select
      vi.event_date,

      count(distinct vi.user_pseudo_id) as view_item_count,  -- count unique users who viewed an item
      count(distinct atc.user_pseudo_id) as add_to_cart_count,  -- count unique users who added to cart

      count(distinct bc.user_pseudo_id) as begin_checkout_count,  -- count unique users who began checkout

      count(distinct p.user_pseudo_id) as purchase_count  -- count unique users who completed a purchase

  from view_item vi

  left join add_to_cart atc on vi.user_pseudo_id = atc.user_pseudo_id and vi.event_date = atc.event_date  -- join 'view_item' with 'add_to_cart'

  left join begin_checkout bc on atc.user_pseudo_id = bc.user_pseudo_id and atc.event_date = bc.event_date  -- join 'add_to_cart' with 'begin_checkout'

  left join purchase p on bc.user_pseudo_id = p.user_pseudo_id and bc.event_date = p.event_date  -- join 'begin_checkout' with 'purchase'

  group by vi.event_date  -- group by event date to get daily counts
)

-- step 7: calculate various funnel metrics such as drop-offs and rates for each stage of the funnel

select
   sum(view_item_count) as total_view_item_count,  -- total number of users who viewed an item

   sum(add_to_cart_count) as total_add_to_cart_count,  -- total number of users who added to cart

   sum(begin_checkout_count) as total_begin_checkout_count,  -- total number of users who began checkout

   sum(purchase_count) as total_purchase_count,  -- total number of users who made a purchase

   -- add to cart rate: percentage of users who added an item to the cart after viewing it
   round(safe_divide(sum(add_to_cart_count), nullif(sum(view_item_count), 0)) * 100, 2) as add_to_cart_rate,

   -- begin checkout rate: percentage of users who began checkout after adding to cart
   round(safe_divide(sum(begin_checkout_count), nullif(sum(add_to_cart_count), 0)) * 100, 2) as begin_checkout_rate,

   -- purchase rate: percentage of users who completed a purchase after beginning checkout
   round(safe_divide(sum(purchase_count), nullif(sum(begin_checkout_count), 0)) * 100, 2) as purchase_rate,

   -- drop-off after view item: percentage of users who viewed an item but didn't add it to the cart
   round(safe_divide(sum(view_item_count) - sum(add_to_cart_count), nullif(sum(view_item_count), 0)) * 100, 2) as drop_off_after_view_item,

   -- drop-off after add to cart: percentage of users who added to cart but didn't begin checkout
   round(safe_divide(sum(add_to_cart_count) - sum(begin_checkout_count), nullif(sum(add_to_cart_count), 0)) * 100, 2) as drop_off_after_add_to_cart,

   -- drop-off after begin checkout: percentage of users who began checkout but didn't purchase
   round(safe_divide(sum(begin_checkout_count) - sum(purchase_count), nullif(sum(begin_checkout_count), 0)) * 100, 2) as drop_off_after_begin_checkout
from funnel;
```

#### Query Result: 

![image](https://github.com/user-attachments/assets/75c3866e-2ea7-4082-b41a-44a9cc8589ed)

![image](https://github.com/user-attachments/assets/09b27d0b-f206-4f72-808f-cfa836b370c0)


#### Summary Insight:

I) High drop-offs are observed after viewing items (80.08%) and adding to cart (57.0%), indicating friction in transitioning users through the purchase funnel. Streamlining navigation, providing clear CTAs, or highlighting value propositions could help reduce drop-offs.

II) The purchase rate is 47.22%, with notable drop-offs after beginning checkout (52.78%). Simplifying the checkout process and addressing user concerns like shipping costs or payment security could improve completion rates.

III) With an add-to-cart rate of 19.92% and a begin-checkout rate of 43.0%, optimizing product pages and emphasizing scarcity or urgency could encourage more users to move deeper into the funnel.


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





