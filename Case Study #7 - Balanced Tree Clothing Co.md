 # Case Study #7 - Balanced Tree Clothing Co

## Introduction

Balanced Tree Clothing Company prides themselves on providing an optimized range of clothing and lifestyle wear for the modern adventurer! Danny, the CEO of this trendy fashion company, has asked you to assist the team's merchandising teams in analyzing their sales performance and generating a basic financial report to share with the wider business.

## Available Data

For this case study, there are a total of 4 datasets. However, you will primarily utilize 2 main tables to solve regular questions, with the additional 2 tables used for the bonus challenge question.

### Product Details

The `balanced_tree.product_details` table includes all information about the entire range that Balanced Clothing sells in their store. It contains the following columns:

- `product_id`
- `price`
- `product_name`
- `category_id`
- `segment_id`
- `style_id`
- `category_name`
- `segment_name`
- `style_name`

### Product Sales

The `balanced_tree.sales` table contains product-level information for all the transactions made for Balanced Tree, including quantity, price, percentage discount, member status, a transaction ID, and the transaction timestamp. It has the following columns:

- `prod_id`
- `qty`
- `price`
- `discount`
- `member`
- `txn_id`
- `start_txn_time`

### Product Hierarchy & Product Price (for Bonus Challenge)

These tables are used for the bonus question to recreate the `balanced_tree.product_details` table.

#### `balanced_tree.product_hierarchy`

- `id`
- `parent_id`
- `level_text`
- `level_name`

#### `balanced_tree.product_prices`

- `id`
- `product_id`
- `price`

## Case Study Questions

### High Level Sales Analysis

1. What was the total quantity sold for all products?
```sql
SELECT SUM(qty) total_quantity_sold_discont 
  FROM [Balanced_Tree].[Balanced_Tree].[sales];
```
result:
total_quantity_sold_discount	45216
2. What is the total generated revenue for all products before discounts?
```sql
SELECT SUM(qty*price) total_quantity_sold_ndiscont 
  FROM [Balanced_Tree].[Balanced_Tree].[sales];
```
result:
total_quantity_sold_ndiscont  1289453
3. What was the total discount amount for all products?
```sql
 SELECT SUM(qty*price* CONVERT(FLOAT,(discount))/100) total__discont_amount
  FROM [Balanced_Tree].[Balanced_Tree].[sales];
```
result: 
 total__discont_amount  156229.14
### Transaction Analysis

4. How many unique transactions were there?
```sql
SELECT count (DISTINCT[txn_id]) unique_transactions
  FROM [Balanced_Tree].[Balanced_Tree].[sales];
```
result: 
 unique_transactions 2500
5. What is the average unique products purchased in each transaction?
```sql
WITH CTE_c_pro AS(  SELECT [txn_id], count (DISTINCT prod_id) total_pro
  FROM [Balanced_Tree].[Balanced_Tree].[sales]
  GROUP BY [txn_id])
  SELECT AVG(total_pro)  average_unique_products FROM CTE_c_pro;
```
result:
average_unique_products: 6
6. What are the 25th, 50th, and 75th percentile values for the revenue per transaction?
```sql
   WITH CTE_revenue AS( SELECT
   txn_id, SUM(qty*price* CONVERT(FLOAT,(discount))/100) revenue_transaction
   FROM [Balanced_Tree].[Balanced_Tree].[sales]
   GROUP BY txn_id) 
   SELECT TOP 1 PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY revenue_transaction )OVER() AS Percentile_25,
   PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY revenue_transaction ) OVER() AS median,
   PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY revenue_transaction ) OVER() AS Percentile_75
  FROM CTE_revenue;
```
result:
| Metric             | Percentile 25th | Median  | Percentile 75th |
| ------------------ | --------------- | ------- | --------------- |
| Value              | 24.9975          | 54.675  | 91.375          |

7. What is the average discount value per transaction?
```sql
SELECT  txn_id, AVG(discount) discount_value
  FROM [Balanced_Tree].[Balanced_Tree].[sales]
 GROUP BY  txn_id ;
```
result:
| txn_id  | discount_value |
| ------- | -------------- |
| d2427c  | 12             |
| 98ab3e  | 8              |
| 64d391  | 15             |
| ...     | ...            |
| 9bb6... | 12             |
| 72ca... | 8              |
| 3d5e... | 10             |

8. What is the percentage split of all transactions for members vs non-members?
```sql
 SELECT  100*(SELECT COUNT(DISTINCT txn_id) con_m FROM [Balanced_Tree].[Balanced_Tree].[sales]
  where member = 't')/ COUNT(DISTINCT  txn_id) per_members, 100*(SELECT COUNT(DISTINCT txn_id) con_m FROM [Balanced_Tree].[Balanced_Tree].[sales]
  where member = 'f')/ COUNT(DISTINCT  txn_id) per_no_members FROM [Balanced_Tree].[Balanced_Tree].[sales];
```
result:
| Percentage for Members | Percentage for Non-Members |
| ----------------------- | -------------------------- |
| 60                      | 39                      |

9. What is the average revenue for member transactions and non-member transactions?
```sql
 SELECT  ROUND( AVG( CASE WHEN  member = 't'  THEN  qty*price*(1 - CONVERT(FLOAT,(discount))/100)ELSE 0 END),2)  AVG_REVENU_M 
  ,ROUND(AVG( CASE WHEN  member = 'F'  THEN  qty*price*(1 - CONVERT(FLOAT,(discount))/100)ELSE 0 END),2)  AVG_REVENU_N 
  FROM [Balanced_Tree].[Balanced_Tree].[sales];
```
result:
| Average Revenue for Members | Average Revenue for Non-Members |
| ---------------------------- | ------------------------------- |
| 45.28                        | 29.79                           |

### Product Analysis

10. What are the top 3 products by total revenue before discount?
```sql
SELECT  TOP 3 p.product_name, SUM(s.qty*s.price) revenue_ndiscont 
  FROM [Balanced_Tree].[Balanced_Tree].[sales] s
  JOIN [Balanced_Tree].[Balanced_Tree].[product_details] p
  ON s.prod_id = p.product_id
  GROUP BY p.product_name
  ORDER BY revenue_ndiscont DESC;
```
| Product Name                      | Revenue (No Discount) |
| --------------------------------- | --------------------- |
| Blue Polo Shirt - Mens            | 217,683               |
| Grey Fashion Jacket - Womens     | 209,304               |
| White Tee Shirt - Mens            | 152,000               |

11. What is the total quantity, revenue, and discount for each segment?
```sql
SELECT p.segment_name , SUM(s.qty)total_quantity,  SUM(qty*s.price*(1 - CONVERT(FLOAT,(discount))/100)) total_revenue, SUM(qty*s.price* CONVERT(FLOAT,(discount))/100) total_discount
  FROM [Balanced_Tree].[Balanced_Tree].[sales] s
  JOIN [Balanced_Tree].[Balanced_Tree].[product_details] p
  ON s.prod_id = p.product_id
  GROUP BY p.segment_name;
```
result:
| Segment Name | Total Quantity | Total Revenue | Total Discount |
| ------------ | -------------- | ------------- | --------------- |
| Jacket       | 11,385         | 322,705.54    | 44,277.46      |
| Jeans        | 11,349         | 183,006.03    | 25,343.97      |
| Shirt        | 11,265         | 356,548.73    | 49,594.27      |
| Socks        | 11,217         | 270,963.56    | 37,013.44      |

12. What is the top-selling product for each segment?
```sql
WITH CTE_SEG AS( SELECT p.segment_name  , product_name , SUM(qty*s.price*(1 - CONVERT(FLOAT,(discount))/100)) selling_segment, 
  ROW_NUMBER() OVER(PARTITION BY p.segment_name ORDER BY SUM(qty*s.price*(1 - CONVERT(FLOAT,(discount))/100)) desc) row_s
  FROM [Balanced_Tree].[Balanced_Tree].[sales] s
  JOIN [Balanced_Tree].[Balanced_Tree].[product_details] p
  ON s.prod_id = p.product_id
  GROUP BY p.segment_name , product_name
  )
  SELECT segment_name , product_name ,selling_segment FROM CTE_SEG
  WHERE row_s =1;
```
result:
| Segment Name | Best-Selling Product                 | Sales Amount |
| ------------ | ----------------------------------- | ------------ |
| Jacket       | Grey Fashion Jacket - Womens        | 183,912.12   |
| Jeans        | Black Straight Jeans - Womens       | 106,407.04   |
| Shirt        | Blue Polo Shirt - Mens              | 190,863.93   |
| Socks        | Navy Solid Socks - Mens             | 119,861.64   |

13. What is the total quantity, revenue, and discount for each category?
```sql
SELECT p.category_name , SUM(s.qty)total_quantity,  SUM(qty*s.price*(1 - CONVERT(FLOAT,(discount))/100)) total_revenue,
   SUM(qty*s.price* CONVERT(FLOAT,(discount))/100) total_discount
  FROM [Balanced_Tree].[Balanced_Tree].[sales] s
  JOIN [Balanced_Tree].[Balanced_Tree].[product_details] p
  ON s.prod_id = p.product_id
  GROUP BY p.category_name;
```
result:
| Category | Total Quantity | Total Revenue    | Total Discount |
| -------- | -------------- | ---------------- | -------------- |
| Mens     | 22,482         | 627,512.29       | 86,607.71      |
| Womens   | 22,734         | 505,711.57       | 69,621.43      |

14. What is the top-selling product for each category?
```sql
WITH CTE_CAT AS( SELECT p.category_name , product_name , SUM(qty*s.price*(1 - CONVERT(FLOAT,(discount))/100)) selling_category,
 ROW_NUMBER() OVER(PARTITION BY p.category_name ORDER BY SUM(qty*s.price*(1 - CONVERT(FLOAT,(discount))/100)) desc) row_s
  FROM [Balanced_Tree].[Balanced_Tree].[sales] s
  JOIN [Balanced_Tree].[Balanced_Tree].[product_details] p
  ON s.prod_id = p.product_id
  GROUP BY p.category_name, product_name)
  SELECT category_name , product_name , selling_category FROM CTE_CAT
  WHERE row_s =1;
```
result:
| Category | Top Selling Product             | Sales Amount |
| -------- | ------------------------------- | ------------ |
| Mens     | Blue Polo Shirt - Mens         | 190,863.93   |
| Womens   | Grey Fashion Jacket - Womens   | 183,912.12   |

15. What is the percentage split of revenue by product for each segment?
```sql
WITH CTE_pro AS (  SELECT segment_name, product_name, SUM(qty*s.price*(1 - CONVERT(FLOAT,(discount))/100)) revenue_pro
  FROM [Balanced_Tree].[Balanced_Tree].[sales] s
  JOIN [Balanced_Tree].[Balanced_Tree].[product_details] p
  ON s.prod_id = p.product_id
  GROUP BY segment_name, product_name),
 CTE_seg AS( SELECT segment_name, SUM(qty*s.price*(1 - CONVERT(FLOAT,(discount))/100)) revvenue_seg
  FROM [Balanced_Tree].[Balanced_Tree].[sales] s
  JOIN [Balanced_Tree].[Balanced_Tree].[product_details] p
  ON s.prod_id = p.product_id
  GROUP BY segment_name)
  SELECT p.segment_name, p.product_name ,ROUND( revenue_pro / revvenue_seg *100, 2 )
  FROM CTE_pro p JOIN  CTE_seg s ON p.segment_name =s.segment_name
  GROUP BY p.segment_name, p.product_name ,revenue_pro, revvenue_seg
  ORDER BY p.segment_name, p.product_name;
```
result:
| Segment | Product Name                   | Revenue Percentage |
| ------- | ------------------------------ | ------------------ |
| Jacket  | Grey Fashion Jacket - Womens  | 56.99%             |
| Jacket  | Indigo Rain Jacket - Womens   | 19.44%             |
| Jacket  | Khaki Suit Jacket - Womens    | 23.57%             |
| Jeans   | Black Straight Jeans - Womens | 58.14%             |
| Jeans   | Cream Relaxed Jeans - Womens  | 17.82%             |
| Jeans   | Navy Oversized Jeans - Womens | 24.04%             |
| Shirt   | Blue Polo Shirt - Mens        | 53.53%             |
| Shirt   | Teal Button Up Shirt - Mens   | 8.99%              |
| Shirt   | White Tee Shirt - Mens        | 37.48%             |
| Socks   | Navy Solid Socks - Mens       | 44.24%             |
| Socks   | Pink Fluro Polkadot Socks - Mens | 35.57%           |
| Socks   | White Striped Socks - Mens    | 20.2%              |

16. What is the percentage split of revenue by segment for each category?
```sql
WITH CTE_segment AS (  SELECT category_name, segment_name, SUM(qty*s.price*(1 - CONVERT(FLOAT,(discount))/100)) revenue_seg
  FROM [Balanced_Tree].[Balanced_Tree].[sales] s
  JOIN [Balanced_Tree].[Balanced_Tree].[product_details] p
  ON s.prod_id = p.product_id
  GROUP BY category_name, segment_name),
 CTE_categ AS( SELECT category_name, SUM(qty*s.price*(1 - CONVERT(FLOAT,(discount))/100)) revvenue_categ
  FROM [Balanced_Tree].[Balanced_Tree].[sales] s
  JOIN [Balanced_Tree].[Balanced_Tree].[product_details] p
  ON s.prod_id = p.product_id
  GROUP BY category_name)
  SELECT s.category_name, s.segment_name ,ROUND( revenue_seg / revvenue_categ *100, 2 )
  FROM CTE_segment s JOIN  CTE_categ c ON s.category_name = c.category_name
  GROUP BY s.category_name, s.segment_name ,revenue_seg, revvenue_categ
  ORDER BY s.category_name, s.segment_name ;

```
result:
| Category | Segment | Revenue Percentage |
| -------- | ------- | ------------------ |
| Mens     | Shirt   | 56.82%             |
| Mens     | Socks   | 43.18%             |
| Womens   | Jacket  | 63.81%             |
| Womens   | Jeans   | 36.19%             |

17. What is the percentage split of total revenue by category?
```sql
 WITH  CTE_categ AS( SELECT category_name, SUM(qty*s.price*(1 - CONVERT(FLOAT,(discount))/100)) revvenue_categ
  FROM [Balanced_Tree].[Balanced_Tree].[sales] s
  JOIN [Balanced_Tree].[Balanced_Tree].[product_details] p
  ON s.prod_id = p.product_id
  GROUP BY category_name)
  SELECT category_name, revvenue_categ,  revvenue_categ / (SELECT SUM(qty* price *(1 - CONVERT(FLOAT,(discount))/100))FROM [Balanced_Tree].[Balanced_Tree].[sales])
*100  FROM  CTE_categ
 GROUP BY  category_name, revvenue_categ ;
```
result:
| Category | Total Revenue | Revenue Percentage |
| -------- | ------------- | ------------------ |
| Mens     | 627,512.29    | 55.37%             |
| Womens   | 505,711.57    | 44.63%             |

18. What is the total transaction "penetration" for each product?
```sql
WITH CTE_penetration AS(SELECT txn_id, count(txn_id)/CONVERT(FLOAT,(SELECT count(DISTINCT txn_id) FROM  [Balanced_Tree].[Balanced_Tree].[sales])) penetration 
	FROM  [Balanced_Tree].[Balanced_Tree].[sales] 
	GROUP BY txn_id)
	SELECT  product_name, ROUND(sum(penetration), 2)
	FROM  [Balanced_Tree].[Balanced_Tree].[sales] s JOIN CTE_penetration pe
	ON s.txn_id = pe.txn_id
	JOIN [Balanced_Tree].[Balanced_Tree].[product_details] p
	ON s.prod_id =p.product_id
	GROUP BY  product_name;
```
result:
| Product Name                      | Penetration |
| --------------------------------- | ----------- |
| Black Straight Jeans - Womens    | 3.25        |
| Blue Polo Shirt - Mens           | 3.31        |
| Cream Relaxed Jeans - Womens     | 3.23        |
| Grey Fashion Jacket - Womens     | 3.36        |
| Indigo Rain Jacket - Womens      | 3.26        |
| Khaki Suit Jacket - Womens       | 3.23        |
| Navy Oversized Jeans - Womens    | 3.29        |
| Navy Solid Socks - Mens          | 3.34        |
| Pink Fluro Polkadot Socks - Mens | 3.27        |
| Teal Button Up Shirt - Mens      | 3.28        |
| White Striped Socks - Mens       | 3.23        |
| White Tee Shirt - Mens           | 3.31        |

19. What is the most common combination of at least 1 quantity of any 3 products in a single transaction?
```sql
WITH CTE_combination AS( SELECT P.product_name, s.txn_id, s.prod_id, s.qty
  FROM [Balanced_Tree].[Balanced_Tree].[sales] s
  JOIN [Balanced_Tree].[Balanced_Tree].[product_details] p
  ON s.prod_id = p.product_id)
	  SELECT top 1
    p1.product_name AS product_name1,
    p2.product_name AS product_name2,
    p3.product_name AS product_name3,
    COUNT(*) AS combination_count
FROM
    CTE_combination p1
JOIN
    CTE_combination p2
ON
    p1.txn_id = p2.txn_id AND p1.prod_id < p2.prod_id AND p1.qty >= 1 AND p2.qty >= 1
JOIN
    CTE_combination p3
ON
    p1.txn_id = p3.txn_id  AND p1.prod_id < p3.prod_id AND p2.prod_id < p3.prod_id AND p3.qty >= 1
	GROUP BY
    p1.product_name, p2.product_name, p3.product_name
ORDER BY
    combination_count DESC;
```
result:
| Product Name 1              | Product Name 2             | Product Name 3           | Combination Count |
| --------------------------- | --------------------------- | ------------------------- | ----------------- |
| White Tee Shirt - Mens     | Grey Fashion Jacket - Womens| Teal Button Up Shirt - Mens | 352              |

### Reporting Challenge

Write a single SQL script that combines all of the previous questions into a scheduled report that the Balanced Tree team can run at the beginning of each month to calculate the previous month's values. Consider generating the data for January and demonstrate that it can be easily adapted for February.

### Bonus Challenge

Use a single SQL query to transform the `product_hierarchy` and `product_prices` datasets to the `product_details` table. You may want to consider using a recursive CTE to solve this problem.

## Conclusion

Sales, transactions, and product exposure are essential aspects of analysis for companies selling products. Being able to navigate product hierarchies and understand the different levels of the structure, as well as joining these details to sales-related datasets, is valuable for anyone working in financial, customer, or exploratory analytics roles. This case study provides exposure to the type of analysis performed in these roles.

For detailed analysis, you can use the embedded SQL instance provided in the document.

 
