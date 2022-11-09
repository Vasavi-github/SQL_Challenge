# [8-Week SQL Challenge] By Danny (https://8weeksqlchallenge.com/) 

# Case Study #1 - Danny's Diner
---

## Problem Statement

> Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.
# Case Study #1 - Danny's Diner

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
ORDER BY total_price DESC;
```

| customer_id | total_price |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

---

### **Q2. How many days has each customer visited the restaurant?**
```sql
SELECT 
    customer_id,
    COUNT(DISTINCT order_date)days_visited
FROM dannys_dinner.sales
GROUP BY customer_id;
```

| customer_id | days_visited |
|-------------|------------|
| A           | 4          |
| B           | 6          |
| C           | 2          |

---

### **Q3. What was the first item from the menu purchased by each customer?**
```sql
WITH first_purchase AS(
SELECT 
    customer_id,
    product_name,
    order_date,
    DENSE_RANK()OVER(PARTITION BY customer_id ORDER BY order_date) AS order_rank
FROM dannys_dinner.sales s
LEFT JOIN dannys_dinner.menu m
on s.product_id = m.product_id
)
SELECT DISTINCT
    customer_id,
    product_name,
FROM first_purchase
WHERE order_rank = 1
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
SELECT
    product_name,
    COUNT(product_name)AS most_purchased
FROM dannys_dinner.sales s
LEFT JOIN dannys_dinner.menu m
ON s.product_id = m.product_id
GROUP BY product_name
ORDER BY most_purchased DESC
LIMIT 1;
```

| product_name | most_purchased |
|-------------|--------------|
| ramen         | 8        |

---
The most popular item on the menu is ramen and it was purchased by customers 8 times.
<br></br>
### **Q5. Which item was the most popular for each customer?**
```sql
WITH popular_item AS(
SELECT 
    s.customer_id,
    product_name,
  
    COUNT(s.product_id) AS popular,
    DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(s.product_id)DESC) AS ranking
FROM dannys_dinner.sales s
INNER JOIN dannys_dinner.menu m
ON s.product_id = m.product_id
GROUP BY s.customer_id, m.product_name
)
SELECT 
    customer_id,
    product_name,
    popular
FROM popular_item
WHERE ranking = 1;
```

| customer_id | product_name | popular |
|-------------|--------------|-------------------|
| A           | ramen        | 3                 |
| B           | curry        | 2                 |
| B           | sushi        | 2                 |
| B           | ramen        | 2                 |
| C           | ramen        | 3                 |
---
Ramen was the most popular for Customer A as this customer had ordered it 3 times. Sushi, curry, and ramen were all popular for Customer B as they all were ordered 2 times. Ramen was popular for Customer C as it was ordered 3 times.
<br></br>

### **Q6. Which item was purchased first by the customer after they became a member?**
```sql
WITH item_purchased AS(
SELECT 
    s.customer_id,
    product_name,
    order_date,
    members.join_date,
    DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS purchase_ranking
FROM dannys_dinner.menu m
INNER JOIN dannys_dinner.sales s
ON m.product_id=s.product_id
INNER JOIN dannys_dinner.members 
ON s.customer_id = members.customer_id
WHERE order_date >= join_date
)
SELECT 
    customer_id,
    order_date,
    product_name
FROM item_purchased
WHERE purchase_ranking =1;
```
| customer_id | order_Date | product_name |
|-------------|------------|--------------|
| A           | 2021-01-07 | curry        |
| B           | 2021-01-11 | sushi        |
---
Customer A purchased curry and B purchased sushi First after they became member.

### **Q7. Which item was purchased just before the customer became a member?**
```sql
WITH item_purchased AS(
SELECT 
    s.customer_id,
    product_name,
    order_date,
    members.join_date,
    DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS purchase_ranking
FROM dannys_dinner.menu m
INNER JOIN dannys_dinner.sales s
ON m.product_id=s.product_id
INNER JOIN dannys_dinner.members 
ON s.customer_id = members.customer_id
WHERE order_date >= join_date
)
SELECT 
    customer_id,
    product_name,
    order_date
FROM item_purchased
WHERE purchase_ranking =1
```

| customer_id | order_date | product_name |
|-------------|------------|--------------|
| A           | 2021-01-01 | sushi        |
| A           | 2021-01-01 | curry        |
| B           | 2021-01-04 | sushi        |
---
Customer A Purchased sushi and curry B purchased sushi on the day before they became member.
### **Q8. What is the total items and amount spent for each member before they became a member?**
```sql
SELECT DISTINCT
    s.customer_id,
    COUNT(*)total_items,
    SUM(price)total_amount
FROM dannys_dinner.menu m
LEFT JOIN dannys_dinner.sales s
ON m.product_id=s.product_id
LEFT JOIN dannys_dinner.members 
ON s.customer_id = members.customer_id
WHERE order_date < join_date
GROUP BY s.customer_id

```

| customer_id | total_items | total_amount |
|-------------|-------------|--------------|
| A           | 2           | 25           |
| B           | 3           | 40           |

---

### **Q9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
```sql
WITH point_table AS(
SELECT 
    s.customer_id,
    product_name,
    price,
CASE WHEN product_name = 'sushi' THEN 2*10*price
    ELSE 10*price END AS points
FROM dannys_dinner.sales s
LEFT JOIN dannys_dinner.menu m
ON s.product_id=m.product_id
)
SELECT 
    customer_id, 
    SUM(points) AS total_spent
FROM point_table
GROUP BY customer_id
ORDER BY customer_id

```

| customer_id | total_spent |
|-------------|--------|
| A           | 860    |
| B           | 940    |
| C           | 360    |

---

### **Q10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**
```sql
WITH point_table AS(
SELECT 
    s.customer_id,
    product_name,
    order_date,
    price,
CASE WHEN s.order_date between members.join_date and (members.join_date+6) THEN 20*m.price
    WHEN m.product_name = 'sushi' THEN m.price * 20
    ELSE 10*price END AS points
FROM dannys_dinner.sales s
INNER JOIN dannys_dinner.menu m
ON s.product_id=m.product_id
INNER JOIN dannys_dinner.members
ON s.customer_id=members.customer_id
WHERE order_date< '2021-01-31'
)
SELECT 
    customer_id, 
    SUM(points) AS total_spent
FROM point_table
GROUP BY customer_id
ORDER BY customer_id;
```
| customer_id | total_spent |
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

WITH member_check AS(
SELECT 
    s.customer_id,
    s.order_date,
    product_name,
    price,
    CASE WHEN s.order_date>= join_date THEN 'Y'
    ELSE 'N' END AS member
   
FROM dannys_dinner.sales s
LEFT JOIN dannys_dinner.menu as m
ON s.product_id=m.product_id
LEFT JOIN dannys_dinner.members
ON s.customer_id=members.customer_id
)
SELECT 
    customer_id, 
    order_date, 
    product_name,
    price,
    member
FROM member_check
ORDER BY customer_id,order_date,price DESC

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

WITH member_check AS(
SELECT 
    s.customer_id,
    s.order_date,
    product_name,
    price,
    CASE WHEN s.order_date>= join_date THEN 'Y'
    ELSE 'N' END AS member 
FROM dannys_dinner.sales s
LEFT JOIN dannys_dinner.menu as m
ON s.product_id=m.product_id
LEFT JOIN dannys_dinner.members
ON s.customer_id=members.customer_id
)
SELECT *,
CASE WHEN member = 'N' THEN NULL
ELSE RANK()OVER(PARTITION BY customer_id, member ORDER BY order_date) END AS ranking
FROM member_check
ORDER BY customer_id,order_date,price DESC

```


