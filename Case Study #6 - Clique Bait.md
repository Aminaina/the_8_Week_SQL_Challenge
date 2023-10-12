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
-Using a single SQL query - create a new output table which has the following details:

How many times was each product viewed?
How many times was each product added to cart?
How many times was each product added to a cart but not purchased (abandoned)?
How many times was each product purchased?
```sql
 -- Create the ProductMetrics table
    CREATE TABLE [ProductMetrics] (
         product_name varchar(14),
         number_views INT,
         number_cart_adds INT,
         number_cart_abandoned INT,
         number_purchased INT
         );

       -- Insert data into the ProductMetrics table using the SELECT query
        WITH CTE_view AS (
            SELECT page_name ,count(*) number_view
            FROM [clique_bait].dbo.events e
            JOIN [clique_bait].dbo.page_hierarchy p ON e.page_id = p.page_id
            where event_type = 1 AND  product_id IS NOT NULL
            GROUP BY page_name
     
                 ),
        CTE_card AS (
        SELECT page_name ,count(*) num_add_card
        FROM [clique_bait].dbo.events e
        JOIN [clique_bait].dbo.page_hierarchy p ON e.page_id = p.page_id
        where e.event_type = 2  
         GROUP BY page_name
    
               ),
          CTE_abon AS (
           SELECT page_name, sum(case when b.event_type is null then 1 else 0 end) num_abondand,sum(case when b.event_type is null then 0 else 1 end) puchased
           FROM
           (SELECT visit_id, cookie_id, page_id, event_type
             FROM [clique_bait].dbo.events e
              where e.event_type = 2) AS a
           left JOIN
             (SELECT visit_id, cookie_id, page_id, event_type
                  FROM [clique_bait].dbo.events e
                  where e.event_type = 3) b
                  on a.visit_id = b.visit_id
                  join  [clique_bait].dbo.page_hierarchy p ON a.page_id = p.page_id
                  GROUP BY  page_name
     
                           )
            INSERT INTO ProductMetrics (product_name, number_views, number_cart_adds,  number_cart_abandoned, number_purchased)
            SELECT V.page_name, number_view, num_add_card, num_abondand, puchased
            FROM CTE_view V
             JOIN CTE_card C ON V.page_name = C.page_name
             JOIN CTE_abon A ON V.page_name = A.page_name;
```
 ```sql
  SELECT * FROM ProductMetrics;
```
result:
# Product Funnel Analysis

## ProductMetrics Table

| product_name     | number_views | number_cart_adds | number_cart_abandoned | number_purchased |
|------------------|--------------|-------------------|------------------------|-------------------|
| Russian Caviar  | 1563         | 946               | 249                    | 697               |
| Salmon           | 1559         | 938               | 227                    | 711               |
| Tuna             | 1515         | 931               | 234                    | 697               |
| Lobster          | 1547         | 968               | 214                    | 754               |
| Crab             | 1564         | 949               | 230                    | 719               |
| Kingfish         | 1559         | 920               | 213                    | 707               |
| Black Truffle    | 1469         | 924               | 217                    | 707               |
| Oyster           | 1568         | 943               | 217                    | 726               |
| Abalone          | 1525         | 932              

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
```sql
     -- Create the Metricscategory table
    CREATE TABLE Metricscategory (
       product_category varchar(14),
       number_views INT,
       number_cart_adds INT,
       number_cart_abandoned INT,
       number_purchased INT
      );

      -- Insert data into the ProductMetrics table using the SELECT query
     WITH CTE_view AS (
     SELECT product_category ,count(*) number_view
     FROM [clique_bait].dbo.events e
     JOIN [clique_bait].dbo.page_hierarchy p ON e.page_id = p.page_id
     where event_type = 1 AND  product_id IS NOT NULL
     GROUP BY product_category
    
     ),
    CTE_card AS (
    SELECT product_category ,count(*) num_add_card
    FROM [clique_bait].dbo.events e
    JOIN [clique_bait].dbo.page_hierarchy p ON e.page_id = p.page_id
    where e.event_type = 2  
    GROUP BY product_category
    
   ),
      CTE_abon AS (
      SELECT product_category, sum(case when b.event_type is null then 1 else 0 end) num_abondand,sum(case when b.event_type is null then 0 else 1 end) puchased
      FROM
          (SELECT visit_id, cookie_id, page_id, event_type
               FROM [clique_bait].dbo.events e
              where e.event_type = 2) AS a
             left JOIN
              (SELECT visit_id, cookie_id, page_id, event_type
               FROM [clique_bait].dbo.events e
                where e.event_type = 3) b
                 on a.visit_id = b.visit_id
                  join  [clique_bait].dbo.page_hierarchy p ON a.page_id = p.page_id
                  GROUP BY  product_category
   
                       )
                   INSERT INTO Metricscategory (product_category, number_views, number_cart_adds,  number_cart_abandoned, number_purchased)
                   SELECT V.product_category, number_view, num_add_card, num_abondand, puchased
                   FROM CTE_view V
                   JOIN CTE_card C ON V.product_category = C.product_category
                    JOIN CTE_abon A ON V.product_category = A.product_category;
 SELECT * FROM Metricscategory;
 ```
result:
# Metricscategory Table

| product_category | number_views | number_cart_adds | number_cart_abandoned | number_purchased |
|------------------|--------------|-------------------|------------------------|-------------------|
| Fish             | 4633         | 2789            | 674                   | 2115              |
| Luxury           | 3032         | 1870            | 466                   | 1404              |
| Shellfish        | 6204         | 3792            | 894                   | 2898              |

## Use your 2 new output tables - answer the following questions:

-Which product had the most views, cart adds and purchases?
```sql
   SELECT top 1 * FROM ProductMetrics
    order by number_views desc, number_cart_adds desc, number_purchased desc;
```
result:
| Product Name    | Number of Views | Number of Cart Adds | Number of Cart Abandoned | Number of Purchases |
|-----------------|-----------------|----------------------|--------------------------|--------------------|
| Oyster          | 1,568           | 943                  | 217                      | 726                |

-Which product was most likely to be abandoned?
```sql
 SELECT top 1 product_name, number_cart_abandoned  FROM ProductMetrics
 order by  number_cart_abandoned desc;
```
result:
| Product Name    | Number of Cart Abandoned |
|-----------------|--------------------------|
| Russian Caviar | 249                      |

-Which product had the highest view to purchase percentage?
```sql
    /*to measure the effectiveness of product pages or advertisements in converting potential customers (views) into actual buyers (purchases)
     helps businesses understand how well a product listing or advertisement is performing in terms of converting people who view the product
      into customers who make a purchase.*/
     SELECT  top 1  product_name, round(100* (number_purchased /convert(float,  number_views)),2) view_purchase_percentage FROM ProductMetrics
    order by view_purchase_percentage desc;
```
result:
| Product Name | View-to-Purchase Percentage |
|--------------|-----------------------------|
| Lobster      | 48.74                     |

-What is the average conversion rate from view to cart add?
```sql
     /* conversion rate from view to cart add refers to the percentage of users who view a product and then proceed to add it
       to their shopping cart. This metric helps businesses understand how effectively 
        they are able to move potential customers from simply viewing a product to taking
          a concrete step towards making a purchase.*/
        with CTE_v_c AS(SELECT   product_name, round(100* (number_cart_adds /convert(float,  number_views)),2) view_cart_adds_percentage FROM ProductMetrics)
        SELECT AVG(view_cart_adds_percentage)  average_conversion_rate
          FROM  CTE_v_c;
```
result:
| Average Conversion Rate from View to Cart Add |
|----------------------------------------------|
| 60.95                                      |

-What is the average conversion rate from cart add to purchase?
```sql
   with CTE_c_p AS(SELECT   product_name, round(100* (number_purchased  /convert(float,  number_cart_adds)),2) cart_adds_purchase_percentage FROM ProductMetrics)
   SELECT Round(AVG(cart_adds_purchase_percentage),2)  Average_Conversion_Rate_from_Cart_Add_to_Purchase
   FROM  CTE_c_p;

```
result:
| Average Conversion Rate from Cart Add to Purchase |
|---------------------------------------------------|
| 75.93%                                            |

## Campaigns Analysis
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
(Optional column) cart_products: a comma separated text value with products added
to the cart sorted by the order they were added to the cart (hint: use the sequence_number)

```sql
   --visit_start_time: the earliest event_time for each visit
 
          SELECT user_id, visit_id, min(TRY_CONVERT(DATETIME, LEFT(event_time, 19)))visit_start_time, 
         SUM(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) number_views,  SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS number_cart_adds,
        SUM(CASE WHEN event_type = 3 THEN 1 ELSE 0 END) number_purchased, c.campaign_name, SUM(CASE WHEN event_type = 4 THEN 1 ELSE 0 END) AS number_impression,
        SUM(CASE WHEN event_type = 5 THEN 1 ELSE 0 END) AS number_click,
        STRING_AGG(
         CASE
            WHEN p.product_id IS NOT NULL AND e.event_type = 2
            THEN p.page_name
            ELSE NULL
        END, 
          ', '
      ) WITHIN GROUP (ORDER BY e.sequence_number) AS cart_products
       FROM  [clique_bait].dbo.users u
       JOIN  [clique_bait].dbo.events e
       ON u.cookie_id  = e.cookie_id
       JOIN [clique_bait].dbo.campaign_identifier c
      ON TRY_CONVERT(DATETIME, LEFT(event_time, 19)) between c.start_date and c.end_date
      JOIN [clique_bait].[dbo].[page_hierarchy] p
      ON e.page_id = p.page_id
      GROUP BY u.user_id, e.visit_id, c.campaign_name
      order by u.user_id;
```
## Conclusion
 These insights provide Clique Bait with valuable information for decision-making. For example, they can focus on optimizing the checkout process to reduce abandonment or consider promoting products from the "Shellfish" category, which seems to be a customer favorite.
 By leveraging data-driven insights, Clique Bait can further enhance its online seafood shopping experience and grow its business in the digital ocean.
