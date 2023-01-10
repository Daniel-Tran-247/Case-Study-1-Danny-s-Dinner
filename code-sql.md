```sql

/* --------------------
   Case Study Questions
   --------------------*/

-- 1. What is the total amount each customer spent at the restaurant?
SELECT SUM(price) FROM dannys_diner.menu;

-- 2. How many days has each customer visited the restaurant?
SELECT customer_id, COUNT(DISTINCT order_date) FROM dannys_diner.sales
GROUP BY customer_id;

-- 3. What was the first item from the menu purchased by each customer?
SELECT customer_id, product_name as first_product_ordered FROM dannys_diner.sales
JOIN dannys_diner.menu
USING(product_id)
WHERE customer_id = 'A'
ORDER BY order_date
LIMIT 1;

SELECT customer_id, product_name as first_product_ordered FROM dannys_diner.sales
JOIN dannys_diner.menu
USING(product_id)
WHERE customer_id = 'B'
ORDER BY order_date
LIMIT 1;

SELECT customer_id, product_name as first_product_ordered FROM dannys_diner.sales
JOIN dannys_diner.menu
USING(product_id)
WHERE customer_id = 'C'
ORDER BY order_date
LIMIT 1;

-- 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
SELECT product_name, COUNT(product_name) as product_count FROM dannys_diner.menu
JOIN dannys_diner.sales
USING(product_id)
GROUP BY product_name
ORDER BY product_count DESC;

-- 5. Which item was the most popular for each customer?
SELECT DISTINCT customer_id, product_name, product_count FROM ( 
  WITH cte_table AS (
    SELECT customer_id, product_name, COUNT(product_name) OVER (PARTITION BY customer_id, product_name) as product_count FROM dannys_diner.menu 
    JOIN dannys_diner.sales
    USING(product_id)
    ORDER BY customer_id, product_count DESC)

  SELECT *, DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY product_count DESC) as ranking
  FROM cte_table) as cte_table
WHERE ranking = 1;

-- 6. Which item was purchased first by the customer after they became a member?
SELECT customer_id, product_name, order_date, join_date FROM (
  SELECT customer_id, product_name, order_date, join_date, RANK() OVER (PARTITION BY customer_id ORDER BY order_date) as ranking FROM dannys_diner.menu
  JOIN dannys_diner.sales
  USING(product_id)
  JOIN dannys_diner.members
  USING(customer_id)
  WHERE join_date < order_date 
  ) as cte_table
 WHERE ranking = 1;

-- 7. Which item was purchased just before the customer became a member?
SELECT customer_id, product_name, order_date, join_date FROM (
  SELECT sales.customer_id, product_name, order_date, join_date, RANK() OVER (PARTITION BY sales.customer_id ORDER BY order_date) as ranking FROM dannys_diner.menu
  JOIN dannys_diner.sales as sales
  USING(product_id)
  LEFT JOIN dannys_diner.members as members
  ON  sales.customer_id = members.customer_id
  WHERE join_date > order_date or join_date is NULL 
  ) as cte_table
 WHERE ranking = 1;

-- 8. What is the total items and amount spent for each member before they became a member?
  SELECT sales.customer_id, product_name, SUM(price) OVER(PARTITION BY sales.customer_id), order_date, join_date FROM dannys_diner.menu
  JOIN dannys_diner.sales as sales
  USING(product_id)
  LEFT JOIN dannys_diner.members as members
  ON  sales.customer_id = members.customer_id
  WHERE join_date > order_date or join_date is NULL;
  
-- 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
SELECT customer_id, SUM(points) FROM (
  SELECT customer_id, product_name, price,
  CASE 
      WHEN product_name = 'sushi' THEN 20*price
      ELSE 10*price
  END AS points
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
  USING(product_id)
  ORDER BY customer_id) as cte_table
GROUP BY customer_id;

-- 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
SELECT customer_id, SUM(points) FROM (
  SELECT *,
  CASE 
      WHEN order_date BETWEEN join_date AND join_date+7 THEN price*20
      WHEN product_name = 'sushi' THEN price*20
      ELSE price*10
  END AS points
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
  USING(product_id)
  JOIN dannys_diner.members
  USING(customer_id)
  WHERE DATE_PART('month',order_date) = 1) as cte_table
GROUP BY customer_id;

```
