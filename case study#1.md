 # Danny's Diner Case Study

Welcome to the Danny's Diner Case Study. This repository contains SQL queries and analyses for solving questions related to customer behavior and preferences at Danny's Diner.

For detailed information about the case study, including example tables and questions, please refer to the [complete case study document](https://8weeksqlchallenge.com/case-study-1/).

## Getting Started
To replicate the analysis and run SQL queries, follow these steps:

1. Install SQL Server and set up the required environment.
2. Download the provided datasets: `sales.csv`, `menu.csv`, and `members.csv`.
3. Load the datasets into your SQL Server instance.
4. Execute the SQL queries provided in the [`queries.sql`](queries.sql) file.

For step-by-step instructions and example queries, visit the [example queries document]( https://8weeksqlchallenge.com/case-study-1/).

## Contact Information
If you have any questions or need assistance, feel free to contact me at [amina.belouadah.94@gmail.com](mailto:your-email@example.com).

##  Case Study Questions
 Each of the following case study questions can be answered using a single SQL statement:

1. What is the total amount each customer spent at the restaurant?
1. How many days has each customer visited the restaurant?
1. What was the first item from the menu purchased by each customer?
1. What is the most purchased item on the menu and how many times was it purchased by all customers?
1. Which item was the most popular for each customer?
1. Which item was purchased first by the customer after they became a member?
1. Which item was purchased just before the customer became a member?
1. What is the total items and amount spent for each member before they became a member?
1. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
1. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

##  Case Study Solutions
1. What is the total amount each customer spent at the restaurant?
  ```sql
SELECT customer_id, SUM(price) total_amount
FROM [Danny's Diner].dbo.sales s
JOIN [Danny's Diner].dbo.menu m ON s.product_id = m.product_id
GROUP BY customer_id;
 ```
the result
| customer_id| total_amount|
|-----:|-------------------|
| A	         | 76          |
| B	         | 74          |
| C	         | 36          |
2. How many days has each customer visited the restaurant?
```sql
 SELECT  customer_id, COUNT(distinct( order_date) )visited_days
 FROM [Danny's Diner].dbo.sales 
 GROUP BY customer_id ;
 ```
the result 
| customer_id|visited_days|
|-----:|-------------------|
| A	         | 4          |
| B	         |6           |
| C	         | 2          |

3. What was the first item from the menu purchased by each customer?
```sql
 SELECT distinct customer_id, product_name
 FROM [Danny's Diner].dbo.sales s
 join ( SELECT  product_id, min(order_date) min_order
 FROM [Danny's Diner].dbo.sales 
 GROUP BY product_id ) c
 on s.product_id  = c.product_id
 AND s.order_date = c.min_order
 join [Danny's Diner].dbo.menu m
 ON  s.product_id = m.product_id ;
```
the result 
| customer_id|product_name|
|-----:|-------------------|
| A	         | curry       |
| B	         |curry        |
| C	         | ramen       |

 4. What is the most purchased item on the menu and how many times was it purchased by each customer?
 ```sql   
 SELECT customer_id, m.product_name, count(m.product_name) times_of_purchased
 FROM [Danny's Diner].dbo.sales s
 JOIN [Danny's Diner].dbo.menu m
 ON  s.product_id = m.product_id
 JOIN  (SELECT top 1 product_name, count(*) count_item
 FROM [Danny's Diner].dbo.sales s
 JOIN [Danny's Diner].dbo.menu  m
 ON  s.product_id = m.product_id
 GROUP BY  product_name
 order by count_item desc) sub
 ON  m.product_name = sub.product_name
  GROUP BY customer_id, m.product_name ;
```
the result
| customer_id|product_name | times_of_purchased |
|-----:|------------------:|--------------------|
| A	         |ramen        | 3                  |
| B	         |ramen        | 2                  |
| C	         | ramen       | 3                  |

5. Which item was the most popular for each customer?
```sql  
 WITH C_ITEM AS (SELECT  customer_id, product_name, count(*) count_item 
 FROM [Danny's Diner].[dbo].[sales] s
 JOIN [Danny's Diner].dbo.menu  m
 ON  s.product_id = m.product_id
 GROUP BY customer_id, product_name)
 SELECT ci.customer_id, product_name  
 FROM C_ITEM ci
 JOIN (
    SELECT customer_id, MAX(count_item) AS max_count_item
    FROM C_ITEM
    GROUP BY customer_id
) subquery ON
 ci.customer_id = subquery.customer_id AND ci.count_item = subquery.max_count_item
 order by customer_id;
 ```

the result
| customer_id|product_name|
|-----:|-------------------|
| A	         |ramen        |
| B	         |sushi        |
| B	         |curry        |
| B	         |ramen        |
| C	         | ramen       |

6. Which item was purchased first by the customer after they became a member?
```sql  
SELECT distinct s.customer_id, product_name 
 FROM [Danny's Diner].dbo.sales s
 join ( SELECT  product_id, min(order_date) min_order
 FROM [Danny's Diner].dbo.sales s
 join  [Danny's Diner].dbo.members me
  on s.customer_id = me.customer_id
  where s.order_date > = me.join_date
 GROUP BY product_id 
 ) c
 on s.product_id  = c.product_id
 AND s.order_date = c.min_order
 join [Danny's Diner].dbo.menu m
 ON  s.product_id = m.product_id ;
 ```
the result
| customer_id|product_name|
|-----:|-------------------|
| A	         | curry       |
| A	         |ramen        |
| B	         | sushi       |

7. Which item was purchased just before the customer became a member?
```sql  
SELECT distinct s.customer_id, product_name  
 FROM [Danny's Diner].dbo.sales s
 join ( SELECT  distinct s.customer_id,   max(order_date) min_order
 FROM [Danny's Diner].dbo.sales s
 join  [Danny's Diner].dbo.members me
  on s.customer_id = me.customer_id
  where s.order_date <  me.join_date
 GROUP BY s.customer_id  
 ) c
 on s.customer_id = c.customer_id
 AND s.order_date = c.min_order
 join [Danny's Diner].dbo.menu m
 ON  s.product_id = m.product_id ;
 ```
the result
| customer_id|product_name|
|-----:|-------------------|
| A	         | curry       |
| A	         |sushi        |
| B	         | sushi       |
8. What is the total items and amount spent for each member before they became a member?
```sql  
SELECT s.customer_id, count(s.product_id) total_item ,sum(price) amount
   FROM [Danny's Diner].dbo.sales s
   JOIN [Danny's Diner].dbo.menu  m
 ON  s.product_id = m.product_id
 join  [Danny's Diner].dbo.members me
  on s.customer_id = me.customer_id
 where s.order_date <  me.join_date
 GROUP BY s.customer_id;
 ```
the result
| customer_id|total_item | amount |
|-----:|------------------:|--------------------|
| A	         |2          | 25                  |
| B	         |3          | 40                  |
 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?  
```sql
WITH points AS ( SELECT  s.customer_id customer_id,s.product_id , IIF(m.product_id = 1, price * 20, price * 10  ) point
 FROM [Danny's Diner].dbo.sales s
   JOIN [Danny's Diner].dbo.menu  m
 ON  s.product_id = m.product_id)
 SELECT customer_id, SUM(point) points
 FROM points
 GROUP BY customer_id;
```
the result
| customer_id|points|
|-----:|-------------------|
| A	         | 860         |
| B	         |940          |
| C	         | 360         |  
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, 
 not just sushi - how many points do customer A and B have at the end of January?
 ```sql
SELECT s.customer_id, SUM( IIF(s.order_date between me.join_date and DATEADD(day, 6, me.join_date), price *20, IIF(m.product_id = 1, price * 20, price * 10  )) ) points
   FROM [Danny's Diner].dbo.sales s
   JOIN [Danny's Diner].dbo.menu  m
 ON  s.product_id = m.product_id
 join  [Danny's Diner].dbo.members me
  on s.customer_id = me.customer_id
  where s.order_date < DATEADD(DAY, -1, EOMONTH(s.order_date)) AND MONTH(s.order_date) = 1
  GROUP BY s.customer_id ;
```
the result
| customer_id|points       |
|-----:|-------------------|
| A	         | 1370        |
| B	         |820          |
  ## Bonus Questions 
  1. Join All The Things ; recreate table contains customer_id, order_date , product_name, price, members  
 ```sql
 CREATE PROCEDURE allThing
 AS
 BEGIN
   SELECT s.customer_id c_id, s.order_date order_de, m.product_name pro_name, m.price pric,  IIF(s.order_date >= me.join_date , 'Y', 'N') members
   FROM [Danny's Diner].dbo.sales s
   JOIN [Danny's Diner].dbo.menu  m
   ON  s.product_id = m.product_id
   LEFT join  [Danny's Diner].dbo.members me
   on s.customer_id = me.customer_id
   END;
   EXEC allThing ;
```

2. RANK customer products FOR member purchases
```sql
  WITH ran AS (SELECT s.customer_id c_id, s.order_date order_de, m.product_name pro_name, m.price pric,  IIF(s.order_date >= me.join_date , 'Y', 'N') members
   FROM [Danny's Diner].dbo.sales s
   JOIN [Danny's Diner].dbo.menu  m
   ON  s.product_id = m.product_id
   LEFT join  [Danny's Diner].dbo.members me
   on s.customer_id = me.customer_id)
  SELECT * , IIF(members = 'N' , NULL   , RANK() OVER(PARTITION BY c_id, members ORDER BY order_de)) ranking
  FROM ran
```
 

   
   
   
   


