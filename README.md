
# 🛒 E-commerce Sales Analysis — SQL Portfolio

## 📌 About This Project
Analysis of an e-commerce sales dataset with 400 orders from January 2022 to February 2023. The goal is to explore revenue trends, customer behavior, and product performance using SQL.

**Tools used:** MySQL v9 | DB Fiddle

**Dataset:** [Kaggle — E-commerce Sales Dataset](https://www.kaggle.com/datasets/abbas829/ecommerce-sales-dataset)

## 📊 Database Structure

The data was normalized into 3 tables:

| Table | Rows | Description |
|-------|------|-------------|
| `customers` | 319 | Customer ID and region |
| `orders` | 400 | Order details, payment method, delivery days, rating |
| `order_items` | 400 | Product category, quantity, price, discount, revenue |

## 📝 SQL Queries

### Query 1 — Monthly Revenue Trend
**Question:** What is the monthly revenue trend? Show year, month, total revenue and total units sold.

```sql
SELECT YEAR(o.order_date) year, MONTH(o.order_date) month,
SUM(oi.revenue) total_revenue, SUM(oi.quantity) total_units_sold
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY YEAR(o.order_date), MONTH(o.order_date)
ORDER BY YEAR(o.order_date), MONTH(o.order_date)

SELECT c.customer_id, SUM(oi.revenue) total_revenue, c.region
FROM order_items oi
JOIN orders o ON oi.order_id = o.order_id
JOIN customers c ON o.customer_id = c.customer_id
GROUP BY c.region, c.customer_id
ORDER BY total_revenue DESC
LIMIT 5

SELECT c.customer_id, c.region, COUNT(order_id) number_of_orders
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.region
HAVING number_of_orders > 1

WITH avg_total AS (
  SELECT customer_id, SUM(revenue) total_rev_cliente
  FROM order_items oi
  JOIN orders o ON oi.order_id = o.order_id
  GROUP BY customer_id)

SELECT avt.customer_id, avt.total_rev_cliente, c.region,
CASE WHEN total_rev_cliente > 5000 THEN 'VIP'
     ELSE 'Regular'
END status
FROM avg_total avt
JOIN customers c ON avt.customer_id = c.customer_id
WHERE total_rev_cliente > (SELECT AVG(total_rev_cliente) FROM avg_total)
ORDER BY total_rev_cliente DESC

SELECT product_category, total_quantity, region
FROM(
  SELECT product_category, SUM(quantity) total_quantity, region,
  ROW_NUMBER() OVER(PARTITION BY region ORDER BY SUM(quantity) DESC) ranking
  FROM order_items oi
  JOIN orders o ON oi.order_id = o.order_id
  JOIN customers c ON c.customer_id = o.customer_id
  GROUP BY product_category, region) tab
WHERE ranking = 1

