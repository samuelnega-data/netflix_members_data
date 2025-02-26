# Netflix Members Data Analysis

```sql
Tools being used for the project: SQL 
```

## Overview
In this project, I will analyze the Netflix dataset to gain deeper insights into Netflix users and identify key features that contribute to user loyalty on the platform.

## Data exploration
After exploring the dataset, I found that no cleaning or transformation is necessary, as it is well-structured and ready for analysis. Here are some key observations specifically related to the dataset.
- Churn Status: 1 = Yes, 0 = No
- Support Queries would indicate the number of times each customer queried for help
- Promotional Offers: 1 = Yes, 0 = No
- Numbers of Profiles Created: I am assuming this is the number of people that are under the users profile

## Data Analysis (Queries)
Lets take a look at the data
```sql
select *
from netflix 
```

**What type of genres have the highest engagement rate?:** Everyone genre seems to have a 5/10 customer satisfaction regardless of genre.
```sql
select Genre_Preference as Genres, avg(Customer_Satisfaction_Score_1_10) as Customer_Satisfaction
from netflix 
group by Genre_Preference
```

**What type of devices do customers have the lowest engagement rate with?:** Everyone device seems to have a 5/10 engagement rate regardless of genre.
```sql
select Device_Used_Most_Often, avg(Engagement_Rate_1_10)
from netflix 
group by Device_Used_Most_Often
```
**What is the daily watch time in hours per genre?:** We can see that Drama has the highest total watch time followed by comedy, and the least watched genres would be action and sci-fi. 
```sql
select Genre_Preference, sum(Daily_Watch_Time_Hours) as Total_Watch_Time
from netflix 
group by Genre_Preference
order by Total_Watch_Time desc
```
**What is the relationship between the subscription length month and numbers of profiles created?:** We can see that for subscriptions 5 profiles created that 569 customers have been subscribed for over a year while only 134 customers have been around for only one year 
```sql
select max(Number_of_Profiles_Created)
from netflix 

with cte as (select Number_of_Profiles_Created, Subscription_Length_Months
from netflix 
where Number_of_Profiles_Created = 5) 
select 
	count(case when Subscription_Length_Months > 12 then 1 else null end) as Sub_over_year,
	count(case when Subscription_Length_Months < 12 then 1 else null end) as Sub_less_year
from cte 
```
**What is the churn rate compared to support queries?:** What was predicted was expected so the higher queries logged accounts did have a higher churn rate, one other interesting insight was the 1&2 support queries requested customers also have a pretty high churn rate which is suprising
```sql
with cte as (select Support_Queries_Logged,
	count(case when churn_status_yes_no = 1 then 1 else null end) as num_of_churns,
	count(case when churn_status_yes_no = 0 then 1 else null end) as num_of_no_churns
from netflix 
group by Support_Queries_Logged)
select *, ((num_of_churns)*1.0/(num_of_churns + num_of_no_churns)) as churn_percentage
from cte 
order by churn_percentage desc
```
**Relaitionship between monthly income and subscription plan?:** One insight i was able to draw is that people making 9-10k a month shouldnt be marketed towards they account for so little in our customers we should focus and target people making 3-9k a month. 
```sql
with cte as (select Subscription_Plan,
case 
    when Monthly_Income > 1000 AND Monthly_Income < 3000 then '1000-3000' 
    when Monthly_Income > 3000 AND Monthly_Income < 6000 then '3000-6000' 
	when Monthly_Income > 6000 AND Monthly_Income < 9000 then '6000-9000' 
	when Monthly_Income > 9000 AND Monthly_Income < 10000 then '9000-10000' else null end as Salary_Range 
from netflix 
)
select Subscription_Plan,
	count (case when Salary_Range = '1000-3000' then 1 else null end) as Sal_1000_3000,
	count (case when Salary_Range = '3000-6000' then 1 else null end) as Sal_3000_6000,
	count (case when Salary_Range = '6000-9000' then 1 else null end) as Sal_6000_9000,
	count (case when Salary_Range = '9000-10000' then 1 else null end) as Sal_9000_10000
from cte 
group by Subscription_Plan
```
**List each genre and the subscription plan for each one?**
```sql
select Genre_Preference, 
	count(case when Subscription_Plan = 'Premium' then 1 else null end) as Premium_Count,
	count(case when Subscription_Plan = 'Standard' then 1 else null end) as Standard_Count,
	count(case when Subscription_Plan = 'Basic' then 1 else null end) as Basic_Count
from netflix 
group by Genre_Preference
```
**Subscription length month and subcription plan**
```sql
select Subscription_Plan, avg(Subscription_Length_Months) as avg_sub_length_months
from netflix 
group by Subscription_Plan
```
**Which regions are most likely to accept promotions?:** We can see that Africa and Oceania are the regions who are most likely to accept promotions.
```sql
with cte as (select Region,
	count(case when Promotional_Offers_Used = '1' then 1 else null end) as Accepted_Promotion,
	count(case when Promotional_Offers_Used = '0' then 1 else null end) as Denied_Promotion
from netflix
group by Region)
select Region, ((Accepted_Promotion)*1.0/(Accepted_Promotion + Denied_Promotion)) as Promotion_acceptance_rate
from cte
order by Promotion_acceptance_rate	desc
```
**What are the average watch hours per genre?:** We see that Drama and Action are the two genres that have the higher average watch time hours
```sql
select Genre_Preference, avg(Daily_Watch_Time_Hours) as Avg_Watch_Time
from netflix 
group by Genre_Preference
order by avg(Daily_Watch_Time_Hours) desc
```
**What is the churn rate by age group?:** We can see that the older the customer the higher the churn rate, this is intersting as we can now think of new creative ways of increasing retention amoung older customers
```sql
SELECT 
    CASE 
        WHEN Age >= 18 AND Age <= 25 THEN '18-25' 
        WHEN Age > 25 AND Age <= 30 THEN '25-30' 
        WHEN Age > 30 AND Age <= 40 THEN '30-40' 
        WHEN Age > 40 AND Age <= 50 THEN '40-50' 
        WHEN Age > 50 AND Age <= 60 THEN '50-60' 
        WHEN Age > 60 AND Age <= 70 THEN '60-71' 
        ELSE NULL 
    END AS Age_Group,
    COUNT(CASE WHEN churn_status_yes_no = '1' THEN 1 END) * 1.0 /
    COUNT(*) AS churn_rate -- Divide by total count for churn rate
FROM netflix 
GROUP BY 
    CASE 
        WHEN Age >= 18 AND Age <= 25 THEN '18-25' 
        WHEN Age > 25 AND Age <= 30 THEN '25-30' 
        WHEN Age > 30 AND Age <= 40 THEN '30-40' 
        WHEN Age > 40 AND Age <= 50 THEN '40-50' 
        WHEN Age > 50 AND Age <= 60 THEN '50-60' 
        WHEN Age > 60 AND Age <= 70 THEN '60-71' 
        ELSE NULL 
    END
ORDER BY Age_Group
```
**Lets take a look at the payment history based on region:** We see that Oceania, South America and Europe are regions that have the largest number of delayed payments we can maybe implement some strategies to make sure clients are paying on time from the region.
```sql
with cte as (select Region,
	count(case when Payment_History_On_Time_Delayed = 'On-Time' then 1 else null end) as On_Time_Payments,
	count(case when Payment_History_On_Time_Delayed = 'Delayed' then 1 else null end) as Delayed_Payments
from netflix 
group by Region)
select Region, Delayed_Payments*1.0 / (On_Time_Payments + Delayed_Payments) as Delayed_Payment_Rate
from cte
order by Delayed_Payment_Rate desc 
```
**Lets take a look at the churn rate based on Device:** I am thinking some user experience based on device may cause customers to want to cancel their membership. Lets take a look. We can see that Tablets and SmartTv devices have the highest churn time maybe we can focus on making these devices more appealing, user friendly or accessable so that customers dont delete their memberships.
```sql
with cte as (select Device_Used_Most_Often,
	count(case when Churn_status_yes_no = '1' then 1 else null end) as Yes_Churn,
	count(case when Churn_status_yes_no = '0' then 1 else null end) as Not_Churnned
from netflix 
group by Device_Used_Most_Often)
select Device_used_most_often, Yes_Churn *1.0 / (Yes_Churn + Not_Churnned) as Churn_Rate
from cte 
order by Churn_Rate
```
