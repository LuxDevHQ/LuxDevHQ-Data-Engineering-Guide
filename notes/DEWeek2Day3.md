# Week 2, Day 3 - Query Optimization and Execution Plans

> **Month 1: Foundations of Data Engineering**
> **Week 2: SQL for Data Engineering**
> **Wednesday**

---

## What we covered yesterday (Day 2 recap)

On Tuesday we learned advanced SQL techniques:

| Concept | What it does |
|---|---|
| Subqueries | Queries nested inside other queries to solve multi-step problems |
| CTEs | Named temporary result sets that make complex logic readable and reusable |
| Window functions | Calculations across rows without collapsing them - ranking, running totals, LAG/LEAD |
| Views | Saved query logic that can be queried like a table |
| Materialized views | Views that store results physically for faster querying |
| Stored procedures | Saved SQL blocks that automate repeated ETL operations |
| Recursive queries | CTEs that reference themselves to traverse hierarchical data |

> **Today we learn how to make SQL fast.** Writing a correct query is step one. Writing a query that runs efficiently on millions or billions of rows - without wasting compute time and cloud money - is what separates a good data engineer from a great one.

---

## Today's objective

By the end of today, you should be able to:

- Explain what a query execution plan is and how PostgreSQL processes a query.
- Read and interpret `EXPLAIN` and `EXPLAIN ANALYZE` output.
- Identify slow query patterns and understand why they are slow.
- Use indexes to speed up queries significantly.
- Apply query optimization techniques used in real data engineering pipelines.
- Understand how these concepts apply to cloud data warehouses like BigQuery, Redshift, and Synapse.

---

## Setup - continue with ecommerce_db

```sql
\c ecommerce_db
```

We will also insert more data to make optimization meaningful:

```sql
-- Insert additional orders to simulate a larger dataset
INSERT INTO raw.orders (customer_id, order_date, status, total_amount)
SELECT
    (RANDOM() * 7 + 1)::INTEGER,
    CURRENT_DATE - (RANDOM() * 365)::INTEGER,
    (ARRAY['delivered', 'shipped', 'pending', 'cancelled'])[FLOOR(RANDOM() * 4 + 1)],
    ROUND((RANDOM() * 100000 + 500)::NUMERIC, 2)
FROM GENERATE_SERIES(1, 10000);
```

>  `GENERATE_SERIES(1, 10000)` generates 10,000 rows. We now have enough data to see meaningful differences between optimized and unoptimized queries.

---

## 1. How PostgreSQL executes a query

Before optimizing queries, you need to understand what happens when PostgreSQL receives one.

### The query execution pipeline

```text
Your SQL query
      ↓
  Parser          → checks syntax, is the SQL valid?
      ↓
  Analyser        → checks semantics, do the tables and columns exist?
      ↓
  Rewriter        → applies rules, expands views
      ↓
  Planner         → generates multiple possible execution plans,
                    estimates the cost of each, picks the cheapest
      ↓
  Executor        → runs the chosen plan and returns results
```

The **planner** is the most important part for optimization. It uses **statistics** about your tables - row counts, column value distributions, table sizes - to estimate the cost of different execution strategies and choose the best one.

### What the planner decides

For every query, the planner chooses:
- **How to scan each table** - full table scan vs index scan.
- **How to join tables** - nested loop, hash join, or merge join.
- **In what order to join tables** - which table to read first.
- **Whether to sort data** - and how.

Understanding these choices is what `EXPLAIN` shows you.

---

## 2. EXPLAIN - reading execution plans

`EXPLAIN` shows you the execution plan PostgreSQL **would use** for a query -without actually running it.

`EXPLAIN ANALYZE` runs the query and shows you the **actual** execution time and row counts alongside the estimates.

### Basic EXPLAIN

```sql
-- See the plan for a simple query
EXPLAIN
SELECT * FROM raw.orders
WHERE status = 'delivered';
```

**Example output:**
```
Seq Scan on orders  (cost=0.00..245.00 rows=2500 width=48)
  Filter: ((status)::text = 'delivered'::text)
```

### Reading the output

Every node in an execution plan shows:

```
Node Type  (cost=startup_cost..total_cost  rows=estimated_rows  width=avg_row_bytes)
```

| Part | What it means |
|---|---|
| **Node type** | The operation being performed (Seq Scan, Index Scan, Hash Join, etc.) |
| **startup_cost** | Cost before the first row can be returned |
| **total_cost** | Estimated total cost to return all rows (arbitrary units) |
| **rows** | Estimated number of rows returned |
| **width** | Average size of each row in bytes |

>  **Cost units are not seconds** - they are arbitrary units used to compare plans to each other. A cost of 100 is not 100 milliseconds. What matters is comparing costs between plans: lower cost = better plan.

### EXPLAIN ANALYZE: actual vs estimated

```sql
-- Run the query and show actual vs estimated
EXPLAIN ANALYZE
SELECT * FROM raw.orders
WHERE status = 'delivered';
```

**Example output:**
```
Seq Scan on orders  (cost=0.00..245.00 rows=2500 width=48)
                    (actual time=0.015..3.842 rows=2487 loops=1)
  Filter: ((status)::text = 'delivered'::text)
  Rows Removed by Filter: 7513
Planning Time: 0.112 ms
Execution Time: 4.021 ms
```

Now you can see:
- **Estimated rows: 2500** vs **Actual rows: 2487** - the planner was accurate here.
- **Execution Time: 4.021 ms** - how long it actually took.
- **Rows Removed by Filter: 7513** - 7,513 rows were read but discarded.

### EXPLAIN options

```sql
-- More detailed output in JSON format
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)
SELECT * FROM raw.orders WHERE status = 'delivered';

-- Show buffer usage (how much was read from disk vs cache)
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM raw.orders WHERE status = 'delivered';
```

---

## 3. Common execution plan node types

You will see these node types regularly in execution plans:

| Node type | What it means | Fast or slow? |
|---|---|---|
| **Seq Scan** | Full table scan - reads every row | Slow on large tables |
| **Index Scan** | Uses an index to find matching rows | Fast for selective queries |
| **Index Only Scan** | All needed data is in the index itself | Very fast |
| **Bitmap Heap Scan** | Uses index to find pages, then reads those pages | Good for moderate selectivity |
| **Nested Loop** | For each row in table A, scans table B | Fast for small tables |
| **Hash Join** | Builds a hash table from one table, probes with the other | Fast for large table joins |
| **Merge Join** | Joins two pre-sorted datasets | Fast when data is already sorted |
| **Sort** | Sorts rows for ORDER BY or merge joins | Can be expensive on large data |
| **Hash Aggregate** | Groups rows using a hash table (for GROUP BY) | Common and generally fast |
| **Limit** | Stops after returning N rows | Makes queries with LIMIT fast |

### Recognizing a problem in a plan

```sql
-- A slow query on a large table
EXPLAIN ANALYZE
SELECT * FROM raw.orders
WHERE order_date = '2025-05-01'
AND status = 'delivered';
```

If you see `Seq Scan` with a large number of `Rows Removed by Filter` - that is a signal that an **index would help**.

---

## 4. Indexes : the most powerful optimization tool

An **index** is a separate data structure that PostgreSQL maintains alongside a table. It stores a sorted copy of one or more columns and pointers back to the full rows - allowing PostgreSQL to find matching rows without reading the entire table.

### Simple analogy

Think of a book. Reading every page to find a topic is a **sequential scan**. Using the **index at the back of the book** to jump directly to the right page is an **index scan**.

### Without an index vs with an index

```sql
-- Without an index: PostgreSQL reads ALL 10,000+ rows
EXPLAIN ANALYZE
SELECT * FROM raw.orders
WHERE order_date = '2025-05-01';
-- Result: Seq Scan, reads entire table
```

```sql
-- Create an index on order_date
CREATE INDEX idx_orders_order_date ON raw.orders(order_date);
```

```sql
-- With an index: PostgreSQL jumps directly to matching rows
EXPLAIN ANALYZE
SELECT * FROM raw.orders
WHERE order_date = '2025-05-01';
-- Result: Index Scan, reads only matching rows
```

### Types of indexes in PostgreSQL

#### B-Tree index (default - most common)

```sql
-- Standard index for equality and range queries
CREATE INDEX idx_orders_status ON raw.orders(status);
CREATE INDEX idx_orders_order_date ON raw.orders(order_date);
CREATE INDEX idx_customers_email ON raw.customers(email);
```

Best for: `=`, `<`, `>`, `BETWEEN`, `ORDER BY`, `LIKE 'prefix%'`

#### Composite index : multiple columns

```sql
-- Index on multiple columns together
-- Column order matters - put the most selective column first
CREATE INDEX idx_orders_status_date ON raw.orders(status, order_date);
```

```sql
-- This query uses the composite index efficiently
SELECT * FROM raw.orders
WHERE status = 'delivered'
AND order_date >= '2025-05-01';
```

>  **Column order in composite indexes:** A composite index on `(status, order_date)` can be used for queries filtering on `status` alone, or `status AND order_date` together. It cannot efficiently serve a query filtering only on `order_date`. Always put the most commonly filtered column first.

#### Partial index : index only a subset of rows

```sql
-- Index only delivered orders (if that is what you query most)
CREATE INDEX idx_orders_delivered ON raw.orders(order_date)
WHERE status = 'delivered';
```

Partial indexes are smaller and faster than full indexes. Excellent for filtering on a specific status, flag, or category that you query constantly.

#### Unique index : enforces uniqueness

```sql
-- Unique index (also created automatically by UNIQUE constraint)
CREATE UNIQUE INDEX idx_customers_email_unique ON raw.customers(email);
```

#### Checking existing indexes

```sql
-- List all indexes on the orders table
SELECT
    indexname,
    indexdef
FROM pg_indexes
WHERE tablename = 'orders'
AND schemaname = 'raw';
```

#### Dropping an index

```sql
DROP INDEX IF EXISTS idx_orders_status;
```

### When NOT to use indexes

Indexes are not always the answer. Avoid indexing when:

| Situation | Why indexing does not help |
|---|---|
| Small tables (< 10,000 rows) | A full scan is faster than using the index overhead |
| Columns with very few distinct values | An index on a boolean column is rarely useful |
| Tables that are written to very frequently | Every INSERT/UPDATE/DELETE must also update the index |
| Queries that return most of the table | If you are returning 80% of rows, a seq scan is faster |

---

## 5. Query optimization techniques

### Technique 1 : Select only the columns you need

```sql
--  Slow - fetches all columns, more data to transfer
SELECT * FROM raw.orders
WHERE status = 'delivered';

--  Fast -fetches only what you need
SELECT order_id, customer_id, order_date, total_amount
FROM raw.orders
WHERE status = 'delivered';
```

### Technique 2 - Filter early, not late

```sql
-- ❌ Slow - joins first, then filters
SELECT c.first_name, o.total_amount
FROM raw.customers c
JOIN raw.orders o ON c.customer_id = o.customer_id
WHERE o.status = 'delivered'
AND o.total_amount > 50000;

-- Fast - filter inside a CTE before joining
WITH delivered_big_orders AS (
    SELECT order_id, customer_id, total_amount
    FROM raw.orders
    WHERE status = 'delivered'
    AND total_amount > 50000
)
SELECT c.first_name, d.total_amount
FROM delivered_big_orders d
JOIN raw.customers c ON d.customer_id = c.customer_id;
```

### Technique 3 - Avoid functions on indexed columns in WHERE

```sql
--  Slow - applying a function to the column breaks index usage
SELECT * FROM raw.orders
WHERE EXTRACT(YEAR FROM order_date) = 2025;

-- Fast - use a range filter so the index can be used
SELECT * FROM raw.orders
WHERE order_date >= '2025-01-01'
AND order_date < '2026-01-01';
```

### Technique 4 - Use EXISTS instead of IN for large subqueries

```sql
--  Slower for large datasets - IN evaluates the entire subquery
SELECT * FROM raw.customers
WHERE customer_id IN (
    SELECT customer_id FROM raw.orders WHERE status = 'delivered'
);

--  Faster - EXISTS stops as soon as it finds the first match
SELECT * FROM raw.customers c
WHERE EXISTS (
    SELECT 1 FROM raw.orders o
    WHERE o.customer_id = c.customer_id
    AND o.status = 'delivered'
);
```

### Technique 5 - Avoid SELECT DISTINCT when you can use GROUP BY or EXISTS

```sql
-- DISTINCT can be expensive - sorts and deduplicates all rows
SELECT DISTINCT customer_id FROM raw.orders;

--  GROUP BY is often faster for this use case
SELECT customer_id FROM raw.orders GROUP BY customer_id;
```

### Technique 6 - Use LIMIT when you only need a sample

```sql
-- Checking data during development - use LIMIT to avoid full scans
SELECT * FROM raw.orders
WHERE status = 'delivered'
ORDER BY order_date DESC
LIMIT 100;
```

### Technique 7 - Avoid correlated subqueries - use JOINs or window functions

A **correlated subquery** runs once for every row in the outer query - it can be extremely slow.

```sql
--  Correlated subquery -runs the inner query once per customer row
SELECT
    customer_id,
    (SELECT SUM(total_amount)
     FROM raw.orders o
     WHERE o.customer_id = c.customer_id) AS total_spent
FROM raw.customers c;

-- Join with aggregation - runs the inner query once
SELECT
    c.customer_id,
    COALESCE(SUM(o.total_amount), 0) AS total_spent
FROM raw.customers c
LEFT JOIN raw.orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id;
```

### Technique 8 : Use VACUUM and ANALYZE to keep statistics fresh

PostgreSQL's planner relies on table statistics to make good decisions. Over time, after many inserts, updates, and deletes, statistics become stale and the planner makes poor choices.

```sql
-- Update table statistics so the planner has accurate information
ANALYZE raw.orders;
ANALYZE raw.customers;

-- Remove dead rows left by updates and deletes, then update statistics
VACUUM ANALYZE raw.orders;
```

>  In production PostgreSQL, `autovacuum` runs automatically in the background. But after a large bulk load (e.g. loading millions of rows via a pipeline), always run `ANALYZE` manually so the planner immediately has accurate statistics for the new data.

---

## 6. Table statistics - what the planner uses

You can inspect the statistics PostgreSQL uses to plan queries:

```sql
-- See the statistics PostgreSQL has collected on the orders table
SELECT
    attname          AS column_name,
    n_distinct,      -- estimated number of distinct values
    correlation      -- how sorted the column is (1 = perfectly sorted, 0 = random)
FROM pg_stats
WHERE tablename = 'orders'
AND schemaname = 'raw';
```

```sql
-- See table-level stats: row count, dead rows, last vacuum/analyze time
SELECT
    relname             AS table_name,
    n_live_tup          AS live_rows,
    n_dead_tup          AS dead_rows,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables
WHERE schemaname = 'raw';
```

---

## 7. Optimization patterns specific to data engineering

### Pattern 1 : Incremental queries with date filters and indexes

```sql
-- Always index the timestamp/date column used for incremental loads
CREATE INDEX idx_orders_created_at ON raw.orders(order_date);

-- Incremental load query - only new records since last run
SELECT *
FROM raw.orders
WHERE order_date > '2025-05-31'   -- last pipeline watermark
ORDER BY order_date;
```

### Pattern 2 - Avoid re-scanning large tables multiple times

```sql
--  Scans the orders table three times
SELECT COUNT(*) FROM raw.orders WHERE status = 'delivered';
SELECT COUNT(*) FROM raw.orders WHERE status = 'pending';
SELECT COUNT(*) FROM raw.orders WHERE status = 'cancelled';

--  Scans the table once using conditional aggregation
SELECT
    COUNT(*) FILTER (WHERE status = 'delivered')    AS delivered,
    COUNT(*) FILTER (WHERE status = 'pending')      AS pending,
    COUNT(*) FILTER (WHERE status = 'cancelled')    AS cancelled
FROM raw.orders;
```

### Pattern 3 Partition large tables by date

For very large tables (millions of rows), **table partitioning** splits the data into smaller physical pieces by date. PostgreSQL can then skip entire partitions when filtering by date - called **partition pruning**.

```sql
-- Create a partitioned table by month
CREATE TABLE raw.orders_partitioned (
    order_id        SERIAL,
    customer_id     INTEGER,
    order_date      DATE NOT NULL,
    status          VARCHAR(50),
    total_amount    NUMERIC(10, 2)
) PARTITION BY RANGE (order_date);

-- Create monthly partitions
CREATE TABLE raw.orders_2025_05
    PARTITION OF raw.orders_partitioned
    FOR VALUES FROM ('2025-05-01') TO ('2025-06-01');

CREATE TABLE raw.orders_2025_06
    PARTITION OF raw.orders_partitioned
    FOR VALUES FROM ('2025-06-01') TO ('2025-07-01');
```

```sql
-- This query only scans the May partition - not the entire table
EXPLAIN ANALYZE
SELECT * FROM raw.orders_partitioned
WHERE order_date >= '2025-05-01'
AND order_date < '2025-06-01';
```

---

## 8. How these concepts apply to cloud data warehouses

The same principles apply on cloud platforms - with platform-specific implementations.

| Concept | PostgreSQL | BigQuery | Redshift | Azure Synapse |
|---|---|---|---|---|
| Execution plans | `EXPLAIN ANALYZE` | Query execution details in console | `EXPLAIN` | Query execution plan in SSMS |
| Indexes | B-Tree, composite, partial | Not used - uses clustering instead | Sort keys / dist keys | Clustered columnstore indexes |
| Partitioning | Range / list partitioning | Partition by date column | Sort key partitioning | Table partitions |
| Statistics | `ANALYZE` / `pg_stats` | Automatic | `ANALYZE` command | Automatic |
| Column selection | `SELECT col1, col2` | Always avoid `SELECT *` - billed by bytes scanned | Always avoid `SELECT *` | Always avoid `SELECT *` |
| Caching | Shared buffer cache | Automatic query result cache | Result cache | Result set cache |

>  **BigQuery billing note:** In BigQuery you are billed by the **amount of data scanned**. `SELECT *` on a 1TB table costs significantly more than `SELECT order_id, total_amount`. Always select only the columns you need - it directly saves money.

### BigQuery-specific optimizations

```sql
-- BigQuery: use partitioned tables and filter on the partition column
SELECT order_id, total_amount
FROM `project.dataset.orders`
WHERE DATE(order_date) = '2025-05-01';   -- uses partition pruning

-- BigQuery: cluster tables by frequently filtered columns
CREATE TABLE `project.dataset.orders`
PARTITION BY DATE(order_date)
CLUSTER BY status, customer_id           -- cluster by common filter columns
AS SELECT * FROM raw_orders;
```

---

## 9. Optimization checklist - use before every pipeline query

Before deploying any query in a production pipeline, run through this checklist:

```text
✅ Run EXPLAIN ANALYZE - understand what the plan is doing
✅ Check for Seq Scans on large tables - add an index if needed
✅ Select only required columns - never SELECT * in production
✅ Filter as early as possible - reduce rows before joining
✅ Avoid functions on indexed columns in WHERE clauses
✅ Use EXISTS instead of IN for large subqueries
✅ Avoid correlated subqueries - replace with JOINs or window functions
✅ Avoid scanning the same table multiple times - use conditional aggregation
✅ Run ANALYZE after large bulk loads
✅ Use date range filters for incremental loads - and index the date column
```

---

## 10. Key terms to know from today

| Term | Meaning |
|---|---|
| Execution plan | The step-by-step strategy PostgreSQL uses to execute a query |
| Planner | The PostgreSQL component that chooses the best execution plan |
| `EXPLAIN` | Shows the execution plan without running the query |
| `EXPLAIN ANALYZE` | Runs the query and shows actual vs estimated costs and row counts |
| Seq Scan | Full table scan - reads every row in the table |
| Index Scan | Uses an index to find matching rows without reading the full table |
| Cost | Arbitrary unit used to compare execution plan options (not milliseconds) |
| Index | A separate data structure that speeds up row lookups |
| B-Tree index | The default index type - best for equality and range queries |
| Composite index | An index on multiple columns |
| Partial index | An index that covers only a subset of rows matching a condition |
| Selectivity | How many rows a filter eliminates - high selectivity = few rows returned |
| Correlated subquery | A subquery that runs once per row in the outer query - usually slow |
| `VACUUM` | Removes dead rows left by updates and deletes |
| `ANALYZE` | Updates table statistics used by the planner |
| Partition pruning | Skipping irrelevant partitions when filtering by the partition key |
| Table partitioning | Splitting a large table into smaller physical pieces by a column value |
| Clustering | Physically ordering data on disk by a column for faster range scans |
| Conditional aggregation | Using `COUNT(*) FILTER (WHERE ...)` to aggregate in one pass |
| Statistics | Data about table contents (row counts, distinct values) used by the planner |

---

## 11. Summary

> **Query optimization** is about understanding how PostgreSQL processes your SQL - and making deliberate choices that reduce the work it has to do. The key tools are `EXPLAIN ANALYZE` to diagnose slow queries, **indexes** to speed up lookups on large tables, and a set of writing patterns that avoid common performance traps.
>
> In data engineering, slow queries mean slow pipelines, delayed dashboards, and - on cloud platforms - expensive bills. Every pipeline query you write should go through an optimization review before it reaches production.

---

## 12. What's coming next

| Day | Topic |
|---|---|
| Thursday | Data modeling - normalization, denormalization, and star schemas |
| Friday | Peer project - SQL optimization challenge |
| Saturday (Lab) | Mini project - optimize a slow pipeline query end to end |

---

*Notes by LuxDevHQ | Month 1 - Foundations of Data Engineering | Week 2, Day 3*