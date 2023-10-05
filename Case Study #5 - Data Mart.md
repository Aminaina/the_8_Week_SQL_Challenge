# Case Study #5 - Data Mart

## Introduction
Data Mart is Danny's latest venture, and he needs your support to analyze the sales performance of his online supermarket that specializes in fresh produce. In June 2020, significant sustainability changes were introduced in the business operations. Danny is looking to quantify the impact of these changes on sales performance and gather insights into various business areas.

### Key Business Questions
1. What was the quantifiable impact of the changes introduced in June 2020?
2. Which platform, region, segment, and customer types were the most impacted by this change?
3. What can we do about future introductions of similar sustainability updates to the business to minimize the impact on sales?

## Available Data
For this case study, we have one primary dataset: `data_mart.weekly_sales`.

### Column Dictionary
Here's a brief explanation of the columns in the dataset:
- `week_date`: The date of the week.
- `region`: The region of operation.
- `platform`: The platform used (Shopify, Retail, etc.).
- `segment`: Customer segment.
- `customer_type`: Type of customer.
- `transactions`: Count of unique purchases.
- `sales`: Total sales amount.

### Example Rows
Here are some example rows from the dataset:

| week_date  | region        | platform | segment | customer_type | transactions | sales      |
|------------|---------------|----------|---------|---------------|--------------|------------|
| 9/9/20     | OCEANIA       | Shopify  | C3      | New           | 610          | 110033.89  |
| 29/7/20    | AFRICA        | Retail   | C1      | New           | 110692       | 3053771.19 |
| ...        | ...           | ...      | ...     | ...           | ...          | ...        |

## Case Study Questions

### 1. Data Cleansing Steps
In a single query, perform the following data cleansing operations and create a new table called `clean_weekly_sales`:

- Convert `week_date` to DATE format.

```sql
 -- Step 1: Create the new table 'weekly_sales_copy' with the same schema as 'weekly_sales'

CREATE TABLE [weekly_sales_copy ]
(
    -- List all the columns and their data types here, as per the schema of 'weekly_sales'
    "week_date" VARCHAR(7),
  "region" VARCHAR(13),
  "platform" VARCHAR(7),
  "segment" VARCHAR(4),
  "customer_type" VARCHAR(8),
  "transactions" INTEGER,
  "sales" INTEGER
);
 
-- Step 2: Insert data from 'weekly_sales' into 'clean_weekly_sales'
INSERT INTO weekly_sales_copy
SELECT *
FROM  [data_mart].[dbo].weekly_sales;
/** change data type of week)date fron varchar(7) tovarchar(10)  let me to convert string to date**/
ALTER TABLE weekly_sales_copy
ALTER COLUMN week_date varchar(20);

SELECT CONVERT(date, week_date, 3)
FROM   weekly_sales_copy;
/*  Convert the week_date to a DATE format */
update  [weekly_sales_copy]
Set week_date = CONVERT(date, week_date, 3);
```
- Add a `week_number` column.
  ```sql
  alter table [weekly_sales_copy]
  add  week_number int ;
  update [weekly_sales_copy]
  set week_number = DATEPART(week, week_date)
  ```
- Add a `month_number` column.
```sql
  alter table [weekly_sales_copy]
  add  month_number int ;
  update [weekly_sales_copy]
  set month_number = DATEPART(MONTH, week_date)
```
- Add a `calendar_year` column.
```sql
  alter table [weekly_sales_copy]
  add  year_number int ;
  update [weekly_sales_copy]
  set year_number = DATEPART(YEAR, week_date)
```
- Add an `age_band` column.
```sql
 ALTER TABLE [weekly_sales_copy]
 ADD age_band varchar(15);
 UPDATE [weekly_sales_copy]
  SET age_band = CASE 
    WHEN RIGHT(segment, 1) IN ('3', '4') THEN 'Retirees' 
    WHEN RIGHT(segment, 1) = '2' THEN 'Middle Aged'
    WHEN RIGHT(segment, 1) = '1' THEN 'Young Adults'
    ELSE NULL  
 END;
```
- Add a `demographic` column.
```sql
  ALTER TABLE [weekly_sales_copy]
  ADD demographic varchar(15);
  UPDATE [weekly_sales_copy]
  SET demographic = CASE 
    WHEN LEFT(segment, 1) = 'C' THEN 'Couples' 
    WHEN LEFT(segment, 1) = 'F' THEN 'Families'

    ELSE NULL 
  END;
```
- Ensure null values are handled appropriately.
```sql
 ALTER TABLE weekly_sales_copy
 ALTER COLUMN segment varchar(20); --- to avoid truncated values
 UPDATE [weekly_sales_copy]
 SET segment = 'unknown',
    age_band = 'unknown',
    demographic = 'unknown'
  WHERE demographic IS NULL AND segment IS NULL AND age_band IS NULL;
  /** it is still other row null*/
  SELECT *
  FROM [weekly_sales_copy];
   UPDATE [weekly_sales_copy]
   SET segment = 'unknown',
    age_band = 'unknown',
    demographic = 'unknown'
    WHERE segment = 'null' ; -- 3024 rows affected with this query
```
- Generate a new `avg_transaction` column.
```sql
  alter table [weekly_sales_copy]
  add   avg_transaction float;
  UPDATE [weekly_sales_copy]
  SET avg_transaction = ROUND(sales / transactions, 2)
```

### 2. Data Exploration
Answer the following questions:
- What day of the week is used for each `week_date` value?
  ```sql
  SELECT  DISTINCT week_date,  DATENAME(weekday, week_date) as day_
  FROM [weekly_sales_copy];   ---- just Monday
  ```
- What range of week numbers are missing from the dataset?
```sql
 /* In this query, we are using the master.dbo.spt_values system table, 
 which usually contains many rows and can act as a source for generating numbers.
 We filter the rows with type = 'P' to get a series of numbers from 0 to 2047. 
 Then we further filter the results to get the range from 1 to 52. */
   -- Generate a list of expected week numbers for the range of your data
 
 WITH week_number_cte AS (
 SELECT Number as week_number
 FROM master.dbo.spt_values 
 WHERE type = 'P'
  AND Number BETWEEN 1 AND 52
)  
SELECT DISTINCT week_n.week_number
FROM week_number_cte AS week_n 
LEFT JOIN [weekly_sales_copy] AS s 
  ON week_n.week_number = s.week_number
WHERE s.week_number IS NULL;
```
| Missing Week Numbers |
| ------- |
|    1    |
|    2    |
|    3    |
|    4    |
|    5    |
|    6    |
|    7    |
|    8    |
|    9    |
|   10    |
|   11    |
|   12    |
|   37    |
|   38    |
|   39    |
|   40    |
|   41    |
|   42    |
|   43    |
|   44    |
|   45    |
|   46    |
|   47    |
|   48    |
|   49    |
|   50    |
|   51    |
|   52    |

- How many total transactions were there for each year in the dataset?
```sql
 SELECT year_number, SUM(transactions) total_transactions
 FROM [weekly_sales_copy]
 GROUP BY year_number;
```
result:
| Year | Total_transactions |
| ---- | ------------------ |
| 2018 |      346,406,460   |
| 2019 |      365,639,285   |
| 2020 |      375,813,651   |

- What is the total sales for each region for each month?
```sql
ALTER TABLE [weekly_sales_copy]
ALTER COLUMN sales BIGINT; --- to avoid eroor Arithmetic overflow error converting expression to data type int

 SELECT region, year_number,  month_number, sum(sales) AS TOTAL_SALES
 FROM [weekly_sales_copy]
GROUP BY region,  year_number, month_number
ORDER BY region,  year_number, month_number;
```
result:
| Region       | Year | Month | Total Sales    |
|--------------|------|-------|----------------|
| AFRICA       | 2018 | 3     | 130,542,213    |
| AFRICA       | 2018 | 4     | 650,194,751    |
| AFRICA       | 2018 | 5     | 522,814,997    |
| ...          | ...  | ...   | ...            |
| USA          | 2020 | 7     | 223,735,311    |
| USA          | 2020 | 8     | 277,361,606    |

- What is the total count of transactions for each platform?
```sql
SELECT platform,    SUM(transactions) AS TOTAL_SALES
 FROM [weekly_sales_copy]
GROUP BY  platform
ORDER BY  platform;
```
result:
| Platform | Total Transactions |
|----------|--------------------|
| Retail   | 1,081,934,227      |
| Shopify  | 5,925,169          |

- What is the percentage of sales for Retail vs. Shopify for each month?
```sql
with pl_per_sales as(SELECT  month_number, platform,    SUM(sales) AS TOTAL_SALES
FROM [weekly_sales_copy]
GROUP BY  month_number, platform)
SELECT  month_number,  platform,    (TOTAL_SALES / (SELECT  CONVERT(FLOAT,SUM(TOTAL_SALES))
FROM pl_per_sales))*100
FROM pl_per_sales
ORDER BY  month_number;

```
result:
| Month | Platform | Percentage of Sales |
|-------|----------|----------------------|
|   3   |  Shopify |      0.1423%         |
|   3   |   Retail |      5.6431%         |
|   4   |  Shopify |      0.4681%         |
|   4   |   Retail |     18.9860%         |
|   5   |   Retail |     16.1641%         |
|   5   |  Shopify |      0.4477%         |
|   6   |   Retail |     17.3032%         |
|   6   |  Shopify |      0.4854%         |
|   7   |  Shopify |      0.5258%         |
|   7   |   Retail |     18.8694%         |
|   8   |   Retail |     17.6505%         |
|   8   |  Shopify |      0.5305%         |
|   9   |   Retail |      2.7109%         |
|   9   |  Shopify |      0.0731%         |

- What is the percentage of sales by demographic for each year?
```sql
 with de_per_sales as( SELECT year_number, demographic, sum(sales) AS TOTAL_SALES
FROM [weekly_sales_copy]
GROUP BY year_number, demographic)
SELECT year_number, demographic, (TOTAL_SALES / (SELECT  CONVERT(FLOAT,SUM(TOTAL_SALES))
FROM de_per_sales))*100 AS  percentage_demo
FROM de_per_sales
ORDER BY year_number;
```
result:
| year_number | demographic | percentage_demo |
|------|-------------|----------------------|
| 2018 |   Unknown   |      13.1786%        |
| 2018 |  Families   |      10.1257%        |
| 2018 |   Couples   |       8.3507%        |
| 2019 |   Unknown   |      13.5797%        |
| 2019 |   Couples   |       9.2021%        |
| 2019 |  Families   |      10.9561%        |
| 2020 |   Unknown   |      13.3427%        |
| 2020 |  Families   |      11.3253%        |
| 2020 |   Couples   |       9.9391%        |

- Which `age_band` and `demographic` values contribute the most to Retail sales?
```sql
SELECT age_band , demographic, SUM(sales) Retail_sales
FROM [weekly_sales_copy]
GROUP BY  age_band , demographic
order by Retail_sales desc;
```
result:
| Age Band     | demographic |Retail_sales     |
|--------------|-------------|--------------------|
| Unknown      | Unknown     | 16,338,612,234     |
| Retirees     | Families    | 6,750,457,132      |
| Retirees     | Couples     | 6,531,115,070      |
| Middle Aged  | Families    | 4,556,141,618      |
| Young Adults | Couples     | 2,679,593,130      |
| Middle Aged  | Couples     | 1,990,499,351      |
| Young Adults | Families    | 1,897,215,692      |

- Can we use the `avg_transaction` column to find the average transaction size for each year for Retail vs. Shopify?
  ```sql
   SELECT year_number,  platform,  
   SUM(sales) / SUM(transactions) AS avg_transaction_size
  FROM [weekly_sales_copy]
  GROUP BY year_number,  platform
  ORDER BY year_number,  platform;
  ```
result:
|  year_number | platform | avg_transaction_size |
|------|----------|--------------------------|
| 2018 |  Retail  |            36            |
| 2018 | Shopify  |           192            |
| 2019 |  Retail  |            36            |
| 2019 | Shopify  |           183            |
| 2020 |  Retail  |            36            |
| 2020 | Shopify  |           179            |


### 3. Before & After Analysis
Analyze the impact of the sustainability changes introduced in June 2020:
- What is the total sales for the 4 weeks before and after 2020-06-15?
- What about the entire 12 weeks before and after?
- How do the sale metrics for these 2 periods compare with the previous years in 2018 and 2019?

### 4. Bonus Question
Identify the areas of the business with the highest negative impact on sales metrics performance in 2020 for the 12-week before and after period:
- Region
- Platform
- Age_band
- Demographic
- Customer_type

Provide recommendations and insights based on the analysis.

## Conclusion
This case study presents a real-life change in retail operations and the impact it had on sales. Analyzing such events is essential for data analytics and provides valuable lessons for addressing similar challenges in the workplace.

[Link to DB Fiddle](insert_link_here) (For interactive SQL instance)
