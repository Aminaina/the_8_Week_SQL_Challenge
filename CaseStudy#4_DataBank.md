#Case Study #4: Data Bank

## Introduction
In the financial industry, a new innovation called Neo-Banks has emerged - digital-only banks without physical branches.
Inspired by this trend, Danny launched Data Bank, a digital bank with a unique twist. Data Bank not only offers banking services
but also provides the world's most secure distributed data storage platform. Customers' cloud data storage limits are directly
linked to their account balances. This case study revolves around calculating metrics, analyzing growth, and helping Data Bank
make informed decisions based on their data.

## Available Data
Data Bank has provided a data model and example rows from their dataset:

### Table 1: Regions
- Contains region_id and region_name.
### Table 2: Customer Nodes
- Randomly distributes customers across nodes based on their region.
- Specifies which node contains both their cash and data.
### Table 3: Customer Transactions
- Stores customer deposits, withdrawals, and purchases made using Data Bank's debit card.
## Case Study Questions

A. **Customer Nodes Exploration**

1. How many unique nodes are there on the Data Bank system?
Answer1:
 ```sql
  SELECT count(DISTINCT node_id) AS count_nodes
  FROM DataBank.dbo.customer_nodes;
 ```
result1:
|   count_nodes   |
|-----------------|
|        5        |

2. What is the number of nodes per region?
   - Answer:
     ```sql
      SELECT region_name, count(   node_id)  number_of_nodes
      FROM DataBank.dbo.customer_nodes n
      join DataBank.dbo.regions r
      on  r.region_id  = n.region_id 
     GROUP BY region_name ;
     
     ```
    result2:
   |  region_name  |  number_of_nodes  |
   |---------------|-------------------|
   |     Africa    |        714        |
   |    America    |        735        |
   |      Asia     |        665        |
   |   Australia   |        770        |
   |     Europe    |        616        |

3. How many customers are allocated to each region?
   - Answer:
     ```sql
     SELECT region_name, count( DISTINCT customer_id) count_customer
     FROM DataBank.dbo.customer_nodes n
     join DataBank.dbo.regions r
     on  r.region_id  = n.region_id 
      GROUP BY region_name ;
     ```
result3:
|  region_name  |  count_customer  |
|---------------|------------------|
|     Africa    |       102        |
|    America    |       105        |
|      Asia     |        95        |
|   Australia   |       110        |
|     Europe    |        88        |


4. How many days on average are customers reallocated to a different node??
   - Answer:
    ```sql
      WITH CTE_day_node AS( SELECT customer_id, node_id,sum( DATEDIFF(day, start_date, end_date)) days_node
     FROM DataBank.dbo.customer_nodes
     where end_date <> '9999-12-31'
     GROUP BY customer_id, node_id
     )
     SELECT AVG(days_node) avg_days_node
     FROM CTE_day_node;
    ```
result4:

|  avg_days_node  |
|-----------------|
|       23        |

 

5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
   - Answer:
     ```sql
         with CTE_R AS (SELECT region_id,  DATEDIFF(day, start_date, end_date)  REALLOCATION 
         FROM  DataBank.dbo.customer_nodes
        where end_date <> '9999-12-31' 
 
          )
         SELECT  DISTINCT region_id, PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY REALLOCATION )  OVER (PARTITION BY  region_id) median,
          PERCENTILE_DISC(0.8) WITHIN GROUP (ORDER BY REALLOCATION )   OVER (PARTITION BY  region_id)'80th',
           PERCENTILE_DISC(0.95) WITHIN GROUP (ORDER BY REALLOCATION )   OVER (PARTITION BY  region_id) '95th'
           FROM  CTE_R;
     ```
     result5:
     |  region_id  |  median  |  80th  |  95th  |
     |-------------|----------|-------|-------|
     |      1      |    15    |   23  |   28  |
     |      2      |    15    |   23  |   28  |
     |      3      |    15    |   24  |   28  |
     |      4      |    15    |   23  |   28  |
     |      5      |    15    |   24  |   28  |

B. **Customer Transactions**
 **first, clean data, remove the redundant rows**
 ```sql
   WITH CTE_RN AS(SELECT  *,  ROW_NUMBER() OVER( PARTITION BY  customer_id, txn_date, txn_type, txn_amount ORDER BY (SELECT 0)) rn
   FROM  DataBank.dbo.customer_transactions)
   DELETE FROM CTE_RN WHERE rn > 1;
 ```
1. What is the unique count and total amount for each transaction type?
   - Answer:
   ```sql
     SELECT txn_type ,  count(distinct customer_id) unique_count, sum(txn_amount) total_amount
     FROM DataBank.dbo.customer_transactions
     GROUP BY txn_type
    ```
  result1:
  |  txn_type  |  unique_count  |  total_amount  |
  |------------|----------------|----------------|
  |  purchase  |      448       |     806,537    |
  | withdrawal |      439       |     793,003    |
  |  deposit   |      500       |   1,359,168    |

2. What is the average total historical deposit counts and amounts for all customers?
   - Answer:
   ```sql
  
     WITH CTE_X AS (SELECT *, ROW_NUMBER() OVER(  ORDER BY txn_type) order_typ
     FROM DataBank.dbo.customer_transactions
     WHERE txn_type = 'deposit'
     )
    SELECT  AVG(order_typ) average_deposit, max(order_typ) amount_deposit_costumer
    FROM CTE_X  c1;
    ```
result2:
|  average_deposit  |  amount_deposit_costumer  |
|-------------------|---------------------------|
|       1336        |            2671            |

3. For each month, how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
   - Answer:
   ```sql
     WITH CTE_ORD_TYPE AS (SELECT * , ROW_NUMBER() OVER(PARTITION BY customer_id, txn_type ORDER BY txn_type) order_type
      FROM DataBank.dbo.customer_transactions)
     SELECT DATENAME(month, c1.txn_date), count( distinct c1.customer_id)
     FROM  CTE_ORD_TYPE c1
     join CTE_ORD_TYPE c2
     on c1.customer_id = c2.customer_id
     WHERE c1.txn_type = 'deposit' and  c1.order_type > 1 and (c2.txn_type = 'purchase' and c2.order_type = 1 or c2.txn_type = 'withdrawal' and c2.order_type = 1 )
    group by DATENAME(month, c1.txn_date)
    order by  DATENAME(month, c1.txn_date) ;
    ```
   result3:
   |   Month    |  Count_of_Customers  |
   |------------|-----------------------|
   |   April    |         186           |
   |  February  |         329           |
   |  January   |         299           |
   |   March    |         344           |

4. What is the closing balance for each customer at the end of the month?
   - Answer:
   ```sql
      SELECT  customer_id ,  DATENAME( MONTH, txn_date) MONTH, sum( case when txn_type <> 'deposit' then - txn_amount ELSE txn_amount END) closing_balance 
      FROM DataBank.dbo.customer_transactions 
      GROUP BY  customer_id , DATENAME( MONTH, txn_date) 
      ORDER BY customer_id , DATENAME( MONTH, txn_date) ;
   ```
  result of 10 rows:
  
  | customer_id |   MONTH   | closing_balance |
  |-------------|-----------|-----------------|
  |      1      |  January  |       312       |
  |      1      |   March   |      -952       |
  |      2      |  January  |       549       |
  |      2      |   March   |        61       |
  |      3      |   April   |       493       |
  |      3      | February  |      -965       |
  |      3      |  January  |       144       |
  |      3      |   March   |      -401       |
  |      4      |  January  |       848       |
  |      4      |   March   |      -193       |

5. What is the percentage of customers who increase their closing balance by more than 5%?
    - Answer:
    ```sql
        With CTE_balance AS (SELECT  customer_id ,EOMONTH(txn_date) end_month, sum( case when txn_type <> 'deposit' then  - txn_amount ELSE txn_amount END) closing_balance
        FROM DataBank.dbo.customer_transactions 
        GROUP BY  customer_id ,EOMONTH(txn_date)),
        CTE_bal  AS ( SELECT *,  LAG(closing_balance, 1, closing_balance ) OVER (ORDER by  customer_id ,end_month )  amount_before,
        closing_balance - (LAG(closing_balance, 1, closing_balance ) OVER (ORDER by  customer_id ,end_month ))  percent_increase
       FROM CTE_balance
       )
       SELECT CAST(count( distinct customer_id) AS FLOAT) / CONVERT(FLOAT, (SELECT  count( distinct customer_id)  
       FROM DataBank.dbo.customer_transactions ) )*100  
       FROM CTE_bal
       WHERE   percent_increase >5;
    ```
    result5:
   The percentage of customers who increase their closing balance by more than 5% is approximately 95.8%.
   
 
