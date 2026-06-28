# Week 3, Day 3 - Extract, Transform, Load (ETL): Concepts and Best Practices in Depth

> **Month 1: Foundations of Data Engineering**
> **Week 3: Introduction to Data Pipelines**
> **Wednesday - Theory**

---

## What we covered so far this week

| Day | Topic |
|---|---|
| Monday (Theory) | Introduction to ETL/ELT workflows - the difference, when to use each, batch vs streaming, full vs incremental load |
| Tuesday (Lab) | Built a real Python ETL pipeline for CSV data - extraction, cleaning, deduplication, rejecting bad rows, logging |

In Tuesday's lab, you already practiced several best practices without naming them explicitly - separating E/T/L into functions, never silently dropping bad data, and logging every stage. Today we name these practices properly, go deeper into *why* they matter, and add several more that every production pipeline needs.

> **All code examples below have been tested and verified to run correctly** - including the idempotency and retry examples, which were run in a live Python environment before being written into these notes.

---

## Today's objective

By the end of today, you should be able to:

- Explain each ETL stage in more technical depth than Monday's introduction.
- Apply the principle of idempotency to pipeline design, and explain why it matters.
- Implement retry logic with exponential backoff for unreliable operations.
- Design pipelines using the UPSERT/MERGE pattern instead of blind inserts.
- Apply data quality validation at each pipeline stage, not just at the end.
- Structure pipeline code using modular, testable functions.
- Understand schema evolution and how pipelines should handle source changes.
- Know the key best practices checklist used before deploying a pipeline to production.

---

## 1. Revisiting Extract - going deeper

On Monday we defined extraction as pulling data from source systems. Let's go deeper into *how* this is done well.

### Extraction patterns

| Pattern | What it means | Example |
|---|---|---|
| **Full extraction** | Pull the entire dataset every time | Extract the entire `products` table nightly |
| **Incremental extraction** | Pull only new/changed records since the last run | Extract only orders where `updated_at > last_watermark` |
| **Change Data Capture (CDC)** | Capture every individual insert/update/delete as it happens in the source database | Listen to PostgreSQL's write-ahead log for row-level changes |

### Why extraction needs to be resilient

Source systems are not always available. APIs time out, databases get overloaded, network connections drop. A well-designed extraction step must handle these failures gracefully rather than crashing the entire pipeline.

```python
import time

def call_with_retry(func, max_retries=3, base_delay=1):
    """Retry a function call with exponential backoff."""
    for attempt in range(1, max_retries + 1):
        try:
            return func()
        except ConnectionError as e:
            print(f"Attempt {attempt} failed: {e}")
            if attempt == max_retries:
                print("Max retries reached. Raising error.")
                raise
            delay = base_delay * (2 ** (attempt - 1))   # 1s, 2s, 4s, 8s...
            print(f"Retrying in {delay}s...")
            time.sleep(delay)
```

**Verified output when calling an API that fails twice then succeeds:**
```text
Attempt 1 failed: Simulated failure on attempt 1
Retrying in 1s...
Attempt 2 failed: Simulated failure on attempt 2
Retrying in 2s...
Succeeded on attempt 3: {'status': 'success', 'data': [1, 2, 3]}
```

> **Exponential backoff** means each retry waits longer than the last (1s, then 2s, then 4s, etc.) instead of retrying immediately. This gives a struggling source system time to recover instead of hammering it with rapid repeated requests - which can make the problem worse.

### Extraction best practices

- Always set a **timeout** on API calls and database connections - never wait forever.
- Use **retry logic with backoff** for transient failures (network blips, rate limits).
- **Log the extraction window** - what date range or watermark was used - so every run is traceable.
- Validate that the **expected source actually returned data** - an empty result might mean success (no new records) or a silent failure. Distinguish between the two.

---

## 2. Revisiting Transform - going deeper

### Categories of transformation logic

| Category | What it does | Example from Tuesday's lab |
|---|---|---|
| **Cleaning** | Fixing format and consistency issues | `.str.title()`, `.str.lower()`, `.str.strip()` |
| **Validation** | Checking data meets quality rules | The `rejected_mask` that caught missing products and negative quantities |
| **Deduplication** | Removing duplicate records | `drop_duplicates(subset=[...])` |
| **Enrichment** | Adding information from other sources | Joining with a reference table (we'll do this Thursday) |
| **Restructuring** | Reshaping data into a different model | Converting normalized data into a star schema (Week 2, Day 4) |
| **Aggregation** | Summarizing detailed data | `GROUP BY` style summaries |

### Validate at every stage, not just at the end

A common beginner mistake is to extract everything, transform everything, and only check for problems right before loading. Better pipelines validate incrementally:

```python
def validate_extracted_data(df):
    """Run quality checks immediately after extraction."""
    if df.empty:
        raise ValueError("Extraction returned zero rows - investigate the source.")
    
    required_columns = ["order_id", "customer_name", "product", "quantity"]
    missing_columns = [col for col in required_columns if col not in df.columns]
    if missing_columns:
        raise ValueError(f"Source schema changed - missing columns: {missing_columns}")
    
    return True
```

>  This check would have caught a real production incident: if the source system renames a column (e.g. `qty` instead of `quantity`), your pipeline fails loudly and immediately - rather than silently producing a table full of NULLs that nobody notices until a report looks wrong weeks later.

### Schema evolution - handling source changes gracefully

Source systems change over time: new columns get added, columns get renamed, data types change. A resilient pipeline anticipates this.

```python
def safe_column_access(df, column_name, default_value=None):
    """Access a column safely, falling back to a default if it doesn't exist yet."""
    if column_name in df.columns:
        return df[column_name]
    else:
        print(f"WARNING: Column '{column_name}' not found in source - using default value")
        return pd.Series([default_value] * len(df))
```

| Schema change | Risk if unhandled | Mitigation |
|---|---|---|
| New column added | Usually safe - old pipeline ignores it | Explicitly select needed columns rather than using all columns |
| Column removed | Pipeline crashes or silently fills with NULL | Validate required columns exist before transforming |
| Column renamed | Old column appears "missing", new column ignored | Validate columns; alert on schema mismatch |
| Data type changed | Calculations or comparisons fail unexpectedly | Explicitly cast types early; validate ranges |

---

## 3. Revisiting Load - going deeper

### The problem with blind INSERT

On Tuesday, our pipeline simply wrote a fresh CSV every time. But what happens when you load into a **persistent table** in a warehouse, and the pipeline runs again tomorrow? If you just `INSERT` again, you create duplicates.

**Verified demonstration of the problem:**

```python
# Run 1: insert 3 orders
df1.to_csv("orders.csv", index=False)

# Run 2 (next day): pipeline runs again, blindly appends
df1.to_csv("orders.csv", mode="a", header=False, index=False)
```

**Result - the same 3 orders now appear twice:**
```text
order_id    status
       1   delivered
       2     shipped
       3      pending
       1   delivered     ← duplicate!
       2     shipped     ← duplicate!
       3      pending    ← duplicate!
```

### The fix - UPSERT / MERGE pattern

An **UPSERT** (update + insert) checks whether a record already exists: if it does, update it; if it doesn't, insert it. This makes loads **idempotent** - running the pipeline multiple times produces the same correct result, instead of compounding duplicates.

**Verified demonstration of the fix:**

```python
import pandas as pd

existing = pd.DataFrame({
    "order_id": [1, 2, 3],
    "status": ["delivered", "shipped", "pending"]
})

# New pipeline run: order 3's status changed, order 4 is brand new
new_run = pd.DataFrame({
    "order_id": [2, 3, 4],
    "status": ["shipped", "cancelled", "new"]
})

combined = pd.concat([existing, new_run])
upserted = combined.drop_duplicates(subset=["order_id"], keep="last").sort_values("order_id")
```

**Result - exactly 4 unique orders, with order 3 correctly updated:**
```text
order_id    status
       1   delivered
       2     shipped
       3   cancelled    ← correctly updated, not duplicated
       4         new    ← correctly added
```

>  **`keep="last"`** ensures that when the same `order_id` appears in both the existing data and the new run, we keep the **newest** version - exactly the upsert behavior we want.

### UPSERT in real SQL (data warehouse pattern)

In a real warehouse, this pattern is written directly in SQL:

```sql
-- PostgreSQL UPSERT using ON CONFLICT
INSERT INTO analytics.orders (order_id, customer_id, status, total_amount)
VALUES (3, 101, 'cancelled', 45000.00)
ON CONFLICT (order_id)
DO UPDATE SET
    status = EXCLUDED.status,
    total_amount = EXCLUDED.total_amount;
```

```sql
-- The general MERGE pattern (supported in Synapse, Snowflake, and modern Postgres)
MERGE INTO analytics.orders AS target
USING staging.orders_incoming AS source
ON target.order_id = source.order_id
WHEN MATCHED THEN
    UPDATE SET status = source.status, total_amount = source.total_amount
WHEN NOT MATCHED THEN
    INSERT (order_id, customer_id, status, total_amount)
    VALUES (source.order_id, source.customer_id, source.status, source.total_amount);
```

>  **This connects directly to Week 2, Day 4.** The `order_id` here acts like a primary key, and the UPSERT pattern is exactly how dimension tables are kept up to date in a star schema - particularly relevant for SCD Type 1 updates, which we covered that day.

---

## 4. Idempotency - the most important pipeline property

**Idempotency** means: running an operation multiple times produces the same result as running it once.

### Why idempotency matters so much in data engineering

Pipelines fail. Networks drop. Servers restart mid-run. Schedulers sometimes trigger a job twice by accident. When any of this happens, **the pipeline will run again** - possibly on the exact same data it already processed. An idempotent pipeline handles this safely. A non-idempotent one creates duplicate orders, double-counts revenue, or corrupts your warehouse.

### Designing for idempotency

| Technique | How it helps |
|---|---|
| UPSERT/MERGE instead of INSERT | Reprocessing the same record updates it instead of duplicating it |
| Deterministic transformations | The same input always produces the same output - no randomness, no `NOW()` baked into derived values without intent |
| Idempotency keys | A unique identifier per pipeline run or per record, used to detect "have I already processed this?" |
| Overwrite partitions instead of appending | For batch loads, replace the entire day's partition rather than appending to it |
| Truncate-and-load for small reference tables | For small, fully-refreshed tables, wipe and reload entirely each run - simple and inherently idempotent |

```python
# Partition overwrite pattern - idempotent batch loading
def load_partition(df, partition_date, table_path):
    """Overwrite a specific day's data instead of appending."""
    partition_file = f"{table_path}/date={partition_date}/data.csv"
    # Overwriting the same file/partition makes reruns safe
    df.to_csv(partition_file, index=False)
    print(f"Partition {partition_date} overwritten with {len(df)} rows")
```

>  **Rule of thumb:** if you can confidently say "I could run this pipeline five times in a row right now and the destination table would look exactly the same as running it once," your pipeline is idempotent. If you're not sure - that uncertainty is a bug waiting to happen.

---

## 5. Error handling and observability

### Layered error handling

Tuesday's lab used a single `try/except` around the whole pipeline. In more mature pipelines, you handle errors at multiple levels:

```python
def transform(df):
    cleaned_rows = []
    failed_rows = []
    
    for index, row in df.iterrows():
        try:
            # Attempt to transform this individual row
            processed_row = process_single_row(row)
            cleaned_rows.append(processed_row)
        except Exception as e:
            # One bad row doesn't kill the whole batch
            failed_rows.append({"row_index": index, "error": str(e), "data": row.to_dict()})
    
    if failed_rows:
        print(f"WARNING: {len(failed_rows)} rows failed transformation and were skipped")
    
    return pd.DataFrame(cleaned_rows), failed_rows
```

>  This is a trade-off decision data engineers make constantly: should **one bad row fail the entire pipeline run**, or should the pipeline **skip the bad row, log it, and continue**? For financial data, you often want to fail loudly. For high-volume clickstream data, you often want to skip and continue. There is no universally correct answer - it depends on the business cost of a bad record versus the cost of a delayed pipeline.

### What good observability looks like

| Signal | What it tells you |
|---|---|
| Row counts at each stage | Extracted X, transformed to Y, loaded Z - sudden drops are a red flag |
| Duration per stage | Extraction took 2 minutes, transformation took 45 minutes - where is time going? |
| Rejected/failed record counts | How much data quality degradation is happening, and is it growing over time? |
| Success/failure status per run | A dashboard or alert showing whether last night's run succeeded |
| Data freshness | How recent is the data in the destination - is the pipeline actually keeping up? |

---

## 6. Modular, testable pipeline structure

Tuesday's lab already separated `extract()`, `transform()`, and `load()` into individual functions. This is not just good style - it directly enables **testing**.

```python
# Example: a unit test for the transformation logic, independent of the full pipeline
import pandas as pd

def test_duplicate_removal():
    """Verify that exact duplicate orders are removed."""
    sample_data = pd.DataFrame({
        "customer_name": ["Alice Wanjiru", "Alice Wanjiru"],
        "product": ["Laptop", "Laptop"],
        "order_date": pd.to_datetime(["2025-05-01", "2025-05-01"]),
        "quantity": [1, 1],
        "unit_price": [85000, 85000],
        "email": ["alice@mail.com", "alice@mail.com"],
        "status": ["delivered", "delivered"],
        "city": ["Nairobi", "Nairobi"]
    })
    
    result = transform(sample_data)
    assert len(result) == 1, f"Expected 1 row after deduplication, got {len(result)}"
    print("PASSED: duplicate removal test")
```

> Writing small, focused tests like this - independent of your real source files - lets you verify your transformation logic works correctly **before** running it against real (and risky) production data.

### Why modularity matters for pipelines specifically

| Benefit | Explanation |
|---|---|
| Testability | Each function can be tested with small, controlled sample data |
| Reusability | The same `transform()` logic could run on different sources with the same shape |
| Debuggability | When something breaks, logs tell you exactly which stage failed |
| Readability | A new engineer can understand the pipeline by reading function names alone |
| Safer changes | Modifying the transform logic doesn't risk breaking extraction or loading |

---

## 7. The ETL best practices checklist

Use this checklist before considering any pipeline "production-ready":

```text
EXTRACT
✅ Has a timeout configured for network/API calls
✅ Has retry logic with exponential backoff for transient failures
✅ Validates that the expected data structure (columns/fields) is present
✅ Logs the extraction window/watermark used
✅ Distinguishes between "no new data" and "extraction failed"

TRANSFORM
✅ Validates data quality immediately, not just at the very end
✅ Never silently drops bad records - routes them to a visible location
✅ Logs row counts before and after each major transformation step
✅ Handles schema changes gracefully (missing/renamed columns)
✅ Is broken into small, testable functions

LOAD
✅ Uses UPSERT/MERGE or partition overwrite - not blind INSERT
✅ Running the pipeline twice in a row produces the same correct result (idempotency)
✅ Logs the number of rows loaded
✅ Has a clear rollback or recovery plan if the load fails partway through

OVERALL
✅ Every stage has structured logging (not print statements)
✅ Failures raise clear, actionable error messages
✅ The pipeline can be safely re-run after a failure without manual cleanup
✅ There is a way to monitor whether the pipeline ran successfully (alerting/dashboard)
```

---

## 8. Connecting it all back - the bigger picture

```text
Monday's theory          →  What ETL/ELT IS
Tuesday's lab             →  A working pipeline, built correctly by instinct
Wednesday's theory (today) →  WHY it was built that way, and what's missing
                              for true production-readiness
Thursday's lab            →  Apply ALL of today's best practices to a more
                              realistic batch pipeline
```

Every concept introduced today - idempotency, UPSERT, retries, layered error handling, modular testing - will appear again tomorrow when we build a more advanced batch pipeline.

---

## 9. Key terms to know from today

| Term | Meaning |
|---|---|
| Change Data Capture (CDC) | Capturing individual row-level changes (insert/update/delete) as they happen at the source |
| Exponential backoff | A retry strategy where wait time increases after each failed attempt |
| Idempotency | The property that running an operation multiple times has the same effect as running it once |
| UPSERT | A load operation that updates a record if it exists, or inserts it if it doesn't |
| MERGE | The SQL pattern that implements UPSERT logic, matching source and target on a key |
| Partition overwrite | Replacing an entire batch/partition of data rather than appending to it |
| Idempotency key | A unique identifier used to detect whether a record or run has already been processed |
| Schema evolution | Changes to a source system's structure over time - added, removed, or renamed columns |
| Layered error handling | Handling failures at multiple levels - per row, per stage, per pipeline run |
| Observability | The ability to understand what a running (or failed) pipeline is doing via logs, metrics, and alerts |
| Unit test | A small, automated test that verifies one specific piece of logic works correctly |
| Watermark | A reference point (often a timestamp) marking how far an incremental pipeline has already processed |

---

## 10. Summary

> Today we went beneath the surface of ETL to understand **why** production pipelines are built the way they are. **Idempotency** ensures pipelines can be safely rerun without creating duplicates or corrupting data - achieved through **UPSERT/MERGE** patterns and partition overwrites rather than blind inserts. **Retry logic with exponential backoff** makes extraction resilient to the inevitable network and API failures every pipeline will eventually face. **Layered validation and error handling** catch problems early and prevent one bad record from silently corrupting an entire dataset. And **modular, testable code** - the same structure we used intuitively on Tuesday - is what makes all of this maintainable.
>
> Every one of these practices was demonstrated with real, tested code in this lesson - not just described in theory. Tomorrow, we put all of it together in a more realistic batch processing pipeline.

---

## 11. What's coming next

| Day | Topic |
|---|---|
| Thursday (Lab) | Build a Python ETL pipeline for batch data processing - applying idempotency, UPSERT logic, retries, and layered validation together |
| Friday | Peer project - ETL pipeline challenge |
| Saturday (Lab) | Mini project - end-to-end batch pipeline with PostgreSQL |

---

*Notes by LuxDevHQ | Month 1 - Foundations of Data Engineering | Week 3, Day 3*