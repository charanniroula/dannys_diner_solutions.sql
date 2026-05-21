# 🍜 Case Study #1: Danny's Diner

## 📊 Business Context
Danny wants to use data to answer simple questions about his customers' visiting patterns, how much money they’ve spent, and which menu items are their favorite. 

## 🛠️ Tech Stack & Skills
- **Database Engine:** PostgreSQL / MySQL 
- **Advanced SQL Concepts:** Common Table Expressions (CTEs), Window Functions (`DENSE_RANK()`), Conditional Logic (`CASE WHEN`), Date/Time Arithmetic (`INTERVAL`), and Relational Joins (`LEFT JOIN`, `INNER JOIN`).

## 🔍 Key Insights Discovered
- **Customer Engagement:** Customer A was the most active participant in the loyalty program, accumulating 1,370 points by the end of January.
- **Product Popularity:** Ramen is the undisputed best-seller across the entire customer canvas, driving the highest volume of traffic.

## 📂 Solutions Directory
All 10 optimized SQL queries along with the schema setup can be found in [dannys_diner_solutions.sql](./dannys_diner_solutions.sql).

/* --------------------
   Case Study Questions
   --------------------*/

## 💻 Case Study Solutions

### 1. What is the total amount each customer spent at the restaurant?

SQL
SELECT
  s.customer_id,
  SUM(m.price) AS total
FROM dannys_diner.menu m 
JOIN dannys_diner.sales s
  ON m.product_id = s.product_id
GROUP BY
  s.customer_id
ORDER BY 
  s.customer_id;
-- > Customer A spent $76, B spent $74, C spent $36

-- 2. How many days has each customer visited the restaurant?
/*
SELECT
  	s.customer_id,
    COUNT(DISTINCT s.order_date) AS total_days
FROM dannys_diner.menu m JOIN dannys_diner.sales s
	ON m.product_id = s.product_id
GROUP BY
	s.customer_id
*/
-- > B - 6, A - 4, C - 2

-- 3. What was the first item from the menu purchased by each customer?

/*
WITH ranked AS (  
  SELECT
      s.customer_id,
      s.order_date,
      m.product_name,
      DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS rnk
  FROM dannys_diner.menu m JOIN dannys_diner.sales s
      ON m.product_id = s.product_id
  GROUP BY
      s.customer_id,
      s.order_date,
      m.product_name
)
SELECT 
	customer_id,
    order_date,
    product_name
FROM ranked
WHERE rnk = 1
*/
-- > A - curry, sushu B - curry C - ramen 

-- 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

/*
SELECT 
	product_name,
	COUNT(product_name) AS times_purchased
FROM
	sales s JOIN menu m
    ON s.product_id = m.product_id
GROUP BY
	product_name
ORDER BY
	times_purchased DESC
*/
-- > Ramen 8 times

-- 5. Which item was the most popular for each customer?

/*
WITH most_popular AS (
  SELECT 
    sales.customer_id, 
    menu.product_name, 
    COUNT(menu.product_id) AS order_count,
    DENSE_RANK() OVER (
      PARTITION BY sales.customer_id 
      ORDER BY COUNT(sales.customer_id) DESC) AS rank
  FROM dannys_diner.menu
  INNER JOIN dannys_diner.sales
    ON menu.product_id = sales.product_id
  GROUP BY sales.customer_id, menu.product_name
)

SELECT 
  customer_id, 
  product_name, 
  order_count
FROM most_popular 
WHERE rank = 1;
*/
-- > Customer A - ramen 3 times, Customer B - ramen, cutty, sushi all 2 times, C - ramen 3 times

-- 6. Which item was purchased first by the customer after they became a member?

/*
WITH RANKED AS(  
  SELECT 
      s.customer_id,
      mem.join_date,
      s.order_date,
      m.product_name,
      DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS rnk
  FROM	
      sales s JOIN menu m
      ON s.product_id = m.product_id
      JOIN members mem 
      ON s.customer_id = mem.customer_id
  WHERE 
      mem.join_date<=s.order_date
)
SELECT 
	customer_id,
    join_date,
    order_date,
    product_name
FROM 
	RANKED
WHERE 
	rnk = 1
*/
-- > Customer A - curry, Customer B - sushi

-- 7. Which item was purchased just before the customer became a member?

/*
WITH RANKED AS(  
  SELECT 
      s.customer_id,
      mem.join_date,
      s.order_date,
      m.product_name,
      DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS rnk
  FROM	
      sales s JOIN menu m
      ON s.product_id = m.product_id
      JOIN members mem 
      ON s.customer_id = mem.customer_id
  WHERE 
      mem.join_date>s.order_date
)
SELECT 
	customer_id,
    join_date,
    order_date,
    product_name
FROM 
	RANKED
WHERE 
	rnk = 1
*/
-- > Customer A - sushi and curry, Customer B - sushi

-- 8. What is the total items and amount spent for each member before they became a member?

/*
SELECT 
   s.customer_id,
   COUNT(s.product_id) AS total_sold,
   SUM(m.price)
FROM	
   sales s JOIN menu m
   ON s.product_id = m.product_id
   JOIN members mem 
   ON s.customer_id = mem.customer_id
WHERE 
    s.order_date < mem.join_date
GROUP BY
	s.customer_id
ORDER BY
	s.customer_id
*/
-- > customer A bought 2 items totaling $25
-- > customer B bought 3 items totaling $40

-- 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

/*
SELECT
	s.customer_id,
	SUM(CASE WHEN m.product_name = 'sushi' THEN 10 * 2 * m.price 
    ELSE 10 * m.price END) AS points
FROM
	sales s JOIN menu m 
    ON s.product_id = m.product_id 
GROUP BY
	s.customer_id
ORDER BY
	s.customer_id
*/
-- > Customer A - 860 points, B - 940 points, C - 360 points   

-- 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

/*
SELECT
	s.customer_id,
    SUM(CASE WHEN s.order_date BETWEEN mem.join_date AND mem.join_date + INTERVAL '6 DAYS' THEN m.price * 20
    WHEN m.product_name = 'sushi' THEN m.price * 20
    ELSE m.price * 10
    END) AS jan_points
FROM
	sales s JOIN menu m 
    ON s.product_id = m.product_id 
    JOIN members mem
    ON s.customer_id = mem.customer_id
WHERE
	s.order_date BETWEEN '2021-01-01' AND '2021-01-31'
GROUP BY
	s.customer_id
ORDER BY
	s.customer_id
*/
-- > January points: Customer A = 1370, Customer B = 820

-- BONUS QUESTIONS

/* The following questions are related creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL. */

/*
SELECT
	s.customer_id,
    s.order_date,
    m.product_name,
    m.price,
    CASE WHEN s.order_date < mem.join_date THEN 'N' 
    WHEN mem.join_date IS NULL THEN 'N'
    ELSE 'Y' END AS member
FROM
	sales s LEFT JOIN menu m 
    ON s.product_id = m.product_id 
    LEFT JOIN members mem
    ON s.customer_id = mem.customer_id
ORDER BY
	s.customer_id,
    s.order_date,
    m.price DESC
*/  

/* Rank All The Things
Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program */

WITH joined_table AS(  
  SELECT
      s.customer_id,
      s.order_date,
      m.product_name,
      m.price,
      CASE WHEN s.order_date < mem.join_date THEN 'N' 
      WHEN mem.join_date IS NULL THEN 'N'
      ELSE 'Y' END AS member
  FROM
      sales s LEFT JOIN menu m 
      ON s.product_id = m.product_id 
      LEFT JOIN members mem
      ON s.customer_id = mem.customer_id
  ORDER BY
      s.customer_id,
      s.order_date,
      m.price DESC
)
SELECT
	*,
    CASE WHEN member = 'N' THEN NULL
    ELSE RANK() OVER(PARTITION BY customer_id, member ORDER BY order_date)
    END AS ranking
FROM
	joined_table
