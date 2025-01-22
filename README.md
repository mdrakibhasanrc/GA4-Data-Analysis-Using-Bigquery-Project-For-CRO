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


### SQL Query 2:  ✅ Analyzing Conversion Funnel Data for eCommerce Optimization

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


### SQL Query 3: ✅ Device Category-Based Conversion Analysis

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












