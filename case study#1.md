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
