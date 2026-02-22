# goit-rdb-hw-04
DML and DDL Commands. Complex SQL Expressions
# Library Management System & Order Analytics (SQL)

This repository contains the implementation of a database schema for a library and advanced analytical SQL queries performed on a retail database.

## 📋 Project Overview

The project is divided into two main parts:
1. **Schema Design**: Building a relational structure for a "Library Management" system with data integrity constraints.
2. **Data Analytics**: Performing complex multi-table joins, aggregations, and performance filtering on an existing order management database.

---

## 🛠 Part 1: Library Management System

### Key Features
* **Relational Integrity**: Implemented using `FOREIGN KEY` constraints across five tables.
* **Historical Data Support**: The `publication_year` column uses the `INT` data type instead of `YEAR` to support historical dates prior to 1901 (e.g., Ivan Franko's works from 1893).
* **Automated Primary Keys**: Utilized `AUTO_INCREMENT` for efficient record indexing.

### Schema Structure


---

## 📊 Part 2: Advanced Order Analytics

### 1. The "Big Join"
A comprehensive query combining 8 tables: `order_details`, `orders`, `customers`, `products`, `categories`, `employees`, `shippers`, and `suppliers`. 
* **Result**: 518 rows.

### 2. Join Type Comparison (INNER vs. LEFT/RIGHT)
* **Inner Join Count**: 518 rows.
* **Mixed Outer Join Count**: 519 rows.
* **Analysis**: The row count increased because `INNER JOIN` excludes records without matches in both tables. `LEFT/RIGHT JOIN` preserves "orphaned" records (e.g., an employee who hasn't made a sale or a product never ordered), filling missing gaps with `NULL`.

### 3. Aggregated Reporting
Calculated the average quantity of products per category for a specific group of employees (IDs 4-10), filtered for high-volume categories (>21 units) and paginated using `OFFSET`.

---

## 🚀 The SQL Script

```sql
/* COMPLETE SQL SCRIPT 
   Includes: Schema Creation, Data Seeding, and Advanced Analytics
*/

-- =============================================
-- SECTION 1: LIBRARY SCHEMA & SEEDING
-- =============================================

CREATE SCHEMA IF NOT EXISTS LibraryManagement;
USE LibraryManagement;

-- Drop tables in reverse order of dependency
DROP TABLE IF EXISTS borrowed_books;
DROP TABLE IF EXISTS books;
DROP TABLE IF EXISTS authors;
DROP TABLE IF EXISTS genres;
DROP TABLE IF EXISTS users;

-- Table Creation
CREATE TABLE authors (
    author_id INT AUTO_INCREMENT PRIMARY KEY,
    author_name VARCHAR(255) NOT NULL
);

CREATE TABLE genres (
    genre_id INT AUTO_INCREMENT PRIMARY KEY,
    genre_name VARCHAR(255) NOT NULL
);

CREATE TABLE books (
    book_id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    publication_year INT, -- Supports dates before 1901
    author_id INT,
    genre_id INT,
    FOREIGN KEY (author_id) REFERENCES authors(author_id),
    FOREIGN KEY (genre_id) REFERENCES genres(genre_id)
);

CREATE TABLE users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL
);

CREATE TABLE borrowed_books (
    borrow_id INT AUTO_INCREMENT PRIMARY KEY,
    book_id INT,
    user_id INT,
    borrow_date DATE,
    return_date DATE,
    FOREIGN KEY (book_id) REFERENCES books(book_id),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

-- Data Seeding (Ivan Franko Example)
INSERT INTO authors (author_name) VALUES ('Ivan Franko');
INSERT INTO genres (genre_name) VALUES ('Drama');
INSERT INTO books (title, publication_year, author_id, genre_id) 
VALUES ('Stolen Happiness', 1893, 1, 1);

INSERT INTO users (username, email) VALUES ('ivan_fan', 'franko_fan@ukr.net');
INSERT INTO borrowed_books (book_id, user_id, borrow_date, return_date) 
VALUES (1, 1, '2024-02-21', '2024-03-21');

-- =============================================
-- SECTION 2: ANALYTICS (W3SCHOOLS DATABASE)
-- =============================================

USE hw3;

-- 4.1 Row Count with INNER JOIN
SELECT COUNT(*) AS inner_join_count
FROM order_details AS od
INNER JOIN orders AS o ON od.order_id = o.id
INNER JOIN customers AS c ON o.customer_id = c.id
INNER JOIN products AS p ON od.product_id = p.id
INNER JOIN categories AS cat ON p.category_id = cat.id
INNER JOIN employees AS e ON o.employee_id = e.employee_id
INNER JOIN shippers AS sh ON o.shipper_id = sh.id
INNER JOIN suppliers AS sup ON p.supplier_id = sup.id;

-- 4.2 Row Count with LEFT/RIGHT JOIN
SELECT COUNT(*) AS outer_join_count
FROM order_details AS od
LEFT JOIN orders AS o ON od.order_id = o.id
LEFT JOIN customers AS c ON o.customer_id = c.id
RIGHT JOIN products AS p ON od.product_id = p.id
LEFT JOIN categories AS cat ON p.category_id = cat.id
RIGHT JOIN employees AS e ON o.employee_id = e.employee_id
LEFT JOIN shippers AS sh ON o.shipper_id = sh.id
LEFT JOIN suppliers AS sup ON p.supplier_id = sup.id;

-- 4.3 - 4.7 Grouping, Filtering, and Pagination
SELECT 
    cat.name AS category_name,
    COUNT(*) AS total_rows,
    AVG(od.quantity) AS average_quantity
FROM order_details AS od
INNER JOIN orders AS o ON od.order_id = o.id
INNER JOIN products AS p ON od.product_id = p.id
INNER JOIN categories AS cat ON p.category_id = cat.id
INNER JOIN employees AS e ON o.employee_id = e.employee_id
WHERE e.employee_id > 3 AND e.employee_id <= 10
GROUP BY cat.name 
HAVING AVG(od.quantity) > 21
ORDER BY total_rows DESC
LIMIT 4 OFFSET 1;
