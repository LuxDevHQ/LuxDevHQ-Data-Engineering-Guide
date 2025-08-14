# MySQL Query Execution Plans: A Complete Guide

## 1. Understanding Query Execution Plans

A **Query Execution Plan** shows you **how** MySQL decides to retrieve data — which indexes it will use, how many rows it will check, and the join strategy. You can get it using:

```sql
EXPLAIN SELECT ...;
```

For deeper insight with actual runtime statistics:

```sql
EXPLAIN ANALYZE SELECT ...;
```

## 2. Example 1 – Basic Table Scan

Let's query all customers.

```sql
EXPLAIN SELECT * FROM customer_info;
```

**Possible output:**

| id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
|----|-------------|-------|------|---------------|-----|---------|-----|------|-------|
| 1 | SIMPLE | customer_info | ALL | NULL | NULL | NULL | NULL | 1000 | Using where |

**Interpretation:**
- **type = ALL** → full table scan (slow for large datasets)
- No indexes used because there's no filtering
- **Optimization:** Avoid `SELECT *` unless necessary. Use `WHERE` + indexed columns

## 3. Example 2 – Using an Index

Suppose we query customers by `customer_id` (indexed as PRIMARY KEY).

```sql
EXPLAIN SELECT full_name FROM customer_info WHERE customer_id = 10;
```

**Possible output:**

| id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
|----|-------------|-------|------|---------------|-----|---------|-----|------|-------|
| 1 | SIMPLE | customer_info | const | PRIMARY | PRIMARY | 4 | const | 1 | NULL |

**Interpretation:**
- **type = const** → MySQL knows it will return at most one row (fast)
- Using `PRIMARY` key (O(1) lookup)
- **Optimization:** Always filter with indexed columns when possible

## 4. Example 3 – Index Usage in Products Table

Query products by `customer_id` (foreign key).

```sql
EXPLAIN SELECT product_name FROM products WHERE customer_id = 3;
```

If `customer_id` is not indexed, **type** will be `ALL` (slow). **Solution:** Add index:

```sql
CREATE INDEX idx_customer_id ON products(customer_id);
```

After indexing, the execution plan might show:

| type | possible_keys | key | rows | Extra |
|------|---------------|-----|------|-------|
| ref | idx_customer_id | idx_customer_id | 5 | Using where |

## 5. Example 4 – Join with Index

Query sales with customer names.

```sql
EXPLAIN
SELECT s.sales_id, s.total_sales, c.full_name
FROM sales s
JOIN customer_info c ON s.customer_id = c.customer_id;
```

If both `sales.customer_id` and `customer_info.customer_id` are indexed:

| id | select_type | table | type | possible_keys | key | ref | rows | Extra |
|----|-------------|-------|------|---------------|-----|-----|------|-------|
| 1 | SIMPLE | c | ALL | PRIMARY | NULL | NULL | 1000 | Using where |
| 1 | SIMPLE | s | ref | idx_customer_id | idx_customer_id | c.customer_id | 10 | NULL |

**Optimization tips:**
- Always index **join columns**
- Make sure both sides of the join use the same data type

## 6. Example 5 – Filtering and Joining

Retrieve all sales above 500 made by customers in "Nairobi".

```sql
EXPLAIN
SELECT s.sales_id, s.total_sales, c.full_name
FROM sales s
JOIN customer_info c ON s.customer_id = c.customer_id
WHERE c.location = 'Nairobi' AND s.total_sales > 500;
```

Possible bottlenecks:
- If `location` isn't indexed → table scan on `customer_info`
- **Solution:** Create index:

```sql
CREATE INDEX idx_location ON customer_info(location);
```

Execution plan should now show **ref** instead of **ALL** for `customer_info`.

## 7. Example 6 – Multi-table Join with Products

```sql
EXPLAIN
SELECT c.full_name, p.product_name, s.total_sales
FROM customer_info c
JOIN products p ON c.customer_id = p.customer_id
JOIN sales s ON p.product_id = s.product_id
WHERE s.total_sales > 1000;
```

Optimization tips:
- Index `products.customer_id`
- Index `sales.product_id`
- Filter early (`WHERE s.total_sales > 1000`) so MySQL processes fewer rows

## 8. Example 7 – Using `EXPLAIN ANALYZE`

```sql
EXPLAIN ANALYZE
SELECT c.full_name, p.product_name
FROM customer_info c
JOIN products p ON c.customer_id = p.customer_id
WHERE p.price > 500;
```

**Benefit:**
- Shows **actual execution time** for each step
- If you see that a join step takes much longer than expected → check indexes

## 9. Best Practices Recap

✅ Index frequently queried columns (especially in `WHERE`, `JOIN`, `ORDER BY`)  
✅ Avoid `SELECT *` for performance  
✅ Use `EXPLAIN` before and after schema changes  
✅ Filter data as early as possible in your query  
✅ Keep an eye on `rows` in EXPLAIN — smaller is better
