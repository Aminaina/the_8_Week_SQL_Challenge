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
   - Answer: [Insert SQL Query and Result]

2. **What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value?**
   - Answer: [Insert SQL Query and Result]

3. **What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name.**
   - Answer: [Insert SQL Query and Result]

4. **What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**
   - Answer: [Insert SQL Query and Result]

5. **How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?**
   - Answer: [Insert SQL Query and Result]

6. **What is the number and percentage of customer plans after their initial free trial?**
   - Answer: [Insert SQL Query and Result]

7. **What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?**
   - Answer: [Insert SQL Query and Result]

8. **How many customers have upgraded to an annual plan in 2020?**
   - Answer: [Insert SQL Query and Result]

9. **How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?**
   - Answer: [Insert SQL Query and Result]

10. **Can you further break down this average value into 30-day periods (i.e., 0-30 days, 31-60 days, etc.)?**
    - Answer: [Insert SQL Query and Result]

11. **How many customers downgraded from a pro monthly to a basic monthly plan in 2020?**
    - Answer: [Insert SQL Query and Result]

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

