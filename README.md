# ClipBoard-Health-Assessment
Write SQL queries to extract agent performance metrics from Zendesk’s data. 

1.  Create an SQL query to determine the average first reply time and average first resolution time by month for the past 12 months. 
2.  Create an SQL query that provides the average satisfaction score (as calculated by Zendesk) and total tickets solved by the agent by month for the last 6 months.


FOR THE FIRST QUERY_

-- This query selects the month from the created_at column and calculates the average first reply time
and average first resolution time for each month.

SELECT
EXTRACT(MONTH FROM CREATED_AT) as month, -- Extracts the month from the created_at column
and aliases it as &quot;month&quot;.
ROUND(AVG(COALESCE(nullif(reply_time_in_minutes, &#39;null&#39;)::decimal, 0))) AS
AVG_FIRST_REPLY_TIME_IN_MINUTE, -- Calculates the rounded average first reply time in minutes,
handling null values and aliasing the result.
ROUND(AVG(COALESCE(nullif(first_resolution_time_in_minutes, &#39;null&#39;)::decimal, 0))) AS
AVG_FIRST_RESOLUTION_TIME_IN_MINUTE -- Calculates the rounded average first resolution time in
minutes, handling null values and aliasing the result.
FROM

mongo_main_app.ZD_TICKETS_METRICS -- Specifies the source table.
WHERE
solved_at IS NOT NULL -- Filters for records where the ticket is solved.
GROUP BY
month; -- Groups the results by month for aggregation.




FOR THE SECOND QUERY_

-- This query selects the month from the created_at column, calculates the satisfaction score, and
counts the total tickets solved for each month.

SELECT
EXTRACT(MONTH FROM CREATED_AT) as month, -- Extracts the month from the created_at column
and aliases it as &quot;month&quot;.
ROUND(
COUNT(CASE WHEN satisfaction_rating =&#39;good&#39; THEN 1 END)::decimal / COUNT(CASE WHEN
satisfaction_rating =&#39;good&#39; or satisfaction_rating =&#39;bad&#39; THEN 1 END)::decimal,2
) * 100 AS satisfaction_score, -- Calculates the rounded satisfaction score as a percentage.
(SELECT COUNT(status) AS Total_tickets_solved FROM mongo_main_app.ZD_Tickets WHERE status =
&#39;solved&#39;) -- Subquery to count the total tickets solved.
FROM
mongo_main_app.ZD_Tickets -- Specifies the source table.
WHERE
EXTRACT(MONTH FROM CREATED_AT) IS NOT NULL -- Filters for records where the month is not null.
GROUP BY
MONTH -- Groups the results by month for aggregation.
ORDER BY

MONTH DESC -- Orders the results by month in descending order.
LIMIT
6; -- Limits the results to the top 6 months.
