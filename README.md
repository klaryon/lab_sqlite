# Lab SQL Lite

```sql
SELECT * FROM users;
SELECT * FROM sessions;
SELECT * FROM chargers;

-- LEVEL 1

-- Question 1: Number of users with sessions
SELECT COUNT(u.id) AS number_users
FROM users AS u;

-- Question 2: Number of chargers used by user with id 1
SELECT u.name, COUNT(c.id) AS number_chargers
FROM users AS u
INNER JOIN sessions AS s
INNER JOIN chargers AS c
ON s.user_id = u.id AND s.charger_id = c.id
WHERE u.id= 1;

-- LEVEL 2

-- Question 3: Number of sessions per charger type (AC/DC):
SELECT c.type, COUNT(s.id) AS number_sessions
FROM sessions AS s
INNER JOIN chargers AS c
ON s.charger_id = c.id
GROUP BY c.type;

-- Question 4: Chargers being used by more than one user
SELECT s.charger_id, s.user_id
FROM users AS u
INNER JOIN sessions AS s
INNER JOIN chargers AS c
ON s.user_id = u.id AND s.charger_id = c.id
GROUP BY s.charger_id
HAVING s.user_id > 1; 

-- Question 5: Average session time per charger
SELECT 
    s.charger_id, 
    ROUND(AVG((strftime('%s', s.end_time) - strftime('%s', s.start_time)))/60, 2) AS avg_session_hours
FROM sessions AS s
INNER JOIN chargers AS c
ON s.charger_id = c.id
GROUP BY s.charger_id;

-- LEVEL 3

-- Question 6: Full username of users that have used more than one charger in one day (NOTE: for date only consider start_time)
SELECT u.name, u.surname, strftime('%Y-%m-%d', s.start_time) AS start_date, COUNT(s.charger_id) AS num_chargers
FROM users AS u
INNER JOIN sessions AS s
ON s.user_id = u.id
GROUP BY s.user_id, start_date
HAVING COUNT(DISTINCT s.charger_id) > 1;

-- Question 7: Top 3 chargers with longer sessions
SELECT 
    s.charger_id, 
    ROUND((strftime('%s', s.end_time) - strftime('%s', s.start_time))/60, 2) AS session_hours
FROM sessions AS s
INNER JOIN chargers AS c
ON s.charger_id = c.id
GROUP BY s.charger_id
ORDER BY session_hours DESC
LIMIT 3;

-- Question 8: Average number of users per charger (per charger in general, not per charger_id specifically)
SELECT AVG(qty_users) AS avg_users
FROM (
    SELECT charger_id,
    COUNT(DISTINCT user_id) AS qty_users
    FROM sessions
    GROUP BY charger_id
);

-- Question 9: Top 3 users with more chargers being used
SELECT u.name, COUNT(s.charger_id) AS used_chargers
FROM users AS u
INNER JOIN sessions AS s
ON u.id = s.user_id
GROUP BY u.name 
ORDER BY used_chargers DESC
LIMIT 3;

-- LEVEL 4

-- Question 10: Number of users that have used only AC chargers, DC chargers or both ********************************
SELECT
COUNT(DISTINCT CASE WHEN charger_type = 'AC Only' THEN user_id END) AS users_ac_only,
COUNT(DISTINCT CASE WHEN charger_type = 'DC Only' THEN user_id END) AS users_dc_only,
COUNT(DISTINCT CASE WHEN charger_type = 'Both' THEN user_id END) AS users_both
FROM (
SELECT
user_id,
CASE
WHEN COUNT(DISTINCT c.type) = 1 AND MIN(c.type) = 'AC' THEN 'AC Only'
WHEN COUNT(DISTINCT c.type) = 1 AND MIN(c.type) = 'DC' THEN 'DC Only'
WHEN COUNT(DISTINCT c.type) = 2
AND MIN(c.type) = 'AC'
AND MAX(c.type) = 'DC' THEN 'Both'
END AS charger_type
FROM sessions s
JOIN chargers c ON s.charger_id = c.id
GROUP BY
user_id
) AS users_charger;

-- Question 11: Monthly average number of users per charger
SELECT 
    s.charger_id, 
    CAST(strftime('%m', s.start_time) AS INTEGER) AS month,
    ROUND(AVG(s.user_id), 2) AS avg_month_users_per_charger
FROM sessions AS s
GROUP BY s.charger_id, month;

-- Question 12: Top 3 users per charger (for each charger, number of sessions)
SELECT  charger_id, 
        user_id, 
        num_sessions,
        order_rank
FROM (
    SELECT charger_id, 
           user_id,
           start_time, 
           COUNT(*) AS num_sessions,
           DENSE_RANK() OVER(PARTITION BY charger_id ORDER BY COUNT(*) DESC) AS order_rank
    FROM sessions
    GROUP BY charger_id, user_id
)
WHERE order_rank <= 3
ORDER BY charger_id, order_rank;

-- LEVEL 5

-- Question 13: Top 3 users with longest sessions per month (consider the month of start_time)
SELECT 
    s.user_id, 
    ROUND((strftime('%s', s.end_time) - strftime('%s', s.start_time))/60, 2) AS session_hours,
    CAST(strftime('%m', s.start_time) AS INTEGER) AS month
FROM sessions AS s
INNER JOIN users AS c
ON s.user_id = c.id
GROUP BY s.user_id, month
ORDER BY session_hours DESC
LIMIT 3;
```
