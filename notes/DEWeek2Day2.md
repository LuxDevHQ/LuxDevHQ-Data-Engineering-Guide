# Week 2, Day 2 - Advanced SQL Techniques: Recursive Queries, Window Functions, Views, Stored Procedures, Subqueries, and CTEs

> **Month 1: Foundations of Data Engineering**
> **Week 2: SQL for Data Engineering**
> **Tuesday**

---

## What we covered yesterday (Day 1 recap)

On Monday we mastered the four core SQL building blocks:

| Concept | What it does |
|---|---|
| `SELECT` | Retrieves exactly the columns you need, with expressions and aliases |
| `WHERE` | Filters rows using conditions, patterns, ranges, and NULL checks |
| `JOIN` | Combines data from multiple tables through shared keys |
| `GROUP BY` | Summarizes rows into aggregated groups with HAVING for filtering |

We also learned the **SQL execution order** (FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT) and wrote real data engineering queries - customer lifetime value, product performance, and pipeline validation.

> **Today we go beyond the basics.** These advanced techniques are what separate a beginner SQL writer from a data engineer who can build real transformation logic inside pipelines and warehouses.

---

## Today's objective

By the end of today, you should be able to:

- Write **subqueries** - queries inside queries - to solve multi-step problems.
- Use **CTEs** to write clean, readable, and reusable transformation logic.
- Use **window functions** to perform calculations across rows without collapsing them.
- Create and use **views** to save reusable query logic in the database.
- Write **stored procedures** to automate repeated database operations.
- Write **recursive queries** to handle hierarchical data like org charts and categories.

---

## Setup: use yesterday's database

We will continue using the `ecommerce_db` database from Monday. Connect to it:

```sql
\c ecommerce_db
```

Verify your tables are there:

```sql
\dt raw.*
```

You should see: `customers`, `products`, `orders`, `order_items`.

---

## 1. Subqueries: queries inside queries

A **subquery** is a query nested inside another query. It is wrapped in parentheses and can appear in the `SELECT`, `FROM`, or `WHERE` clause.

Subqueries let you solve problems in multiple steps - first get an intermediate result, then use it in the outer query.

### Subquery in WHERE - filtering with a derived value

```sql
-- Find all orders where the total is above the average order value
SELECT
    order_id,
    customer_id,
    order_date,
    total_amount
FROM raw.orders
WHERE total_amount > (
    SELECT AVG(total_amount)
    FROM raw.orders
    WHERE status != 'cancelled'
)
ORDER BY total_amount DESC;
```

> The inner query `SELECT AVG(total_amount) FROM raw.orders` runs first and returns a single number. The outer query then uses that number in the `WHERE` filter.

### Subquery with IN - filtering against a list

```sql
-- Find customers who have placed at least one delivered order
SELECT
    customer_id,
    first_name,
    last_name,
    city
FROM raw.customers
WHERE customer_id IN (
    SELECT DISTINCT customer_id
    FROM raw.orders
    WHERE status = 'delivered'
);
```

```sql
-- Find customers who have NEVER placed an order
SELECT
    customer_id,
    first_name,
    last_name,
    city
FROM raw.customers
WHERE customer_id NOT IN (
    SELECT DISTINCT customer_id
    FROM raw.orders
);
```

### Subquery in FROM - derived tables

A subquery in the `FROM` clause creates a **temporary table** (also called a derived table) that the outer query reads from.

```sql
-- Get the top 3 customers by total spend
SELECT
    customer_name,
    total_spent
FROM (
    SELECT
        c.first_name || ' ' || c.last_name  AS customer_name,
        SUM(o.total_amount)                 AS total_spent
    FROM raw.customers c
    JOIN raw.orders o ON c.customer_id = o.customer_id
    WHERE o.status != 'cancelled'
    GROUP BY c.customer_id, c.first_name, c.last_name
) AS customer_totals
ORDER BY total_spent DESC
LIMIT 3;
```

### Subquery in SELECT- scalar subqueries

A subquery in the `SELECT` clause returns a single value per row.

```sql
-- Show each order alongside the overall average order value
SELECT
    order_id,
    customer_id,
    total_amount,
    (SELECT ROUND(AVG(total_amount), 2) FROM raw.orders WHERE status != 'cancelled') AS overall_avg,
    total_amount - (SELECT AVG(total_amount) FROM raw.orders WHERE status != 'cancelled') AS diff_from_avg
FROM raw.orders
WHERE status != 'cancelled'
ORDER BY total_amount DESC;
```

>  **When to use subqueries vs CTEs:** Subqueries are fine for simple cases. When your logic has multiple steps or you need to reference the same intermediate result more than once, use a CTE — they are much easier to read and debug.

---

## 2. CTEs - Common Table Expressions

A **CTE** (Common Table Expression) is a named, temporary result set that you define at the top of a query using the `WITH` keyword. You can then reference it by name in the main query — just like a table.

CTEs are one of the most important tools in a data engineer's SQL toolkit because they make complex transformation logic **readable, maintainable, and debuggable**.

### Basic CTE syntax

```sql
WITH cte_name AS (
    -- your query here
)
SELECT * FROM cte_name;
```

### Single CTE example

```sql
-- Step 1: Calculate total spend per customer
-- Step 2: Filter to only high-value customers

WITH customer_spend AS (
    SELECT
        c.customer_id,
        c.first_name || ' ' || c.last_name  AS customer_name,
        c.city,
        c.segment,
        COUNT(o.order_id)                   AS total_orders,
        SUM(o.total_amount)                 AS total_spent
    FROM raw.customers c
    LEFT JOIN raw.orders o ON c.customer_id = o.customer_id
    WHERE o.status != 'cancelled'
    GROUP BY c.customer_id, c.first_name, c.last_name, c.city, c.segment
)
SELECT *
FROM customer_spend
WHERE total_spent > 50000
ORDER BY total_spent DESC;
```

### Multiple CTEs - chaining transformation steps

This is the data engineering pattern - each CTE is one step in your transformation logic.

```sql
-- Multi-step transformation: daily revenue → monthly summary → final report

WITH daily_revenue AS (
    -- Step 1: aggregate to daily level
    SELECT
        order_date,
        COUNT(*)            AS daily_orders,
        SUM(total_amount)   AS daily_revenue
    FROM raw.orders
    WHERE status != 'cancelled'
    GROUP BY order_date
),

monthly_summary AS (
    -- Step 2: aggregate daily to monthly
    SELECT
        DATE_TRUNC('month', order_date)     AS month,
        SUM(daily_orders)                   AS total_orders,
        SUM(daily_revenue)                  AS total_revenue,
        ROUND(AVG(daily_revenue), 2)        AS avg_daily_revenue
    FROM daily_revenue
    GROUP BY DATE_TRUNC('month', order_date)
)

-- Step 3: final output
SELECT
    TO_CHAR(month, 'Month YYYY')        AS month_name,
    total_orders,
    total_revenue,
    avg_daily_revenue
FROM monthly_summary
ORDER BY month;
```

### CTE for data quality checks in pipelines

```sql
-- Identify and report data quality issues in one query

WITH null_checks AS (
    SELECT 'missing_email' AS issue, COUNT(*) AS count
    FROM raw.customers
    WHERE email IS NULL

    UNION ALL

    SELECT 'missing_city', COUNT(*)
    FROM raw.customers
    WHERE city IS NULL
),

value_checks AS (
    SELECT 'negative_amount' AS issue, COUNT(*) AS count
    FROM raw.orders
    WHERE total_amount <= 0

    UNION ALL

    SELECT 'cancelled_high_value', COUNT(*)
    FROM raw.orders
    WHERE status = 'cancelled' AND total_amount > 50000
),

all_issues AS (
    SELECT * FROM null_checks
    UNION ALL
    SELECT * FROM value_checks
)

SELECT issue, count
FROM all_issues
WHERE count > 0
ORDER BY count DESC;
```

> -**CTEs vs subqueries - key difference:** A subquery can only be used once and is embedded inside another query. A CTE is named and can be referenced multiple times in the same query. For complex pipelines with 3+ steps, always use CTEs.

---

## 3. Window Functions- calculations across rows

A **window function** performs a calculation across a set of rows that are related to the current row - without collapsing those rows into a single result the way `GROUP BY` does.

This is the single most powerful SQL concept for data engineers. Window functions let you:
- Rank records within groups.
- Calculate running totals and moving averages.
- Compare each row to the previous or next row.
- Calculate each row's share of a total.

### Basic window function syntax

```sql
function_name() OVER (
    PARTITION BY column    -- divide rows into groups (optional)
    ORDER BY column        -- define the order within each group
)
```

- **PARTITION BY** - divides rows into groups (like GROUP BY, but rows are kept).
- **ORDER BY** - defines the order within each partition.
- **OVER()** - defines the window (the set of rows the function looks at).

### Ranking functions

#### ROW_NUMBER() - unique sequential number per row

```sql
-- Number each customer's orders from oldest to newest
SELECT
    c.first_name || ' ' || c.last_name      AS customer_name,
    o.order_id,
    o.order_date,
    o.total_amount,
    ROW_NUMBER() OVER (
        PARTITION BY o.customer_id
        ORDER BY o.order_date
    )                                        AS order_number
FROM raw.orders o
JOIN raw.customers c ON o.customer_id = c.customer_id
ORDER BY customer_name, order_date;
```

#### RANK() and DENSE_RANK() - ranking with ties

```sql
-- Rank products by revenue (ties get the same rank)
SELECT
    p.product_name,
    p.category,
    SUM(oi.subtotal)                        AS total_revenue,
    RANK() OVER (ORDER BY SUM(oi.subtotal) DESC)        AS revenue_rank,
    DENSE_RANK() OVER (ORDER BY SUM(oi.subtotal) DESC)  AS dense_rank
FROM raw.order_items oi
JOIN raw.products p ON oi.product_id = p.product_id
GROUP BY p.product_id, p.product_name, p.category
ORDER BY revenue_rank;
```

| Function | Behaviour with ties | Example ranks |
|---|---|---|
| `ROW_NUMBER()` | No ties — always unique | 1, 2, 3, 4 |
| `RANK()` | Ties get same rank, next rank skips | 1, 2, 2, 4 |
| `DENSE_RANK()` | Ties get same rank, no skipping | 1, 2, 2, 3 |

#### Rank within a partition — top N per group

```sql
-- Get the top 2 most expensive products per category
WITH ranked_products AS (
    SELECT
        product_name,
        category,
        unit_price,
        RANK() OVER (
            PARTITION BY category
            ORDER BY unit_price DESC
        ) AS price_rank
    FROM raw.products
)
SELECT *
FROM ranked_products
WHERE price_rank <= 2
ORDER BY category, price_rank;
```

>  **Top N per group** is one of the most common data engineering patterns. You will use this constantly — top customers per region, top products per category, most recent record per customer, etc.

### Running totals and cumulative aggregates

```sql
-- Running total of revenue day by day
SELECT
    order_date,
    SUM(total_amount)           AS daily_revenue,
    SUM(SUM(total_amount)) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    )                           AS running_total
FROM raw.orders
WHERE status != 'cancelled'
GROUP BY order_date
ORDER BY order_date;
```

### LAG() and LEAD() - comparing to previous/next rows

```sql
-- Compare each day's revenue to the previous day
WITH daily AS (
    SELECT
        order_date,
        SUM(total_amount) AS daily_revenue
    FROM raw.orders
    WHERE status != 'cancelled'
    GROUP BY order_date
)
SELECT
    order_date,
    daily_revenue,
    LAG(daily_revenue)  OVER (ORDER BY order_date)  AS prev_day_revenue,
    LEAD(daily_revenue) OVER (ORDER BY order_date)  AS next_day_revenue,
    daily_revenue - LAG(daily_revenue) OVER (ORDER BY order_date) AS day_over_day_change
FROM daily
ORDER BY order_date;
```

>  `LAG()` looks **back** at the previous row. `LEAD()` looks **forward** at the next row. These are used heavily in time-series analysis - growth rates, day-over-day changes, month-over-month comparisons.

### Percentage of total

```sql
-- Each order's share of total revenue
SELECT
    order_id,
    customer_id,
    order_date,
    total_amount,
    SUM(total_amount) OVER ()                           AS grand_total,
    ROUND(
        total_amount / SUM(total_amount) OVER () * 100
    , 2)                                                AS pct_of_total
FROM raw.orders
WHERE status != 'cancelled'
ORDER BY total_amount DESC;
```

---

## 4. Views - saving reusable query logic

A **view** is a saved SQL query stored in the database with a name. You query it exactly like a table - but it runs the underlying query every time you call it.

Views are used in data engineering to:
- Create a clean **analytics layer** on top of raw tables.
- Hide complex JOIN and transformation logic from analysts.
- Build the foundation for dashboard queries.
- Enforce consistent business logic across all consumers of the data.

### Creating a view

```sql
-- Create a view for the customer order summary in the analytics schema
CREATE VIEW analytics.vw_customer_summary AS
SELECT
    c.customer_id,
    c.first_name || ' ' || c.last_name      AS customer_name,
    c.city,
    c.segment,
    COUNT(DISTINCT o.order_id)              AS total_orders,
    COALESCE(SUM(o.total_amount), 0)        AS total_spent,
    COALESCE(MIN(o.order_date), NULL)       AS first_order_date,
    COALESCE(MAX(o.order_date), NULL)       AS last_order_date
FROM raw.customers c
LEFT JOIN raw.orders o
    ON c.customer_id = o.customer_id
    AND o.status != 'cancelled'
GROUP BY c.customer_id, c.first_name, c.last_name, c.city, c.segment;
```

```sql
-- Query the view just like a table
SELECT * FROM analytics.vw_customer_summary
ORDER BY total_spent DESC;
```

### Replacing a view

```sql
-- Update the view definition
CREATE OR REPLACE VIEW analytics.vw_customer_summary AS
SELECT
    c.customer_id,
    c.first_name || ' ' || c.last_name      AS customer_name,
    c.city,
    c.country,
    c.segment,
    COUNT(DISTINCT o.order_id)              AS total_orders,
    COALESCE(SUM(o.total_amount), 0)        AS total_spent
FROM raw.customers c
LEFT JOIN raw.orders o
    ON c.customer_id = o.customer_id
    AND o.status != 'cancelled'
GROUP BY c.customer_id, c.first_name, c.last_name, c.city, c.country, c.segment;
```

### Dropping a view

```sql
DROP VIEW IF EXISTS analytics.vw_customer_summary;
```

### Materialized views - performance upgrade

A regular view re-runs its query every time it is called. A **materialized view** stores the result physically - like a cached snapshot.

```sql
-- Create a materialized view for heavy aggregations
CREATE MATERIALIZED VIEW analytics.mv_monthly_revenue AS
SELECT
    DATE_TRUNC('month', order_date)     AS month,
    COUNT(*)                            AS total_orders,
    SUM(total_amount)                   AS total_revenue,
    ROUND(AVG(total_amount), 2)         AS avg_order_value
FROM raw.orders
WHERE status != 'cancelled'
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;

-- Refresh the materialized view (run this after each pipeline load)
REFRESH MATERIALIZED VIEW analytics.mv_monthly_revenue;

-- Query it
SELECT * FROM analytics.mv_monthly_revenue;
```

| Type | Stores data? | Auto-updates? | Best for |
|---|---|---|---|
| View | No — reruns every time | Yes | Simple logic, small data |
| Materialized view | Yes — cached snapshot | No — must refresh manually | Heavy aggregations, dashboards |

---

## 5. Stored Procedures - automating database operations

A **stored procedure** is a saved block of SQL code that can be executed by name. It can contain multiple SQL statements, conditional logic, loops, and error handling.

Data engineers use stored procedures to:
- Automate repetitive ETL operations inside the database.
- Run multi-step transformations in a single call.
- Implement data quality checks as part of a pipeline.
- Encapsulate business logic that should always run together.

### Basic stored procedure

```sql
-- Create a procedure that moves delivered orders to a staging table
CREATE OR REPLACE PROCEDURE staging.load_delivered_orders()
LANGUAGE plpgsql
AS $$
BEGIN
    -- Create the staging table if it doesn't exist
    CREATE TABLE IF NOT EXISTS staging.delivered_orders (
        order_id        INTEGER,
        customer_id     INTEGER,
        order_date      DATE,
        total_amount    NUMERIC(10, 2),
        loaded_at       TIMESTAMP DEFAULT NOW()
    );

    -- Insert only new delivered orders not already in staging
    INSERT INTO staging.delivered_orders (order_id, customer_id, order_date, total_amount)
    SELECT
        order_id,
        customer_id,
        order_date,
        total_amount
    FROM raw.orders
    WHERE status = 'delivered'
    AND order_id NOT IN (SELECT order_id FROM staging.delivered_orders);

    RAISE NOTICE 'Delivered orders loaded successfully at %', NOW();
END;
$$;
```

```sql
-- Call the procedure
CALL staging.load_delivered_orders();
```

### Stored procedure with parameters

```sql
-- Procedure to load orders by status (flexible)
CREATE OR REPLACE PROCEDURE staging.load_orders_by_status(p_status VARCHAR)
LANGUAGE plpgsql
AS $$
DECLARE
    v_count INTEGER;
BEGIN
    INSERT INTO staging.delivered_orders (order_id, customer_id, order_date, total_amount)
    SELECT order_id, customer_id, order_date, total_amount
    FROM raw.orders
    WHERE status = p_status
    AND order_id NOT IN (SELECT order_id FROM staging.delivered_orders);

    GET DIAGNOSTICS v_count = ROW_COUNT;
    RAISE NOTICE '% rows loaded for status: %', v_count, p_status;
END;
$$;

-- Call with a parameter
CALL staging.load_orders_by_status('shipped');
```

>  **Stored procedures vs functions in PostgreSQL:**
> - A **procedure** (`CREATE PROCEDURE`) performs actions - INSERT, UPDATE, DELETE. Called with `CALL`.
> - A **function** (`CREATE FUNCTION`) returns a value and can be used in a SELECT. Called inline.
> For ETL and pipeline automation, procedures are the standard choice.

---

## 6. Recursive Queries - handling hierarchical data

A **recursive query** is a CTE that references itself. It is used to traverse **hierarchical** or **tree-structured** data - where each row has a parent row.

Common use cases in data engineering:
- **Organisational hierarchies** - managers and their reports.
- **Product categories** - parent and child categories.
- **Geographic hierarchies** - country → region → city.
- **Bill of materials** - a product made of components, each made of sub-components.

### Setup - create a category hierarchy table

```sql
CREATE TABLE raw.categories (
    category_id     SERIAL PRIMARY KEY,
    category_name   VARCHAR(100),
    parent_id       INTEGER REFERENCES raw.categories(category_id)
);

INSERT INTO raw.categories (category_name, parent_id) VALUES
('All Products',    NULL),      -- id 1 — root
('Electronics',     1),         -- id 2
('Furniture',       1),         -- id 3
('Stationery',      1),         -- id 4
('Phones',          2),         -- id 5
('Computers',       2),         -- id 6
('Laptops',         6),         -- id 7
('Desktops',        6),         -- id 8
('Office Chairs',   3),         -- id 9
('Desks',           3),         -- id 10
('Pens',            4),         -- id 11
('Notebooks',       4);         -- id 12
```

### Recursive CTE syntax

```sql
WITH RECURSIVE cte_name AS (
    -- Base case: the starting point (the root or top-level rows)
    SELECT ...

    UNION ALL

    -- Recursive case: join the CTE back to itself to get the next level
    SELECT ...
    FROM table
    JOIN cte_name ON ...
)
SELECT * FROM cte_name;
```

### Traversing the full hierarchy

```sql
-- Show the full category tree with levels and paths
WITH RECURSIVE category_tree AS (
    -- Base case: start from the root (no parent)
    SELECT
        category_id,
        category_name,
        parent_id,
        0                           AS level,
        category_name               AS full_path
    FROM raw.categories
    WHERE parent_id IS NULL

    UNION ALL

    -- Recursive case: join children to their parent
    SELECT
        c.category_id,
        c.category_name,
        c.parent_id,
        ct.level + 1,
        ct.full_path || ' > ' || c.category_name
    FROM raw.categories c
    JOIN category_tree ct ON c.parent_id = ct.category_id
)
SELECT
    level,
    REPEAT('    ', level) || category_name  AS indented_name,
    full_path
FROM category_tree
ORDER BY full_path;
```

**Output:**
```
level | indented_name               | full_path
------|-----------------------------|----------------------------------
0     | All Products                | All Products
1     |     Electronics             | All Products > Electronics
2     |         Computers           | All Products > Electronics > Computers
3     |             Laptops         | All Products > Electronics > Computers > Laptops
3     |             Desktops        | All Products > Electronics > Computers > Desktops
1     |     Furniture               | All Products > Furniture
...
```

### Finding all descendants of a node

```sql
-- Find all subcategories under 'Electronics' (id = 2)
WITH RECURSIVE subcategories AS (
    SELECT category_id, category_name, parent_id
    FROM raw.categories
    WHERE category_id = 2           -- start from Electronics

    UNION ALL

    SELECT c.category_id, c.category_name, c.parent_id
    FROM raw.categories c
    JOIN subcategories s ON c.parent_id = s.category_id
)
SELECT * FROM subcategories;
```

---

## 7. Putting it all together - a full transformation pipeline in SQL

This is what a real data engineering transformation looks like — using CTEs, window functions, and views together.

```sql
-- Build a complete customer analytics table using advanced SQL

CREATE OR REPLACE VIEW analytics.vw_customer_360 AS

WITH order_stats AS (
    -- Step 1: basic order metrics per customer
    SELECT
        customer_id,
        COUNT(*)                                AS total_orders,
        SUM(total_amount)                       AS lifetime_value,
        MIN(order_date)                         AS first_order_date,
        MAX(order_date)                         AS last_order_date,
        AVG(total_amount)                       AS avg_order_value
    FROM raw.orders
    WHERE status != 'cancelled'
    GROUP BY customer_id
),

order_rankings AS (
    -- Step 2: rank customers by lifetime value
    SELECT
        customer_id,
        lifetime_value,
        RANK() OVER (ORDER BY lifetime_value DESC)  AS value_rank,
        NTILE(4) OVER (ORDER BY lifetime_value DESC) AS value_quartile
        -- quartile 1 = top 25% customers
    FROM order_stats
),

recency AS (
    -- Step 3: calculate days since last order
    SELECT
        customer_id,
        last_order_date,
        CURRENT_DATE - last_order_date          AS days_since_last_order
    FROM order_stats
)

-- Final output: join everything together
SELECT
    c.customer_id,
    c.first_name || ' ' || c.last_name          AS customer_name,
    c.city,
    c.segment,
    os.total_orders,
    os.lifetime_value,
    ROUND(os.avg_order_value, 2)                AS avg_order_value,
    os.first_order_date,
    os.last_order_date,
    r.days_since_last_order,
    orr.value_rank,
    CASE orr.value_quartile
        WHEN 1 THEN 'Platinum'
        WHEN 2 THEN 'Gold'
        WHEN 3 THEN 'Silver'
        ELSE        'Bronze'
    END                                         AS customer_tier
FROM raw.customers c
LEFT JOIN order_stats    os  ON c.customer_id = os.customer_id
LEFT JOIN order_rankings orr ON c.customer_id = orr.customer_id
LEFT JOIN recency        r   ON c.customer_id = r.customer_id
ORDER BY os.lifetime_value DESC NULLS LAST;

-- Query the final view
SELECT * FROM analytics.vw_customer_360;
```

---

## 8. Key terms to know from today

| Term | Meaning |
|---|---|
| Subquery | A query nested inside another query |
| Derived table | A subquery used in the FROM clause as a temporary table |
| Scalar subquery | A subquery that returns exactly one value |
| CTE | Common Table Expression — a named temporary result set defined with `WITH` |
| Chained CTEs | Multiple CTEs defined in sequence, each building on the previous |
| Window function | A function that calculates across a set of rows without collapsing them |
| `PARTITION BY` | Divides rows into groups for a window function |
| `ROW_NUMBER()` | Assigns a unique sequential number to each row within a partition |
| `RANK()` | Ranks rows with gaps for ties |
| `DENSE_RANK()` | Ranks rows without gaps for ties |
| `NTILE(n)` | Divides rows into n equal buckets |
| `LAG()` | Accesses the value of a previous row |
| `LEAD()` | Accesses the value of a next row |
| Running total | Cumulative sum calculated row by row using a window function |
| View | A saved SQL query that can be queried like a table |
| Materialized view | A view that stores its results physically for faster querying |
| Stored procedure | A saved block of SQL code executed by name using `CALL` |
| Recursive CTE | A CTE that references itself to traverse hierarchical data |
| `UNION ALL` | Combines the results of two queries including duplicates |
| `CASE WHEN` | Conditional logic inside a SQL query |

---

## 9. Summary

> **Subqueries** let you solve multi-step problems by nesting queries. **CTEs** make that logic clean, readable, and reusable — each CTE is one step in your transformation. **Window functions** are the most powerful SQL tool for analytics — ranking, running totals, and row-by-row comparisons without losing detail. **Views** save reusable transformation logic as named objects in the database. **Stored procedures** automate repeated ETL operations. **Recursive queries** handle hierarchical data that flat queries cannot.
>
> These are the tools that power real data warehouse transformation layers — the kind built with dbt, Synapse, and BigQuery — and mastering them makes you a significantly stronger data engineer.

---

## 10. What's coming next

| Day | Topic |
|---|---|
| Wednesday | Query optimization and execution plans |
| Thursday | Data modeling — normalization, denormalization, and star schemas |
| Friday | Peer project — advanced SQL transformation challenge |
| Saturday (Lab) | Mini project — build a transformation layer using CTEs and window functions |

---

*Notes by LuxDevHQ | Month 1 - Foundations of Data Engineering | Week 2, Day 2*