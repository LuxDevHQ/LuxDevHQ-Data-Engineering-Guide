# Week 2, Day 4 - Data Modeling: Normalization, Denormalization, and Star Schemas

> **Month 1: Foundations of Data Engineering**
> **Week 2: SQL for Data Engineering**
> **Thursday**

---

## What we covered yesterday (Day 3 recap)

On Wednesday we learned how to make SQL fast:

| Concept | What it does |
|---|---|
| `EXPLAIN` / `EXPLAIN ANALYZE` | Shows the execution plan and actual runtime of a query |
| Seq Scan vs Index Scan | Full table read vs targeted row lookup using an index |
| Indexes | B-Tree, composite, partial - speed up row lookups on large tables |
| Optimization techniques | Filter early, avoid functions on indexed columns, use EXISTS over IN |
| VACUUM / ANALYZE | Keeps table statistics fresh for the planner |
| Table partitioning | Splits large tables into smaller physical pieces for faster scans |

> **Today we zoom out from writing queries to designing the tables themselves.** How you structure your data - the shape of your tables, how they relate, and how they are organized - has a bigger impact on pipeline performance and analytics speed than any single query optimization.

---

## Today's objective

By the end of today, you should be able to:

- Explain what data modeling is and why it matters in data engineering.
- Understand the normal forms and apply normalization to eliminate data redundancy.
- Explain denormalization and when it is the right choice.
- Design a star schema for a data warehouse.
- Understand the difference between a star schema and a snowflake schema.
- Know when to use each modeling approach - OLTP vs OLAP.

---

## 1. What is data modeling?

**Data modeling** is the process of defining how data is structured, organized, and related within a database or data warehouse.

A data model answers:
- What **tables** exist?
- What **columns** does each table have?
- What **data types** are used?
- How do tables **relate** to each other?
- What **constraints** enforce data integrity?

### Why data modeling matters for data engineers

The data model you design directly affects:
- **Query performance** - a well-modeled warehouse runs analytics queries in seconds instead of minutes.
- **Pipeline complexity** - a clear model makes transformation logic straightforward.
- **Data quality** - good models prevent duplicate, inconsistent, or orphaned data.
- **Scalability** - the right model handles millions of rows without redesign.
- **Analyst experience** - analysts can self-serve without needing a data engineer to explain the structure every time.

### Two worlds of data modeling

| | OLTP | OLAP |
|---|---|---|
| **Full name** | Online Transaction Processing | Online Analytical Processing |
| **Purpose** | Running the business - recording transactions | Analyzing the business - reporting and insights |
| **Examples** | E-commerce orders, banking transactions, hospital records | Data warehouses, BI dashboards, sales reports |
| **Optimized for** | Fast writes - INSERT, UPDATE, DELETE | Fast reads - complex SELECT queries |
| **Model used** | Normalized (3NF) | Denormalized (star schema) |
| **Table count** | Many small tables | Fewer, wider tables |
| **Tools** | PostgreSQL, MySQL, SQL Server | BigQuery, Redshift, Synapse, Snowflake |

>  As a data engineer you work in **both** worlds. You extract from OLTP systems (source databases) and load into OLAP systems (data warehouses). Understanding both models tells you what you are pulling from and what you are building toward.

---

## 2. Normalization - designing clean source databases

**Normalization** is the process of organizing a database to reduce data redundancy and improve data integrity by dividing data into related tables and defining relationships between them.

### Simple analogy

Imagine a spreadsheet where every order row also contains the customer's full name, email, phone number, and address. If a customer changes their phone number, you have to update it in every single order row - and if you miss one, your data is inconsistent.

Normalization solves this by storing the customer's details **once** in a customers table and referencing it by ID in the orders table.

### The normal forms

#### First Normal Form (1NF) - eliminate repeating groups

Rules:
- Every column contains **atomic** (indivisible) values - no lists or arrays in a cell.
- Every row is **unique**.
- Every column has a **single data type**.

** Violates 1NF - multiple values in one cell:**

| order_id | customer_name | products |
|---|---|---|
| 1 | Alice Wanjiru | Laptop, Mouse, Headphones |
| 2 | Brian Otieno | Smartphone |

** Satisfies 1NF - one value per cell:**

| order_id | customer_name | product |
|---|---|---|
| 1 | Alice Wanjiru | Laptop |
| 1 | Alice Wanjiru | Mouse |
| 1 | Alice Wanjiru | Headphones |
| 2 | Brian Otieno | Smartphone |

#### Second Normal Form (2NF) - eliminate partial dependencies

Rules:
- Must already be in 1NF.
- Every non-key column must depend on the **entire** primary key - not just part of it.
- Applies when the primary key is composite (made of multiple columns).

** Violates 2NF - customer_name depends only on customer_id, not the full key:**

| order_id | customer_id | customer_name | product | quantity |
|---|---|---|---|---|
| 1 | 101 | Alice Wanjiru | Laptop | 1 |
| 1 | 101 | Alice Wanjiru | Mouse | 2 |
| 2 | 102 | Brian Otieno | Smartphone | 1 |

** Satisfies 2NF - split into separate tables:**

**customers table:**
| customer_id | customer_name |
|---|---|
| 101 | Alice Wanjiru |
| 102 | Brian Otieno |

**order_items table:**
| order_id | customer_id | product | quantity |
|---|---|---|---|
| 1 | 101 | Laptop | 1 |
| 1 | 101 | Mouse | 2 |
| 2 | 102 | Smartphone | 1 |

#### Third Normal Form (3NF) - eliminate transitive dependencies

Rules:
- Must already be in 2NF.
- No non-key column should depend on another non-key column.
- Every column must depend **only** on the primary key.

** Violates 3NF - city depends on zip_code, not on customer_id:**

| customer_id | customer_name | zip_code | city |
|---|---|---|---|
| 101 | Alice Wanjiru | 00100 | Nairobi |
| 102 | Brian Otieno | 40100 | Kisumu |
| 103 | Carol Muthoni | 00100 | Nairobi |

** Satisfies 3NF - extract zip_code and city to a separate table:**

**customers table:**
| customer_id | customer_name | zip_code |
|---|---|---|
| 101 | Alice Wanjiru | 00100 |
| 102 | Brian Otieno | 40100 |
| 103 | Carol Muthoni | 00100 |

**locations table:**
| zip_code | city |
|---|---|
| 00100 | Nairobi |
| 40100 | Kisumu |

### A fully normalized schema (3NF) in SQL

```sql
-- Normalized OLTP schema for our e-commerce system

CREATE TABLE raw.locations (
    zip_code        VARCHAR(10) PRIMARY KEY,
    city            VARCHAR(100),
    country         VARCHAR(100)
);

CREATE TABLE raw.customers (
    customer_id     SERIAL PRIMARY KEY,
    first_name      VARCHAR(100),
    last_name       VARCHAR(100),
    email           VARCHAR(255) UNIQUE NOT NULL,
    phone           VARCHAR(20),
    zip_code        VARCHAR(10) REFERENCES raw.locations(zip_code),
    segment         VARCHAR(50),
    created_at      TIMESTAMP DEFAULT NOW()
);

CREATE TABLE raw.categories (
    category_id     SERIAL PRIMARY KEY,
    category_name   VARCHAR(100),
    parent_id       INTEGER REFERENCES raw.categories(category_id)
);

CREATE TABLE raw.products (
    product_id      SERIAL PRIMARY KEY,
    product_name    VARCHAR(255),
    category_id     INTEGER REFERENCES raw.categories(category_id),
    unit_price      NUMERIC(10, 2),
    stock_qty       INTEGER
);

CREATE TABLE raw.orders (
    order_id        SERIAL PRIMARY KEY,
    customer_id     INTEGER REFERENCES raw.customers(customer_id),
    order_date      DATE,
    status          VARCHAR(50),
    created_at      TIMESTAMP DEFAULT NOW()
);

CREATE TABLE raw.order_items (
    item_id         SERIAL PRIMARY KEY,
    order_id        INTEGER REFERENCES raw.orders(order_id),
    product_id      INTEGER REFERENCES raw.products(product_id),
    quantity        INTEGER,
    unit_price      NUMERIC(10, 2),
    subtotal        NUMERIC(10, 2)
);
```

### Benefits and trade-offs of normalization

| Benefit | Trade-off |
|---|---|
| No data redundancy - update in one place | Queries need many JOINs |
| Data integrity - no inconsistent values | More complex queries |
| Storage efficient - no repeated data | Can be slower for analytics |
| Easy to update, insert, and delete | Not ideal for data warehouses |

>  **Normalization is the right choice for OLTP systems** - your source databases where the application writes transactions. It is NOT the right choice for data warehouses where analysts run complex analytical queries across millions of rows.

---

## 3. Denormalization - designing fast analytical tables

**Denormalization** is the deliberate process of combining normalized tables into fewer, wider tables by introducing controlled redundancy - trading storage efficiency for query speed.

In a data warehouse, analysts need to answer questions like:
- "What was the total revenue by product category and customer city last month?"

In a fully normalized schema, this requires joining 5+ tables. In a denormalized warehouse table, it is a single table scan with a GROUP BY.

### Denormalization in practice

```sql
-- A denormalized orders fact table for the data warehouse
-- All the information an analyst needs in one wide table

CREATE TABLE analytics.orders_flat (
    order_id            INTEGER,
    order_date          DATE,
    order_month         VARCHAR(20),
    order_year          INTEGER,
    order_status        VARCHAR(50),

    -- Customer details (denormalized from customers + locations)
    customer_id         INTEGER,
    customer_name       VARCHAR(200),
    customer_email      VARCHAR(255),
    customer_city       VARCHAR(100),
    customer_country    VARCHAR(100),
    customer_segment    VARCHAR(50),

    -- Product details (denormalized from products + categories)
    product_id          INTEGER,
    product_name        VARCHAR(255),
    category_name       VARCHAR(100),

    -- Measures
    quantity            INTEGER,
    unit_price          NUMERIC(10, 2),
    subtotal            NUMERIC(10, 2)
);
```

```sql
-- Populate the denormalized table using a transformation query
INSERT INTO analytics.orders_flat
SELECT
    o.order_id,
    o.order_date,
    TO_CHAR(o.order_date, 'Month YYYY')     AS order_month,
    EXTRACT(YEAR FROM o.order_date)         AS order_year,
    o.status                                AS order_status,
    c.customer_id,
    c.first_name || ' ' || c.last_name     AS customer_name,
    c.email                                 AS customer_email,
    l.city                                  AS customer_city,
    l.country                               AS customer_country,
    c.segment                               AS customer_segment,
    p.product_id,
    p.product_name,
    cat.category_name,
    oi.quantity,
    oi.unit_price,
    oi.subtotal
FROM raw.orders o
JOIN raw.customers   c   ON o.customer_id  = c.customer_id
JOIN raw.locations   l   ON c.zip_code     = l.zip_code
JOIN raw.order_items oi  ON o.order_id     = oi.order_id
JOIN raw.products    p   ON oi.product_id  = p.product_id
JOIN raw.categories  cat ON p.category_id  = cat.category_id
WHERE o.status != 'cancelled';
```

Now an analyst can answer complex questions with simple queries:

```sql
-- Revenue by category and city - no JOINs needed
SELECT
    category_name,
    customer_city,
    SUM(subtotal)       AS total_revenue,
    COUNT(DISTINCT order_id) AS total_orders
FROM analytics.orders_flat
WHERE order_year = 2025
GROUP BY category_name, customer_city
ORDER BY total_revenue DESC;
```

---

## 4. Star schema - the standard data warehouse model

The **star schema** is the most widely used data modeling pattern for data warehouses and analytics systems. It is a denormalized structure organized around two types of tables:

- **Fact tables** - store measurable business events (sales, transactions, clicks, payments).
- **Dimension tables** - store descriptive context about those events (who, what, where, when).

The name comes from the shape: one central fact table surrounded by dimension tables - like a star.

```text
                    ┌─────────────────┐
                    │  dim_date       │
                    │─────────────────│
                    │ date_id (PK)    │
                    │ full_date       │
                    │ day_of_week     │
                    │ month           │
                    │ quarter         │
                    │ year            │
                    └────────┬────────┘
                             │
  ┌──────────────┐           │           ┌──────────────────┐
  │ dim_customer │           │           │  dim_product     │
  │──────────────│           │           │──────────────────│
  │ customer_id  │           │           │ product_id (PK)  │
  │ customer_name│           │           │ product_name     │
  │ city         │           │           │ category         │
  │ country      │           │           │ unit_price       │
  │ segment      │           │           └────────┬─────────┘
  └──────┬───────┘           │                    │
         │                   │                    │
         │          ┌────────┴──────────┐         │
         └─────────►│   fact_orders     │◄────────┘
                    │───────────────────│
                    │ order_id (PK)     │
                    │ date_id (FK)      │
                    │ customer_id (FK)  │
                    │ product_id (FK)   │
                    │ quantity          │
                    │ unit_price        │
                    │ subtotal          │
                    │ total_amount      │
                    └───────────────────┘
```

### Fact tables

A **fact table** records individual business events - one row per transaction, one row per order item, one row per click.

Fact tables contain:
- **Foreign keys** - linking to each dimension table.
- **Measures** - the numeric values you want to analyze (revenue, quantity, duration, count).

```sql
-- Create the fact table
CREATE TABLE analytics.fact_orders (
    order_id        INTEGER,
    date_id         INTEGER,        -- FK to dim_date
    customer_id     INTEGER,        -- FK to dim_customer
    product_id      INTEGER,        -- FK to dim_product

    -- Measures
    quantity        INTEGER,
    unit_price      NUMERIC(10, 2),
    subtotal        NUMERIC(10, 2),
    discount_amt    NUMERIC(10, 2)  DEFAULT 0
);
```

### Dimension tables

**Dimension tables** provide the descriptive context - the who, what, where, and when of each fact.

```sql
-- Date dimension - one of the most important dimensions in any warehouse
CREATE TABLE analytics.dim_date (
    date_id         INTEGER PRIMARY KEY,    -- e.g. 20250501
    full_date       DATE,
    day_of_week     VARCHAR(10),            -- 'Monday', 'Tuesday', etc.
    day_num         INTEGER,                -- 1-7
    week_num        INTEGER,                -- 1-52
    month_num       INTEGER,                -- 1-12
    month_name      VARCHAR(10),            -- 'January', etc.
    quarter         INTEGER,                -- 1-4
    year            INTEGER,
    is_weekend      BOOLEAN,
    is_public_holiday BOOLEAN DEFAULT FALSE
);

-- Customer dimension
CREATE TABLE analytics.dim_customer (
    customer_id     INTEGER PRIMARY KEY,
    customer_name   VARCHAR(200),
    email           VARCHAR(255),
    city            VARCHAR(100),
    country         VARCHAR(100),
    segment         VARCHAR(50),
    -- SCD fields (slowly changing dimension)
    valid_from      DATE,
    valid_to        DATE,
    is_current      BOOLEAN DEFAULT TRUE
);

-- Product dimension
CREATE TABLE analytics.dim_product (
    product_id      INTEGER PRIMARY KEY,
    product_name    VARCHAR(255),
    category        VARCHAR(100),
    sub_category    VARCHAR(100),
    unit_price      NUMERIC(10, 2)
);
```

### Populating the date dimension

The date dimension is unique - it is not sourced from a transaction system. You generate it once and it covers all time periods.

```sql
-- Generate a date dimension for 2024-2026
INSERT INTO analytics.dim_date (
    date_id, full_date, day_of_week, day_num, week_num,
    month_num, month_name, quarter, year, is_weekend
)
SELECT
    TO_CHAR(d, 'YYYYMMDD')::INTEGER         AS date_id,
    d                                        AS full_date,
    TO_CHAR(d, 'Day')                        AS day_of_week,
    EXTRACT(DOW FROM d)                      AS day_num,
    EXTRACT(WEEK FROM d)                     AS week_num,
    EXTRACT(MONTH FROM d)                    AS month_num,
    TO_CHAR(d, 'Month')                      AS month_name,
    EXTRACT(QUARTER FROM d)                  AS quarter,
    EXTRACT(YEAR FROM d)                     AS year,
    EXTRACT(DOW FROM d) IN (0, 6)            AS is_weekend
FROM GENERATE_SERIES(
    '2024-01-01'::DATE,
    '2026-12-31'::DATE,
    '1 day'::INTERVAL
) AS d;
```

### Querying the star schema

With a star schema in place, analytical queries become clean and fast:

```sql
-- Monthly revenue by product category
SELECT
    dd.month_name,
    dd.year,
    dp.category,
    SUM(fo.subtotal)            AS total_revenue,
    SUM(fo.quantity)            AS units_sold,
    COUNT(DISTINCT fo.order_id) AS total_orders
FROM analytics.fact_orders fo
JOIN analytics.dim_date     dd ON fo.date_id     = dd.date_id
JOIN analytics.dim_product  dp ON fo.product_id  = dp.product_id
WHERE dd.year = 2025
GROUP BY dd.month_name, dd.year, dd.month_num, dp.category
ORDER BY dd.month_num, total_revenue DESC;
```

```sql
-- Top customers by segment and city
SELECT
    dc.segment,
    dc.city,
    dc.customer_name,
    SUM(fo.subtotal)            AS lifetime_value,
    COUNT(DISTINCT fo.order_id) AS total_orders
FROM analytics.fact_orders fo
JOIN analytics.dim_customer dc ON fo.customer_id = dc.customer_id
JOIN analytics.dim_date     dd ON fo.date_id      = dd.date_id
WHERE dd.year = 2025
GROUP BY dc.segment, dc.city, dc.customer_name
ORDER BY lifetime_value DESC
LIMIT 20;
```

---

## 5. Snowflake schema - normalized dimensions

A **snowflake schema** is an extension of the star schema where dimension tables are further normalized into sub-dimension tables.

```text
Star schema:                    Snowflake schema:

fact_orders                     fact_orders
    → dim_product                   → dim_product
         (category inside)               → dim_category
                                              → dim_category_group
```

```sql
-- Snowflake: product and category are separate tables
CREATE TABLE analytics.dim_category (
    category_id     INTEGER PRIMARY KEY,
    category_name   VARCHAR(100),
    category_group  VARCHAR(100)
);

CREATE TABLE analytics.dim_product_snowflake (
    product_id      INTEGER PRIMARY KEY,
    product_name    VARCHAR(255),
    category_id     INTEGER REFERENCES analytics.dim_category(category_id),
    unit_price      NUMERIC(10, 2)
);
```

### Star schema vs snowflake schema

| | Star schema | Snowflake schema |
|---|---|---|
| **Dimension structure** | Denormalized - flat | Normalized - split into sub-tables |
| **Query complexity** | Simple - fewer JOINs | More complex - more JOINs |
| **Query speed** | Faster for analytics | Slightly slower due to extra JOINs |
| **Storage** | Uses more space (redundancy) | Uses less space |
| **Maintenance** | Easier - update one dimension table | Harder - changes cascade through sub-tables |
| **Best for** | Most data warehouse use cases | Large dimensions with high redundancy |
| **Used in** | BigQuery, Redshift, most BI tools | Snowflake (the platform), some enterprise DWs |

> **In practice, star schema is the default choice** for data warehouses. It is simpler, faster for analytics, and works better with BI tools like Power BI, Looker, and Tableau. Most modern cloud data warehouses are designed around the star schema pattern.

---

## 6. Slowly Changing Dimensions (SCD)

A **Slowly Changing Dimension (SCD)** handles how dimension data changes over time. For example - a customer moves from Nairobi to Mombasa. What do you store in `dim_customer`?

### SCD Type 1 - Overwrite (no history)

```sql
-- Simply update the record - history is lost
UPDATE analytics.dim_customer
SET city = 'Mombasa'
WHERE customer_id = 101;
```

Use when: history does not matter and you always want the current value.

### SCD Type 2 - Add a new row (full history)

```sql
-- Close the old record
UPDATE analytics.dim_customer
SET valid_to = CURRENT_DATE - 1,
    is_current = FALSE
WHERE customer_id = 101
AND is_current = TRUE;

-- Insert the new record
INSERT INTO analytics.dim_customer
    (customer_id, customer_name, email, city, country, segment, valid_from, valid_to, is_current)
VALUES
    (101, 'Alice Wanjiru', 'alice@mail.com', 'Mombasa', 'Kenya', 'online',
     CURRENT_DATE, '9999-12-31', TRUE);
```

Use when: you need to know what the dimension value was **at the time of the transaction**. For example - was Alice in Nairobi or Mombasa when she placed that order in March?

```sql
-- Query that respects SCD Type 2 history
SELECT
    fo.order_id,
    dd.full_date,
    dc.customer_name,
    dc.city                     AS city_at_time_of_order,
    fo.subtotal
FROM analytics.fact_orders fo
JOIN analytics.dim_date     dd ON fo.date_id     = dd.date_id
JOIN analytics.dim_customer dc ON fo.customer_id = dc.customer_id
                               AND dd.full_date BETWEEN dc.valid_from AND dc.valid_to;
```

### SCD Type 3 - Add a column (limited history)

```sql
-- Store both current and previous value in the same row
ALTER TABLE analytics.dim_customer
ADD COLUMN previous_city VARCHAR(100);

UPDATE analytics.dim_customer
SET previous_city = city,
    city = 'Mombasa'
WHERE customer_id = 101;
```

Use when: you only need to track one previous value - current and one step back.

---

## 7. How data modeling connects to the full pipeline

```text
Source Systems (OLTP)               Data Warehouse (OLAP)
─────────────────────               ─────────────────────
Normalized (3NF)                    Star Schema
Many small tables                   Fact + Dimension tables
Optimized for writes                Optimized for reads
PostgreSQL / MySQL / SQL Server      BigQuery / Redshift / Synapse

         ↓  Extract (SQL SELECT)
         ↓  Transform (CTEs, window functions, dbt)
         ↓  Load (INSERT INTO fact and dim tables)

Analytics Layer
───────────────
Views / Materialized Views
Power BI / Looker / Tableau
```

As a data engineer, your job is to:
1. **Understand** the normalized source schema (OLTP).
2. **Design** the star schema in the warehouse (OLAP).
3. **Build** the ETL/ELT pipeline that transforms one into the other.
4. **Maintain** dimension history with SCD patterns.
5. **Expose** clean views and aggregated tables for analysts.

---

## 8. Key terms to know from today

| Term | Meaning |
|---|---|
| Data modeling | Defining how data is structured, organized, and related in a database |
| OLTP | Online Transaction Processing - systems that record business transactions |
| OLAP | Online Analytical Processing - systems that analyze business data |
| Normalization | Organizing tables to reduce redundancy and improve integrity |
| 1NF | First Normal Form - atomic values, no repeating groups |
| 2NF | Second Normal Form - no partial dependencies on a composite key |
| 3NF | Third Normal Form - no transitive dependencies between non-key columns |
| Denormalization | Combining tables into wider ones for faster analytical queries |
| Star schema | A warehouse model with one central fact table and surrounding dimension tables |
| Fact table | Stores measurable business events - transactions, orders, clicks |
| Dimension table | Stores descriptive context - who, what, where, when |
| Measures | Numeric values in a fact table that can be aggregated - revenue, quantity |
| Grain | The level of detail each row in a fact table represents |
| Snowflake schema | A star schema where dimension tables are further normalized |
| Date dimension | A pre-generated dimension table covering every date in a time range |
| SCD | Slowly Changing Dimension - how to handle changes to dimension data over time |
| SCD Type 1 | Overwrite - update the record, lose history |
| SCD Type 2 | Add a new row - preserve full history with valid_from / valid_to dates |
| SCD Type 3 | Add a column - store current and one previous value in the same row |
| Surrogate key | A system-generated primary key (e.g. SERIAL) used in dimension tables |

---

## 9. Summary

> **Data modeling** is the foundation everything else sits on. A well-designed model makes pipelines simpler, queries faster, and analysts more self-sufficient.
>
> **Normalization** (3NF) is the right approach for OLTP source systems - it eliminates redundancy, enforces integrity, and makes writes fast. **Denormalization** and the **star schema** are the right approach for OLAP data warehouses - they reduce JOINs, speed up analytics, and make BI tools work naturally.
>
> As a data engineer, you read from normalized sources and write to denormalized warehouses. The transformation in between - from 3NF to star schema - is one of the core jobs you will do every day.

---

## 10. What's coming next

| Day | Topic |
|---|---|
| Friday | Peer project - SQL and data modeling challenge |
| Saturday (Lab) | Mini project - design and build a star schema from a normalized source |
| Week 3 | Python for data engineering - scripting, automation, and pipeline code |

---

*Notes by LuxDevHQ | Month 1 - Foundations of Data Engineering | Week 2, Day 4*