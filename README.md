# [8-Week SQL Challenge] By Danny (https://8weeksqlchallenge.com/) 

# Case Study #1 - Danny's Diner
---

## Problem Statement

> Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.
# Case Study #1 - Danny's Diner
<p align="center">
<img src="/IMG/org-1.png" width=40% height=40%>
  
---

## 🛠️ Problem Statement

> Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

 <br /> 

---

## 📂 Dataset
Danny has shared with you 3 key datasets for this case study:

### **```sales```**

<details>
<summary>
View table
</summary>

The sales table captures all ```customer_id``` level purchases with an corresponding ```order_date``` and ```product_id``` information for when and what menu items were ordered.

|customer_id|order_date|product_id|
|-----------|----------|----------|
|A          |2021-01-01|1         |
|A          |2021-01-01|2         |
|A          |2021-01-07|2         |
|A          |2021-01-10|3         |
|A          |2021-01-11|3         |
|A          |2021-01-11|3         |
|B          |2021-01-01|2         |
|B          |2021-01-02|2         |
|B          |2021-01-04|1         |
|B          |2021-01-11|1         |
|B          |2021-01-16|3         |
|B          |2021-02-01|3         |
|C          |2021-01-01|3         |
|C          |2021-01-01|3         |
|C          |2021-01-07|3         |

 </details>

### **```menu```**

<details>
<summary>
View table
</summary>

The menu table maps the ```product_id``` to the actual ```product_name``` and price of each menu item.

|product_id |product_name|price     |
|-----------|------------|----------|
|1          |sushi       |10        |
|2          |curry       |15        |
|3          |ramen       |12        |

</details>

### **```members```**

<details>
<summary>
View table
</summary>

The final members table captures the ```join_date``` when a ```customer_id``` joined the beta version of the Danny’s Diner loyalty program.

|customer_id|join_date |
|-----------|----------|
|A          |1/7/2021  |
|B          |1/9/2021  |

 </details>

 ##  Solutions

### **Q1. What is the total amount each customer spent at the restaurant?**
```sql
SELECT
    s.customer_id, 
    SUM(m.price)total_price
FROM dannys_dinner.sales s
LEFT JOIN dannys_dinner.menu m
ON s.product_id=m.product_id
GROUP BY s.customer_id
ORDER BY total_price DESC
```

| customer_id | total_price |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

---

### **Q2. How many days has each customer visited the restaurant?**
```sql
select customer_id, count(distinct order_date) as total_days 
from dannys_diner.sales
group by customer_id;
```

| customer_id | total_days |
|-------------|------------|
| A           | 4          |
| B           | 6          |
| C           | 2          |

---

### **Q3. What was the first item from the menu purchased by each customer?**
```sql
with ordered_sales as(
    select a.customer_id, a.order_date, a.product_id,b.product_name, 
    dense_rank() over(partition by a.customer_id order by a.order_date) ranking 
    from dannys_diner.sales a 
    left join dannys_diner.menu b
    on a.product_id = b.product_id
)
select distinct customer_id, product_name
from ordered_sales 
where ranking = 1;
```

| customer_id | product_name |
|-------------|--------------|
| A           | sushi        |
| A           | curry        |
| B           | curry        |
| C           | ramen        |

---

### **Q4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
```sql
select b.product_name, count(a.order_date) as item_bought 
from dannys_diner.sales a
left join dannys_diner.menu b
on a.product_id = b.product_id
group by b.product_name
order by item_bought desc
limit 1;
```

| product_name | item_bought |
|-------------|--------------|
| ramen         | 8        |

---

### **Q5. Which item was the most popular for each customer?**
```sql
with temp as(
    select a.customer_id, b.product_name, count(a.order_date) as item_bought_count,
    dense_rank() over (partition by a.customer_id order by count(a.order_date) desc) ranking 
    from dannys_diner.sales a 
    left join dannys_diner.menu b
    on a.product_id = b.product_id
    group by a.customer_id, b.product_name
)

SELECT customer_id, product_name, item_bought_count
FROM temp
WHERE ranking = 1;

```

| customer_id | product_name | item_bought_count |
|-------------|--------------|-------------------|
| A           | ramen        | 3                 |
| B           | curry        | 2                 |
| B           | sushi        | 2                 |
| B           | ramen        | 2                 |
| C           | ramen        | 3                 |

---


### **Q6. Which item was purchased first by the customer after they became a member?**
```sql
with temp_cte as(
    select a.customer_id, a.order_date,b.product_name,
    dense_rank() over(partition by a.customer_id order by a.order_date) ranking 
    from dannys_diner.sales a
    inner join dannys_diner.members c
    on a.customer_id = c.customer_id 
    left join dannys_diner.menu b 
    on a.product_id = b.product_id
    where a.order_date >= c.join_date
) 

select customer_id, order_Date, product_name
from temp_cte where ranking = 1;

```

| customer_id | order_Date | product_name |
|-------------|------------|--------------|
| A           | 2021-01-07 | curry        |
| B           | 2021-01-11 | sushi        |

---

### **Q7. Which item was purchased just before the customer became a member?**
```sql
with temp_cte as(
    select a.customer_id, a.order_Date, b.product_name,
    dense_rank() over(partition by a.customer_id order by order_date desc) ranking
    from dannys_diner.sales a
    inner join dannys_diner.members c
    on a.customer_id = c.customer_id 
    left join dannys_diner.menu b 
    on a.product_id = b.product_id
    where a.order_date < c.join_date
)

select customer_id, order_date,product_name
from temp_cte where ranking = 1;


```

| customer_id | order_date | product_name |
|-------------|------------|--------------|
| A           | 2021-01-01 | sushi        |
| A           | 2021-01-01 | curry        |
| B           | 2021-01-04 | sushi        |

---

### **Q8. What is the total items and amount spent for each member before they became a member?**
```sql
select a.customer_id, count(a.product_id) as total_items, sum(b.price) as total_amount
from dannys_diner.sales a
left join dannys_diner.menu b
on a.product_id = b.product_id
left join dannys_diner.members c
on a.customer_id = c.customer_id
where a.order_date < c.join_date
group by a.customer_id;

```

| customer_id | total_items | total_amount |
|-------------|-------------|--------------|
| A           | 2           | 25           |
| B           | 3           | 40           |

---

### **Q9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
```sql
with pointer_cte as (
    select product_id,
    case when product_id = 1 then price*20
    else price*10 end as multiplier
    from dannys_diner.menu
)
select a.customer_id,sum(b.multiplier) as points
from dannys_diner.sales a
left join pointer_cte b
on a.product_id = b.product_id
group by a.customer_id;

```

| customer_id | points |
|-------------|--------|
| A           | 860    |
| B           | 940    |
| C           | 360    |

---

### **Q10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**
```sql
with date_cte as (
    select *,
    DATE_ADD(join_date, INTERVAL 6 DAY) as valid_date
    from dannys_diner.members
)

select d.customer_id,
sum(case
when b.product_id = 1 then 20*b.price
when a.order_date between d.join_date and d.valid_date then 20*b.PRICE
else 10*b.price end) as pointer
from date_cte d
left join dannys_diner.sales a
on d.customer_id = a.customer_id
left join dannys_diner.menu b 
on a.product_id = b.product_id
where a.order_date < '2021-01-31'
group by a.customer_id;
```
| customer_id | pointer |
|-------------|---------|
| A           | 1370    |
| B           | 820     |

---

### **Bonus Questions**

> Recreate the following table output using the available data:</br>

| customer_id | order_date | product_name | price | member |
|-------------|------------|--------------|-------|--------|
| A           | 2021-01-01 | curry        | 15    | N      | 
| A	          | 2021-01-01 | sushi	      | 10	  | N      |
| A           |	2021-01-07 | curry	      | 15	  | Y      |
| A	          | 2021-01-10 | ramen        |	12    |	Y      | 
| A	          | 2021-01-11 | ramen        |	12    |	Y      |
| A	          | 2021-01-11 | ramen        |	12	  | Y      |
| B	          | 2021-01-01 | curry        |	15    |	N      |
| B	          | 2021-01-02 | curry	      | 15    |	N      |
| B	          | 2021-01-04 | sushi        |	10    |	N      |
| B	          | 2021-01-11 | sushi        |	10    |	Y      |
| B	          | 2021-01-16 | ramen	      |12	  | Y      |
| B	          | 2021-02-01 | ramen	      |12	  | Y      |
| C	          | 2021-01-01 | ramen	      |12	  | N      |
| C	          | 2021-01-01 | ramen	      |12	  | N      |
| C	          | 2021-01-07 | ramen	      |12	  | N      |


```sql

select a.customer_id, a.order_date, b.product_name,
b.price,
CASE
WHEN c.join_date > a.order_date THEN 'N'
WHEN c.join_date <= a.order_date THEN 'Y'
ELSE 'N' end as member
from dannys_diner.sales a
left join dannys_diner.menu b
on a.product_id = b.product_id
left join dannys_diner.members C
on a.customer_id = c.customer_id
order by a.customer_id, a.order_date, b.product_name;
```

---

> Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.

| customer_id | order_date | product_name | price | member | ranking | 
|-------------|------------|--------------|-------|--------|---------| 
| A	          | 2021-01-01 | curry	      | 15	  | N	   | null |
| A	          | 2021-01-01 | sushi	      | 10	  | N	   | null |
| A	          | 2021-01-07 | curry	      | 15	  | Y	   | 1 |
| A	          | 2021-01-10 | ramen	      | 12	  | Y	   | 2 |
| A	          | 2021-01-11 | ramen	      | 12    | Y	   | 3 |
| A	          | 2021-01-11 | ramen	      | 12	  | Y	   | 3 |
| B	          | 2021-01-01 | curry	      | 15	  | N	   | null |
| B	          | 2021-01-02 | curry	      | 15	  | N	   | null |
| B	          | 2021-01-04 | sushi	      | 10	  | N	   | null |
| B	          | 2021-01-11 | sushi	      | 10	  | Y	   | 1 |
| B	          | 2021-01-16 | ramen	      | 12	  | Y	   | 2 |
| B	          | 2021-02-01 | ramen	      | 12	  | Y	   | 3 |
| C	          | 2021-01-01 | ramen	      | 12	  | N	   | null |
| C	          | 2021-01-01 | ramen	      | 12	  | N	   | null |
| C	          | 2021-01-07 | ramen	      | 12	  | N	   | null |


```sql

with temp_cte as(select a.customer_id, a.order_date, b.product_name,
b.price,
CASE
WHEN c.join_date > a.order_date THEN 'N'
WHEN c.join_date <= a.order_date THEN 'Y'
ELSE 'N' end as member
from dannys_diner.sales a
left join dannys_diner.menu b
on a.product_id = b.product_id
left join dannys_diner.members C
on a.customer_id = c.customer_id
order by a.customer_id, a.order_date, b.product_name)

select *, CASE
when member = 'N' then 'null'
else rank() over(PARTITION by customer_id, member order by order_date) end as ranking
from temp_cte;

```


