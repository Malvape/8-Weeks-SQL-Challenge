# 🍕 Case Study #1: Danny's Diner 

![image](https://github.com/Malvape/8-Weeks-SQL-Challenge/assets/41355722/6c4a7781-055d-4fd4-a25b-b3a3cdfb7115)


## 📚 Index
- [Introduction](#introduction)
- [Problem Statement](#problem-statement)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Problems Solution](#problems-solution)

***

## Introduction
Because Danny had a few years of experience as a data scientist - he was very aware that data collection was going to be critical for his business’ growth.

He has prepared for us an entity relationship diagram of his database design but requires further assistance to clean his data and apply some basic calculations so he can better direct his runners and optimise Pizza Runner’s operations.

All datasets exist within the pizza_runner database schema - be sure to include this reference within your SQL scripts as you start exploring the data and answering the case study questions.

***

## Problem Statement

This case study has LOTS of questions - they are broken up by area of focus including:

Pizza Metrics
Runner and Customer Experience
Ingredient Optimisation
Pricing and Ratings
Bonus DML Challenges (DML = Data Manipulation Language)
Each of the following case study questions can be answered using a single SQL statement.

Again, there are many questions in this case study - please feel free to pick and choose which ones you’d like to try!

Before you start writing your SQL queries however - you might want to investigate the data, you may want to do something with some of those null values and data types in the customer_orders and runner_orders tables!

***

## Entity Relationship Diagram

![image](https://github.com/Malvape/8-Weeks-SQL-Challenge/assets/41355722/1faef5ef-f3dd-4a8a-8a2d-072fa40b4ebb)


***
## Problems Solution

### A. Pizza Metrics  

<details>
  <summary><b>1. How many pizzas were ordered?</b></summary>

  #### Query
  ````sql
select 
count(*) as pizzas_ordered 
from customer_orders co ;
````

  #### Result
| number_of_pizza_ordered |
| ----------------------- |
| 14                      |
            
</details>

***
<details>
  <summary><b>2. How many unique customer orders were made?</b></summary>

### Query
````sql
select 
count(distinct order_id) as unique_customers_orders
from customer_orders co ;
````
### Result
|unique_customers_orders|
|-----------------------|
|                     10|

</details>

***
<details>
  <summary><b>3. How many successful orders were delivered by each runner?</b></summary>

### Query
````sql
select
runner_id ,
count(distinct order_id) as delivered_orders 
from  runner_orders ro 
where pickup_time <> 'null'
group by runner_id ;
````
### Result
|runner_id|delivered_orders|
|---------|----------------|
|        1|               4|
|        2|               3|
|        3|               1|

</details>

***
<details>
  <summary><b>4. How many of each type of pizza was delivered?</b></summary>

### Query
````sql
select
pn.pizza_name  ,
count(co.pizza_id) as pizzas_delivered 
--count(distinct order_id) as delivered_orders 
from  runner_orders ro 
inner join customer_orders co  on ro.order_id = co.order_id 
inner join pizza_names pn on co.pizza_id =pn.pizza_id 
where pickup_time <> 'null'
group by pn.pizza_name ;
````
### Result
|pizza_name|pizzas_delivered|
|----------|----------------|
|Meatlovers|               9|
|Vegetarian|               3|

</details>

***
<details>
  <summary><b>5. How many Vegetarian and Meatlovers were ordered by each customer?</b></summary>

### Query
````sql
select
co.customer_id,
pn.pizza_name ,
 
count(co.pizza_id) as pizzas_ordered 
--count(distinct order_id) as delivered_orders 
from  customer_orders co  
inner join pizza_names pn on co.pizza_id =pn.pizza_id 

group by pn.pizza_name,
co.customer_id;
````
### Result
|customer_id|pizza_name|pizzas_ordered|
|-----------|----------|--------------|
|        101|Vegetarian|             1|
|        103|Meatlovers|             3|
|        105|Vegetarian|             1|
|        104|Meatlovers|             3|
|        101|Meatlovers|             2|
|        103|Vegetarian|             1|
|        102|Meatlovers|             2|
|        102|Vegetarian|             1|

</details>

***
<details>
  <summary><b>6. What was the maximum number of pizzas delivered in a single order?</b></summary>

### Query
````sql
select 
co.order_id,
count(co.pizza_id) as pizzas_ordered
from customer_orders co 
inner join runner_orders ro on co.order_id = ro.order_id 
where pickup_time <> 'null'
group by co.order_id 
order by count(co.pizza_id) desc
limit 1; 
````
### Result
|order_id|pizzas_ordered|
|--------|--------------|
|       4|             3|

</details>

***
<details>
  <summary><b>7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?</b></summary>

### Query
````sql

select 
co.customer_id,
SUM(case
	when (
		(exclusions is not null and exclusions <>'null' and LENGTH(exclusions) >0)
		or (extras is not null and extras <> 'null' and LENGTH(extras)>0)
		) = true 
	then 1 
	else 0 
end) as changes,
SUM(case
	when (
		(exclusions is not null and exclusions <>'null' and LENGTH(exclusions) >0)
		or (extras is not null and extras <> 'null' and LENGTH(extras)>0)
		) = true 
	then 0 
	else 1 
end) as no_changes
from customer_orders co 
inner join runner_orders ro on ro.order_id = co.order_id 
where pickup_time <> 'null'
group by co.customer_id
order by co.customer_id ASC;

````
### Result
|customer_id|changes|no_changes|
|-----------|-------|----------|
|        101|      0|         2|
|        102|      0|         3|
|        103|      3|         0|
|        104|      2|         1|
|        105|      1|         0|

</details>

***
<details>
  <summary><b>8. How many pizzas were delivered that had both exclusions and extras?</b></summary>

### Query
````sql

select 
count(pizza_id) as pizzas_delivered_with_exclusions_and_extras
from customer_orders co 
inner join runner_orders ro on ro.order_id = co.order_id 
where pickup_time <> 'null'
and (exclusions is not null and exclusions <>'null' and LENGTH(exclusions) >0)
		and (extras is not null and extras <> 'null' and LENGTH(extras)>0);

````
### Result
|pizzas_delivered_with_exclusions_and_extras|
|-------------------------------------------|
|                                          1|

</details>

***
<details>
  <summary><b>9. What was the total volume of pizzas ordered for each hour of the day?</b></summary>

### Query
````sql

select 
DATE_PART('hour',order_time) as hour,
COUNT(pizza_id) as pizzas_ordered
from customer_orders co 
group by date_part('hour', order_time); 

````
### Result
|hour|pizzas_ordered|
|----|--------------|
|18.0|             3|
|23.0|             3|
|21.0|             3|
|11.0|             1|
|19.0|             1|
|13.0|             3|

</details>

***
<details>
  <summary><b>10. What was the volume of orders for each day of the week?</b></summary>

### Query
````sql
SELECT
  TO_CHAR(order_time, 'Day') AS day_of_week,
  COUNT(*) AS number_of_pizzas
FROM 
  customer_orders
GROUP BY 
  TO_CHAR(order_time, 'Day'),
  EXTRACT('dow' FROM order_time)
ORDER BY 
  EXTRACT('dow' FROM order_time);

````
### Result
|day_of_week|number_of_pizzas|
|-----------|----------------|
|Wednesday  |               5|
|Thursday   |               3|
|Friday     |               1|
|Saturday   |               5|

</details>

***
