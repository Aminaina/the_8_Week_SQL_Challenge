# Case Study #8 - Fresh Segments

## Introduction

Danny created Fresh Segments, a digital marketing agency that helps other businesses analyze trends in online ad click behavior for their unique customer base.

Clients share their customer lists with the Fresh Segments team who then aggregate interest metrics and generate a single dataset worth of metrics for further analysis.

In particular, the composition and rankings for different interests are provided for each client showing the proportion of their customer list who interacted with online assets related to each interest for each month.

Danny has asked for your assistance to analyze aggregated metrics for an example client and provide some high-level insights about the customer list and their interests.

## Available Data

For this case study, there are two datasets that you will need to use to solve the questions:

### Interest Metrics

This table contains information about aggregated interest metrics for a specific major client of Fresh Segments, which makes up a large proportion of their customer base.

| _month | _year | month_year | interest_id | composition | index_value | ranking | percentile_ranking |
|--------|-------|------------|-------------|-------------|-------------|---------|--------------------|
| 7      | 2018  | 07-2018    | 32486       | 11.89       | 6.19        | 1       | 99.86              |
| 7      | 2018  | 07-2018    | 6106        | 9.93        | 5.31        | 2       | 99.73              |
| ...    | ...   | ...        | ...         | ...         | ...         | ...     | ...                |

### Interest Map

This mapping table links the interest_id with their relevant interest information.

| id  | interest_name                | interest_summary                             | created_at         | last_modified      |
|----|-----------------------------|----------------------------------------------|--------------------|--------------------|
| 1  | Fitness Enthusiasts         | Consumers using fitness tracking apps and websites. | 2016-05-26 14:57:59 | 2018-05-23 11:30:12 |
| 2  | Gamers                      | Consumers researching game reviews and cheat codes. | 2016-05-26 14:57:59 | 2018-05-23 11:30:12 |
| ...| ...                         | ...                                          | ...                | ...                |

## Case Study Questions

### Data Exploration and Cleansing

1. Update the `fresh_segments.interest_metrics` table by modifying the `month_year` column to be a date data type with the start of the month. 
```sql
 ALTER TABLE [Fresh_Segments].[dbo].[interest_metrics]
ALTER COLUMN month_year VARCHAR(10);
 UPDATE [Fresh_Segments].[dbo].[interest_metrics]
SET month_year = FORMAT(CONVERT(DATE, '01-' + month_year, 103), 'yyyy-MM-dd')
WHERE month_year LIKE '%2%';
```
2. What is the count of records in the `fresh_segments.interest_metrics` for each `month_year` value sorted in chronological order (earliest to latest) with the null values appearing first?
```sql
SELECT month_year, count(*) records
FROM [Fresh_Segments].[dbo].[interest_metrics]
 GROUP BY month_year
 ORDER BY ISNULL(month_year, '01-01-1990') asc ;
```
result2:
| month_year  | records |
|-------------|---------|
| NULL        | 1194    |
| 2018-07-01  | 729     |
| 2018-08-01  | 767     |
| 2018-09-01  | 780     |
| 2018-10-01  | 857     |
| 2018-11-01  | 928     |
| 2018-12-01  | 995     |
| 2019-01-01  | 973     |
| 2019-02-01  | 1121    |
| 2019-03-01  | 1136    |
| 2019-04-01  | 1099    |
| 2019-05-01  | 857     |
| 2019-06-01  | 824     |
| 2019-07-01  | 864     |
| 2019-08-01  | 1149    |

3. What do you think we should do with these null values in the `fresh_segments.interest_metrics`?
Leave Them As Is: If null values signify missing or unknown data and their presence doesn't impact the analysis,
  they can be left untouched. This approach is suitable when working with available data is the priority.
Impute Values: Depending on the nature of the analysis, null values can be replaced with meaningful default values
or calculated estimates. For example, the average value for a specific column or related record can be imputed to ensure continuity in the dataset.
Exclude from Analysis: If null values are irrelevant to the analysis or would distort the outcomes,
it might be prudent to exclude rows with null values. This ensures that results accurately reflect the data's patterns.
Investigate Data Collection Process: Unexpected null values could stem from errors in the data collection process.
Investigating and rectifying these issues at the data source is crucial to maintain data integrity and accuracy.
Special Treatment: Null values can hold specific meanings in certain cases. Treating them as distinct categories can be helpful,
especially when they indicate events that haven't occurred yet or represent unique conditions.
Consult Domain Experts: Collaborating with domain experts who understand the data and its context can provide 
insights into the significance of null values. Their input can guide decisions on how to handle these values effectively.
The chosen approach should be guided by the objectives of the analysis, the impact of null values on results,
and the transparency required for stakeholders. Ultimately, ensuring accurate and reliable insights is paramount.
```sql
SELECT interest_id, COUNT(*) AS null_count
FROM [Fresh_Segments].[dbo].[interest_metrics]
WHERE interest_id IS NULL
GROUP BY interest_id;
SELECT month_year, COUNT(*) AS null_count
FROM [Fresh_Segments].[dbo].[interest_metrics]
WHERE month_year IS NULL
GROUP BY month_year;

```
4. How many `interest_id` values exist in the `fresh_segments.interest_metrics` table but not in the `fresh_segments.interest_map` table? What about the other way around?
```sql
SELECT  count( distinct me.interest_id)  MissingInterestIDsCount
FROM  [Fresh_Segments].[dbo].[interest_metrics] me
left JOIN [Fresh_Segments].[dbo].[interest_map] ma
ON me.interest_id = ma.id
WHERE ma.id IS NULl ;
 --other method
 SELECT count( distinct interest_id) MissingInterestIDsCount FROM [Fresh_Segments].[dbo].[interest_metrics]
 WHERE interest_id not in(select id FROM  [Fresh_Segments].[dbo].[interest_map]);
```
result4:
| MissingInterestIDsCount |
|-------------------------|
|           0             |

5. Summarize the `id` values in the `fresh_segments.interest_map` by its total record count in this table.
```sql
select id, count(*) AS Total_Record_Count FROM  [Fresh_Segments].[dbo].[interest_map]
group by id ;
```
6. What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where `interest_id = 21246` in your joined output and include
 all columns from `fresh_segments.interest_metrics` and all columns from `fresh_segments.interest_map` except from the `id` column.
```sql
SELECT  * 
FROM  [Fresh_Segments].[dbo].[interest_metrics] me
 JOIN [Fresh_Segments].[dbo].[interest_map] ma
ON me.interest_id = ma.id
where me.interest_id = 21246;
```
result6:
| _month | _year | month_year | interest_id | composition | index_value | ranking | percentile_ranking | interest_name           | interest_summary                                      | created_at            | last_modified          |
|--------|-------|------------|-------------|-------------|-------------|---------|-------------------|-------------------------|-------------------------------------------------------|-----------------------|-----------------------|
| 7      | 2018  | 2018-07-01 | 21246       | 2.26        | 0.65        | 722     | 0.96              | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |
| 8      | 2018  | 2018-08-01 | 21246       | 2.13        | 0.59        | 765     | 0.26              | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |
| 9      | 2018  | 2018-09-01 | 21246       | 2.06        | 0.61        | 774     | 0.77              | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |
| 10     | 2018  | 2018-10-01 | 21246       | 1.74        | 0.58        | 855     | 0.23              | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |
| 11     | 2018  | 2018-11-01 | 21246       | 2.25        | 0.78        | 908     | 2.16              | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |
| 12     | 2018  | 2018-12-01 | 21246       | 1.97        | 0.7         | 983     | 1.21              | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |
| 1      | 2019  | 2019-01-01 | 21246       | 2.05        | 0.76        | 954     | 1.95              | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |
| 2      | 2019  | 2019-02-01 | 21246       | 1.84        | 0.68        | 1109    | 1.07              | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |
| 3      | 2019  | 2019-03-01 | 21246       | 1.75        | 0.67        | 1123    | 1.14              | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |
| 4      | 2019  | 2019-04-01 | 21246       | 1.58        | 0.63        | 1092    | 0.64              | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |
| NULL   | NULL  | NULL       | 21246       | 1.61        | 0.68        | 1191    | 0.25              | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |

7. Are there any records in your joined table where the `month_year` value is before the `created_at` value from the `fresh_segments.interest_map` table? Do you think these values are valid, and why?
```sql
SELECT  * 
FROM  [Fresh_Segments].[dbo].[interest_metrics] me
 JOIN [Fresh_Segments].[dbo].[interest_map] ma
ON me.interest_id = ma.id
WHERE  month_year < created_at ;
```
### Interest Analysis

1. Which interests have been present in all `month_year` dates in our dataset?
```sql
SELECT  interest_name, count(distinct month_year)
FROM  [Fresh_Segments].[dbo].[interest_metrics] me
 JOIN [Fresh_Segments].[dbo].[interest_map] ma
ON me.interest_id = ma.id
GROUP by  interest_name 
HAVING count(distinct month_year) = 14;
```
2. Using this same `total_months` measure, calculate the cumulative percentage of all records starting at 14 months - which `total_months` value passes the 90% cumulative percentage value?
3. If we were to remove all `interest_id` values which are lower than the `total_months` value we found in the previous question, how many total data points would we be removing?
4. Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have fewer months present from a segment perspective.

### Segment Analysis

12. Using our filtered dataset by removing the interests with less than 6 months' worth of data, which are the top 10 and bottom 10 interests which have the largest composition values in any `month_year`? Only use the maximum composition value for each interest but you must keep the corresponding `month_year`.
13. Which 5 interests had the lowest average ranking value?
14. Which 5 interests had the largest standard deviation in their `percentile_ranking` value?
15. For the 5 interests found in the previous question, what was the minimum and maximum `percentile_ranking` values for each interest and its corresponding `year_month` value? Can you describe what is happening for these 5 interests?
16. How would you describe our customers in this segment based on their composition and ranking values? What sort of products or services should we show to these customers and what should we avoid?

### Index Analysis

17. The `index_value` is a measure which can be used to reverse calculate the average composition for Fresh Segments’ clients. Average composition can be calculated by dividing the `composition` column by the `index_value` column rounded to 2 decimal places.
18. What is the top 10 interests by the average composition for each month?
19. For all of these top 10 interests, which interest appears the most often?
20. What is the average of the average composition for the top 10 interests for each month?
21. What is the 3-month rolling average of the max average composition value from September 2018 to August 2019 and include the previous top-ranking interests in the same output shown below.
22. Provide a possible reason why the max average composition might change from month to month? Could it signal something is not quite right with the overall business model for Fresh Segments?

## Conclusion

Customer segments or marketing segments are essential in digital marketing. Businesses like Google, Facebook, Instagram, LinkedIn, and others rely on these segments to target advertising effectively.

Traditional businesses upload customer data into digital marketing systems to generate matches based on interests and traits. Understanding customer segments is crucial for successful marketing strategies.
