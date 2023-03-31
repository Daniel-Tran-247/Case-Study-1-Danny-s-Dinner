# 🍜 Case Study #1 - Danny's Diner
[Challenge Site](https://8weeksqlchallenge.com/case-study-1/)
## 📖 Table of Contents
- 🎯 [Problem Statement](#problem-statement)
- 📂 [Dataset](#dataset)
- 🚀 [Questions & Solutions](#questions-&-solutions)

## 🎯 Problem Statement
> Danny wants to use the data to answer a few simple questions about his customers, 
> especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. 
> Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.


## 📁 Dataset
### Entity Relationship Diagram
![image](https://user-images.githubusercontent.com/75436284/211835554-bdf7c43b-8be9-446a-b9e7-858ddbbb4682.png)

### ```sales```
```sql
SELECT * FROM dannys_diner.sales;
```
<details>
<summary>
  Result
</summary>

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
 
 ### ```Menu```
 ```sql
 SELECT * FROM dannys_diner.menu;
 ```
 
<details>
<summary>
  Result
</summary>

|product_id |product_name|price     |
|-----------|------------|----------|
|1          |sushi       |10        |
|2          |curry       |15        |
|3          |ramen       |12        |

</details>
 
### ```Members```
```sql
SELECT * FROM danny_diner.members;
```

<details>
<summary>
  Result
</summary>

|customer_id|join_date |
|-----------|----------|
|A          |1/7/2021  |
|B          |1/9/2021  |

 </details>

## 🚀 Questions & Solutions
### Q1. What is the total amount each customer spent at the restaurant?
```sql
SELECT 
  customer_id,
  SUM(price) AS total_spent
FROM dannys_diner.sales
JOIN dannys_diner.menu
USING (product_id)
GROUP BY customer_id
ORDER BY customer_id;
```

**Result**
| customer_id | total_spent |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |


### Q2. How many days has each customer visited the restaurant?
```sql
SELECT
  customer_id,
  COUNT (DISTINCT order_date) AS num_days_vist
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id;
```
**Result**
|customer_id|num_days_visit|
|-----------|------------  |
|A          |4             |
|B          |6             |
|C          |2             |

### Q3. What was the first item from the menu purchased by each customer?
```sql
WITH cte_table AS (
  SELECT
    customer_id,
    product_name,
    DENSE_RANK() OVER(
     PARTITION BY customer_id
     ORDER BY order_date  
    ) AS item_order
    FROM dannys_diner.sales
    JOIN dannys_diner.menu
    USING (product_id)
)
SELECT * FROM cte_table
WHERE item_order = 1;
```

**Result:**
| customer_id | product_name | item_order |
| ----------- | ------------ | ---------- |
| A           | curry        | 1          |
| A           | sushi        | 1          |
| B           | curry        | 1          |
| C           | ramen        | 1          |

***Note:***
A quick note here is that there is no record about hours/minutes/seconds in the timestamp of order_date.
Therefore, I used **```DENSE_RANK```** instead of **```ROW_NUMBER```** or **```RANK```** to rank the first item ordered by each customer. 
So it is possible that two items can be the first order (They were both ordered on the first day) such as ```curry``` and ```sushi``` for the ```customer A``` 

### Q4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT
  product_name,
  COUNT(product_id) AS item_count
FROM dannys_diner.sales
JOIN dannys_diner.menu
USING (product_id)
GROUP BY product_name
ORDER BY item_count DESC
LIMIT 1;
```

**Result**
|product_name|item_count |
|------------|-----------|
|ramen       |8          |


### Q5. Which item was the most popular for each customer?
```sql
WITH cte_table AS (
  SELECT 
      customer_id,
      product_name,
      COUNT(product_name) AS order_count,
      DENSE_RANK() OVER (
        PARTITION BY customer_id
        ORDER BY COUNT(product_name) DESC 
        ) AS ranking
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
  USING (product_id)
  GROUP BY customer_id, product_name
)
SELECT 
	customer_id,
    product_name, 
    order_count
FROM cte_table
WHERE ranking = 1;
```

**Result**
| customer_id | product_name | product_count |
| ----------- | ------------ | ------------- |
| A           | ramen        | 3             |
| B           | curry        | 2             |
| B           | ramen        | 2             |
| B           | sushi        | 2             |
| C           | ramen        | 3             |

### Q6. Which item was purchased first by the customer after they became a member?
```sql
WITH cte_table AS (
SELECT 
  	customer_id,
  	product_name, 
  	order_date, 
  	join_date, RANK() OVER (
      PARTITION BY customer_id 
      ORDER BY order_date
    ) AS ranking 
  FROM dannys_diner.menu
  JOIN dannys_diner.sales
  USING(product_id)
  JOIN dannys_diner.members
  USING(customer_id)
  WHERE join_date <= order_date 
) 
SELECT 
  customer_id,
  product_name, 
  order_date,
  join_date
FROM cte_table
WHERE ranking = 1;
```
**Result**
| customer_id | product_name | order_date               | join_date      |
| ----------- | ------------ | ------------------------ | -------------- |
| A           | curry        | 2021-01-07T00:00:00.000Z | 2021-01-07T00:00:00.000Z |             |
| B           | sushi        | 2021-01-11T00:00:00.000Z | 2021-01-09T00:00:00.000Z |   

***Note:***
As we can see, ```customer A``` became a member and ordered ```curry``` on the same day. 
We don't really know they order before which event came first, so we just assume that
```customer A``` became a member before he/she paid the bill for the ```curry```

### Q7. Which item was purchased just before the customer became a member?
```sql
WITH cte_table AS (
  SELECT 
  	sales.customer_id,
  	product_name, 
  	order_date, 
  	join_date, 
  	RANK() OVER (
      PARTITION BY sales.customer_id 
      ORDER BY order_date
    ) AS ranking 
  FROM dannys_diner.menu
  JOIN dannys_diner.sales AS sales
  USING(product_id)
  LEFT JOIN dannys_diner.members AS members
  USING (customer_id)
  WHERE join_date > order_date 
  ) 
SELECT 
	customer_id,
    product_name,
    order_date,
    join_date
FROM cte_table
WHERE ranking = 1;
```

**Result**
| customer_id | product_name | order_date               | join_date |
| ----------- | ------------ | ------------------------ | -------------- |
| A           | sushi        | 2021-01-01T00:00:00.000Z | 2021-01-07T00:00:00.000Z	            |
| A           | curry        | 2021-01-01T00:00:00.000Z | 2021-01-07T00:00:00.000Z              |
| B           | sushi        | 2021-01-04T00:00:00.000Z | 2021-01-09T00:00:00.000Z              |

### Q8. What is the total items and amount spent for each member before they became a member?
```sql
SELECT 
    customer_id,
    COUNT(product_name) AS total_items, 
    SUM(price) AS total_spent
FROM dannys_diner.sales
JOIN dannys_diner.menu
USING (product_id)
JOIN dannys_diner.members
USING (customer_id)
WHERE order_date < join_date
GROUP BY customer_id;
```

**Result**
| customer_id | total_items | total_spent |
| ----------- | ----------- | ----------- |
| A           | 2           | 25          |
| B           | 2           | 40          |

### Q9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
WITH cte_table AS (
  SELECT 
      customer_id,
      CASE WHEN product_name = 'sushi' THEN price * 20
      ELSE price * 10
      END AS points
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
  USING (product_id)
 )
 SELECT 
    customer_id, 
    SUM(points) AS points
 FROM cte_table 
 GROUP BY customer_id
 ORDER BY customer_id;
```

**Result**
| customer_id | points	     |
| ----------- | ------------ |
| A           | 860          |
| B           | 940          |
| C	      | 360	     |

### Q10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
WITH cte_table AS (
  SELECT 
      customer_id,
      order_date,
      CASE WHEN order_date BETWEEN join_date AND join_date+6 THEN price*20
      WHEN product_name = 'sushi' THEN price * 20
      ELSE price * 10
      END AS points
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
  USING (product_id)
  JOIN dannys_diner.members
  USING (customer_id)
  WHERE DATE_PART('month', order_date) = 1
)
SELECT 
    customer_id,
    SUM(points) AS total_points
FROM cte_table
WHERE customer_id != 'C'
GROUP BY customer_id
ORDER BY customer_id;
```

**Result**
| customer_id | total_points |
| ----------- | ------------ |
| A           | 1370         |
| B           | 820          |

### [Bonus #1 - Join All The Things] Recreate the following table output using the available data
| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | -------------| ----- | ------ |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

```sql
SELECT 
    customer_id, 
    order_date,
    product_name, 
    price,
    CASE WHEN order_date >= join_date THEN 'Y'
    ELSE 'N'
    END AS member 
FROM dannys_diner.sales
JOIN dannys_diner.menu
USING (product_id)
LEFT JOIN dannys_diner.members
USING (customer_id)
ORDER BY customer_id, order_date;
```

### [Bonus #2 - Rank All The Things] Danny also requires further information about the ```ranking``` of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ```ranking``` values for the records when customers are not yet part of the loyalty program.
| customer_id | order_date | product_name | price | member | ranking | 
| ----------- | ---------- | -------------| ----- | ------ |-------- |
| A           | 2021-01-01 | sushi        | 10    | N      | NULL
| A           | 2021-01-01 | curry        | 15    | N      | NULL
| A           | 2021-01-07 | curry        | 15    | Y      | 1
| A           | 2021-01-10 | ramen        | 12    | Y      | 2
| A           | 2021-01-11 | ramen        | 12    | Y      | 3
| A           | 2021-01-11 | ramen        | 12    | Y      | 3
| B           | 2021-01-01 | curry        | 15    | N      | NULL
| B           | 2021-01-02 | curry        | 15    | N      | NULL
| B           | 2021-01-04 | sushi        | 10    | N      | NULL
| B           | 2021-01-11 | sushi        | 10    | Y      | 1
| B           | 2021-01-16 | ramen        | 12    | Y      | 2
| B           | 2021-02-01 | ramen        | 12    | Y      | 3
| C           | 2021-01-01 | ramen        | 12    | N      | NULL
| C           | 2021-01-01 | ramen        | 12    | N      | NULL
| C           | 2021-01-07 | ramen        | 12    | N      | NULL

```sql
WITH cte_table AS (
  SELECT 
      customer_id, 
      order_date,
      product_name, 
      price,
      CASE WHEN order_date >= join_date THEN 'Y'
      ELSE 'N'
      END AS member
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
  USING (product_id)
  LEFT JOIN dannys_diner.members
  USING (customer_id)
  ORDER BY customer_id, order_date
  ) 
SELECT 
    *,
    CASE WHEN member = 'N' THEN NULL
    ELSE DENSE_RANK() OVER (
      PARTITION BY customer_id, member
      ORDER BY order_date
      )
    END AS ranking
FROM cte_table
```

👉 [View all my SQL code here](https://github.com/Daniel-Tran-247/Case-Study-1-Danny-s-Dinner/blob/main/code.sql)

