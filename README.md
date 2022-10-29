# SQL_Challenge
This is an 8 week SQL Challenge from Danny ma's Weekly challenge
Case Study Questions
# 1.What is the total amount each customer spent at the restaurant?
```SQL
SELECT s.customer_id,
	SUM(m.price) as amount_spent
FROM sales as s
INNER JOIN menu as m
on s.product_id=m.product_id
GROUP BY s.customer_id
ORDER BY amount_spent DESC
```
| Tables        | Are           |
| ------------- |:-------------:|
| 	A	|	76	|
|	B	|	74	|
|	C	|	36	| 
