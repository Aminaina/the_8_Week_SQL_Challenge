# Case Study #6 - Clique Bait

## Introduction

Clique Bait is not like your regular online seafood store - the founder and CEO Danny, was also a part of a digital data analytics team and wanted to expand his knowledge into the seafood industry! In this case study, you are required to support Danny’s vision and analyze his dataset and come up with creative solutions to calculate funnel fallout rates for the Clique Bait online store.

## Available Data

For this case study, there are five datasets that you will need to combine to solve all of the questions.

### Users
- user_id
- cookie_id
- start_date
### Events
- visit_id
- cookie_id
- page_id
- event_type
- sequence_number
- event_time
### Event Identifier
- event_type
- event_name
### Campaign Identifier
- campaign_id
- products
- campaign_name
- start_date
- end_date

### Page Hierarchy
- page_id
- page_name
- product_category
- product_id

## 1. Digital Analysis

- How many users are there?

```sql
  SELECT count(distinct user_id) as users
   FROM [clique_bait].dbo.users ;
```
result: 500  users
-How many cookies does each user have on average?
```sql
  SELECT  AVG(users)avg_cookie_id
  FROM
  (SELECT distinct user_id, count(cookie_id) as users
   FROM [clique_bait].dbo.users
   GROUP BY user_id)AS counts;
```
result: the average of cookies is 3 
-What is the unique number of visits by all users per month?
```sql
SELECT MONTH(TRY_CONVERT(DATETIME, LEFT(event_time, 19)))AS month_datetime, count(distinct visit_id)  number_of_visits
FROM [clique_bait].dbo.events
GROUP BY MONTH(TRY_CONVERT(DATETIME, LEFT(event_time, 19)))
ORDER BY MONTH(TRY_CONVERT(DATETIME, LEFT(event_time, 19)));
```
reuslt:
| Month     | Number of Visits |
|-----------|------------------|
| January   | 876              |
| February  | 1,488            |
| March     | 916              |
| April     | 248              |
| May       | 36               |
-What is the number of events for each event type?
```sql
  SELECT event_type, COUNT(*) n_event
   FROM [clique_bait].dbo.events
   GROUP BY event_type
    ORDER BY event_type;
```
result:
| Event Type    | Number of Events |
|---------------|-------------------|
| 1 (Page View) | 20,928            |
| 2 (Add to Cart)| 8,451            |
| 3 (Purchase)  | 1,777             |
| 4 (Ad Impression) | 876          |
| 5 (Ad Click)  | 702              |
-What is the percentage of visits which have a purchase event?
```sql
select ROUND(count(visit_id) /CONVERT(float, (select count(DISTINCT visit_id)  from [clique_bait].dbo.events) )*100,2) percentage_visits_purchase
from [clique_bait].dbo.events e
join [clique_bait].dbo.event_identifier i
on e.event_type = i.event_type
where i.event_name = 'Purchase';
```
result:
The percentage of visits that have a purchase event is approximately 49.86%.

-What is the percentage of visits which view the checkout page but do not have a purchase event?
```sql
SELECT ROUND(count(c.visit_id)/CONVERT(float, (select count(DISTINCT visit_id)  from [clique_bait].dbo.events) )*100,2)
FROM(
 SELECT visit_id, cookie_id, page_id, event_type
 FROM [clique_bait].dbo.events
 WHERE  page_id = 12) c
 left JOIN(
  SELECT visit_id, cookie_id, page_id, event_type
 FROM [clique_bait].dbo.events
 WHERE  event_type = 3) pu
 ON c.visit_id = pu.visit_id
 WHERE  pu.event_type is null
```
Approximately 9.15% of visits view the checkout page but do not have a purchase event.
-What are the top 3 pages by number of views?
```sql
SELECT  TOP 3 page_name, COUNT(*) number_views
FROM [clique_bait].dbo.events e
JOIN  [clique_bait].dbo.page_hierarchy p
ON e.page_id = p.page_id
WHERE event_type = 1 
GROUP BY   page_name
ORDER BY number_views DESC;
```
The top 3 pages by the number of views are as follows:

1. "All Products" with 3,174 views
2. "Checkout" with 2,103 views
3. "Home Page" with 1,782 views
-What is the number of views and cart adds for each product category?
```sql
SELECT product_category, SUM(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) number_views,  SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS number_cart_adds
FROM [clique_bait].dbo.events e
JOIN  [clique_bait].dbo.page_hierarchy p
ON e.page_id = p.page_id
WHERE product_category IS NOT NULL
GROUP BY product_category
ORDER BY product_category;
```
Here are the counts of views and cart adds for each product category:

| Product Category | Number of Views | Number of Cart Adds |
|------------------|-----------------|----------------------|
| Fish             | 4,633           | 2,789                |
| Luxury           | 3,032           | 1,870                |
| Shellfish        | 6,204           | 3,792                |
-What are the top 3 products by purchases?
```sql
SELECT product_category, COUNT(*) number_purchases 
FROM [clique_bait].dbo.events e
JOIN  [clique_bait].dbo.page_hierarchy p
ON e.page_id = p.page_id
where event_type IN( 3, 2) AND product_category IS NOT NULL 
GROUP BY product_category
ORDER BY product_category desc;
```
The top 3 products by the number of purchases are as follows:

1. Shellfish with 3,792 purchases
2. Luxury with 1,870 purchases
3. Fish with 2,789 purchases
## Product Funnel Analysis
-How many times was each product viewed?
```sql
```
-How many times was each product added to the cart?
```sql
```
How many times was each product added to a cart but not purchased (abandoned)?
```sql
```
How many times was each product purchased?
```sql
```
Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.

Use your 2 new output tables to answer the following questions:

Which product had the most views, cart adds, and purchases?
```sql
```
Which product was most likely to be abandoned?
```sql
```
Which product had the highest view to purchase percentage?
```sql
```
What is the average conversion rate from view to cart add?
```sql
```
What is the average conversion rate from cart add to purchase?
```sql
```
Campaigns Analysis
Generate a table that has 1 single row for every unique visit_id record and has the following columns:

user_id
visit_id
visit_start_time: the earliest event_time for each visit
page_views: count of page views for each visit
cart_adds: count of product cart add events for each visit
purchase: 1/0 flag if a purchase event exists for each visit
campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
impression: count of ad impressions for each visit
click: count of ad clicks for each visit
(Optional column) cart_products: a comma-separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)
