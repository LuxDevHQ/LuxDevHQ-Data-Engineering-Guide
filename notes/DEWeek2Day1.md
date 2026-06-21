# Week 2, Day 1: Core SQL Concepts (`SELECT`, `WHERE`, `JOIN`, `GROUP BY`)

> **Month 1: Foundations of Data Engineering**
> **Week 2: SQL for Data Engineering**
> **Monday**

---

## What we covered last week (Week 1 recap)

Last week we built the foundations of data engineering:

| Day | Topic |
|---|---|
| Day 1 | What data engineering is - pipelines, ETL, data warehouses |
| Day 2 | Cloud computing - AWS, Azure, and GCP and their core services |
| Day 3 | Data governance, security, compliance, and access control |
| Day 4 | Introduction to SQL and PostgreSQL setup |
| Day 5 (Lab) | Built a basic pipeline with PostgreSQL and Azure Blob Storage |

On Day 4 we installed PostgreSQL, connected using psql and pgAdmin, and wrote our first SQL queries - creating tables, inserting data, and running basic SELECT statements.

> **This week we go deeper into SQL** - the language you will use every single day as a data engineer, in every pipeline, in every warehouse, on every cloud platform.

---

## Today's objective

By the end of today, you should be able to:

- Write powerful `SELECT` queries to retrieve exactly the data you need.
- Use `WHERE` with multiple conditions to filter data precisely.
- Use all major types of `JOIN` to combine data from multiple tables.
- Use `GROUP BY` with aggregate functions to summarize and analyze data.
- Combine all four concepts together to answer real business questions.
- Apply these skills in the context of data engineering pipelines and warehouses.

---

## Setup: create today's working database

Before we start, let us set up the database and tables we will use throughout today's session. Open **psql** or **pgAdmin** and run the following:

```sql
-- Create and connect to the database
CREATE DATABASE ecommerce_db;
\c ecommerce_db

-- Create schemas
CREATE SCHEMA raw;
CREATE SCHEMA staging;
CREATE SCHEMA analytics;
```

```sql
-- Customers table
CREATE TABLE raw.customers (
    customer_id   SERIAL PRIMARY KEY,
    first_name    VARCHAR(100),
    last_name     VARCHAR(100),
    email         VARCHAR(255) UNIQUE,
    city          VARCHAR(100),
    country       VARCHAR(100),
    segment       VARCHAR(50),       -- 'retail', 'wholesale', 'online'
    created_at    TIMESTAMP DEFAULT NOW()
);

-- Products table
CREATE TABLE raw.products (
    product_id    SERIAL PRIMARY KEY,
    product_name  VARCHAR(255),
    category      VARCHAR(100),
    unit_price    NUMERIC(10, 2),
    stock_qty     INTEGER
);

-- Orders table
CREATE TABLE raw.orders (
    order_id      SERIAL PRIMARY KEY,
    customer_id   INTEGER REFERENCES raw.customers(customer_id),
    order_date    DATE,
    status        VARCHAR(50),       -- 'pending', 'shipped', 'delivered', 'cancelled'
    total_amount  NUMERIC(10, 2)
);

-- Order items table (one order can have many items)
CREATE TABLE raw.order_items (
    item_id       SERIAL PRIMARY KEY,
    order_id      INTEGER REFERENCES raw.orders(order_id),
    product_id    INTEGER REFERENCES raw.products(product_id),
    quantity      INTEGER,
    unit_price    NUMERIC(10, 2),
    subtotal      NUMERIC(10, 2)
);
```

```sql
-- Insert customers
INSERT INTO raw.customers (first_name, last_name, email, city, country, segment) VALUES
('Alice',   'Wanjiru',  'alice@mail.com',   'Nairobi',  'Kenya',    'online'),
('Brian',   'Otieno',   'brian@mail.com',   'Kisumu',   'Kenya',    'retail'),
('Carol',   'Muthoni',  'carol@mail.com',   'Nairobi',  'Kenya',    'wholesale'),
('David',   'Kimani',   'david@mail.com',   'Nakuru',   'Kenya',    'online'),
('Eve',     'Achieng',  'eve@mail.com',     'Mombasa',  'Kenya',    'retail'),
('Frank',   'Mwangi',   'frank@mail.com',   'Nairobi',  'Kenya',    'wholesale'),
('Grace',   'Njeri',    'grace@mail.com',   'Nairobi',  'Kenya',    'online'),
('Henry',   'Odhiambo', 'henry@mail.com',   'Kisumu',   'Kenya',    'retail');

-- Insert products
INSERT INTO raw.products (product_name, category, unit_price, stock_qty) VALUES
('Laptop',          'Electronics',  85000.00,   50),
('Smartphone',      'Electronics',  45000.00,   120),
('Headphones',      'Electronics',  3500.00,    200),
('Office Chair',    'Furniture',    18000.00,   30),
('Standing Desk',   'Furniture',    35000.00,   15),
('Notebook (A4)',   'Stationery',   150.00,     1000),
('Ballpoint Pens',  'Stationery',   50.00,      5000),
('Backpack',        'Accessories',  4500.00,    80);

-- Insert orders
INSERT INTO raw.orders (customer_id, order_date, status, total_amount) VALUES
(1, '2025-05-01', 'delivered',  85000.00),
(2, '2025-05-03', 'delivered',  3500.00),
(3, '2025-05-05', 'delivered',  53000.00),
(1, '2025-05-10', 'shipped',    4500.00),
(4, '2025-05-12', 'delivered',  45000.00),
(5, '2025-05-14', 'cancelled',  18000.00),
(6, '2025-05-15', 'delivered',  70000.00),
(7, '2025-05-18', 'pending',    150.00),
(2, '2025-05-20', 'delivered',  9000.00),
(3, '2025-05-22', 'shipped',    35000.00),
(8, '2025-05-25', 'delivered',  4500.00),
(1, '2025-06-01', 'delivered',  45000.00);

-- Insert order items
INSERT INTO raw.order_items (order_id, product_id, quantity, unit_price, subtotal) VALUES
(1,  1, 1, 85000.00, 85000.00),
(2,  3, 1,  3500.00,  3500.00),
(3,  2, 1, 45000.00, 45000.00),
(3,  4, 1, 18000.00, 18000.00) ,
(4,  8, 1,  4500.00,  4500.00),
(5,  2, 1, 45000.00, 45000.00),
(6,  1, 1, 85000.00, 85000.00),
(7,  6, 1,   150.00,   150.00),
(8,  3, 2,  3500.00,  7000.00),
(9,  3, 2,  3500.00,  7000.00),
(10, 5, 1, 35000.00, 35000.00),
(11, 8, 1,  4500.00,  4500.00),
(12, 2, 1, 45000.00, 45000.00);
```

> You now have 4 related tables - customers, products, orders, and order_items. This is a realistic e-commerce data model used in real pipelines.

---

## 1. `SELECT` - retrieving data with precision

`SELECT` is the most used SQL statement. In data engineering you use it to:
- Inspect data at every stage of a pipeline.
- Extract data from source systems.
- Query data warehouses for reports and dashboards.

### Basic SELECT

```sql
-- Get everything (use only for exploration)
SELECT * FROM raw.customers;

-- Get only the columns you need (best practice in pipelines)
SELECT customer_id, first_name, last_name, city, segment
FROM raw.customers;
```

> **Why avoid `SELECT *` in pipelines?**
> When you write `SELECT *`, your pipeline will break silently if someone adds or removes a column from the source table. Always name your columns explicitly in production pipelines.

### SELECT with expressions and aliases

```sql
-- Calculate a discounted price on the fly
SELECT
    product_name,
    category,
    unit_price,
    unit_price * 0.9                        AS discounted_price,
    unit_price - (unit_price * 0.9)         AS discount_amount
FROM raw.products;
```

```sql
-- Concatenate first and last name
SELECT
    customer_id,
    first_name || ' ' || last_name          AS full_name,
    email,
    city
FROM raw.customers;
```

### SELECT DISTINCT - removing duplicates

```sql
-- What cities do our customers come from?
SELECT DISTINCT city
FROM raw.customers
ORDER BY city;

-- What categories of products do we have?
SELECT DISTINCT category
FROM raw.products
ORDER BY category;
```

### Useful SELECT functions for data engineering

```sql
-- String functions
SELECT
    UPPER(first_name)                       AS name_upper,
    LOWER(email)                            AS email_lower,
    LENGTH(email)                           AS email_length,
    TRIM('  hello  ')                       AS trimmed
FROM raw.customers;

-- Date functions
SELECT
    order_id,
    order_date,
    EXTRACT(YEAR  FROM order_date)          AS order_year,
    EXTRACT(MONTH FROM order_date)          AS order_month,
    EXTRACT(DOW   FROM order_date)          AS day_of_week,  -- 0=Sunday
    NOW()                                   AS current_timestamp,
    order_date + INTERVAL '30 days'         AS due_date
FROM raw.orders;

-- NULL handling
SELECT
    customer_id,
    first_name,
    COALESCE(city, 'Unknown')               AS city
FROM raw.customers;
```

> **COALESCE()** is one of the most useful functions in data engineering. It replaces NULL with a default value - critical for cleaning data in pipelines.

---

## 2. `WHERE` - filtering data precisely

`WHERE` filters rows before they are returned. In data engineering, you use `WHERE` to:
- Extract only the data you need from a large source table.
- Isolate records with quality issues (NULLs, bad values, duplicates).
- Filter by date ranges in incremental pipeline loads.

### Basic WHERE

```sql
-- Orders that were delivered
SELECT * FROM raw.orders
WHERE status = 'delivered';

-- Customers from Nairobi
SELECT * FROM raw.customers
WHERE city = 'Nairobi';

-- Products priced above 10,000
SELECT * FROM raw.products
WHERE unit_price > 10000;
```

### WHERE with multiple conditions

```sql
-- AND — both conditions must be true
SELECT * FROM raw.orders
WHERE status = 'delivered'
AND total_amount > 10000;

-- OR — at least one condition must be true
SELECT * FROM raw.orders
WHERE status = 'pending'
OR status = 'shipped';

-- NOT — exclude a condition
SELECT * FROM raw.orders
WHERE NOT status = 'cancelled';
```

### WHERE with IN and NOT IN

```sql
-- IN — match any value in a list (cleaner than multiple ORs)
SELECT * FROM raw.orders
WHERE status IN ('pending', 'shipped');

-- NOT IN — exclude a list of values
SELECT * FROM raw.customers
WHERE city NOT IN ('Nairobi', 'Mombasa');
```

### WHERE with BETWEEN

```sql
-- Orders placed in May 2025
SELECT * FROM raw.orders
WHERE order_date BETWEEN '2025-05-01' AND '2025-05-31';

-- Products in a price range
SELECT * FROM raw.products
WHERE unit_price BETWEEN 1000 AND 50000;
```

### WHERE with LIKE - pattern matching

```sql
-- Customers whose email ends with @mail.com
SELECT * FROM raw.customers
WHERE email LIKE '%@mail.com';

-- Products whose name starts with 'S'
SELECT * FROM raw.products
WHERE product_name LIKE 'S%';

-- Case-insensitive match (PostgreSQL specific)
SELECT * FROM raw.customers
WHERE email ILIKE '%ALICE%';
```

| Pattern | Meaning |
|---|---|
| `LIKE 'A%'` | Starts with A |
| `LIKE '%a'` | Ends with a |
| `LIKE '%oo%'` | Contains oo anywhere |
| `LIKE '_at'` | Any single character followed by 'at' |

### WHERE with NULL checks

```sql
-- Find rows where city is missing
SELECT * FROM raw.customers
WHERE city IS NULL;

-- Find rows where city is NOT missing
SELECT * FROM raw.customers
WHERE city IS NOT NULL;
```

>  **Never use `= NULL`** - it does not work in SQL. Always use `IS NULL` or `IS NOT NULL`.

### WHERE for incremental pipeline loads

This is a critical data engineering pattern. Instead of loading all data every time, you only load **new or changed records**:

```sql
-- Load only orders created in the last 24 hours (incremental load)
SELECT * FROM raw.orders
WHERE order_date >= NOW() - INTERVAL '1 day';

-- Load only orders updated since the last pipeline run
SELECT * FROM raw.orders
WHERE order_date > '2025-05-31';
```

> **Incremental loading** is one of the most important patterns in data engineering. Instead of re-processing millions of rows every time, you filter with `WHERE` to process only what is new. This saves time, money, and compute resources.

---

## 3. `JOIN` - combining data from multiple tables

`JOIN` is the most powerful SQL concept for data engineers. Real-world data is always spread across multiple tables  and JOINs let you bring it together.

### How JOINs work

A JOIN connects two tables through a **shared key** -usually a primary key in one table and a foreign key in another.

```text
raw.customers          raw.orders
─────────────          ──────────
customer_id ◄──────── customer_id
first_name             order_id
last_name              order_date
email                  total_amount
city                   status
```

### INNER JOIN - only matching rows

Returns rows where there is a match in **both** tables. Rows with no match are excluded.

```sql
-- Show all orders with customer details
SELECT
    o.order_id,
    c.first_name || ' ' || c.last_name  AS customer_name,
    c.city,
    o.order_date,
    o.status,
    o.total_amount
FROM raw.orders o
INNER JOIN raw.customers c ON o.customer_id = c.customer_id
ORDER BY o.order_date DESC;
```

> **Table aliases** (`o` for orders, `c` for customers) make queries shorter and easier to read. Always use them when joining multiple tables.

### LEFT JOIN - all rows from the left table

Returns **all rows from the left table**, plus matching rows from the right. If there is no match, NULLs appear for the right table's columns.

```sql
-- Show ALL customers, including those who have never placed an order
SELECT
    c.customer_id,
    c.first_name || ' ' || c.last_name  AS customer_name,
    c.city,
    c.segment,
    COUNT(o.order_id)                   AS total_orders,
    COALESCE(SUM(o.total_amount), 0)    AS total_spent
FROM raw.customers c
LEFT JOIN raw.orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name, c.city, c.segment
ORDER BY total_spent DESC;
```

>  **LEFT JOIN is the most used JOIN in data engineering.** When building pipelines, you almost always want to preserve all records from the primary table and enrich them with data from secondary tables.

### RIGHT JOIN - all rows from the right table

Returns all rows from the right table, plus matching rows from the left. Less commonly used - most engineers rewrite these as LEFT JOINs for clarity.

```sql
-- Show all orders, even if the customer record is somehow missing
SELECT
    o.order_id,
    o.order_date,
    o.total_amount,
    c.first_name,
    c.city
FROM raw.customers c
RIGHT JOIN raw.orders o ON c.customer_id = o.customer_id;
```

### FULL OUTER JOIN - all rows from both tables

Returns all rows from both tables. NULLs appear wherever there is no match on either side. Useful for **reconciliation** — comparing two datasets.

```sql
-- Find mismatches between two versions of a customer table
SELECT
    a.customer_id   AS id_in_table_a,
    b.customer_id   AS id_in_table_b,
    a.email         AS email_a,
    b.email         AS email_b
FROM raw.customers a
FULL OUTER JOIN staging.customers b ON a.customer_id = b.customer_id
WHERE a.customer_id IS NULL OR b.customer_id IS NULL;
```

### Joining more than two tables

In real pipelines you often join three or four tables together.

```sql
-- Full order details: customer + order + items + product
SELECT
    c.first_name || ' ' || c.last_name  AS customer_name,
    c.city,
    o.order_id,
    o.order_date,
    o.status,
    p.product_name,
    p.category,
    oi.quantity,
    oi.unit_price,
    oi.subtotal
FROM raw.orders o
JOIN raw.customers  c  ON o.order_id    = o.order_id
                       AND c.customer_id = o.customer_id
JOIN raw.order_items oi ON oi.order_id  = o.order_id
JOIN raw.products    p  ON p.product_id = oi.product_id
ORDER BY o.order_date DESC, o.order_id;
```

---

## 4. `GROUP BY` - summarizing data

`GROUP BY` groups rows that share the same value in one or more columns, so you can apply aggregate functions to each group.

In data engineering, `GROUP BY` is used constantly to:
- Build summary tables that feed dashboards.
- Aggregate raw transactional data into daily, weekly, or monthly reports.
- Validate data - checking record counts per category or date.

### Basic GROUP BY

```sql
-- How many orders per status?
SELECT
    status,
    COUNT(*) AS order_count
FROM raw.orders
GROUP BY status
ORDER BY order_count DESC;
```

```sql
-- Total revenue per customer segment
SELECT
    c.segment,
    COUNT(DISTINCT o.order_id)  AS total_orders,
    SUM(o.total_amount)         AS total_revenue,
    ROUND(AVG(o.total_amount), 2) AS avg_order_value
FROM raw.orders o
JOIN raw.customers c ON o.customer_id = c.customer_id
GROUP BY c.segment
ORDER BY total_revenue DESC;
```

### GROUP BY with multiple columns

```sql
-- Revenue by city and status
SELECT
    c.city,
    o.status,
    COUNT(*)            AS order_count,
    SUM(o.total_amount) AS revenue
FROM raw.orders o
JOIN raw.customers c ON o.customer_id = c.customer_id
GROUP BY c.city, o.status
ORDER BY c.city, revenue DESC;
```

### GROUP BY with date truncation - time series reporting

This is one of the most important patterns in data engineering - aggregating data by time periods for dashboards and reports.

```sql
-- Daily revenue
SELECT
    order_date,
    COUNT(*)            AS orders_placed,
    SUM(total_amount)   AS daily_revenue
FROM raw.orders
WHERE status != 'cancelled'
GROUP BY order_date
ORDER BY order_date;

-- Monthly revenue
SELECT
    DATE_TRUNC('month', order_date)     AS month,
    COUNT(*)                            AS orders_placed,
    SUM(total_amount)                   AS monthly_revenue
FROM raw.orders
WHERE status != 'cancelled'
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;
```

> **DATE_TRUNC()** is extremely useful for time-series aggregations. It rounds a timestamp down to the nearest year, month, week, day, hour, etc. You will use this in almost every reporting pipeline.

### HAVING - filtering groups

`WHERE` filters rows **before** grouping. `HAVING` filters groups **after** grouping.

```sql
-- Show only customers who have spent more than 50,000 total
SELECT
    c.customer_id,
    c.first_name || ' ' || c.last_name  AS customer_name,
    SUM(o.total_amount)                 AS total_spent
FROM raw.orders o
JOIN raw.customers c ON o.customer_id = c.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
HAVING SUM(o.total_amount) > 50000
ORDER BY total_spent DESC;
```

```sql
-- Show product categories where average price exceeds 10,000
SELECT
    category,
    COUNT(*)                        AS product_count,
    ROUND(AVG(unit_price), 2)       AS avg_price
FROM raw.products
GROUP BY category
HAVING AVG(unit_price) > 10000
ORDER BY avg_price DESC;
```

| Clause | When it filters | Used for |
|---|---|---|
| `WHERE` | Before grouping | Filtering individual rows |
| `HAVING` | After grouping | Filtering aggregated results |

---

## 5. Putting it all together - real data engineering queries

These are the kinds of queries you will write in real pipelines and warehouse transformations.

### Query 1 - Customer lifetime value report

```sql
SELECT
    c.customer_id,
    c.first_name || ' ' || c.last_name          AS customer_name,
    c.city,
    c.segment,
    COUNT(DISTINCT o.order_id)                  AS total_orders,
    SUM(o.total_amount)                         AS lifetime_value,
    ROUND(AVG(o.total_amount), 2)               AS avg_order_value,
    MIN(o.order_date)                           AS first_order_date,
    MAX(o.order_date)                           AS last_order_date
FROM raw.customers c
LEFT JOIN raw.orders o
    ON c.customer_id = o.customer_id
    AND o.status != 'cancelled'
GROUP BY c.customer_id, c.first_name, c.last_name, c.city, c.segment
ORDER BY lifetime_value DESC NULLS LAST;
```

### Query 2 - Product sales performance

```sql
SELECT
    p.category,
    p.product_name,
    SUM(oi.quantity)                            AS units_sold,
    SUM(oi.subtotal)                            AS total_revenue,
    ROUND(AVG(oi.unit_price), 2)                AS avg_selling_price,
    p.stock_qty                                 AS remaining_stock
FROM raw.order_items oi
JOIN raw.products p ON oi.product_id = p.product_id
JOIN raw.orders o   ON oi.order_id   = o.order_id
WHERE o.status != 'cancelled'
GROUP BY p.product_id, p.category, p.product_name, p.stock_qty
ORDER BY total_revenue DESC;
```

### Query 3 - Daily pipeline validation query

```sql
-- Run this at the end of every pipeline to validate the day's load
SELECT
    order_date,
    COUNT(*)                                    AS records_loaded,
    COUNT(DISTINCT customer_id)                 AS unique_customers,
    SUM(total_amount)                           AS total_value,
    COUNT(*) FILTER (WHERE status IS NULL)      AS null_status_count,
    COUNT(*) FILTER (WHERE total_amount <= 0)   AS bad_amount_count
FROM raw.orders
WHERE order_date = CURRENT_DATE - INTERVAL '1 day'
GROUP BY order_date;
```

> **Pipeline validation queries** like this are something every data engineer runs after a load. You check that the right number of records arrived, no critical fields are NULL, and no suspicious values exist. If any check fails, you raise an alert before bad data reaches the warehouse.

---

## 6. The SQL query execution order

Understanding **how SQL actually runs** your query helps you write better queries and debug errors faster.

SQL does **not** execute in the order you write it. The actual execution order is:

```text
1. FROM          → which tables to read from
2. JOIN          → combine the tables
3. WHERE         → filter individual rows
4. GROUP BY      → group the remaining rows
5. HAVING        → filter the groups
6. SELECT        → choose which columns to return
7. DISTINCT      → remove duplicates
8. ORDER BY      → sort the results
9. LIMIT         → restrict the number of rows
```

This explains common beginner errors:

```sql
--  This fails - you cannot use a SELECT alias in WHERE
SELECT total_amount * 0.1 AS tax
FROM raw.orders
WHERE tax > 1000;            -- ERROR: "tax" does not exist yet

--  This works , repeat the expression in WHERE
SELECT total_amount * 0.1 AS tax
FROM raw.orders
WHERE total_amount * 0.1 > 1000;
```

---

## 7. Key terms to know from today

| Term | Meaning |
|---|---|
| `SELECT` | Retrieves data from one or more tables |
| `SELECT DISTINCT` | Retrieves unique values only |
| Alias (`AS`) | A temporary name given to a column or table in a query |
| `WHERE` | Filters rows before grouping or returning results |
| `IN` | Matches any value in a list |
| `BETWEEN` | Matches values within a range (inclusive) |
| `LIKE` / `ILIKE` | Pattern matching on strings |
| `IS NULL` / `IS NOT NULL` | Checks for the absence or presence of a value |
| `COALESCE()` | Returns the first non-NULL value |
| `INNER JOIN` | Returns only rows with a match in both tables |
| `LEFT JOIN` | Returns all rows from the left table, NULLs for unmatched right rows |
| `RIGHT JOIN` | Returns all rows from the right table |
| `FULL OUTER JOIN` | Returns all rows from both tables |
| `GROUP BY` | Groups rows by one or more columns for aggregation |
| `HAVING` | Filters groups after aggregation |
| `DATE_TRUNC()` | Truncates a timestamp to a specified time unit |
| `EXTRACT()` | Pulls a specific part (year, month, day) from a date |
| Incremental load | Loading only new or changed records using a date filter |
| Pipeline validation | Running checks after a data load to verify quality and completeness |

---

## 8. Summary

> `SELECT` retrieves exactly the columns you need. `WHERE` filters rows with precision. `JOIN` brings together data spread across multiple tables. `GROUP BY` summarizes raw transactional data into meaningful aggregations.
>
> Together these four SQL concepts form the backbone of almost every query a data engineer writes — whether extracting from a source database, transforming inside a warehouse, validating a pipeline load, or preparing data for a dashboard.
>
> Master these and you have the foundation for everything else in SQL.


*Notes by LuxDevHQ | Month 1 - Foundations of Data Engineering | Week 2, Day 1*