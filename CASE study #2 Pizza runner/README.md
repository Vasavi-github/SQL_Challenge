# 🍕 Case Study #2 Pizza Runner

<img src="https://user-images.githubusercontent.com/81607668/127271856-3c0d5b4a-baab-472c-9e24-3c1e3c3359b2.png" alt="Image" width="500" height="520">

## 🛠️ Problem Statement
## Introduction
> Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”
> 
> Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so **Pizza Runner** was launched!
> 
> Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.
## Available Data
>Danny had a few years of experience as a data scientist - he was very aware that data collection was going to be critical for his business’ growth.
>He has prepared for us an entity relationship diagram of his database design but requires further assistance to clean his data and apply some basic calculations so he  can better direct his runners and optimise Pizza Runner’s operations.


---

## 📂 Dataset
Danny has shared with you 6 key datasets for this case study:

### **```runners```**
<details>
<summary>
View table
</summary>

The runners table shows the **```registration_date```** for each new runner.


|runner_id|registration_date|
|---------|-----------------|
|1        |1/1/2021         |
|2        |1/3/2021         |
|3        |1/8/2021         |
|4        |1/15/2021        |

</details>


### **```customer_orders```**

<details>
<summary>
View table
</summary>

Customer pizza orders are captured in the **```customer_orders```** table with 1 row for each individual pizza that is part of the order.

|order_id|customer_id|pizza_id|exclusions|extras|order_time        |
|--------|---------|--------|----------|------|------------------|
|1  |101      |1       |          |      |44197.75349537037 |
|2  |101      |1       |          |      |44197.79226851852 |
|3  |102      |1       |          |      |44198.9940162037  |
|3  |102      |2       |          |*null* |44198.9940162037  |
|4  |103      |1       |4         |      |44200.558171296296|
|4  |103      |1       |4         |      |44200.558171296296|
|4  |103      |2       |4         |      |44200.558171296296|
|5  |104      |1       |null      |1     |44204.87533564815 |
|6  |101      |2       |null      |null  |44204.877233796295|
|7  |105      |2       |null      |1     |44204.88922453704 |
|8  |102      |1       |null      |null  |44205.99621527778 |
|9  |103      |1       |4         |1, 5  |44206.47429398148 |
|10 |104      |1       |null      |null  |44207.77417824074 |
|10 |104      |1       |2, 6      |1, 4  |44207.77417824074 |

</details>

### **```runner_orders```**

<details>
<summary>
View table
</summary>

After each orders are received through the system - they are assigned to a runner - however not all orders are fully completed and can be cancelled by the restaurant or the customer.

The **```pickup_time```** is the timestamp at which the runner arrives at the Pizza Runner headquarters to pick up the freshly cooked pizzas. 

The **```distance```** and **```duration```** fields are related to how far and long the runner had to travel to deliver the order to the respective customer.



|order_id|runner_id|pickup_time|distance  |duration|cancellation      |
|--------|---------|-----------|----------|--------|------------------|
|1       |1        |1/1/2021 18:15|20km      |32 minutes|                  |
|2       |1        |1/1/2021 19:10|20km      |27 minutes|                  |
|3       |1        |1/3/2021 0:12|13.4km    |20 mins |*null*             |
|4       |2        |1/4/2021 13:53|23.4      |40      |*null*             |
|5       |3        |1/8/2021 21:10|10        |15      |*null*             |
|6       |3        |null       |null      |null    |Restaurant Cancellation|
|7       |2        |1/8/2020 21:30|25km      |25mins  |null              |
|8       |2        |1/10/2020 0:15|23.4 km   |15 minute|null              |
|9       |2        |null       |null      |null    |Customer Cancellation|
|10      |1        |1/11/2020 18:50|10km      |10minutes|null              |

</details>

### **```pizza_names```**

<details>
<summary>
View table
</summary>

|pizza_id|pizza_name|
|--------|----------|
|1       |Meat Lovers|
|2       |Vegetarian|

</details>

### **```pizza_recipes```**

<details>
<summary>
View table
</summary>

Each **```pizza_id```** has a standard set of **```toppings```** which are used as part of the pizza recipe.


|pizza_id|toppings |
|--------|---------|
|1       |1, 2, 3, 4, 5, 6, 8, 10| 
|2       |4, 6, 7, 9, 11, 12| 

</details>

### **```pizza_toppings```**

<details>
<summary>
View table
</summary>

This table contains all of the **```topping_name```** values with their corresponding **```topping_id```** value.


|topping_id|topping_name|
|----------|------------|
|1         |Bacon       | 
|2         |BBQ Sauce   | 
|3         |Beef        |  
|4         |Cheese      |  
|5         |Chicken     |     
|6         |Mushrooms   |  
|7         |Onions      |     
|8         |Pepperoni   | 
|9         |Peppers     |   
|10        |Salami      | 
|11        |Tomatoes    | 
|12        |Tomato Sauce|

</details>
## ♻️ Data Cleaning

### **Data Issues**

Data issues in the existing schema include:

* **```customer_orders``` table**
  - ```null``` values entered as text
  - using both ```NaN``` and ```null``` values
* **```runner_orders``` table**
  - ```null``` values entered as text
  - using both ```NaN``` and ```null``` values
  - units manually entered in ```distance``` and ```duration``` columns

### **Data Cleaning**

**```customer_orders```**
- Converting ```null``` and ```NaN``` values into blanks ```''``` in ```exclusions``` and ```extras```
  - Blanks indicate that the customer requested no extras/exclusions for the pizza, whereas ```null``` values would be ambiguous.
- Saving the transformations in a temporary table
  - We want to avoid permanently changing the raw data via ```UPDATE``` commands if possible.

**```runner_orders```**

- Converting ```'null'``` text values into null values for ```pickup_time```, ```distance``` and ```duration```
- Extracting only numbers and decimal spaces for the distance and duration columns
  - Use regular expressions and ```NULLIF``` to convert non-numeric entries to null values
- Converting blanks, ```'null'``` and ```NaN``` into null values for cancellation
- Saving the transformations in a temporary table

## Result of Data cleaning
Our course of action to clean the table:
- Create a temporary table with all the columns
- Remove null values in `exlusions` and `extras` columns and replace with blank space ' '.

````sql
CREATE TEMP TABLE customer_orders_temp AS
SELECT 
    order_id,
    customer_id,
    pizza_id,
    CASE 
        WHEN exclusions IS NULL OR exclusions LIKE 'null' THEN ''
        ELSE exclusions
        END AS exclusions,
    CASE 
        WHEN extras = 'null' OR extras IS NULL OR extras ='NaN' THEN ''
        ELSE extras END AS extras,
     order_time
FROM pizza_runner.customer_orders;
`````
This is how the clean `customers_orders_temp` table looks like and we will use this table to run all our queries.
---------------------
|order_id|customer_id|pizza_id|exclusions|extras|order_time         |
|--------|-----------|--------|----------|------|-------------------|
|1       |101        |1       |          |      |2020-01-01T18:05:02|
|2       |101        |1       |          |      |2020-01-01T19:00:52|
|3       |102        |1       |          |      |2020-01-02T12:51:23|
|3       |102        |2       |          |      |2020-01-02T12:51:23|
|4       |103        |1       |4         |      |2020-01-04T13:23:46|
|4       |103        |1       |4         |      |2020-01-04T13:23:46|
|4       |103        |2       |4         |      |2020-01-04T13:23:46|
|5       |104        |1       |          |1     |2020-01-08T21:00:29|
|6       |101        |2       |          |      |2020-01-08T21:03:13|
|7       |105        |2       |          |1     |2020-01-08T21:20:29|
|8       |102        |1       |          |      |2020-01-09T23:54:33|
|9       |103        |1       |4         |1, 5  |2020-01-10T11:22:59|
|10      |104        |1       |          |      |2020-01-11T18:34:49|
|10      |104        |1       |2, 6      |1, 4  |2020-01-11T18:34:49|
----------------

### 🔨 Table: runner_orders
````sql
DROP TABLE IF EXISTS temp_runner_orders;
CREATE TEMP TABLE temp_runner_orders AS
SELECT
  order_id,
  runner_id,
  CAST(CASE WHEN pickup_time = 'null' THEN NULL 
       ELSE pickup_time END AS TIMESTAMP) + INTERVAL '1 year' AS pickup_time,
  CAST (
    CASE
	  WHEN distance = 'null' THEN NULL
	  WHEN distance LIKE '%km' THEN TRIM(distance, 'km') ELSE distance
    END AS FLOAT),
  CAST(
	CASE
	  WHEN duration = 'null' THEN NULL
	  WHEN duration LIKE '%mins' THEN TRIM(duration, 'mins')
	  WHEN duration LIKE '%minute' THEN TRIM(duration, 'minute')
	  WHEN duration LIKE '%minutes' THEN TRIM(duration, 'minutes') ELSE duration
    END AS FLOAT),
  CASE WHEN cancellation = 'null' or cancellation = '' THEN NULL ELSE cancellation END
FROM pizza_runner.runner_orders;
```````
--------
``````````sql
SELECT * FROM temp_runner_orders
``````````
----------------
| order_id | runner_id | pickup_time         | distance | duration | cancellation            |
|----------|-----------|---------------------|----------|----------|-------------------------|
| 1        | 1         | 2020-01-01 18:15:34 | 20       | 32       |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20       | 27       |                         |
| 3        | 1         | 2020-01-02 00:12:37 | 13.4     | 20       |                         |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40       |                         |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15       |                         |
| 6        | 3         |                     |          |          | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25       | 25       |                         |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4     | 15       |                         |
| 9        | 2         |                     |          |          | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10       | 10       |                         |
--------
# 🍕 Case Study #2 - Pizza Runner

## 🍝 A. Pizza Metrics -Solution

### 1. How many pizzas were ordered?

````sql
SELECT COUNT(*) AS pizza_order_count
FROM customer_orders_temp;
````
![image](https://user-images.githubusercontent.com/107137479/201217117-610abecf-0953-4aad-95e4-238a93f578dd.png)
- 14 total pizzas ordered.
### 2. How many unique customer orders were made?
`````sql
SELECT 
COUNT(DISTINCT order_id)unique_orders_count
FROM customer_orders_temp;
```````
![1](https://user-images.githubusercontent.com/107137479/201216127-06276d15-bbb7-4880-9e44-9ef813435293.jpg)
- There are 10 unique customer orders
### 3. How many successful orders were delivered by each runner?
````sql
SELECT
runner_id,
COUNT(order_id)suceessful_orders
FROM temp_runner_orders
WHERE distance IS NOT NULL
GROUP BY runner_id
ORDER BY runner_id;
``````
![image](https://user-images.githubusercontent.com/107137479/201219315-4405ba4f-b009-45f7-98dd-e15bc483229a.png)
-  runner_id 1, 2, 3 delivered 4,3,1 successful orders respectively.
###  4. How many of each type of pizza was delivered?
``````sql
SELECT 
     p.pizza_name,
     COUNT(c.pizza_id)count_of_pizza
FROM customer_orders_temp c
LEFT JOIN pizza_runner.pizza_names p
ON c.pizza_id=p.pizza_id
LEFT JOIN temp_runner_orders
ON c.order_id=temp_runner_orders.order_id
WHERE distance IS NOT NULL
GROUP BY c.pizza_id,pizza_name;
````````
![image](https://user-images.githubusercontent.com/107137479/201222250-dea54124-b5d7-458d-88a3-3020a58ab7b0.png)
-  9 Meatlovers and 3 Vegetarian pizaas were delivered.
###  5. How many Vegetarian and Meatlovers were ordered by each customer?
```````sql
SELECT 
    customer_id,
SUM(CASE WHEN p.pizza_name='Meatlovers' THEN 1 ELSE 0 END)AS meatlovers,
SUM(CASE WHEN p.pizza_name='Vegetarian' THEN 1 ELSE 0 END)AS vegetarian 
FROM customer_orders_temp c
LEFT JOIN pizza_runner.pizza_names p
ON c.pizza_id=p.pizza_id
GROUP BY c.customer_id
ORDER BY customer_id
``````````
![image](https://user-images.githubusercontent.com/107137479/201225146-7dba1fa8-f815-43b4-856f-ced895790a4f.png)
-  Customer 101 ordered 2 Meatlovers pizzas and 1 Vegetarian pizza.
-  Customer 102 ordered 2 Meatlovers pizzas and 2 Vegetarian pizzas.
-  Customer 103 ordered 3 Meatlovers pizzas and 1 Vegetarian pizza.
-  Customer 104 ordered 3 Meatlovers pizza.
-  Customer 105 ordered 1 Vegetarian pizza.
---------------------------------------------

















