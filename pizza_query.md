# MySQL Data Analysis for Pizza Start-Up

### Data Cleaning

#### 1. Cleaning data in  **customer_orders** table:

Original `customer_orders` table structure.

````sql
SELECT * FROM customer_orders
LIMIT 5;
````

**Results:**

order_id|customer_id|pizza_id|exclusions|extras|order_time             |
--------|-----------|--------|----------|------|-----------------------|
 1|        101|       1|          |      |2020-01-01 18:05:02.000|
2|        101|       1|          |      |2020-01-01 19:00:52.000|
3|        102|       1|          |      |2020-01-02 23:51:23.000|
3|        102|       2|          |      |2020-01-02 23:51:23.000|
4|        103|       1|4         |      |2020-01-04 13:23:46.000|

We need to address the inconsistent data types in the customer_order table before proceeding with any queries. The exclusions and extras columns have mixed data types with values such as 'null' (text), null (data type), and '' (empty).
To clean the data, we will create a temporary table that converts all types of null values to null (data type)

````
DROP TABLE IF EXISTS new_customer_orders;

CREATE TEMPORARY TABLE new_customer_orders AS (
  SELECT order_id,
    customer_id,
    pizza_id,
    CASE
      WHEN exclusions = ''
      OR exclusions LIKE 'null' THEN NULL
      ELSE exclusions
    END AS exclusions,
    CASE
      WHEN extras = ''
      OR extras LIKE 'null' THEN NULL
      ELSE extras
    END AS extras,
    order_time
  FROM customer_orders
);

SELECT *
FROM new_customer_orders
LIMIT 5;
````

**Results:**

order_id|customer_id|pizza_id|exclusions|extras|order_time             |
--------|-----------|--------|----------|------|-----------------------|
1|        101|       1|          |      |2020-01-01 18:05:02.000|
2|        101|       1|          |      |2020-01-01 19:00:52.000|
3|        102|       1|          |      |2020-01-02 23:51:23.000|
3|        102|       2|          |      |2020-01-02 23:51:23.000|
4|        103|       1|4         |      |2020-01-04 13:23:46.000|

#### 2. Cleaning data in  **runner_orders** table:

Original `runner_orders` table structure:

````
SELECT * FROM runner_orders
LIMIT 5;
````

**Results:**

order_id|runner_id|pickup_time        |distance|duration  |cancellation           |
--------|---------|-------------------|--------|----------|-----------------------|
1|        1|2020-01-01 18:15:34|20km    |32 minutes|                       |
2|        1|2020-01-01 19:10:54|20km    |27 minutes|                       |
3|        1|2020-01-03 00:12:37|13.4km  |20 mins   |                       |
4|        2|2020-01-04 13:53:03|23.4    |40        |                       |
5|        3|2020-01-08 21:10:57|10      |15        |                       |

The distance and duration columns have mixed data types with text and numerical values. To address this, we will remove the text values and convert them to numeric values. In addition, the cancellation column has 'null' (text) and 'NaN' values, which we will convert to null (data type). Lastly, we will convert the pickup_time column (currently stored as varchar) to a timestamp data type.

````
DROP TABLE IF EXISTS new_runner_orders;

CREATE TEMPORARY TABLE new_runner_orders AS (
  SELECT order_id,
    runner_id,
    CASE
      WHEN pickup_time LIKE 'null' THEN NULL
      ELSE pickup_time
    END AS pickup_time,
    -- Return null value if both arguments are equal
    -- Use regex to match only numeric values and decimal point.
    -- Convert to numeric datatype
    NULLIF(REGEXP_REPLACE(distance, '[^0-9.]', ''), '') AS distance,
    NULLIF(REGEXP_REPLACE(duration, '[^0-9.]', ''), '') AS duration,
    CASE
      WHEN cancellation LIKE 'null'
      OR cancellation LIKE 'NaN'
      OR cancellation LIKE '' THEN NULL
      ELSE cancellation
    END AS cancellation
  FROM runner_orders
);

ALTER TABLE new_runner_orders
MODIFY COLUMN duration INT,
MODIFY COLUMN distance DECIMAL,
MODIFY COLUMN pickup_time TIMESTAMP;

SELECT *
FROM new_runner_orders
LIMIT 5;
````

**Results:**

| order_id | runner_id | pickup_time         | distance | duration | cancellation |
|----------|-----------|---------------------|----------|----------|--------------|
|        1 |         1 | 2020-01-01 18:15:34 | 20       | 32       | NULL         |
|        2 |         1 | 2020-01-01 19:10:54 | 20       | 27       | NULL         |
|        3 |         1 | 2020-01-03 00:12:37 | 13.4     | 20       | NULL         |
|        4 |         2 | 2020-01-04 13:53:03 | 23.4     | 40       | NULL         |
|        5 |         3 | 2020-01-08 21:10:57 | 10       | 15       | NULL         |

### Data Analysis

#### 1. Customerwise order count 

````
SELECT customer_id,
  count(*) as n_orders
FROM new_customer_orders
GROUP BY customer_id
ORDER BY n_orders DESC;
````

**Results:**

| customer_id | n_orders |
|-------------|----------|
|         103 |        4 |
|         101 |        3 |
|         102 |        3 |
|         104 |        3 |
|         105 |        1 |


#### 2. Total orders

````
SELECT count(DISTINCT order_id) AS n_orders
FROM new_customer_orders;
````

**Results:**

n_orders|
--------|
10|


#### 3. Successful orders delivered by each runner

Only orders which are not cancelled must be considered as successful orders

````
SELECT runner_id,
	count(order_id) AS n_orders
FROM new_runner_orders
WHERE cancellation IS NULL
GROUP BY runner_id
ORDER BY n_orders DESC;
````

**Results:**

runner_id|n_orders|
---------|--------|
1|       4|
2|       3|
3|       1|

#### 4. Quantity of each type of pizza that was delivered

Here, we filter out any cancelled orders.

````
SELECT p.pizza_name,
  COUNT(c.order_id) AS n_pizza_type
FROM new_customer_orders AS c
  JOIN pizza_names AS p ON p.pizza_id = c.pizza_id
  JOIN new_runner_orders AS r ON c.order_id = r.order_id
WHERE cancellation IS NULL
GROUP BY p.pizza_name
ORDER BY n_pizza_type DESC;
````

**Results:**

pizza_name|n_pizza_type|
----------|------------|
Meatlovers|           9|
Vegetarian|           3|

#### 5. Number of Vegetarian and Meatlovers pizzas ordered by each customer

````
SELECT customer_id,
	sum(
		CASE
			WHEN pizza_id = 1 THEN 1
			ELSE 0
		END
	) AS meat_lovers,
	sum(
		CASE
			WHEN pizza_id = 2 THEN 1
			ELSE 0
		END
	) AS vegetarian
FROM new_customer_orders
GROUP BY customer_id
ORDER BY customer_id;
````

**Results:**

customer_id|meat_lovers|vegetarian|
-----------|-----------|----------|
101|          2|         1|
102|          2|         1|
103|          3|         1|
104|          3|         0|
105|          0|         1|

#### 6. Maximum number of pizzas delivered in a single order

````
WITH cte_order_count AS (
	SELECT c.order_id,
		count(c.pizza_id) AS n_orders
	FROM new_customer_orders AS c
		JOIN new_runner_orders AS r ON c.order_id = r.order_id
	WHERE r.cancellation IS NULL
	GROUP BY c.order_id
)
SELECT max(n_orders) AS max_n_orders
FROM cte_order_count;
````

**Results:**

max_n_orders|
------------|
3|

#### 7. Count of delivered pizzas with at least one change and with no changes for each customer

````
SELECT c.customer_id,
	sum(
		CASE
			WHEN c.exclusions IS NOT NULL
			OR c.extras IS NOT NULL THEN 1
			ELSE 0
		END
	) AS has_changes,
	sum(
		CASE
			WHEN c.exclusions IS NULL
			AND c.extras IS NULL THEN 1
			ELSE 0
		END
	) AS no_changes
FROM new_customer_orders AS c
	JOIN new_runner_orders AS r ON c.order_id = r.order_id
WHERE r.cancellation IS NULL
GROUP BY c.customer_id
ORDER BY c.customer_id;
````

**Results:**

customer_id|has_changes|no_changes|
-----------|-----------|----------|
101|          0|         2|
102|          0|         3|
103|          3|         0|
104|          2|         1|
105|          1|         0|

#### 8. Count of pizzas that were delivered with both exclusions and extras

````
SELECT sum(
		CASE
			WHEN c.exclusions IS NOT NULL
			AND c.extras IS NOT NULL THEN 1
			ELSE 0
		END
	) AS n_pizzas
FROM new_customer_orders AS c
	JOIN new_runner_orders AS r ON c.order_id = r.order_id
WHERE r.cancellation IS NULL;
````

**Results:**

|both_changes|
|--------|
|1|

#### 9. Order volume for each hour of the day

````
SELECT EXTRACT(
    HOUR
    FROM order_time
  ) AS hour_of_day,
  COUNT(*) AS n_pizzas
FROM new_customer_orders
WHERE order_time IS NOT NULL
GROUP BY hour_of_day
ORDER BY hour_of_day;
````

**Results:**

| hour_of_day | n_pizzas |
|-------------|----------|
|          11 |        1 |
|          13 |        3 |
|          18 |        3 |
|          19 |        1 |
|          21 |        3 |
|          23 |        3 |

#### 10. Daywise order volume

````
SELECT DAYNAME(order_time) AS weekday,
  COUNT(*) AS n_pizzas
FROM new_customer_orders
WHERE order_time IS NOT NULL
GROUP BY weekday
ORDER BY n_pizzas DESC;
````

**Results:**

| weekday   | n_pizzas |
|-----------|----------|
| Wednesday |        5 |
| Saturday  |        5 |
| Thursday  |        3 |
| Friday    |        1 |

### Runner Experience

#### 1. Number of runners registering each week starting from 2021-01-01

````
WITH runner_signups AS (
  SELECT runner_id,
    registration_date,
    DATE_ADD(
      '2021-01-01',
      INTERVAL DATEDIFF(registration_date, '2021-01-01') DIV 7 WEEK
    ) AS starting_week
  FROM runners
)
SELECT starting_week,
  count(runner_id) AS n_runners
from runner_signups
GROUP BY starting_week
ORDER BY starting_week;
````

**Results:**

starting_week|n_runners|
-------------|---------|
   2021-01-01|        2|
   2021-01-08|        1|
   2021-01-15|        1|

#### 2. Average time (in minutes) it took for a runner to reach the Pizza Runner HQ and pick up the order

````
WITH runner_time AS (
  SELECT r.runner_id,
    r.order_id,
    TIMESTAMPDIFF(SECOND, c.order_time, r.pickup_time) AS runner_arrival_time
  FROM new_runner_orders AS r
    JOIN new_customer_orders AS c ON r.order_id = c.order_id
  WHERE r.pickup_time IS NOT NULL
    AND c.order_time IS NOT NULL
)
SELECT runner_id,
  ROUND(AVG(runner_arrival_time) / 60, 2) AS avg_arrival_time
FROM runner_time
GROUP BY runner_id
ORDER BY runner_id;
````

**Results:**

| runner_id | avg_arrival_time |
|-----------|------------------|
|         1 |            15.68 |
|         2 |            23.72 |
|         3 |            10.47 |

#### 3. Number of pizzas ordered vs average preparation time (in minutes) 

````
WITH cte_prep AS (
  SELECT COUNT(c.order_id) AS n_pizza,
    AVG(
      TIMESTAMPDIFF(SECOND, c.order_time, r.pickup_time)
    ) AS prep_time
  FROM new_runner_orders AS r
    JOIN new_customer_orders AS c ON r.order_id = c.order_id
  WHERE r.pickup_time IS NOT NULL
    AND c.order_time IS NOT NULL
  GROUP BY c.order_id
)
SELECT n_pizza,
  ROUND(AVG(prep_time) / 60, 2) AS avg_prep_time
FROM cte_prep
GROUP BY n_pizza
ORDER BY n_pizza;
````

**Results:**

| n_pizza | avg_prep_time |
|---------|---------------|
|       1 |         12.36 |
|       2 |         18.38 |
|       3 |         29.28 |

#### 4. Average distance traveled for each customer

````
SELECT c.customer_id,
  ROUND(AVG(r.distance), 2) AS avg_distance
FROM new_runner_orders AS r
  JOIN new_customer_orders AS c ON c.order_id = r.order_id
GROUP BY customer_id
ORDER BY customer_id;
````

**Results:**

| customer_id | avg_distance |
|-------------|--------------|
|         101 |        20.00 |
|         102 |        16.33 |
|         103 |        23.00 |
|         104 |        10.00 |
|         105 |        25.00 |

#### 5. Average distance travelled by each runner

````
SELECT runner_id,
  ROUND(AVG(distance), 2) AS avg_distance
FROM new_runner_orders
GROUP BY runner_id
ORDER BY runner_id;
````

**Results:**

| runner_id | avg_distance |
|-----------|--------------|
|         1 |        15.75 |
|         2 |        23.67 |
|         3 |        10.00 |

#### 6. Difference between the longest and shortest delivery times (in minutes)

````
SELECT
	MIN(duration) AS min_time,
	MAX(duration) AS max_time,
	MAX(duration) - MIN(duration) AS time_diff
FROM new_runner_orders;
````

**Results:**

| min_time | max_time | time_diff |
|----------|----------|-----------|
|       10 |       40 |        30 |

#### 7.  Successful delivery percentage for each runner

````
WITH cte_runner AS (
	SELECT runner_id, 
		SUM(CASE WHEN cancellation IS NOT NULL THEN 1 ELSE 0 END) AS delivered, 
		COUNT(*) AS total_orders
	FROM new_runner_orders
	GROUP BY runner_id
)
SELECT runner_id, delivered, total_orders, ROUND((delivered/total_orders)*100,2) AS percent_delivered
FROM cte_runner;
````

**Results:**

| runner_id | delivered | total_orders | percent_delivered |
|-----------|-----------|--------------|-------------------|
|         1 |         0 |            4 |              0.00 |
|         2 |         1 |            4 |             25.00 |
|         3 |         1 |            2 |             50.00 |

