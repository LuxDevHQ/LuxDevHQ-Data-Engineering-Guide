# Week 1, Day 4 - Introduction to SQL for Data Engineering and PostgreSQL Setup

> **Month 1: Foundations of Data Engineering**
> **Week 1: Onboarding and Environment Setup**
> **Thursday**

---

## What we covered yesterday (Day 3 recap)

On Day 3 we learned about **data governance, security, compliance, and access control** - the rules, technical controls, and legal requirements that govern how data is managed and protected in cloud systems.

We learned that data engineers are directly responsible for building systems that comply with laws like **GDPR**, **HIPAA**, and **Kenya's Data Protection Act**, and that tools like **Microsoft Purview**, **AWS IAM**, and **Cloud Audit Logs** help enforce and track data governance policies.

> **Quick recap definition:**
> Data governance is the rulebook. Data security is the locks and guards. Compliance is the inspector who checks that your locks meet the law.

---

## Today's objective

By the end of today, you should be able to:

- Explain what SQL is and why it is essential for data engineering.
- Understand the difference between SQL and PostgreSQL.
- Install and set up PostgreSQL on your local machine.
- Connect to PostgreSQL using psql and pgAdmin.
- Write and run basic SQL queries for data engineering tasks.
- Understand the most important SQL concepts used in pipelines and data warehouses.

---

## 1. What is SQL?

**SQL** stands for **Structured Query Language**. It is the standard language used to communicate with relational databases.

SQL allows you to:
- **Create** tables and databases.
- **Insert** data into tables.
- **Query** (retrieve) data from tables.
- **Update** existing data.
- **Delete** data.
- **Join** multiple tables together.
- **Aggregate** data ; count, sum, average, and group it.

### Simple analogy

Think of a database as a very large, organized Excel workbook.

- Each **sheet** in the workbook is a **table** in the database.
- Each **column** is a **field** (like Customer Name, Order Amount, Date).
- Each **row** is a **record** (one customer, one order, one transaction).

SQL is the language you use to ask questions about that workbook - but at a scale Excel cannot handle. SQL databases can manage **millions and billions of rows** efficiently.

### Why SQL is essential for data engineers

As a data engineer, SQL is one of the most important tools in your toolkit. You will use it to:

- Query source databases to understand the data before building pipelines.
- Write transformation logic inside data warehouses (Redshift, Synapse, BigQuery).
- Validate data quality - checking for nulls, duplicates, and anomalies.
- Build views and aggregations that analysts and dashboards consume.
- Debug pipelines by querying intermediate and final data.
- Work with tools like **dbt**, which uses SQL to transform data inside warehouses.

> **SQL is not just for analysts.** Data engineers write SQL every single day - in pipelines, in warehouses, in data quality checks, and in transformation logic.

---

## 2. SQL vs PostgreSQL - what is the difference?

This is a common question for beginners.

| | SQL | PostgreSQL |
|---|---|---|
| **What it is** | A language | A database system |
| **Purpose** | Used to communicate with databases | A specific database that understands SQL |
| **Analogy** | English (the language) | A person who speaks English |

**SQL** is the language. **PostgreSQL** is one of many database systems that use SQL as their language.

Other database systems that use SQL include:
- **MySQL**
- **Microsoft SQL Server**
- **SQLite**
- **Amazon Redshift** (uses PostgreSQL-compatible SQL)
- **Google BigQuery** (uses standard SQL)
- **Azure Synapse Analytics** (uses T-SQL)

### Why PostgreSQL specifically?

PostgreSQL (often called **Postgres**) is one of the most powerful and widely used open-source relational databases in the world. Data engineers use it because:

- It is **free and open-source**.
- It is **highly reliable** and used in production systems worldwide.
- It supports **advanced data types** - JSON, arrays, UUID, and more.
- **Amazon RDS, Aurora, Azure Database, and Cloud SQL** all offer managed PostgreSQL — so learning it locally translates directly to cloud work.
- Many data engineering pipelines use PostgreSQL as a **source database** - the starting point of a pipeline.

---

## 3. Core SQL concepts for data engineering

Before writing queries, understand these fundamental concepts.

### A. Tables, rows, and columns

A **table** is where data is stored in a database. It has:
- **Columns** - the fields/attributes (e.g. customer_id, name, email, created_at).
- **Rows** - the individual records (one row = one customer, one order, one event).

### B. Data types

Every column in a table has a data type that defines what kind of data it can hold.

| Data type | What it stores | Example |
|---|---|---|
| `INTEGER` / `INT` | Whole numbers | 1, 42, 1000 |
| `BIGINT` | Very large whole numbers | 9823748237 |
| `NUMERIC` / `DECIMAL` | Numbers with decimals | 199.99, 3.14 |
| `VARCHAR(n)` | Text up to n characters | 'Nairobi', 'John Doe' |
| `TEXT` | Unlimited text | Long descriptions |
| `BOOLEAN` | True or false | TRUE, FALSE |
| `DATE` | A date | 2025-06-08 |
| `TIMESTAMP` | Date and time | 2025-06-08 14:30:00 |
| `UUID` | Unique identifier | a3f2c1d4-... |
| `JSON` / `JSONB` | JSON data | {"key": "value"} |

### C. Primary keys and foreign keys

| Concept | What it means | Example |
|---|---|---|
| **Primary key** | A unique identifier for each row in a table | `customer_id` in the customers table |
| **Foreign key** | A column that references the primary key of another table | `customer_id` in the orders table (linking back to customers) |

This is how tables **relate** to each other - which is why these are called **relational** databases.

### D. Schemas

A **schema** is a namespace inside a database - a way of organizing tables into logical groups.

```text
Database: retail_db
  ├── Schema: raw        → raw ingested data (from pipelines)
  ├── Schema: staging    → cleaned and validated data
  └── Schema: analytics  → final tables used by dashboards
```

This pattern is very common in data engineering - you will use schemas to separate raw, staging, and analytics layers inside a single database or warehouse.

---

## 4. Setting up PostgreSQL

### Step 1: Download and install PostgreSQL

Go to the official PostgreSQL download page:
```
https://www.postgresql.org/download/
```

Choose your operating system:
- **Windows** - download the installer from EDB (EnterpriseDB).
- **Mac** -use the installer or Homebrew: `brew install postgresql`.
- **Linux (Ubuntu)** - use apt: `sudo apt install postgresql`.

During installation on Windows:
- Set a **password** for the default `postgres` superuser - remember this password.
- Keep the default port: **5432**.
- Install **pgAdmin 4** when offered - it is the graphical interface for PostgreSQL.

### Step 2: Verify PostgreSQL is running

Open your terminal (Git Bash on Windows, Terminal on Mac/Linux) and run:

```bash
psql --version
```

You should see something like:
```
psql (PostgreSQL) 16.x
```

### Step 3: Connect to PostgreSQL using psql

**psql** is the command-line tool for PostgreSQL. Connect as the default superuser:

```bash
psql -U postgres
```

Enter your password when prompted. You will see the psql prompt:

```
postgres=#
```

You are now inside PostgreSQL and can run SQL commands.

### Step 4: Connect using pgAdmin (graphical interface)

**pgAdmin 4** is the graphical interface for PostgreSQL - useful for beginners and for visualizing tables.

1. Open **pgAdmin 4** from your applications.
2. In the left panel, click **Servers → PostgreSQL**.
3. Enter the password you set during installation.
4. You are now connected and can see databases, tables, and run queries visually.

---

## 5. Essential psql commands to know

These are commands you run inside the psql prompt (they start with `\`):

| Command | What it does |
|---|---|
| `\l` | List all databases |
| `\c database_name` | Connect to a specific database |
| `\dt` | List all tables in the current database |
| `\d table_name` | Describe a table (show its columns and types) |
| `\dn` | List all schemas |
| `\du` | List all users and roles |
| `\q` | Quit psql |
| `\i file.sql` | Run a SQL file |

---

## 6. Core SQL queries for data engineering

### A. Creating a database and connecting to it

```sql
-- Create a new database
CREATE DATABASE retail_db;

-- Connect to it in psql
\c retail_db
```

### B. Creating a schema

```sql
-- Create schemas for raw, staging, and analytics layers
CREATE SCHEMA raw;
CREATE SCHEMA staging;
CREATE SCHEMA analytics;
```

### C. Creating a table

```sql
-- Create a customers table in the raw schema
CREATE TABLE raw.customers (
    customer_id   SERIAL PRIMARY KEY,
    first_name    VARCHAR(100),
    last_name     VARCHAR(100),
    email         VARCHAR(255) UNIQUE,
    phone         VARCHAR(20),
    country       VARCHAR(100),
    created_at    TIMESTAMP DEFAULT NOW()
);
```

```sql
-- Create an orders table with a foreign key to customers
CREATE TABLE raw.orders (
    order_id      SERIAL PRIMARY KEY,
    customer_id   INTEGER REFERENCES raw.customers(customer_id),
    product_name  VARCHAR(255),
    quantity      INTEGER,
    unit_price    NUMERIC(10, 2),
    total_amount  NUMERIC(10, 2),
    order_date    DATE,
    status        VARCHAR(50)
);
```

### D. Inserting data

```sql
-- Insert customers
INSERT INTO raw.customers (first_name, last_name, email, phone, country)
VALUES
    ('Alice', 'Wanjiru', 'alice@email.com', '+254700000001', 'Kenya'),
    ('Brian', 'Otieno', 'brian@email.com', '+254700000002', 'Kenya'),
    ('Carol', 'Muthoni', 'carol@email.com', '+254700000003', 'Kenya');

-- Insert orders
INSERT INTO raw.orders (customer_id, product_name, quantity, unit_price, total_amount, order_date, status)
VALUES
    (1, 'Laptop', 1, 85000.00, 85000.00, '2025-06-01', 'delivered'),
    (2, 'Headphones', 2, 3500.00, 7000.00, '2025-06-02', 'delivered'),
    (1, 'Mouse', 1, 1200.00, 1200.00, '2025-06-03', 'pending'),
    (3, 'Keyboard', 1, 2500.00, 2500.00, '2025-06-04', 'shipped');
```

### E. Querying data (SELECT)

```sql
-- Get all customers
SELECT * FROM raw.customers;

-- Get specific columns
SELECT first_name, last_name, email, country
FROM raw.customers;

-- Filter with WHERE
SELECT * FROM raw.orders
WHERE status = 'delivered';

-- Filter with multiple conditions
SELECT * FROM raw.orders
WHERE status = 'delivered'
AND order_date >= '2025-06-01';
```

### F. Sorting and limiting results

```sql
-- Sort orders by total amount (highest first)
SELECT * FROM raw.orders
ORDER BY total_amount DESC;

-- Get the 3 most recent orders
SELECT * FROM raw.orders
ORDER BY order_date DESC
LIMIT 3;
```

### G. Aggregations: COUNT, SUM, AVG, MIN, MAX

```sql
-- Count total number of orders
SELECT COUNT(*) AS total_orders
FROM raw.orders;

-- Total revenue
SELECT SUM(total_amount) AS total_revenue
FROM raw.orders;

-- Average order value
SELECT ROUND(AVG(total_amount), 2) AS avg_order_value
FROM raw.orders;

-- Revenue by status
SELECT status, COUNT(*) AS order_count, SUM(total_amount) AS revenue
FROM raw.orders
GROUP BY status
ORDER BY revenue DESC;
```

### H. JOIN: combining tables

This is one of the most important SQL skills for data engineers.

```sql
-- Join customers and orders to see who ordered what
SELECT
    c.first_name,
    c.last_name,
    c.country,
    o.product_name,
    o.total_amount,
    o.order_date,
    o.status
FROM raw.orders o
JOIN raw.customers c ON o.customer_id = c.customer_id
ORDER BY o.order_date DESC;
```

### Types of JOINs

| Join type | What it returns |
|---|---|
| `INNER JOIN` | Only rows that have a match in both tables |
| `LEFT JOIN` | All rows from the left table, and matching rows from the right (NULLs if no match) |
| `RIGHT JOIN` | All rows from the right table, and matching rows from the left |
| `FULL OUTER JOIN` | All rows from both tables (NULLs where there is no match) |

```sql
-- LEFT JOIN example: show all customers, even those with no orders
SELECT
    c.first_name,
    c.last_name,
    COUNT(o.order_id) AS total_orders,
    COALESCE(SUM(o.total_amount), 0) AS total_spent
FROM raw.customers c
LEFT JOIN raw.orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total_spent DESC;
```

### I. Data quality checks - used constantly in pipelines

```sql
-- Check for NULL values in critical columns
SELECT COUNT(*) AS null_emails
FROM raw.customers
WHERE email IS NULL;

-- Check for duplicate emails
SELECT email, COUNT(*) AS count
FROM raw.customers
GROUP BY email
HAVING COUNT(*) > 1;

-- Check for negative or zero prices (data quality issue)
SELECT * FROM raw.orders
WHERE unit_price <= 0;

-- Check the date range of the data
SELECT MIN(order_date) AS earliest, MAX(order_date) AS latest
FROM raw.orders;
```

### J. Creating views - reusable queries

```sql
-- Create a view in the analytics schema for the dashboard
CREATE VIEW analytics.customer_order_summary AS
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    c.country,
    COUNT(o.order_id)          AS total_orders,
    SUM(o.total_amount)        AS total_spent,
    MAX(o.order_date)          AS last_order_date
FROM raw.customers c
LEFT JOIN raw.orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name, c.country;

-- Query the view
SELECT * FROM analytics.customer_order_summary
ORDER BY total_spent DESC;
```

>  **Views in data engineering:** Views are like saved queries. Instead of writing a complex JOIN every time, you save it as a view and query it like a table. This is heavily used in data warehouses and BI layers.

---

## 7. How SQL fits into a data engineering pipeline

```text
Source Database (PostgreSQL)
        ↓
Extract with SQL SELECT queries
        ↓
Load into Cloud Storage (S3 / Data Lake / GCS)
        ↓
Transform with SQL (dbt / Glue / Synapse / BigQuery)
        ↓
Load into Data Warehouse (Redshift / Synapse / BigQuery)
        ↓
Query with SQL → Power BI / Looker dashboards
```

SQL is used at **multiple stages** of every pipeline:
- **Extraction** - querying the source database.
- **Transformation** - cleaning, joining, and reshaping data.
- **Validation** - running data quality checks.
- **Analytics** - querying the final warehouse for reports.

---

## 8. Common SQL mistakes beginners make

| Mistake | Why it matters | Fix |
|---|---|---|
| `SELECT *` in production | Fetches all columns - slow and expensive on large tables | Always specify only the columns you need |
| No `WHERE` on `DELETE` or `UPDATE` | Deletes or updates every row in the table | Always double-check your WHERE clause first |
| Forgetting `GROUP BY` columns | Query fails or gives wrong results | Every non-aggregated column in SELECT must be in GROUP BY |
| Not handling NULLs | NULLs cause unexpected results in aggregations and JOINs | Use `IS NULL`, `IS NOT NULL`, and `COALESCE()` |
| Joining on the wrong column | Returns incorrect or duplicate rows | Always verify foreign key relationships before joining |

---

## 9. Key terms to know from today

| Term | Meaning |
|---|---|
| SQL | Structured Query Language - the language used to communicate with relational databases |
| PostgreSQL | A powerful open-source relational database system that uses SQL |
| Table | A structured collection of data organized in rows and columns |
| Schema | A namespace that organizes tables into logical groups within a database |
| Primary key | A unique identifier for each row in a table |
| Foreign key | A column that references the primary key of another table |
| `SELECT` | Retrieve data from a table |
| `WHERE` | Filter rows based on a condition |
| `GROUP BY` | Group rows for aggregation |
| `JOIN` | Combine rows from two or more tables based on a related column |
| `ORDER BY` | Sort results |
| `LIMIT` | Restrict the number of rows returned |
| Aggregation | Computing summary values — COUNT, SUM, AVG, MIN, MAX |
| View | A saved SQL query that can be queried like a table |
| NULL | The absence of a value in a column |
| `COALESCE()` | Returns the first non-NULL value from a list |
| psql | The command-line interface for PostgreSQL |
| pgAdmin | The graphical interface for PostgreSQL |

---

## 10. Summary

> **SQL** is the most important language in data engineering. It is used to query source databases, transform data inside warehouses, run data quality checks, and feed analytics dashboards.
>
> **PostgreSQL** is one of the most widely used open-source relational databases in the world — and the one we will use throughout this program. Understanding how to create tables, insert data, write queries, join tables, and check data quality in PostgreSQL is a foundational skill that applies directly to cloud platforms like Amazon RDS, Azure Database, and Google Cloud SQL.



*Notes by LuxDevHQ | Month 1 - Foundations of Data Engineering | Week 1, Day 4*