# E-commerce Data Analysis SQL Project

## Project Overview

**Project Title**: E-commerce Data Analysis  
**Database**: `amazon_project`

This project is designed to demonstrate SQL skills and techniques typically used by data analysts to explore and analyze e-commerce sales data. The project involves setting up a database, performing exploratory data analysis (EDA), and answering specific business questions through SQL queries.

## Objectives

1. **Set up an e-commerce sales database**: Create and populate an e-commerce sales database with the provided .csv files.
2. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
3. **Business Analysis**: Use SQL to answer specific business questions and derive insights from the sales data.

## Project Structure

### 1. Schema
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
