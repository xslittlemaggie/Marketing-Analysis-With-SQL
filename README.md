# Marketing-Analysis-with-SQL
# Title: Marketing Attribution: First-and Last-Touch Attribution
### The customer journey: 
- --> landing_page 
- --> shopping_cart 
- --> checkout
- --> purchase

The funnel analysis illustrates the theoretical customer journey from the landing_page towards the purchase of a product. Besides the funnel analysis of the number of customers from a series of steps, the primary purpose of this projects is to identify the most influential **first-touch** and **last touch**. 

**First-touch** attribution only considers the first source for each customer. This is a good way of knowing how visitors initially discover a website.

**Last-touch** attribution only considers the last source for each customer. This a a good way of knowing how visitors are drawn back to a website, especially for making a final purchase.

## Step 1. Get familar with the tables

There are 1986 users participating the survey--> 1000 users participate the quiz --> 750 users have the home-try-on --> 495 users make the purchae
```sql
/*
SELECT * FROM page_visits 
WHERE user_id = 10069;
*/
```
|page_name	|timestamp	|user_id	|utm_campaign	|utm_source|
|-----------|-----------|-----------|-----------|-----------|
|1 - landing_page	|2018-01-02 23:14:01	|10069	|ten-crazy-cool-tshirts-facts	|buzzfeed
|2 - shopping_cart	|2018-01-02 23:55:01	|10069	|ten-crazy-cool-tshirts-facts	|buzzfeed
|3 - checkout	|2018-01-04 08:12:01	|10069	|retargetting-ad	|facebook
|4 - purchase	|2018-01-04 08:13:01	|10069	|retargetting-ad	|facebook

From the example above, the user used the utm_source **buzzfeed** for **step_1: landing_page** and **step_2: shopping_chart**, and the utm_source **facebook** for **step_3: checkout** and **step_4: purchase**. 
The question is: if we want to increase sales, should we count on **buzzfeed** or **facebook**?

There are 2 ways of analyzing this:

1. First-touch attribution is **buzzfeed**
2. Last-touch attribution is **facebook**

The results can be crucial to improve a company's marketing and online presence. Most companies analyze both 
first-touch and last-touch attribution and display the results separately.


## Part 2: The campiagn contribute to the first touch
```sql
WITH first_touch AS (
SELECT user_id, MIN(timestamp) AS first_touch
FROM page_visits
GROUP BY user_id)

SELECT pv.page_name, pv.utm_campaign, COUNT(*)
FROM first_touch AS ft
JOIN page_visits AS pv
ON ft.first_touch = pv.timestamp AND ft.user_id = pv.user_id
WHERE pv.page_name LIKE '1%'
GROUP BY pv.utm_campaign
ORDER BY 1, 3 DESC
LIMIT 100;
```
### Output:
|page_name	|utm_campaign	|COUNT(*)
|----------|-------------|----------|
|1 - landing_page	|interview-with-cool-tshirts-founder	|622
|1 - landing_page	|getting-to-know-cool-tshirts	|612|
|1 - landing_page	|ten-crazy-cool-tshirts-facts	|576|
|1 - landing_page	|cool-tshirts-search	|169|

From the output, the first three campaigns contributes a lot to the first touch (91.5%). Thus, it is efficient to target the first three campaigns as the first touch marketing. 

## Part 3: The campiagn contribute to the last touch
```sql
WITH first_touch AS (
SELECT user_id, MAX(timestamp) AS first_touch
FROM page_visits
GROUP BY user_id)

SELECT pv.page_name, pv.utm_campaign, COUNT(*)
FROM first_touch AS ft
JOIN page_visits AS pv
ON ft.first_touch = pv.timestamp AND ft.user_id = pv.user_id
WHERE pv.page_name LIKE '4%'
GROUP BY pv.utm_campaign
ORDER BY 1, 3 DESC
LIMIT 100;
```
### Output:
|page_name|	utm_campaign	|COUNT(*)|
|----------|-------------|----------|
|4 - purchase	|weekly-newsletter	|114
|4 - purchase	|retargetting-ad	|112
|4 - purchase	|retargetting-campaign	|53
|4 - purchase	|paid-search	|52
|4 - purchase	|getting-to-know-cool-tshirts	|9
|4 - purchase	|ten-crazy-cool-tshirts-facts	|9
|4 - purchase	|interview-with-cool-tshirts-founder	|7
|4 - purchase	|cool-tshirts-search	|2

From the output, the combinaiton of the first two campaigns (weekly-newsletter, retargetting-ad) contributes 63% to the total of the purchase. Thus, it is efficient to target on the first two campaigns for the last touch to improve purchase rate. 

## Discussion

1. Find the funnel from the page 1 (**page landing**)through to page 4 (**purchase**), and the percentage from each page to the next one, and find the percentages with least values, which needs improvement
2. Find the source contribute to the first touch
3. Find the source contribute to the last touch
4. The contribution to the first and last touch could be different. 

## Appendix: The whole query
```sql
WITH first_touch AS (
    SELECT user_id,
        MIN(timestamp) as first_touch_at
    FROM page_visits
    GROUP BY user_id), 
    status AS (
SELECT ft.user_id,
    ft.first_touch_at,
    pv.utm_source,
		pv.utm_campaign
FROM first_touch ft
JOIN page_visits pv
    ON ft.user_id = pv.user_id
    AND ft.first_touch_at = pv.timestamp)
    
    SELECT utm_campaign, COUNT(DISTINCT user_id)
    FROM status
    GROUP BY  utm_campaign;
    
    
    WITH first_touch AS (
    SELECT user_id, 
        MAX(timestamp) as first_touch_at
    FROM page_visits
    GROUP BY user_id), 
    status AS (
SELECT ft.user_id, 
    ft.first_touch_at,
      pv.page_name,
    pv.utm_source,
		pv.utm_campaign
FROM first_touch ft
JOIN page_visits pv
    ON ft.user_id = pv.user_id
    AND ft.first_touch_at = pv.timestamp)
    
    SELECT page_name, utm_campaign, COUNT(DISTINCT user_id)
    FROM status
    GROUP BY page_name, utm_campaign;
```
