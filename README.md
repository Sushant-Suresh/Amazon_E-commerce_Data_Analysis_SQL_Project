# E-commerce Data Analysis SQL Project

## Project Overview

**Project Title**: E-commerce Data Analysis  
**Database**: `amazon_project`

This project is designed to demonstrate SQL skills and techniques typically used by data analysts to explore and analyze e-commerce sales data for Amazon. The project involves setting up a database, performing exploratory data analysis (EDA), and answering specific business questions through SQL queries.

## Objectives

1. **Set up an e-commerce sales database**: Create and populate an e-commerce sales database with the provided .csv files.
2. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
3. **Business Analysis**: Use SQL to answer specific business questions and derive insights from the sales data.

## Project Structure

### 1. Database Setup

- **Database Creation**: The project starts by creating a database named `amazon_project`.
- **Table Creation**: The following tables are created - `Customers`, `Sellers`, `Products`, `Orders`, `Returns`.
```sql
-- Creating Database
CREATE DATABASE amazon_project;

-- Creating `Customers` Table
DROP TABLE IF EXISTS customers;
CREATE TABLE customers (customer_id VARCHAR(25) PRIMARY KEY, customer_name VARCHAR(25), state VARCHAR(25));

-- Creating `Sellers` Table
DROP TABLE IF EXISTS sellers;
CREATE TABLE sellers (seller_id VARCHAR(25) PRIMARY KEY, seller_name VARCHAR(25));

-- Creating `Products` Table
DROP TABLE IF EXISTS products;
CREATE TABLE products (product_id VARCHAR(25) PRIMARY KEY, product_name VARCHAR(255), price FLOAT, cogs FLOAT);

-- Creating `Orders` Table
DROP TABLE IF EXISTS orders;
CREATE TABLE orders (order_id VARCHAR(25) PRIMARY KEY, order_date DATE, customer_id VARCHAR(25), state VARCHAR(25), category VARCHAR(25),
sub_category VARCHAR(25), product_id VARCHAR(25), price_per_unit FLOAT, quantity INT, sale FLOAT, seller_id VARCHAR(25),

                     CONSTRAINT fk_customers FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
                     CONSTRAINT fk_products  FOREIGN KEY (product_id)  REFERENCES products(product_id),    
                     CONSTRAINT fk_sellers   FOREIGN KEY (seller_id)   REFERENCES sellers(seller_id));

-- Creating `Returns` Table
DROP TABLE IF EXISTS returns;
CREATE TABLE returns (return_id VARCHAR(25) PRIMARY KEY, order_id VARCHAR(25),

CONSTRAINT fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id));
```
### 2. Data Analysis & Findings

The following SQL queries were used to answer specific business questions:

1. **What Is the Total Sale and Average Sale for Goa, Uttarakhand & Bihar (Round-Off to 2 Decimal Places)?**
```sql
SELECT state,
       ROUND(SUM(sale)::numeric, 2) AS total_sale,
       ROUND(AVG(sale)::numeric, 2) AS avg_sale
FROM orders
WHERE state IN ('Goa', 'Uttarakhand', 'Bihar')
GROUP BY state;
```
2. **What Is the Total Revenue Generated by Each State?**
```sql
SELECT state,
       ROUND(SUM(sale)::numeric, 1) AS revenue
FROM orders
WHERE state IS NOT NULL
GROUP BY state;
```
3. **How Many Orders Were Placed by Each Customer, and What Is Their Average Order Quantity?**
```sql
SELECT customer_id,
       COUNT(order_id) AS orders_placed,
	   ROUND(AVG(quantity)::numeric, 1) AS avg_quantity_ordered
FROM orders
GROUP BY customer_id
```
4. **Which Category Has the Second Highest Average Sale Amount Per Order?**
```sql
SELECT category,
       ROUND(AVG(sale)::numeric, 2) AS avg_sale
FROM orders
WHERE category IS NOT NULL
GROUP BY category
ORDER BY avg_sale DESC
OFFSET 1
LIMIT 1;
```
5. **Identify the Top 3 Best-Selling Products (Sub-Categories) in Terms of Total Quantity Sold.**
```sql
SELECT sub_category,
	   SUM(quantity) AS total_quantity
FROM orders
GROUP BY sub_category
ORDER BY total_quantity DESC
LIMIT 3;
```
6. **Find Top 3 Products Which Generated Revenue > 10000.**
```sql
SELECT product_id,
       SUM(sale) AS revenue
FROM orders
GROUP BY product_id
HAVING SUM(sale) > 10000
ORDER BY revenue DESC
LIMIT 3;
```
7. **Find the Best Selling Month in 2022 Based on Revenue.**
```sql
SELECT TO_CHAR(order_date, 'Month') AS month,
       SUM(sale) AS revenue
FROM orders
WHERE EXTRACT(YEAR FROM order_date) = 2022
GROUP BY TO_CHAR(order_date, 'Month')
ORDER BY revenue DESC
LIMIT 1;
```
8. **Find Customer Names Having Total Orders > 100 and Sort Them by Their Total Revenue in Descending Order.**
```sql
SELECT o.customer_id,
       c.customer_name,
       COUNT(o.order_id) AS total_orders,
	   ROUND(SUM(o.sale)::numeric, 2) AS revenue
FROM orders AS o
INNER JOIN customers AS c
ON o.customer_id = c.customer_id
GROUP BY 1,2
HAVING COUNT(o.order_id) > 100
ORDER BY revenue DESC;
```
9. **Identify All Orders That Have Been Returned, Along With the Details of the Returns (If Available).**
```sql
SELECT *
FROM orders AS o
RIGHT JOIN returns AS r
ON o.order_id = r.order_id;
```
10. **Find All the Instances Where Products Have Been Sold, Returned, or Both, Along With the Associated Details.**
```sql
SELECT p.product_id,
       p.product_name,
       COUNT(DISTINCT o.order_id) AS total_orders,
       COUNT(DISTINCT r.return_id) AS total_returns
FROM products AS p
LEFT JOIN orders AS o 
ON p.product_id = o.product_id
LEFT JOIN returns AS r 
ON o.order_id = r.order_id
GROUP BY p.product_id, p.product_name;
```
11. **Find Each Customer’s Latest and Second Latest Order Amount.**
```sql
WITH RankedOrders AS (SELECT o.customer_id,
                             o.order_id,
                             ROUND(SUM(o.quantity * o.price_per_unit)::numeric, 2) AS order_amount,
                             ROW_NUMBER() OVER (PARTITION BY o.customer_id ORDER BY o.order_date DESC) AS order_rank
                      FROM orders AS o
                      GROUP BY o.customer_id, o.order_id, o.order_date)
SELECT customer_id,
       MAX(CASE WHEN order_rank = 1 THEN order_amount END) AS latest_order_amount,
       MAX(CASE WHEN order_rank = 2 THEN order_amount END) AS second_latest_order_amount
FROM RankedOrders
GROUP BY customer_id;
```
12. **Identify Top-Selling Products by Revenue for Each Category.**
```sql
SELECT category, product_name, total_sales
FROM (SELECT o.category,
             p.product_name,
             ROUND(SUM(o.sale)::numeric, 2) AS total_sales,
             ROW_NUMBER() OVER (PARTITION BY o.category ORDER BY SUM(o.sale) DESC) AS rn
      FROM orders AS o
      JOIN products AS p ON o.product_id = p.product_id
      GROUP BY o.category, p.product_name) AS ranked_sales
WHERE rn = 1;
```
13. **Identify Customers With Orders Exceeding the Average Order Value.**
```sql
SELECT DISTINCT o.customer_id, c.customer_name
FROM orders AS o 
JOIN customers AS c ON o.customer_id = c.customer_id
WHERE o.sale > (SELECT AVG(sale) FROM orders);
```
14. **Calculate the Return Rate Per Seller.**
```sql
SELECT s.seller_id,
       s.seller_name,
       COALESCE(ROUND((COUNT(r.return_id) * 100.0 / COUNT(o.order_id)), 2), 0) AS return_rate
FROM sellers AS s
LEFT JOIN orders AS o ON s.seller_id = o.seller_id
LEFT JOIN returns AS r ON o.order_id = r.order_id
GROUP BY s.seller_id, s.seller_name;
```
15. **Identify States With Highest Sales in Each Product Category.**
```sql
SELECT category, 
       state, 
       total_sales
FROM (SELECT o.category,
             o.state,
             ROUND(SUM(o.sale)::numeric, 2) AS total_sales,
             ROW_NUMBER() OVER (PARTITION BY o.category ORDER BY SUM(o.sale) DESC) AS rn
      FROM orders AS o
      GROUP BY o.category, o.state) AS ranked_sales
WHERE rn = 1;
```
16. **Identify Products With Profit Margin Below Average.**
```sql
SELECT product_id, 
       product_name, 
	   price, 
	   cogs, 
	   ROUND((price - cogs)::numeric, 2) AS profit_margin
FROM products
WHERE (price - cogs) < (SELECT AVG(price - cogs) FROM products);
```
17. **Identify Top 3 Products Frequently Returned Across All Categories. If Multiple Products Have the Same Number of Returns, They Should Be Ranked by the Total Sale Value.**
```sql
SELECT product_name, 
       category, 
	   return_count, 
	   total_sales
FROM (SELECT p.product_name,
             o.category,
             COUNT(r.return_id) AS return_count,
             ROUND(SUM(o.sale)::numeric, 2) AS total_sales,
             RANK() OVER (ORDER BY COUNT(r.return_id) DESC, SUM(o.sale) DESC) AS rn
      FROM orders AS o
      JOIN products AS p ON o.product_id = p.product_id
      LEFT JOIN returns AS r ON o.order_id = r.order_id
      GROUP BY p.product_name, o.category) AS ranked_returns
WHERE rn <= 3;
```
## Conclusion

This project serves as a comprehensive introduction to SQL for data analysts, covering database setup, data cleaning, exploratory data analysis, and business-driven SQL queries. The findings from this project can help drive business decisions by understanding sales patterns, customer behavior, and product demand.

