
### Project Danny's Diner

## Introduction

This project is a case study created by Danny Ma found [here](https://8weeksqlchallenge.com/case-study-2/). Full credit to Danny Ma and the Data With Danny virtual data apprenticeship program for providing the dataset and case study questions to be answered. All analysis was done using PostgreSQL in PG Admin4. This case study tested many important SQL concepts including Aggregation, CTE's, Joins, Window Functions, Case Statements, String Manipulation and more!


## Creating The Tables

Below is how the tables are defined, this was taken from [here](https://8weeksqlchallenge.com/case-study-2/).

~~~~sql
CREATE SCHEMA pizza_runner;
SET search_path = pizza_runner;

DROP TABLE IF EXISTS runners;
CREATE TABLE runners (
  "runner_id" INTEGER,
  "registration_date" DATE
);
INSERT INTO runners
  ("runner_id", "registration_date")
VALUES
  (1, '2021-01-01'),
  (2, '2021-01-03'),
  (3, '2021-01-08'),
  (4, '2021-01-15');


DROP TABLE IF EXISTS customer_orders;
CREATE TABLE customer_orders (
  "order_id" INTEGER,
  "customer_id" INTEGER,
  "pizza_id" INTEGER,
  "exclusions" VARCHAR(4),
  "extras" VARCHAR(4),
  "order_time" TIMESTAMP
);

INSERT INTO customer_orders
  ("order_id", "customer_id", "pizza_id", "exclusions", "extras", "order_time")
VALUES
  ('1', '101', '1', '', '', '2020-01-01 18:05:02'),
  ('2', '101', '1', '', '', '2020-01-01 19:00:52'),
  ('3', '102', '1', '', '', '2020-01-02 23:51:23'),
  ('3', '102', '2', '', NULL, '2020-01-02 23:51:23'),
  ('4', '103', '1', '4', '', '2020-01-04 13:23:46'),
  ('4', '103', '1', '4', '', '2020-01-04 13:23:46'),
  ('4', '103', '2', '4', '', '2020-01-04 13:23:46'),
  ('5', '104', '1', 'null', '1', '2020-01-08 21:00:29'),
  ('6', '101', '2', 'null', 'null', '2020-01-08 21:03:13'),
  ('7', '105', '2', 'null', '1', '2020-01-08 21:20:29'),
  ('8', '102', '1', 'null', 'null', '2020-01-09 23:54:33'),
  ('9', '103', '1', '4', '1, 5', '2020-01-10 11:22:59'),
  ('10', '104', '1', 'null', 'null', '2020-01-11 18:34:49'),
  ('10', '104', '1', '2, 6', '1, 4', '2020-01-11 18:34:49');


DROP TABLE IF EXISTS runner_orders;
CREATE TABLE runner_orders (
  "order_id" INTEGER,
  "runner_id" INTEGER,
  "pickup_time" VARCHAR(19),
  "distance" VARCHAR(7),
  "duration" VARCHAR(10),
  "cancellation" VARCHAR(23)
);

INSERT INTO runner_orders
  ("order_id", "runner_id", "pickup_time", "distance", "duration", "cancellation")
VALUES
  ('1', '1', '2020-01-01 18:15:34', '20km', '32 minutes', ''),
  ('2', '1', '2020-01-01 19:10:54', '20km', '27 minutes', ''),
  ('3', '1', '2020-01-03 00:12:37', '13.4km', '20 mins', NULL),
  ('4', '2', '2020-01-04 13:53:03', '23.4', '40', NULL),
  ('5', '3', '2020-01-08 21:10:57', '10', '15', NULL),
  ('6', '3', 'null', 'null', 'null', 'Restaurant Cancellation'),
  ('7', '2', '2020-01-08 21:30:45', '25km', '25mins', 'null'),
  ('8', '2', '2020-01-10 00:15:02', '23.4 km', '15 minute', 'null'),
  ('9', '2', 'null', 'null', 'null', 'Customer Cancellation'),
  ('10', '1', '2020-01-11 18:50:20', '10km', '10minutes', 'null');


DROP TABLE IF EXISTS pizza_names;
CREATE TABLE pizza_names (
  "pizza_id" INTEGER,
  "pizza_name" TEXT
);
INSERT INTO pizza_names
  ("pizza_id", "pizza_name")
VALUES
  (1, 'Meatlovers'),
  (2, 'Vegetarian');


DROP TABLE IF EXISTS pizza_recipes;
CREATE TABLE pizza_recipes (
  "pizza_id" INTEGER,
  "toppings" TEXT
);
INSERT INTO pizza_recipes
  ("pizza_id", "toppings")
VALUES
  (1, '1, 2, 3, 4, 5, 6, 8, 10'),
  (2, '4, 6, 7, 9, 11, 12');


DROP TABLE IF EXISTS pizza_toppings;
CREATE TABLE pizza_toppings (
  "topping_id" INTEGER,
  "topping_name" TEXT
);
INSERT INTO pizza_toppings
  ("topping_id", "topping_name")
VALUES
  (1, 'Bacon'),
  (2, 'BBQ Sauce'),
  (3, 'Beef'),
  (4, 'Cheese'),
  (5, 'Chicken'),
  (6, 'Mushrooms'),
  (7, 'Onions'),
  (8, 'Pepperoni'),
  (9, 'Peppers'),
  (10, 'Salami'),
  (11, 'Tomatoes'),
  (12, 'Tomato Sauce');
~~~~ 

### Table 1: Runners

This table shows the date that each runner registered


| runner_id | registration_date |
|-----------|-------------------|
| 1         | 2021-01-01        |
| 2         | 2021-01-03        |
| 3         | 2021-01-08        |
| 4         | 2021-01-15        |

### Table 2: Customer_Orders

This table shows the pizzas ordered for each customer, each row represents one pizza. order_id describes the order number, customer_id is the id of the customer, pizza_id relates to one of the two pizza types, exclusions and extras describe any topping additions or exclusions to existing pizzas and finally order_time shows the time the order was placed



| order_id | customer_id | pizza_id | exclusions | extras | order_time          |
|----------|-------------|----------|------------|--------|---------------------|
| 1        | 101         | 1        |            |        | 2020-01-01 18:05:02 |
| 2        | 101         | 1        |            |        | 2020-01-01 19:00:52 |
| 3        | 102         | 1        |            |        | 2020-01-02 23:51:23 |
| 3        | 102         | 2        |            | NULL   | 2020-01-02 23:51:23 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 2        | 4          |        | 2020-01-04 13:23:46 |
| 5        | 104         | 1        | null       | 1      | 2020-01-08 21:00:29 |
| 6        | 101         | 2        | null       | null   | 2020-01-08 21:03:13 |
| 7        | 105         | 2        | null       | 1      | 2020-01-08 21:20:29 |
| 8        | 102         | 1        | null       | null   | 2020-01-09 23:54:33 |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10 11:22:59 |
| 10       | 104         | 1        | null       | null   | 2020-01-11 18:34:49 |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11 18:34:49 |



### Table 3: Runner_orders

This table maps an assigned runner or employee to an order represented by runner_id. pickup_time describes when the order was picked up, distance describes how far the runner needed to drive to the customer, duration is how long the drive was and cancellation refers to if an order was canceled.


| order_id | runner_id | pickup_time         | distance | duration   | cancellation            |
|----------|-----------|---------------------|----------|------------|-------------------------|
| 1        | 1         | 2020-01-01 18:15:34 | 20km     | 32 minutes |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20km     | 27 minutes |                         |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4km   | 20 mins    | NULL                    |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40         | NULL                    |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15         | NULL                    |
| 6        | 3         | null                | null     | null       | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25km     | 25mins     | null                    |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4 km  | 15 minute  | null                    |
| 9        | 2         | null                | null     | null       | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10km     | 10minutes  | null                    |


### Table 4: Pizza_names

Table shows the two available pizzas at Danny's Diner marked with both a name and id.

| pizza_id | pizza_name | 
|----------|------------|
| 1        | Meatlovers | 
| 2        | Vegetarian | 


### Table 5: Pizza_recipes

This table shows the ingredients for each pizza represented in a comma seperated list where each value corresponds to a specific ingredient found in the pizza_toppings table

| pizza_id | toppings                |                                                   
|----------|-------------------------|
| 1        | 1, 2, 3, 4, 5, 6, 8, 10 | 
| 2        | 4, 6, 7, 9, 11, 12      |            

### Table 6: Pizza_toppings

This assigns a topping name to each topping id


| topping_id | topping_name |
|------------|--------------|
| 1          | Bacon        |
| 2          | BBQ Sauce    |
| 3          | Beef         |
| 4          | Cheese       |
| 5          | Chicken      |
| 6          | Mushrooms    |
| 7          | Onions       |
| 8          | Pepperoni    |
| 9          | Peppers      |
| 10         | Salami       |
| 11         | Tomatoes     |
| 12         | Tomato Sauce |



## Background

Below is a data model of all the relevant tables: 
![entity relationship](https://github.com/Tyler-Wilson-326/Project_1/assets/141818698/de510fe8-1977-4cdc-8db5-84bc9cacc4af)

Here we can see we have 6 tables total which can be related to each other.

This case study is split into 4 groups of questions which are as follows: Pizza Metrics, Runner/Customer Experience,  Ingredient Optimization, and Pricing/Ratings. But before we get to answering questions we need to start cleaning up the data.


## Data Cleaning 
The current tables as is need to be cleaned first before we can do any analysis on them, if you would like to see what the original tables look like copy the code  from the "Creating Tables" section. We will be adjusting 3 tables: the customer table, runner table and pizza names tables.

### Customer Order Table


~~~~sql

CREATE TABLE customer_orders_n as
WITH split_values AS (
  SELECT
    order_id,
    customer_id,
	pizza_id,
	exclusions,
    CASE
      WHEN exclusions IS NULL OR exclusions = 'null' OR TRIM(exclusions) = '' THEN NULL
      WHEN exclusions LIKE '%,%' THEN split_part(exclusions, ',', 1)
	  ELSE exclusions
    END AS exclusions_1,
    CASE
      WHEN exclusions IS NULL OR exclusions = 'null' OR TRIM(exclusions) = '' THEN NULL
	  WHEN exclusions LIKE '%,%' THEN TRIM(substring(exclusions FROM position(',' IN exclusions) + 1))
      ELSE NULL
    END AS exclusions_2,
	extras,
	    CASE
      WHEN extras IS NULL OR extras = 'null' OR TRIM(extras) = '' THEN NULL
      WHEN extras LIKE '%,%' THEN  split_part(extras, ',', 1)
	  ELSE extras
    END AS extras_1,
    CASE
      WHEN extras IS NULL OR extras = 'null' OR TRIM(extras) = '' THEN NULL
	  WHEN extras LIKE '%,%' THEN TRIM(substring(extras FROM position(',' IN extras) + 1))
      ELSE NULL
    END AS extras_2,
	order_time
	
    
  FROM
    customer_orders
	ORDER BY order_id ASC
)
SELECT
*
FROM
  split_values;
~~~~
~~~~sql
ALTER  TABLE customer_orders_n

ALTER COLUMN exclusions_1 TYPE INT

USING exclusions_1::integer
~~~~
~~~~sql
ALTER  TABLE customer_orders_n

ALTER COLUMN exclusions_2 TYPE INT

USING exclusions_2::integer
~~~~
~~~~sql
ALTER  TABLE customer_orders_n

ALTER COLUMN extras_1 TYPE INT

USING extras_1::integer
~~~~
~~~~sql
ALTER  TABLE customer_orders_n

ALTER COLUMN extras_2 TYPE INT

USING extras_2::integer
~~~~

~~~~sql
UPDATE customer_orders_n

set exclusions = NULL

where (exclusions = 'null') or (exclusions = '')
~~~~


~~~~sql
UPDATE customer_orders_n

set extras = NULL

where (extras = 'null') or (extras = '')
~~~~



Here we can see some issues in the original customer table, we have NULL, null and " " values which I want to assign to just NULL which is done above. We then have some exclusions/extras that have 2 toppings listed separated by a comma, I want to extract these and put them in their own columns indicated by a comma delimiter. Since we have a max of two exclusions/extras this method should be fine. We then assign new value types more appropriate than before, also assigning null values to the original exclusions and extras columns to help for later. By doing all of this we get a new cleaned table below:

| order_id | customer_id | pizza_id | exclusions | exclusions_1 | exclusions_2 | extras | extras_1 | extras_2 | order_time          |
|----------|-------------|----------|------------|--------------|--------------|--------|----------|----------|---------------------|
| 1        | 101         | 1        | NULL       | NULL         | NULL         | NULL   | NULL     | NULL     | 2020-01-01 18:05:02 |
| 2        | 101         | 1        | NULL       | NULL         | NULL         | NULL   | NULL     | NULL     | 2020-01-01 19:00:52 |
| 3        | 102         | 1        | NULL       | NULL         | NULL         | NULL   | NULL     | NULL     | 2020-01-02 23:51:23 |
| 3        | 102         | 2        | NULL       | NULL         | NULL         | NULL   | NULL     | NULL     | 2020-01-02 23:51:23 |
| 4        | 103         | 1        | 4          | 4            | NULL         | NULL   | NULL     | NULL     | 2020-01-04 13:23:46 |
| 4        | 103         | 1        | 4          | 4            | NULL         | NULL   | NULL     | NULL     | 2020-01-04 13:23:46 |
| 4        | 103         | 2        | 4          | 4            | NULL         | NULL   | NULL     | NULL     | 2020-01-04 13:23:46 |
| 5        | 104         | 1        | NULL       | NULL         | NULL         | 1      | 1        | NULL     | 2020-01-08 21:00:29 |
| 6        | 101         | 2        | NULL       | NULL         | NULL         | NULL   | NULL     | NULL     | 2020-01-08 21:03:13 |
| 7        | 105         | 2        | NULL       | NULL         | NULL         | 1      | 1        | NULL     | 2020-01-08 21:20:29 |
| 8        | 102         | 1        | NULL       | NULL         | NULL         | NULL   | NULL     | NULL     | 2020-01-09 23:54:33 |
| 9        | 103         | 1        | 4          | 4            | NULL         | 1, 5   | 1        | 5        | 2020-01-10 11:22:59 |
| 10       | 104         | 1        | 2, 6       | 2            | 6            | 1, 4   | 1        | 4        | 2020-01-11 18:34:49 |
| 10       | 104         | 1        | NULL       | NULL         | NULL         | NULL   | NULL     | NULL     | 2020-01-11 18:34:49 |


### Runner Order Table
~~~~sql
CREATE TABLE runner_orders_n as
WITH runners_fixed AS (
SELECT *, 
CASE WHEN distance = 'null' THEN NULL
	WHEN distance LIKE '%km%' THEN TRIM(split_part(distance, 'k', 1))
	ELSE distance
	END AS distance_km,

CASE WHEN duration = 'null' THEN NULL
	 WHEN duration LIKE '%mins%' OR  duration LIKE '%minute%' OR  duration LIKE '%minutes%' THEN TRIM(split_part(duration, 'm', 1))
	 ELSE duration
	 END AS duration_mins,
	
CASE WHEN  cancellation IS NULL OR cancellation = 'null' OR TRIM(cancellation) = '' THEN NULL
	ELSE cancellation
	END AS cancellation_fixed,
	
	CASE WHEN  pickup_time IS NULL OR pickup_time = 'null' OR TRIM(pickup_time) = '' THEN NULL
	ELSE pickup_time
	END AS pickup_time_fixed

	FROM runner_orders
	ORDER BY order_id ASC
)	
select order_id, runner_id, pickup_time_fixed, distance_km,duration_mins,cancellation_fixed from runners_fixed
~~~~
~~~~sql
ALTER  TABLE runner_orders_n

ALTER COLUMN pickup_time TYPE TIMESTAMP  

USING pickup_time::TIMESTAMP without TIME ZONE

~~~~
~~~~sql
ALTER  TABLE runner_orders_n

ALTER COLUMN  duration_mins TYPE int

USING duration_mins::int
~~~~
~~~~sql
ALTER  TABLE runner_orders_n

ALTER COLUMN  distance_km TYPE numeric

USING distance_km::numeric 
~~~~

Here we need to also update the original runners_orders table. We have some duration and distance columns that have some inconsistent formatting, we use a split part to only include the numbers for both duration and distance. I am assuming here that distance if not specified is in km and duration is in minutes. We also remove any blanks, "null” and NULL to just be NULL values. Finally, we ensure if an order is not canceled we have NULL there. After this we make sure to update the columns to more appropriate data types and we get the result below.

| order_id | runner_id | pickup_time         | distance_km | duration_mins | cancellation_fixed      |
|----------|-----------|---------------------|-------------|---------------|-------------------------|
| 1        | 1         | 2020-01-01 18:15:34 | 20          | 32            | NULL                    |
| 2        | 1         | 2020-01-01 19:10:54 | 20          | 27            | NULL                    |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4        | 20            | NULL                    |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4        | 40            | NULL                    |
| 5        | 3         | 2020-01-08 21:10:57 | 10          | 15            | NULL                    |
| 6        | 3         | NULL                | NULL        | NULL          | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25          | 25            | NULL                    |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4        | 15            | NULL                    |
| 9        | 2         | NULL                | NULL        | NULL          | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10          | 10            | NULL                    |


### Pizza_names

~~~~sql
ALTER TABLE pizza_names
ADD list_of_ingredients VARCHAR
~~~~
~~~~sql
UPDATE  pizza_names

SET list_of_ingredients = case when pizza_id = 1 then 'Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami'
else 'Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce' end
~~~~

| pizza_id | pizza_name | list_of_ingredients                                                   |
|----------|------------|-----------------------------------------------------------------------|
| 1        | Meatlovers | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 2        | Vegetarian | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce            |

Finally we update the pizza_names table to add a comma seperated list ordered alphabetically of the list of ingredients for each pizza type, this will make it much easier to answers questions in part c.


Now with all of data cleaned up we are ready to start analyzing the data and answer some questions.



## Questions  


### Part A:  Pizza Metrics
**1. How many pizzas were ordered?**

~~~~sql
SELECT COUNT(*) AS num_pizas FROM customer_orders_n
~~~~

| num_pizas |
|---------------|
| 14          |

This is a simple count aggregate function to count all pizzas, each row entry indicates one pizza in the customer_orders_n table.


**2. How many unique customer orders were made?**
~~~~sql
SELECT COUNT(DISTINCT order_id) FROM customer_orders_n
~~~~

| unique_pizzas |
|-------|
| 10    |

Very simple distinct count aggregate here.

**3. How many successful orders were delivered by each runner?**
~~~~sql
SELECT runner_id,COUNT(order_id) AS num_orders FROM runner_orders_n

WHERE cancellation_fixed IS null

GROUP BY runner_id
~~~~
| runner_id | num_orders |
|-----------|------------|
| 1         | 4          |
| 2         | 3          |
| 3         | 1          |

Simple group by and count making sure to filter out where there were any cancellations.

**4. How many of each type of pizza was delivered?**
~~~~sql
SELECT pizza_name, COUNT(pizza_names) AS num_pizza FROM runner_orders_n JOIN customer_orders_n ON runner_orders_n.order_id = customer_orders_n.order_id

JOIN pizza_names ON customer_orders_n.pizza_id = pizza_names.pizza_id

WHERE cancellation_fixed IS null

GROUP BY pizza_name

ORDER BY num_pizza DESC
~~~~

| pizza_name | num_pizza |
|------------|-----------|
| Meatlovers | 9         |
| Vegetarian | 3         |

Simple join, count aggregate and group by pizza_name


**5. How many Vegetarian and Meatlovers were ordered by each customer?**


~~~~sql
 SELECT customer_id, pizza_name, COUNT(pizza_names) AS num_pizza 
 
 FROM customer_orders_n  JOIN pizza_names ON customer_orders_n.pizza_id = pizza_names.pizza_id
 
 GROUP BY customer_id,pizza_name
 
 ORDER BY customer_id ASC
~~~~


| customer_id | pizza_name | num_pizza |
|-------------|------------|-----------|
| 101         | Meatlovers | 2         |
| 101         | Vegetarian | 1         |
| 102         | Meatlovers | 2         |
| 102         | Vegetarian | 1         |
| 103         | Meatlovers | 3         |
| 103         | Vegetarian | 1         |
| 104         | Meatlovers | 3         |
| 105         | Vegetarian | 1         |

Almost identical to last question, instead including customer_id

**6. What was the maximum number of pizzas delivered in a single order?**
~~~~sql
WITH cte AS (SELECT customer_orders_n.order_id, COUNT(*) AS num_pizzas FROM customer_orders_n
 
JOIN runner_orders_n ON customer_orders_n.order_id= runner_orders_n.order_id

WHERE cancellation_fixed IS null
			GROUP BY customer_orders_n.order_id
			 
			)

SELECT MAX(num_pizzas) AS max_pizzas FROM cte

~~~~
| max_pizzas |
|------------|
| 3          |


Here we use a CTE to count the number of pizzas grouped by order id and then select the max value from that.
Another better solution would be to use a window function like below, since the first solution wont account for any ties

~~~~sql
WITH cte AS (SELECT customer_orders_n.order_id, COUNT(*) AS num_pizzas, DENSE_RANK() OVER (ORDER BY COUNT(*) DESC) AS RANK FROM customer_orders_n
 
JOIN runner_orders_n ON customer_orders_n.order_id= runner_orders_n.order_id

WHERE cancellation_fixed IS null
			GROUP BY customer_orders_n.order_id)

SELECT num_pizzas FROM cte

WHERE RANK = 1
~~~~
**7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**
~~~~sql
WITH cte AS (SELECT * , 
			CASE WHEN exclusions_1 IS null AND exclusions_2 IS null AND extras_1 IS null AND extras_2 IS null  
			
			THEN 1 ELSE 0 END AS no_changes, 
			 
			CASE WHEN exclusions IS NOT null OR extras IS NOT null THEN 1 ELSE 0 END AS pizza_changes 
			 
			FROM customer_orders_n
			JOIN runner_orders_n ON customer_orders_n.order_id = runner_orders_n.order_id
			WHERE cancellation_fixed IS null
			)

SELECT customer_id, SUM(no_changes) as total_pizzas_with_no_changes,SUM(pizza_changes) as total_pizzas_with_changes  FROM cte

GROUP BY customer_id
ORDER BY customer_id
~~~~
| customer_id | total_pizzas_with_no_changes | total_pizzas_with_changes |
|-------------|------------------------------|---------------------------|
| 101         | 2                            | 0                         |
| 102         | 3                            | 0                         |
| 103         | 0                            | 3                         |
| 104         | 1                            | 2                         |
| 105         | 0                            | 1                         |

Here we do two case statements one for a count if there are any changes and another for if there are no changes. We then sum the values up and essentially count the number of each type of pizza grouped by the customer_id. We also make sure to filter cases where pizzas were not delivered.

**8. How many pizzas were delivered that had both exclusions and extras?**
~~~~sql
SELECT COUNT(*) AS num_pizzas FROM customer_orders_n JOIN runner_orders_n ON customer_orders_n.order_id = runner_orders_n.order_id 

WHERE (cancellation_fixed IS null) AND (exclusions_1 IS NOT null OR exclusions_2 IS NOT null) AND (extras_1 IS NOT null OR extras_2 IS NOT null)

~~~~

| num_pizzas |
|------------|
| 1          |

here we use a count and filter based on if there was both a an exclusion and an extra.


**9. What was the total volume of pizzas ordered for each hour of the day?**
~~~~sql
SELECT to_char(order_time,'HH24') AS HOUR,COUNT(*) AS num_pizzas FROM customer_orders_n 
GROUP BY to_char(order_time,'HH24')

ORDER BY HOUR ASC
~~~~
| hour | num_pizzas |
|------|------------|
| 11   | 1          |
| 13   | 3          |
| 18   | 3          |
| 19   | 1          |
| 21   | 3          |
| 23   | 3          |

Here we use PostgreSQL's to_char to extract the hour in 24 hours and then count the amount of rows falling under each hour grouping by the hour.


**10. What was the volume of orders for each day of the week?**
~~~~sql
SELECT to_char(order_time,'Day') AS DAY,COUNT (DISTINCT order_id) AS num_pizzas FROM customer_orders_n 
GROUP BY to_char(order_time,'Day')

ORDER BY num_pizzas DESC
~~~~

| day       | num_pizzas |
|-----------|------------|
| Wednesday | 5          |
| Saturday  | 2          |
| Thursday  | 2          |
| Friday    | 1          |

Here it is very similar to last question except we use day of the week and a distinct count of the order_id since it asks for total orders not pizzas and there can be entries with the same order_id.


### Part B:  Runner/Customer Experience
**1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)**
~~~~sql

SELECT COUNT(runner_id), EXTRACT(WEEK FROM registration_date + INTERVAL '3 days') AS week_num FROM runners

GROUP BY 2

ORDER BY week_num 

~~~~


| count | week_num |
|-------|----------|
| 2     | 1        |
| 1     | 2        |
| 1     | 3        |


So here it would initially seem like an easy extract however at dates close to the start of January they can be misidentified as week 51/52, this is because ISO 8601 defines the first week of a year to contain January 4th, because of this we need to add an interval of 3 days so that 2021-01-01 is labeled correctly as week 1.

**2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**
~~~~sql

WITH cte AS (SELECT runner_orders_n.order_id,runner_id, (AVG(pickup_time::TIME)::TIME - AVG(order_time::TIME)::TIME)::TIME  AS avg_time

FROM runner_orders_n JOIN customer_orders_n ON runner_orders_n.order_id = customer_orders_n.order_id
WHERE cancellation_fixed IS null
GROUP BY runner_orders_n.order_id,runner_id

ORDER BY runner_orders_n.order_id ASC)

SELECT runner_id, EXTRACT(MINUTE FROM(AVG(avg_time)::TIME)) AS avg_time FROM cte

GROUP BY runner_id

~~~~
| runner_id | avg_time_minutes |
|-----------|------------------|
| 1         | 14               |
| 2         | 20               |
| 3         | 10               |

Here we use a cte to find the average pickup time and avg ordertime grouping by order_id and runner_id this ensures that if we have multiple entries for one order, we get one time value. PostgreSQL is a bit finicky with aggregate operations on time values, converting automatically to intervals which is why i typecasted to TIME many times. We then extract the minute from the avg_time and get the average time in minutes.


**3. Is there any relationship between the number of pizzas and how long the order takes to prepare?**
~~~~SQL
SELECT runner_orders_n.order_id, AVG(pickup_time - order_time) AS avg_time,COUNT(*) AS num_pizzas 

FROM runner_orders_n JOIN customer_orders_n ON runner_orders_n.order_id = customer_orders_n.order_id

GROUP BY runner_orders_n.order_id

HAVING AVG(pickup_time - order_time) IS NOT null

ORDER BY num_pizzas desc
~~~~
| order_id | avg_time | num_pizzas |
|----------|----------|------------|
| 4        | 00:29:17 | 3          |
| 10       | 00:15:31 | 2          |
| 3        | 00:21:14 | 2          |
| 7        | 00:10:16 | 1          |
| 1        | 00:10:32 | 1          |
| 8        | 00:20:29 | 1          |
| 5        | 00:10:28 | 1          |
| 2        | 00:10:02 | 1          |


Here we can see a general trend where as num_pizzas increases so does the avg time it takes to prepare. There is one outlier here that we can see at order id 10 where it took much less then the other order id of 3 at 15 minutes instead of 21 minutes. However, we can see all the 1 pizza orders at around 10 minutes consistently. There is not enough data to fully confirm this trend but logically and based on what we have there seems to be a trend.


**4. What was the average distance travelled for each customer?**
~~~~SQL
SELECT customer_id, round(AVG(distance_km),1) AS avg_distance_km
FROM customer_orders_n JOIN runner_orders_n ON customer_orders_n.order_id = runner_orders_n.order_id
WHERE cancellation_fixed IS  null
GROUP BY customer_id
ORDER BY customer_id ASC
~~~~
| customer_id | avg_distance_km |
|-------------|-----------------|
| 101         | 20.0            |
| 102         | 16.7            |
| 103         | 23.4            |
| 104         | 10.0            |
| 105         | 25.0            |

Here we do a simple join and average grouping by customer_id ensuring to remove any cancellations.

**5. What was the difference between the longest and shortest delivery times for all orders?**
~~~~sql
SELECT  MAX(duration_mins) - MIN(duration_mins) AS delivery_difference FROM runner_orders_n 
~~~~

| delivery_difference |
|---------------------|
| 30                  |

This was a very simple max-min calculation


**6. What was the average speed for each runner for each delivery and do you notice any trend for these values?**
~~~~sql
SELECT runner_id,round(AVG(distance_km),1) as avg_distance_km ,round(AVG(duration_mins),1) AS avg_time_mins

FROM runner_orders_n

GROUP BY runner_id

ORDER BY runner_id ASC
~~~~

| runner_id | avg_distance_km | avg_time_mins |
|-------------|-------------------|-----------------|
| 1           | 15.9            | 22.3          |
| 2           | 23.9            | 26.7          |
| 3           | 10.0            | 15.0          |

Yes, there is quite a noticeable trend here, as the distance traveled increases so does the actual time to drive to the location which is quite expected logically.

**7. What is the successful delivery percentage for each runner?**
~~~~sql

WITH cte AS (SELECT *, CASE WHEN cancellation_fixed IS null THEN 1.0 ELSE 0.0 END AS VALUE FROM runner_orders_n)


SELECT runner_id, round((SUM(VALUE)/(COUNT(*))) * 100.0,1) AS delivery_percentage FROM cte

GROUP BY runner_id

ORDER BY delivery_percentage DESC

~~~~

| runner_id | delivery_percentage |
|-----------|---------------------|
| 1         | 100.0               |
| 2         | 75.0                |
| 3         | 50.0                |

Here we used a case statement to count only values that had no cancelations and then divided that by a count of the entire set.



### Part C:  Ingredient Optimization
**1. What are the standard ingredients for each pizza?**
~~~~sql
WITH cte AS (SELECT  *, TRIM(UNNEST(string_to_array(toppings, ','))) AS topping 

FROM pizza_recipes JOIN pizza_names ON pizza_recipes.pizza_id = pizza_names.pizza_id)

SELECT pizza_name,topping_name FROM cte JOIN pizza_toppings
 
ON pizza_toppings.topping_id = cte.topping::int

ORDER BY pizza_name
~~~~

| pizza_name | topping_name |
|------------|--------------|
| Meatlovers | BBQ Sauce    |
| Meatlovers | Pepperoni    |
| Meatlovers | Cheese       |
| Meatlovers | Salami       |
| Meatlovers | Chicken      |
| Meatlovers | Bacon        |
| Meatlovers | Mushrooms    |
| Meatlovers | Beef         |
| Vegetarian | Tomato Sauce |
| Vegetarian | Cheese       |
| Vegetarian | Mushrooms    |
| Vegetarian | Onions       |
| Vegetarian | Peppers      |
| Vegetarian | Tomatoes     |

For this question i used unnest and string_to_array on the topping’s column field using a comma as a delimiter to collapse the comma separated topping list into a multi row list matching each topping to each pizza name. This is going to make it possible to answer future questions while answering this one

**2. What was the most commonly added extra?**
~~~~sql
WITH cte AS (SELECT TRIM(regexp_split_to_table(extras, ',')) AS toppings  FROM customer_orders_n)
SELECT topping_name, COUNT(*) AS topping_count FROM cte
JOIN pizza_toppings ON pizza_toppings.topping_id = cte.toppings::int
GROUP BY topping_name
ORDER BY topping_count DESC

~~~~
| topping_name | topping_count |
|--------------|---------------|
| Bacon        | 4             |
| Chicken      | 1             |
| Cheese       | 1             |

Here we use a regular expression to collapse each topping into its own row separated  by commas, we then count and group by each topping name, here we can see bacon has the most.


**3. What was the most common exclusion?**
~~~~sql
WITH cte AS (SELECT TRIM(regexp_split_to_table(exclusions, ',')) AS toppings  FROM customer_orders_n)
SELECT topping_name, COUNT(*) AS topping_count FROM cte
JOIN pizza_toppings ON pizza_toppings.topping_id = cte.toppings::int
GROUP BY topping_name

ORDER BY topping_count DESC

~~~~

| topping_name | topping_count |
|--------------|---------------|
| Cheese       | 4             |
| Mushrooms    | 1             |
| BBQ Sauce    | 1             |

Very Similar to last question, just replacing extras with exclusions.

**4. Generate an order item for each record in the customers_orders table in the format of one of the following:
  Meat Lovers
  Meat Lovers - Exclude Beef
  Meat Lovers - Extra Bacon
  Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers**
~~~~sql

WITH cte AS (SELECT order_id,customer_id,pizza_name, exclusions,exclusions_1_name.topping_name 
			 AS exclusions_1_name, exclusions_2_name.topping_name AS exclusions_2_name, extras,

extras_1_name.topping_name AS extras_1_name, extras_2_name.topping_name AS extras_2_name, order_time

FROM  customer_orders_n JOIN pizza_names ON pizza_names.pizza_id = customer_orders_n.pizza_id

LEFT JOIN pizza_toppings exclusions_1_name ON exclusions_1_name.topping_id = customer_orders_n.exclusions_1

LEFT JOIN pizza_toppings exclusions_2_name ON exclusions_2_name.topping_id = customer_orders_n.exclusions_2

LEFT JOIN pizza_toppings extras_1_name ON extras_1_name.topping_id = customer_orders_n.extras_1

LEFT JOIN pizza_toppings extras_2_name ON extras_2_name.topping_id = customer_orders_n.extras_2

ORDER BY order_id ASC)

SELECT order_id, CASE WHEN exclusions IS null AND extras IS null THEN pizza_name

WHEN exclusions_1_name IS NOT null AND exclusions_2_name IS null and extras IS null THEN concat(pizza_name, ' ','-',' ','Exclude',' ', exclusions_1_name )

WHEN exclusions_1_name IS NOT null AND exclusions_2_name IS not null and extras IS null THEN concat(pizza_name, ' ','-',' ','Exclude',' ', exclusions_1_name, ',', ' ',exclusions_2_name )

WHEN extras_1_name IS NOT null AND extras_2_name IS null AND exclusions IS null THEN concat(pizza_name, ' ','-',' ','Extra',' ', extras_1_name )

WHEN extras_1_name IS NOT null AND extras_2_name IS NOT null AND exclusions IS null THEN concat(pizza_name, ' ','-',' ','Extra',' ', extras_1_name, ',', ' ',extras_2_name )

WHEN exclusions_1_name IS NOT null AND extras_1_name IS NOT null AND exclusions_2_name IS null AND extras_2_name IS null THEN concat(pizza_name, ' ','-',' ','Exclude',' ', exclusions_1_name, '-', ' ', 'Extra', ' ', extras_1_name )

WHEN exclusions_1_name IS NOT null AND  exclusions_2_name IS NOT null AND extras_1_name IS NOT null AND extras_2_name IS null  THEN concat(pizza_name, ' ','-',' ','Exclude',' ', exclusions_1_name, ',', ' ',exclusions_2_name, ' ', '-', ' ', 'Extra', ' ', extras_1_name )

WHEN exclusions_1_name IS NOT null AND  exclusions_2_name IS null AND extras_1_name IS NOT null AND extras_2_name IS NOT null  THEN concat(pizza_name, ' ','-',' ','Exclude',' ', exclusions_1_name, ' ', '-', ' ', 'Extra', ' ', extras_1_name, ',', ' ', extras_2_name )

ELSE  concat(pizza_name, ' ','-',' ','Exclude',' ', exclusions_1_name, ',', ' ',exclusions_2_name, ' ', '-', ' ', 'Extra', ' ', extras_1_name, ',', ' ', extras_2_name) END AS order_item FROM cte

~~~~

| order_id | order_item                                                      |
|----------|-----------------------------------------------------------------|
| 1        | Meatlovers                                                      |
| 2        | Meatlovers                                                      |
| 3        | Vegetarian                                                      |
| 3        | Meatlovers                                                      |
| 4        | Meatlovers - Exclude Cheese                                     |
| 4        | Meatlovers - Exclude Cheese                                     |
| 4        | Vegetarian - Exclude Cheese                                     |
| 5        | Meatlovers - Extra Bacon                                        |
| 6        | Vegetarian                                                      |
| 7        | Vegetarian - Extra Bacon                                        |
| 8        | Meatlovers                                                      |
| 9        | Meatlovers - Exclude Cheese - Extra Bacon, Chicken              |
| 10       | Meatlovers                                                      |
| 10       | Meatlovers - Exclude BBQ Sauce, Mushrooms - Extra Bacon, Cheese |

Here is when things started ramping up in difficulty. I assumed this question asked for an order_item in the exact same format they provided in the question so items without any additions or exclusions should just contain the pizza name, any exclusions or extras should also be formatted the way they specified.
The way I set this up I have 2 extras columns and 2 exclusions columns since we have a max number of exclusions and extras of 2 each. I then join 4 times so that I can match each exclusion/extra id with its corresponding topping name. I then go into a detailed case statement which looks at every possible combination since we have 2 values max of either exclusion/extras, this isn’t too bad, any more would make this much longer and possibly another solution would need to be implemented, it is still possible using case statements but would be quite tedious to code. based on these conditions I concat the appropriate formatting, ending up with what we have above.




**5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
   For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"**

~~~~sql
WITH cte AS (SELECT order_id,customer_id,pizza_name,list_of_ingredients, exclusions,exclusions_1_name.topping_name AS exclusions_1_name, exclusions_2_name.topping_name AS exclusions_2_name, extras,

extras_1_name.topping_name AS extras_1_name, extras_2_name.topping_name AS extras_2_name, order_time

FROM  customer_orders_n JOIN pizza_names ON pizza_names.pizza_id = customer_orders_n.pizza_id

LEFT JOIN pizza_toppings exclusions_1_name ON exclusions_1_name.topping_id = customer_orders_n.exclusions_1

LEFT JOIN pizza_toppings exclusions_2_name ON exclusions_2_name.topping_id = customer_orders_n.exclusions_2

LEFT JOIN pizza_toppings extras_1_name ON extras_1_name.topping_id = customer_orders_n.extras_1

LEFT JOIN pizza_toppings extras_2_name ON extras_2_name.topping_id = customer_orders_n.extras_2

ORDER BY order_id ASC), cte2 AS(

SELECT *, CASE WHEN exclusions IS  null AND extras IS null and extras IS null THEN concat(pizza_name,':',list_of_ingredients)

WHEN extras IS  null AND exclusions_1_name IS NOT null AND exclusions_2_name IS null THEN concat(pizza_name,':',REPLACE(list_of_ingredients, CONCAT(exclusions_1_name, ','), ''))

WHEN extras IS  null AND exclusions_1_name IS NOT null AND exclusions_2_name IS NOT null THEN concat(pizza_name,':',REPLACE(list_of_ingredients, CONCAT(exclusions_1_name, ','), ''),REPLACE(REPLACE(list_of_ingredients, CONCAT(exclusions_1_name, ','), ''), CONCAT(exclusions_2_name, ','), ''))

WHEN extras_1_name IS NOT null AND extras_2_name IS NOT null AND exclusions_1_name IS NOT null AND exclusions_2_name IS null 

THEN concat(pizza_name,':',replace(list_of_ingredients, CONCAT(exclusions_1_name, ','), ''))

WHEN extras_1_name IS NOT null AND extras_2_name IS NOT null AND exclusions_1_name IS NOT null AND exclusions_2_name IS NOT null THEN concat(pizza_name,':',REPLACE(REPLACE(list_of_ingredients, CONCAT(exclusions_1_name, ','), ''), CONCAT(exclusions_2_name, ','), ''))

ELSE concat(pizza_name,':',list_of_ingredients)	

END AS exclusions_removed FROM cte),

cte3 AS (SELECT *, CASE WHEN extras_1_name IS NOT null AND exclusions_removed ILIKE '%' || extras_1_name || '%'  THEN REPLACE(exclusions_removed,extras_1_name,concat('2x',extras_1_name))
		 
		 WHEN extras_1_name IS NOT null AND exclusions_removed NOT ILIKE '%' || extras_1_name || '%'  THEN REPLACE(exclusions_removed,extras_1_name,concat(extras_1_name))
		 
		 ELSE exclusions_removed
		 END AS extras_1_added

FROM cte2)

SELECT order_id, CASE WHEN extras_2_name IS NOT null AND exclusions_removed ILIKE '%' || extras_2_name || '%'  THEN REPLACE(extras_1_added,extras_2_name,concat('2x',extras_2_name)) 

		 WHEN extras_2_name IS NOT null AND exclusions_removed NOT ILIKE '%' || extras_2_name || '%'  THEN REPLACE(extras_1_added,extras_2_name,concat(extras_2_name)) 

ELSE extras_1_added 

END AS ingregdients_all

FROM cte3
~~~~

| order_id | ingregdients_all                                                                   |
|----------|------------------------------------------------------------------------------------|
| 1        | Meatlovers:Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 2        | Meatlovers:Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 3        | Vegetarian:Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce              |
| 3        | Meatlovers:Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 4        | Meatlovers:Bacon, BBQ Sauce, Beef,  Chicken, Mushrooms, Pepperoni, Salami          |
| 4        | Meatlovers:Bacon, BBQ Sauce, Beef,  Chicken, Mushrooms, Pepperoni, Salami          |
| 4        | Vegetarian: Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce                     |
| 5        | Meatlovers:2xBacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 6        | Vegetarian:Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce              |
| 7        | Vegetarian:Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce              |
| 8        | Meatlovers:Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 9        | Meatlovers:2xBacon, BBQ Sauce, Beef,  2xChicken, Mushrooms, Pepperoni, Salami      |
| 10       | Meatlovers:Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 10       | Meatlovers:2xBacon,  Beef, 2xCheese, Chicken,  Pepperoni, Salami                   |

This again just like the previous question was very difficult to construct, we start with the same process as the previous question by doing 4 joins to get the toppings names to each exclusion and extra column we also join with the pizza table to get the pizza names as well. We then construct 3 CTE's where each CTE takes care of a specific task.

**CTE 1 :** Join all the toppings_id's with their corresponding names to each extra and exclusions column, also joins the pizza names together

**CTE 2 :** Removes all the exclusions for every possible combination between extras 1, extras 2, exclusions 1 and exclusions 2 columns. This is done using replace and concat clauses.

**CTE 3 :** adds any extras in extras 1 to the list of ingredients This is done using replace and concat clauses along with ILIKE

**Main Query :** adds any extras in extras 2 to the list of ingredients This is done using replace and concat clauses along with ILIKE

Now there is probably a more elegant solution to this, but this code achieves the desired result and i like splitting into multiple CTE's so i and others can follow my logic like "steps"


**6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?**
~~~~sql
WITH cte AS (SELECT order_id,customer_id,pizza_name,list_of_ingredients, exclusions,exclusions_1_name.topping_name AS exclusions_1_name, 
			 exclusions_2_name.topping_name AS exclusions_2_name, extras,

extras_1_name.topping_name AS extras_1_name, extras_2_name.topping_name AS extras_2_name, order_time

FROM  customer_orders_n JOIN pizza_names ON pizza_names.pizza_id = customer_orders_n.pizza_id

LEFT JOIN pizza_toppings exclusions_1_name ON exclusions_1_name.topping_id = customer_orders_n.exclusions_1

LEFT JOIN pizza_toppings exclusions_2_name ON exclusions_2_name.topping_id = customer_orders_n.exclusions_2

LEFT JOIN pizza_toppings extras_1_name ON extras_1_name.topping_id = customer_orders_n.extras_1

LEFT JOIN pizza_toppings extras_2_name ON extras_2_name.topping_id = customer_orders_n.extras_2

ORDER BY order_id ASC), cte2 AS(

SELECT *, CASE WHEN exclusions IS  null AND extras IS null AND extras IS null THEN concat(list_of_ingredients)

WHEN extras IS  null AND exclusions_1_name IS NOT null AND exclusions_2_name IS null THEN concat(REPLACE(list_of_ingredients, CONCAT(exclusions_1_name, ','), ''))

WHEN extras IS  null AND exclusions_1_name IS NOT null AND exclusions_2_name IS NOT null THEN concat(REPLACE(list_of_ingredients, CONCAT(exclusions_1_name, ','), ''),REPLACE(REPLACE(list_of_ingredients, CONCAT(exclusions_1_name, ','), ''), CONCAT(exclusions_2_name, ','), ''))

WHEN extras_1_name IS NOT null AND extras_2_name IS NOT null AND exclusions_1_name IS NOT null AND exclusions_2_name IS null 

THEN concat(replace(list_of_ingredients, CONCAT(exclusions_1_name, ','), ''))

WHEN extras_1_name IS NOT null and extras_2_name IS NOT null AND exclusions_1_name IS NOT null AND exclusions_2_name IS NOT null THEN concat(REPLACE(REPLACE(list_of_ingredients, CONCAT(exclusions_1_name, ','), ''), CONCAT(exclusions_2_name, ','), ''))

ELSE concat(list_of_ingredients)	

END AS exclusions_removed FROM cte),
cte3 AS (SELECT *, CASE WHEN extras_1_name IS NOT null AND extras_2_name IS null THEN concat(exclusions_removed,',',extras_1_name)
		 
		 WHEN extras_1_name IS NOT null AND extras_2_name IS NOT null THEN concat(exclusions_removed,',',extras_1_name,',',extras_2_name)
		 
		 ELSE exclusions_removed
		 END AS updated_list FROM cte2),cte4 AS(
		 
		 SELECT *, regexp_split_to_table(updated_list, ',') AS toppings
			 
			 FROM cte3
		 )
SELECT TRIM(toppings) as topping, COUNT(*) AS topping_count FROM cte4
GROUP BY TRIM(toppings)
ORDER BY topping_count DESC
~~~~
| Topping        | topping_count |
|--------------|---------------|
| Bacon        | 14            |
| Mushrooms    | 13            |
| Chicken      | 11            |
| Cheese       | 11            |
| Pepperoni    | 10            |
| Salami       | 10            |
| Beef         | 10            |
| BBQ Sauce    | 9             |
| Tomato Sauce | 4             |
| Onions       | 4             |
| Tomatoes     | 4             |
| Peppers      | 4             |

Here again we have 3 CTE's and one main query.

**CTE 1:** Just like the last two questions this CTE goes and joins all the topping id names to each extra and exclusion column while also joining the pizza name

**CTE 2:** Same as previous question it removes all toppings that are excluded for every possible combination

**CTE 3:** Slightly modified from the last questions since we are not concerned with adding 2x in front of relevant ingredients we don’t have to use ILIKE, but this just adds the extras

These updated ingredient lists also do not include the pizza name like the previous questions.

**Main Query:**  Uses a regular expression to collapse each topping name into its own row, we then count these and group them by topping name to get a count of all toppings used.



### Part D: Pricing and Ratings
**1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?**
~~~~sql

WITH cte AS(SELECT CASE WHEN pizza_id = 1 THEN 12 ELSE 10 END AS COST FROM customer_orders_n

JOIN runner_orders_n ON customer_orders_n.order_id = runner_orders_n.order_id

WHERE cancellation_fixed IS null

ORDER BY customer_orders_n.order_id ASC)

SELECT SUM(COST) AS money_made FROM cte
~~~~
| money_made |
|------------|
| 138        |

This is a simple case assignment for each type of pizza assigning the appropriate cost to each pizza type then summing it up making sure to remove any non delivered pizzas, assuming no cost is associated with this.

**2. What if there was an additional $1 charge for any pizza extras?
   Add cheese is $1 extra**

~~~~sql
WITH cte AS(SELECT CASE WHEN pizza_id = 1 THEN 12 ELSE 10 END AS initial_pizza_cost, 
			
CASE WHEN extras_1 IS NOT null AND extras_2 IS null THEN 1 
			
WHEN extras_1 IS NOT null AND extras_2 IS NOT null THEN 2

ELSE 0 END AS extras_cost

FROM customer_orders_n

JOIN runner_orders_n ON customer_orders_n.order_id = runner_orders_n.order_id

WHERE cancellation_fixed IS null

ORDER BY customer_orders_n.order_id ASC)

SELECT SUM(initial_pizza_cost+extras_cost) AS money_made FROM cte
~~~~

| money_made |
|------------|
| 142        |

Very simple add on from the last question just making sure to add extras costs in the case statement.



**3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and      insert your own data for ratings for each successful customer order between 1 to 5.**
~~~~SQL
CREATE TABLE customer_satisfaction (
  "order_id" INTEGER,
  "runner_id" INTEGER,
  "customer_rating" INTEGER)
  
  
  
 INSERT INTO customer_satisfaction ("order_id","runner_id","customer_rating")
  
 VALUES
 
 (1,1,3),
 (2,1,4),
 (3,1,5),
 (4,2,2),
 (5,3,5),
 (6,3,0),
 (7,2,3),
 (8,2,4),
 (9,2,0),
 (10,1,1)
~~~~

| order_id | runner_id | customer_rating |
|----------|-----------|-----------------|
| 1        | 1         | 3               |
| 2        | 1         | 4               |
| 3        | 1         | 5               |
| 4        | 2         | 2               |
| 5        | 3         | 5               |
| 6        | 3         | 0               |
| 7        | 2         | 3               |
| 8        | 2         | 4               |
| 9        | 2         | 0               |
| 10       | 1         | 1               |

Here I didn't add a new schema as I didn’t think it was necessary for one table, so I just created a table and randomly assigned values to fill up the table.




**4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
     customer_id
     order_id
     runner_id
     rating
     order_time
     pickup_time
     Time between order and pickup
     Delivery duration
     Average speed
     Total number of pizzas**

~~~~sql
WITH cte AS (SELECT customer_id,runner_orders_n.order_id,runner_orders_n.runner_id,customer_rating,order_time::TIME,pickup_time::TIME,distance_km,duration_mins, 
			 (pickup_time-order_time)::TIME AS delivery_duration, 
			 ROW_NUMBER() OVER (PARTITION BY runner_orders_n.order_id ORDER BY runner_orders_n.order_id ASC) AS pizza_num

FROM customer_satisfaction 

JOIN runner_orders_n ON customer_satisfaction.order_id =runner_orders_n.order_id

JOIN customer_orders_n ON customer_orders_n.order_id = runner_orders_n.order_id)


SELECT customer_id,order_id,runner_id,round(AVG(customer_rating),1) AS average_rating, AVG(order_time)::TIME AS order_time,

AVG(pickup_time)::TIME AS pickup_time, (AVG(pickup_time)::TIME -  AVG(order_time)::TIME)::TIME  AS time_to_receive_pizza,

round(AVG(distance_km),1) AS avg_distance_km ,round(AVG(duration_mins),1) AS avg_time_mins, MAX(pizza_num) AS num_pizzas

FROM cte


GROUP BY runner_id, order_id, customer_id

HAVING round(AVG(customer_rating),1) != 0

ORDER BY order_id ASC
~~~~

| customer_id | order_id | runner_id | average_rating | order_time | pickup_time | time_to_receive_pizza | avg_distance_km | avg_time_mins | num_pizzas |
|-------------|----------|-----------|----------------|------------|-------------|-----------------------|-----------------|---------------|------------|
| 101         | 1        | 1         | 3.0            | 18:05:02   | 18:15:34    | 00:10:32              | 20.0            | 32.0          | 1          |
| 101         | 2        | 1         | 4.0            | 19:00:52   | 19:10:54    | 00:10:02              | 20.0            | 27.0          | 1          |
| 102         | 3        | 1         | 5.0            | 23:51:23   | 00:12:37    | 00:21:14              | 13.4            | 20.0          | 2          |
| 103         | 4        | 2         | 2.0            | 13:23:46   | 13:53:03    | 00:29:17              | 23.4            | 40.0          | 3          |
| 104         | 5        | 3         | 5.0            | 21:00:29   | 21:10:57    | 00:10:28              | 10.0            | 15.0          | 1          |
| 105         | 7        | 2         | 3.0            | 21:20:29   | 21:30:45    | 00:10:16              | 25.0            | 25.0          | 1          |
| 102         | 8        | 2         | 4.0            | 23:54:33   | 00:15:02    | 00:20:29              | 23.4            | 15.0          | 1          |
| 104         | 10       | 1         | 1.0            | 18:34:49   | 18:50:20    | 00:15:31              | 10.0            | 10.0          | 2          |


Here we just join a bunch of other fields that we have calculated in previous questions to one table. Our new additions are avg rating which is a simple average and num_pizzas which allocates that amount of pizzas per order using a row_number window function

**5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?**

~~~~sql
WITH cte AS (SELECT CASE WHEN pizza_id = 1 THEN 12.0 ELSE 10.0 END AS fixed_cost, CASE WHEN distance_km IS NOT null THEN distance_km*0.30 END AS variable_costs

FROM customer_satisfaction 

JOIN runner_orders_n ON customer_satisfaction.order_id =runner_orders_n.order_id

JOIN customer_orders_n ON customer_orders_n.order_id = runner_orders_n.order_id
WHERE distance_km IS NOT null)
			

SELECT round(SUM(fixed_cost::numeric)-SUM(variable_costs::numeric),2) AS profit FROM cte

~~~~

| profit |
|--------|
| 73.38  |

Here we take the money made in pizzas (fixed costs) - the money paid out to the runners (variable costs), we then sum that difference and we get a profit


## Conclusion

Thats all the questions, this was an amazing case study where i really could demonstrate my SQL knowledge across multiple tables. It tested many concepts like joins, aggregations, case statements, window functions, string manipulation, time data and more. If anyone spots anything that seems wrong or incorrect or just want to let me know of a more proficient way to solve the query, please don’t hesitate to reach out to me at wilsontyle123@gmail.com where i look forward to hearing from you. Once again thank you to Danny Ma for providing the data and case questions. I learned a lot from this especially string manipulation functions.


