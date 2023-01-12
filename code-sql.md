```sql

/* --------------------
   Case Study Questions
   --------------------*/

-- 1. What is the total amount each customer spent at the restaurant?

SELECT 
  customer_id,
  SUM(price) AS total_spent
FROM dannys_diner.sales
JOIN dannys_diner.menu
USING (product_id)
GROUP BY customer_id
ORDER BY customer_id;

-- 2. How many days has each customer visited the restaurant?

SELECT
  customer_id,
  COUNT (DISTINCT order_date) AS num_days_vist
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id;

-- 3. What was the first item from the menu purchased by each customer?

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

-- 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

SELECT
  product_name,
  COUNT(product_id) AS item_count
FROM dannys_diner.sales
JOIN dannys_diner.menu
USING (product_id)
GROUP BY product_name
ORDER BY item_count DESC
LIMIT 1;

-- 5. Which item was the most popular for each customer?

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

-- 6. Which item was purchased first by the customer after they became a member?

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

-- 7. Which item was purchased just before the customer became a member?

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

-- 8. What is the total items and amount spent for each member before they became a member?

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

-- 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

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

-- 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

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

-- [Bonus #1 - Join All The Things] 

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

-- [Bonus #2 - Rank All The Things]

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
FROM cte_table;
```
