# Case Study #3: Foodie-Fi

## Introduction
Subscription-based businesses are booming, and Danny saw an opportunity to create a streaming service exclusively for food-related content. Foodie-Fi, launched in 2020, offers monthly and annual subscriptions that provide unlimited on-demand access to exclusive food videos from around the world. Danny built Foodie-Fi with a data-driven mindset, aiming to make all future investment decisions and feature enhancements based on data. This case study explores the use of subscription-style digital data to answer crucial business questions.

## Available Data
Danny has provided data for Foodie-Fi, focusing on two main tables within the "foodie_fi" database schema. The data design and table descriptions are as follows:

### Table 1: plans
- Customers can select different plans when signing up for Foodie-Fi.
- Basic plan: Limited access, monthly subscription at $9.90.
- Pro plan: Unlimited access, offline video downloads, starting at $19.90/month or $199 annually.
- Customers can begin with a 7-day free trial and then switch to a paid plan.
- When customers cancel, they have a churn plan record with a null price but continue until the billing period ends.

### Table 2: subscriptions
- Customer subscriptions include the start date of their specific plan.
- Downgrading or canceling a subscription retains the higher plan until the period ends.
- Upgrading to a higher plan takes effect immediately.
- When customers churn, they retain access until the end of the current billing period.

## Case Study Questions

A. **Customer Journey**

Based on the 8 sample customers provided in the subscriptions table, here is a brief description of each customer's onboarding journey:

- Customer 1: Joined with a basic monthly plan, continued monthly.
- Customer 2: Started with a pro annual plan in September 2020.
- Customer 13: Joined with a basic monthly plan in December 2020.
- Customer 15: Started with a pro monthly plan in March 2020.
- Customer 16: Joined with a basic monthly plan and upgraded to a pro annual plan in October 2020.
- Customer 18: Started with a pro monthly plan in July 2020.
- Customer 19: Started with a pro monthly plan in June 2020 and upgraded to a pro annual plan in August 2020.

B. **Data Analysis Questions**

Here are the responses to the data analysis questions:

1. **How many customers has Foodie-Fi ever had?**
   ```sql
   SELECT count(distinct customer_id) count_customer
   FROM  [foodie_fi].[dbo].subscriptions;
   ```
   
result1:
|   count_customer   |
|-------------------|
|        909        |

2. **What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value?**

 ```sql
   SELECT FORMAT(start_date, 'yyyy-MM') AS month, COUNT(*) AS trail_plan
   FROM [foodie_fi].[dbo].subscriptions
   WHERE plan_id = 0
   GROUP BY FORMAT(start_date, 'yyyy-MM')
   ORDER BY FORMAT(start_date, 'yyyy-MM');
```
result2:
|     month     | trail_plan |
|--------------|------------|
| 2020-01-01   |     76     |
| 2020-02-01   |     64     |
| 2020-03-01   |     82     |
| 2020-04-01   |     74     |
| 2020-05-01   |     84     |
| 2020-06-01   |     67     |
| 2020-07-01   |     82     |
| 2020-08-01   |     81     |
| 2020-09-01   |     82     |
| 2020-10-01   |     74     |
| 2020-11-01   |     67     |
| 2020-12-01   |     75     |


3. **What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name.**
   ```sql
    SELECT plan_id, COUNT(*) AS event_count
    FROM [foodie_fi].dbo.subscriptions
    WHERE YEAR(start_date) > 2020
    GROUP BY plan_id;
   ```
   result3:
   | Plan ID | Event Count |
|---------|-------------|
|    1    |      8      |
|    2    |     54      |
|    3    |     58      |
|    4    |     67      |


4. **What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**
  ```sql
    SELECT COUNT(*) AS CustomerCount, ROUND((COUNT(*) * 100.0) / (SELECT COUNT(*) FROM [foodie_fi].dbo.subscriptions), 1) AS PercentageChurned
FROM [foodie_fi].dbo.subscriptions
WHERE plan_id = 4;

   ```
result4:
| COUNT(*) | count_churned |
|----------|---------------|
|   285    |     11.8      |

5. **How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?**
  ```sql
  
    WITH cte_rank AS (
        SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY (SELECT 0)) AS rank_plan
       FROM [foodie_fi].[dbo].subscriptions
     )

     SELECT COUNT(*) AS CustomerCount, ROUND((COUNT(*) * 100.0) / (SELECT COUNT(DISTINCT customer_id) FROM [foodie_fi].[dbo].subscriptions), 0) AS 
     PercentageChurned
    FROM cte_rank
     WHERE plan_id = 4 AND rank_plan = 2;

   ```
result5:
| CustomerCount | PercentageChurned |
|---------------|-------------------|
|      86       |         9         |


6. **What is the number and percentage of customer plans after their initial free trial?**
   ```sql
 
      WITH cte_rank AS (
        SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY (SELECT 0)) AS rank_plan
       FROM [foodie_fi].[dbo].subscriptions
      )

      SELECT
      plan_id AS "Plan ID",
      COUNT(*) AS "Number of Customers",
      ROUND((COUNT(*) * 100.0) / (SELECT COUNT(DISTINCT customer_id) FROM [foodie_fi].[dbo].subscriptions), 0) AS "Percentage of Plans"
     FROM cte_rank
     WHERE rank_plan = 2
    GROUP BY plan_id
   ORDER BY plan_id;

   ```
result6:
| Plan ID | Number of Customers | Percentage of Plans |
|---------|---------------------|---------------------|
|    1    |         493         |          54         |
|    2    |         297         |          33         |
|    3    |          32         |           4         |
|    4    |          86         |           9         |

7. **What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?**
   ```sql
     WITH max_rank_plan AS (
        SELECT
          customer_id,
          plan_id,
          start_date,
          ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY (SELECT 0)) AS rank_plan
            FROM [foodie_fi].[dbo].subscriptions
        WHERE start_date <= '2020-12-31'
            )

      SELECT
      t1.plan_id AS "Plan ID",
          COUNT(t2.customer_id) AS "Customer Count",
        ROUND((COUNT(t2.customer_id) * 100.0) / (SELECT COUNT(customer_id) FROM max_rank_plan), 2) AS "Percentage"
            FROM max_rank_plan t1
            JOIN (
          SELECT customer_id, MAX(rank_plan) AS max_rank
          FROM max_rank_plan
          GROUP BY customer_id
          ) t2 ON t1.customer_id = t2.customer_id AND t1.rank_plan = t2.max_rank
        GROUP BY t1.plan_id
         ORDER BY t1.plan_id;

    ```
   result7:
   
   | Plan ID | Customer Count | Percentage |
   |---------|----------------|------------|
   |    0    |       17       |    0.76    |
   |    1    |      197       |    8.81    |
   |    2    |      297       |   13.29    |
   |    3    |      180       |    8.05    |
   |    4    |      218       |    9.75    |


8. **How many customers have upgraded to an annual plan in 2020?**
   ```sql
    SELECT count(DISTINCT customer_id) as "Upgraded Customers"
    FROM  [foodie_fi].[dbo] .subscriptions
    where DATEPART(year, start_date) = '2020' and plan_id = 3;
   ```
result 8:
| Upgraded Customers |
|-------------------|
|        180        |

9. **How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?**
   ```sql
    WITH CTE_1  AS(SELECT * FROM [foodie_fi].[dbo] .subscriptions WHERE plan_id = 0 ) ,
    CTE_2 AS (SELECT * FROM [foodie_fi].[dbo] .subscriptions WHERE plan_id = 3  )
    SELECT AVG( DATEDIFF(day, t1.start_date, t2.start_date) )  as "Average Days to Upgrade"
    FROM CTE_1  t1
    JOIN  CTE_2 t2
    ON  t1.customer_id = t2.customer_id;   
   ```
   result 9:
   | Average Days to Upgrade |
   |-------------------------|
   |          106            |

11. **How many customers downgraded from a pro monthly to a basic monthly plan in 2020?**
   ```sql
        WITH CTE_rank as( SELECT customer_id, plan_id, start_date, ROW_NUMBER() over(partition  by  customer_id order by     (select 0)   ) rank_plan
        FROM [foodie_fi].[dbo] .subscriptions
	     WHERE    start_date <= '2020-12-31')
	     SELECT * from  CTE_rank c1
	     join CTE_rank c2
	     on c1.start_date <= c2.start_date
	     and c1.customer_id = c2.customer_id
	     where c1.plan_id = 2 and  c2.plan_id = 1;
   ```
result 11:
| Customer ID | Plan ID (Start) | Start Date (Start)          | Plan ID (End) | Start Date (End)            |
|-------------|-----------------|-----------------------------|---------------|-----------------------------|
|             |                 |                             |               |                             |


C. **Challenge Payment Question**

The Foodie-Fi team wants you to create a new payments table for the year 2020 with the specified requirements. Here's an example output for this table:

- Customer_id | Plan_id | Plan_name       | Payment_date | Amount | Payment_order
- 1          | 1       | basic monthly   | 2020-08-08   | 9.90   | 1
- 1          | 1       | basic monthly   | 2020-09-08   | 9.90   | 2
- ... (continue with example data)

D. **Outside The Box Questions**

These open-ended questions are designed to encourage critical thinking:

1. **How would you calculate the rate of growth for Foodie-Fi?**

2. **What key metrics would you recommend Foodie-Fi management to track over time to assess the performance of their overall business?

3. **What are some key customer journeys or experiences that you would analyze further to improve customer retention?

4. **If the Foodie-Fi team were to create an exit survey shown to customers who wish to cancel their subscription, what questions would you include in the survey?

5. **What business levers could the Foodie-Fi team use to reduce the customer churn rate? How would you validate the effectiveness of your ideas?**

Feel free to replace "[Insert SQL Query and Result]" with the actual SQL queries and results as you perform the analysis. This extended Markdown document now covers a wider range of questions and challenges for the Foodie-Fi case study.

