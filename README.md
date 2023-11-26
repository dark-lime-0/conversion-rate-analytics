# Calculating Free-to-Paid Conversion Rate

## Project Overview

The objective is to estimate the free-to-paid conversion rate among students who engage with video content on the 365 platform. This involves working with three tables containing information about students' registration dates, engagement dates, and subscription purchase dates. The project not only focuses on calculating the conversion rate but also includes the analysis of additional key metrics to draw meaningful conclusions from the data. 


## Exploratory Data Analysis

Insights to uncover from the tables

1. What is the free-to-paid conversion rate of students who have watched a lecture on the 365 platform?
2. What is the average duration between the registration date and when a student has watched a lecture for the first time (date of first-time engagement)?
3. What is the average duration between the date of first-time engagement and when a student purchases a subscription for the first time (date of first-time purchase)?


## Data Analysis

~~~ sql
USE db_course_conversions;

SELECT 
	ROUND(
        COUNT(DISTINCT CASE WHEN first_date_purchased IS NOT NULL THEN student_id END) /
        COUNT(DISTINCT student_id) * 100,
        2
    ) AS conversion_rate,
    ROUND(
        SUM(date_diff_reg_watch)/ 
        COUNT(DISTINCT student_id),
        2
    ) AS av_reg_watch,
    ROUND(
        SUM(date_diff_watch_purch)/ 
        COUNT(date_diff_watch_purch),
        2
    ) AS av_watch_purch
    FROM(
    
    SELECT 
    i.student_id,
    i.date_registered,
    MIN(e.date_watched) AS first_date_watched,
    COALESCE(MIN(p.date_purchased), NULL) AS first_date_purchased,
    ABS(DATEDIFF(
            i.date_registered,
            MIN(e.date_watched))) AS date_diff_reg_watch,
    ABS(datediff(MIN(e.date_watched), COALESCE(MIN(p.date_purchased), NULL))) AS date_diff_watch_purch
FROM
    student_info AS i
        JOIN
    student_engagement AS e ON i.student_id = e.student_id
        LEFT JOIN
    student_purchases AS p ON i.student_id = p.student_id
    GROUP BY i.student_id, i.date_registered
    HAVING
    MIN(e.date_watched) <= COALESCE(MIN(p.date_purchased), MIN(e.date_watched))
 )   T;

  SELECT 
    MIN(student_engagement.date_watched) AS first_date_watched, 
    MIN(student_purchases.date_purchased) AS first_date_purchased
    FROM student_engagement
    LEFT JOIN student_purchases
    ON student_engagement.student_id = student_purchases.student_id
    WHERE student_engagement.student_id = 268727;
~~~

## Interpretation

- The free-to-paid conversion rate on the 365 platform stands at approximately 11%, indicating that about 11 out of every 100 students who engage with lectures go on to purchase a subscription. While this figure may seem modest initially, a deeper examination reveals various factors influencing this rate. The platform's broad audience, potentially lacking a specific focus on data science enthusiasts, and uncertainties about where to begin in the learning journey could contribute to the relatively low conversion rate. To address these challenges, an onboarding sequence was introduced in August 2023, tailoring learning paths for individual students. However, external factors such as exam periods for students or time constraints for working individuals may still affect the conversion rate. The importance of continuous feedback, identifying shortcomings, and striving for product improvement is emphasized as a crucial aspect of maintaining and enhancing user engagement.

- On average, students take between three and four days to begin watching a lecture after registering on the 365 platform. Ideally, early engagement is desired, and the lessons are easily accessible compared to other platform elements.


## Data Source

[Project File](https://learn.365datascience.com/projects/calculating-free-to-paid-conversion-rate-with-sql/?tab=description)
