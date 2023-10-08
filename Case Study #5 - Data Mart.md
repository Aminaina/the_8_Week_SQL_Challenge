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
```sql
/*the week number of 2020-06-15 is 25  in 2020 so before this week it would be  before_change and other wise after_change */
SELECT *, case when week_number < 25  AND (year_number = 2020) then 'before_change'
          else 'after_change' end as period
FROM [weekly_sales_copy];
 /*  In this query, we're using the CASE statement to categorize 
 the data into two periods: "Before Change" and "After Change" based on the week_date. */
```
- What is the total sales for the 4 weeks before and after 2020-06-15?
 ```sql
 With CTE_period as (SELECT *, case when week_number < 25  AND (year_number = 2020) then 'before_change'
          else 'after_change' end as period
FROM [weekly_sales_copy]) 
 SELECT period , sum(sales) total_sales_4w
from CTE_period
where week_number  between  21 and  28
AND (year_number = 2020)
group by period ;
```
result:
|    Period     | Total Sales for 4 Weeks |
|---------------|--------------------------|
| before_change |       2,345,878,357      |
| after_change  |       2,318,994,169      |

-What is the growth or reduction rate in actual values and percentage of sales?
```sql
With CTE_period as (SELECT *, case when week_number < 25  AND (year_number = 2020) then 'before_change'
          else 'after_change' end as period
FROM [weekly_sales_copy]) ,
CTE_RA as (
 SELECT period , sum(sales) total_sales_4w
from CTE_period
where week_number  between  21 and  28
AND (year_number = 2020)
group by period )
SELECT top 1 total_sales_4w - lead(total_sales_4w) OVER (ORDER BY  period) as rate_sales, (total_sales_4w - lead(total_sales_4w) OVER (ORDER BY  period))/ 
 convert (FLOAT, lead(total_sales_4w) OVER (ORDER BY  period) )*100 percentage_sales
 FROM CTE_RA;
```
result:
| Rate Sales | Percentage Sales |
|------------|-------------------|
| -26,884,188|     -1.15         |

- What about the entire 12 weeks before and after?
  ```sql
   With CTE_period as (SELECT *, case when week_number < 25  AND (year_number = 2020)  then 'before_change'
          else 'after_change' end as period
  FROM [weekly_sales_copy]) ,
  CTE_RA as (
  SELECT period , sum(sales) total_sales_12w
  from CTE_period
    where week_number   between 13 and  36
  AND (year_number = 2020)
  group by period )
   SELECT top 1 total_sales_12w - lead(total_sales_12w) OVER (ORDER BY  period) as rate_sales, (total_sales_12w - lead(total_sales_12w) OVER (ORDER BY  period))/ 
  convert (FLOAT, lead(total_sales_12w) OVER (ORDER BY  period) )*100 percentage_sales
  FROM CTE_RA;
   ```
 result:
 | Rate Sales  | Percentage Sales |
|-------------|-------------------|
| -152,325,394|     -2.14         |

These results further emphasize the negative impact of the sustainable packaging change on sales:

Magnitude of Impact: The larger reduction in total sales over the 12-week period 
compared to the 4-week period   suggests that the negative impact on sales intensified as time went on.

Consistency of Impact: Both results indicate a reduction in sales, with the percentage reduction being
higher over the 12-week period. This consistency suggests that the impact was not just a short-term fluctuation but had a more sustained effect.

Confirmation of Negative Impact: The negative values for both the reduction rate and 
the percentage reduction confirm that the sustainable packaging change had a detrimental effect on sales during the specified periods. 
  
- How do the sale metrics for these 2 periods compare with the previous years in 2018 and 2019?
```sql
 WITH CTE_period AS (
    SELECT *, CASE WHEN week_number < 25 THEN 'before_change' ELSE 'after_change' END AS period
    FROM [weekly_sales_copy]
),
CTE_RA AS (
    SELECT
        year_number,
        period,
        SUM(sales) total_sales_12w,
        ROW_NUMBER() OVER (PARTITION BY year_number ORDER BY year_number, period) AS row_num
    FROM CTE_period
    WHERE week_number BETWEEN 13 AND 36
    GROUP BY year_number, period
),
CTE_R AS (SELECT row_num,
    year_number,
    total_sales_12w - LEAD(total_sales_12w) OVER (PARTITION BY year_number ORDER BY period) AS rate_sales,
    (total_sales_12w - LEAD(total_sales_12w) OVER (PARTITION BY year_number ORDER BY period)) / 
    CONVERT(FLOAT, LEAD(total_sales_12w) OVER (PARTITION BY year_number ORDER BY period)) * 100 AS percentage_sales
FROM CTE_RA)
SELECT  year_number, rate_sales, percentage_sales
FROM  CTE_R
where row_num = 1;
```
result:
| Year | Rate Sales   | Percentage Sales |
|------|--------------|-------------------|
| 2018 | 104,256,193  | 1.63          |
| 2019 | -20,740,294  | -0.30          |
| 2020 | -152,325,394 | -2.14          |
comment
Positive Trend in 2018 and 2019: Both 2018 and 2019 exhibited positive growth in sales,
albeit at relatively modest rates. This suggests that the business was doing reasonably well and experiencing gradual growth during these years.

Significant Decline in 2020: The sales metrics for 2020, however, show a starkly different pattern.
There was a substantial reduction in sales, resulting in a negative growth rate of -1.146%.
This negative trend strongly indicates that something impactful happened in 2020 that caused a decline in sales.

Impact of Sustainable Packaging Change: Considering the context of the sustainable packaging change in 2020,
it's reasonable to infer that this change likely contributed to the significant decline in sales during this period. 
The sales metrics align with this hypothesis, as the negative growth observed in 2020 stands in contrast to
the positive growth observed in the previous years.

In summary, the results of the sales metrics analysis provide clear evidence that the sustainable packaging 
change negatively impacted sales in 2020. The comparison with sales from previous years highlights
the distinctiveness of the situation in 2020 and suggests a potential correlation between the packaging change and the observed sales decline.

### 4. Bonus Question
Identify the areas of the business with the highest negative impact on sales metrics performance in 2020 for the 12-week before and after period:
- Region
  ```sql
  WITH CTE_period AS (
    SELECT *, CASE WHEN week_number < 25 THEN 'before_change' ELSE 'after_change' END AS period
    FROM [weekly_sales_copy]
),
CTE_RA AS (
    SELECT
        region,
        period,
        SUM(sales) total_sales_12w,
        ROW_NUMBER() OVER (PARTITION BY region ORDER BY region, period  ) AS row_num
    FROM CTE_period
    WHERE week_number BETWEEN 13 AND 36
    GROUP BY region, period
),
CTE_R AS (SELECT row_num,
    region,
  (total_sales_12w - LEAD(total_sales_12w) OVER (PARTITION BY region ORDER BY period)) / 
    CONVERT(FLOAT, LEAD(total_sales_12w) OVER (PARTITION BY region ORDER BY period)) * 100 AS percentage_sales
FROM CTE_RA)
SELECT  region, round(percentage_sales,2)
FROM  CTE_R
where row_num = 1
order by round(percentage_sales,2)  ;

  ```
result:
|     Region      | Percentage Sales |
|-----------------|-------------------|
|      ASIA       |       -1.33      |
|     OCEANIA     |       -0.87      |
|     CANADA      |       -0.85     |
|       USA       |       -0.37      |
| SOUTH AMERICA   |       -0.34      |
|     AFRICA      |        1.10      |
|     EUROPE      |        4.96      |

- Platform
```sql
  WITH CTE_period AS (
    SELECT *, CASE WHEN week_number < 25 THEN 'before_change' ELSE 'after_change' END AS period
    FROM [weekly_sales_copy]
    ),
  CTE_RA AS (
    SELECT
         platform,
        period,
        SUM(sales) total_sales_12w,
        ROW_NUMBER() OVER (PARTITION BY  platform ORDER BY  platform, period  ) AS row_num
      FROM CTE_period
     WHERE week_number BETWEEN 13 AND 36
     GROUP BY  platform, period
    ),
   CTE_R AS (SELECT row_num,
     platform,
     (total_sales_12w - LEAD(total_sales_12w) OVER (PARTITION BY platform ORDER BY period)) / 
    CONVERT(FLOAT, LEAD(total_sales_12w) OVER (PARTITION BY platform ORDER BY period)) * 100 AS percentage_sales
   FROM CTE_RA)
     SELECT   platform, round(percentage_sales,2) 
    FROM  CTE_R
   where row_num = 1
  order by  round(percentage_sales,2)  ;
```
result:
|  Platform  | Percentage Sales |
|------------|-------------------|
|   Retail   |      -0.59%       |
|  Shopify   |       9.35%       |

- Age_band
  ```sql
    WITH CTE_period AS (
      SELECT *, CASE WHEN week_number < 25 THEN 'before_change' ELSE 'after_change' END AS period
      FROM [weekly_sales_copy]
      ),
    CTE_RA AS (
    SELECT
           age_band,
           period,
           SUM(sales) total_sales_12w,
           ROW_NUMBER() OVER (PARTITION BY  age_band ORDER BY  age_band, period  ) AS row_num
           FROM CTE_period
          WHERE week_number BETWEEN 13 AND 36
         GROUP BY  age_band, period
    ),
     CTE_R AS (SELECT row_num,
         age_band,
     (total_sales_12w - LEAD(total_sales_12w) OVER (PARTITION BY age_band ORDER BY period)) / 
      CONVERT(FLOAT, LEAD(total_sales_12w) OVER (PARTITION BY age_band ORDER BY period)) * 100 AS percentage_sales
      FROM CTE_RA)
       SELECT   age_band, round(percentage_sales,2) 
       FROM  CTE_R
      where row_num = 1
      order by  round(percentage_sales,2);
  ```
result:
|   Age Band   | Percentage Sales |
|--------------|-------------------|
|   Unknown    |      -0.55      |
| Middle Aged  |      -0.22      |
| Young Adults |      -0.21      |
|   Retirees   |      -0.18      |

- Demographic
```sql
     WITH CTE_period AS (
       SELECT *, CASE WHEN week_number < 25 THEN 'before_change' ELSE 'after_change' END AS period
     FROM [weekly_sales_copy]
      ),
     CTE_RA AS (
       SELECT
         demographic,
        period,
        SUM(sales) total_sales_12w,
        ROW_NUMBER() OVER (PARTITION BY  demographic ORDER BY  demographic, period  ) AS row_num
       FROM CTE_period
      WHERE week_number BETWEEN 13 AND 36
      GROUP BY  demographic, period
        ),
     CTE_R AS (SELECT row_num,
       demographic,
      (total_sales_12w - LEAD(total_sales_12w) OVER (PARTITION BY demographic ORDER BY period)) / 
      CONVERT(FLOAT, LEAD(total_sales_12w) OVER (PARTITION BY demographic ORDER BY period)) * 100 AS percentage_sales
      FROM CTE_RA)
     SELECT   demographic, round(percentage_sales,2) 
     FROM  CTE_R
       where row_num = 1
      order by  round(percentage_sales,2);
```
result:
|  Demographic  | Percentage Sales |
|---------------|-------------------|
|    Unknown    |      -0.55%      |
|    Couples    |      -0.29%      |
|   Families    |      -0.12%      |

- Customer_type
```sql
      WITH CTE_period AS (
      SELECT *, CASE WHEN week_number < 25 THEN 'before_change' ELSE 'after_change' END AS period
      FROM [weekly_sales_copy]
       ),
       CTE_RA AS (
        SELECT
          customer_type,
          period,
          SUM(sales) total_sales_12w,
        ROW_NUMBER() OVER (PARTITION BY  customer_type ORDER BY  customer_type, period  ) AS row_num
       FROM CTE_period
       WHERE week_number BETWEEN 13 AND 36
       GROUP BY  customer_type, period
         ),
     CTE_R AS (SELECT row_num,
     customer_type,
      (total_sales_12w - LEAD(total_sales_12w) OVER (PARTITION BY customer_type ORDER BY period)) / 
       CONVERT(FLOAT, LEAD(total_sales_12w) OVER (PARTITION BY customer_type ORDER BY period)) * 100 AS percentage_sales
       FROM CTE_RA)
    SELECT    customer_type, round(percentage_sales,2) 
   FROM  CTE_R
    where row_num = 1
   order by  round(percentage_sales,2);
```
result:
| Customer Type | Percentage Sales |
|---------------|-------------------|
|   Existing    |      -0.51%      |
|     Guest     |      -0.46%      |
|      New      |       0.69%      |

Provide recommendations and insights based on the analysis.
age_band has the highest negative impact in sales metrics
4.2 the recommendations based on the negative impact on sales for different age bands:

Unknown Age Band: Address missing or incomplete age data to improve accuracy and better understand customer demographics.

Middle Aged and Young Adults: Tailor marketing efforts and product offerings to align with preferences of these age groups, potentially boosting engagement.

Retirees: Explore external factors affecting purchasing behavior among retirees and engage directly to gain insights into their needs.

In-Depth Analysis: Investigate specific products, promotions, or channels that might be contributing to negative impacts within each age band.

Customer Input: Collect feedback through surveys and interactions to gain qualitative insights into age group preferences and concerns.

Experiment and Refine: Implement targeted strategies for each age band and closely monitor results, iterating as needed based on performance.

Holistic Understanding: Analyze how age bands intersect with other dimensions to uncover potential combined influences on sales metrics.

These recommendations offer actionable steps to address the challenges highlighted by the analysis, ultimately driving improved sales performance 
across different age bands.

## Conclusion
This case study presents a real-life change in retail operations and the impact it had on sales. Analyzing such events is essential for data analytics and provides valuable lessons for addressing similar challenges in the workplace.

[Link to DB Fiddle](insert_link_here) (For interactive SQL instance)
