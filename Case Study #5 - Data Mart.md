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
- Add a `week_number` column.
- Add a `month_number` column.
- Add a `calendar_year` column.
- Add an `age_band` column.
- Add a `demographic` column.
- Ensure null values are handled appropriately.
- Generate a new `avg_transaction` column.

### 2. Data Exploration
Answer the following questions:
- What day of the week is used for each `week_date` value?
- What range of week numbers are missing from the dataset?
- How many total transactions were there for each year in the dataset?
- What is the total sales for each region for each month?
- What is the total count of transactions for each platform?
- What is the percentage of sales for Retail vs. Shopify for each month?
- What is the percentage of sales by demographic for each year?
- Which `age_band` and `demographic` values contribute the most to Retail sales?
- Can we use the `avg_transaction` column to find the average transaction size for each year for Retail vs. Shopify?

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
