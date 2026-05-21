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

```sql
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
  
Answer:

Customer A: $76

Customer B: $74

Customer C: $36

2. How many days has each customer visited the restaurant?
SQL

SELECT
  s.customer_id,
  COUNT(DISTINCT s.order_date) AS total_days
FROM dannys_diner.menu m 
JOIN dannys_diner.sales s
  ON m.product_id = s.product_id
GROUP BY
  s.customer_id
ORDER BY 
  total_days DESC;
Answer:

Customer B: 6 days

Customer A: 4 days

Customer C: 2 days

3. What was the first item from the menu purchased by each customer?
SQL

WITH ranked AS (  
  SELECT
      s.customer_id,
      s.order_date,
      m.product_name,
      DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS rnk
  FROM dannys_diner.menu m 
  JOIN dannys_diner.sales s
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
WHERE rnk = 1;
Answer:

Customer A: Curry and Sushi

Customer B: Curry

Customer C: Ramen

4. What is the most purchased item on the menu and how many times was it purchased by all customers?
SQL

SELECT 
  m.product_name,
  COUNT(s.product_id) AS times_purchased
FROM dannys_diner.sales s 
JOIN dannys_diner.menu m
  ON s.product_id = m.product_id
GROUP BY
  m.product_name
ORDER BY
  times_purchased DESC
LIMIT 1;
Answer:

Ramen is the most purchased item, bought 8 times in total.

5. Which item was the most popular for each customer?
SQL

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
Answer:

Customer A: Ramen (3 times)

Customer B: Ramen, Curry, and Sushi (2 times each)

Customer C: Ramen (3 times)

6. Which item was purchased first by the customer after they became a member?
SQL

WITH RANKED AS (  
  SELECT 
      s.customer_id,
      mem.join_date,
      s.order_date,
      m.product_name,
      DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS rnk
  FROM sales s 
  JOIN menu m ON s.product_id = m.product_id
  JOIN members mem ON s.customer_id = mem.customer_id
  WHERE mem.join_date <= s.order_date
)
SELECT 
    customer_id,
    join_date,
    order_date,
    product_name
FROM RANKED
WHERE rnk = 1;
Answer:

Customer A: Curry (on join date 2021-01-07)

Customer B: Sushi (on 2021-01-11)

7. Which item was purchased just before the customer became a member?
SQL

WITH RANKED AS (  
  SELECT 
      s.customer_id,
      mem.join_date,
      s.order_date,
      m.product_name,
      DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS rnk
  FROM sales s 
  JOIN menu m ON s.product_id = m.product_id
  JOIN members mem ON s.customer_id = mem.customer_id
  WHERE mem.join_date > s.order_date
)
SELECT 
    customer_id,
    join_date,
    order_date,
    product_name
FROM RANKED
WHERE rnk = 1;
Answer:

Customer A: Sushi and Curry (on 2021-01-01)

Customer B: Sushi (on 2021-01-04)

8. What is the total items and amount spent for each member before they became a member?
SQL

SELECT 
   s.customer_id,
   COUNT(s.product_id) AS total_sold,
   SUM(m.price) AS total_spent
FROM sales s 
JOIN menu m ON s.product_id = m.product_id
JOIN members mem ON s.customer_id = mem.customer_id
WHERE s.order_date < mem.join_date
GROUP BY s.customer_id
ORDER BY s.customer_id;
Answer:

Customer A: 2 items, totaling $25

Customer B: 3 items, totaling $40

9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
SQL

SELECT
  s.customer_id,
  SUM(CASE WHEN m.product_name = 'sushi' THEN 10 * 2 * m.price 
      ELSE 10 * m.price END) AS points
FROM sales s 
JOIN menu m ON s.product_id = m.product_id 
GROUP BY s.customer_id
ORDER BY s.customer_id;
Answer:

Customer A: 860 points

Customer B: 940 points

Customer C: 360 points

10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
SQL

SELECT
  s.customer_id,
  SUM(CASE 
      WHEN s.order_date BETWEEN mem.join_date AND mem.join_date + INTERVAL '6 DAYS' THEN m.price * 20
      WHEN m.product_name = 'sushi' THEN m.price * 20
      ELSE m.price * 10
  END) AS jan_points
FROM sales s 
JOIN menu m ON s.product_id = m.product_id 
JOIN members mem ON s.customer_id = mem.customer_id
WHERE s.order_date BETWEEN '2021-01-01' AND '2021-01-31'
GROUP BY s.customer_id
ORDER BY s.customer_id;
Answer:

Customer A: 1,370 points

Customer B: 820 points

💎 Bonus Questions
Bonus 1: Join All The Things
SQL

SELECT
  s.customer_id,
  s.order_date,
  m.product_name,
  m.price,
  CASE 
    WHEN s.order_date >= mem.join_date THEN 'Y'
    ELSE 'N' 
  END AS member
FROM sales s 
LEFT JOIN menu m ON s.product_id = m.product_id 
LEFT JOIN members mem ON s.customer_id = mem.customer_id
ORDER BY
  s.customer_id,
  s.order_date,
  m.price DESC;
Bonus 2: Rank All The Things
SQL

WITH joined_table AS (  
  SELECT
      s.customer_id,
      s.order_date,
      m.product_name,
      m.price,
      CASE 
        WHEN s.order_date >= mem.join_date THEN 'Y'
        ELSE 'N' 
       END AS member
  FROM sales s 
  LEFT JOIN menu m ON s.product_id = m.product_id 
  LEFT JOIN members mem ON s.customer_id = mem.customer_id
)
SELECT
  *,
  CASE 
    WHEN member = 'N' THEN NULL
    ELSE RANK() OVER(PARTITION BY customer_id, member ORDER BY order_date)
  END AS ranking
FROM joined_table
ORDER BY
  customer_id,
  order_date;
