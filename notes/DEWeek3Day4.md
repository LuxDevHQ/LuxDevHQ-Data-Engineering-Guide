# Week 3, Day 4 - Lab: Build a Python ETL Pipeline for Batch Data Processing

> **Month 1: Foundations of Data Engineering**
> **Week 3: Introduction to Data Pipelines**
> **Thursday - Lab**

---

## What we covered so far this week

| Day | Topic |
|---|---|
| Monday (Theory) | ETL/ELT workflows, batch vs streaming, full vs incremental load |
| Tuesday (Lab) | A simple single-file CSV ETL pipeline - extract, clean, reject bad rows, load |
| Wednesday (Theory) | Idempotency, UPSERT/MERGE, retry logic, layered validation, modular pipeline design |

> **Today we combine everything.** We are building a pipeline that processes an entire **batch of files at once** - the way real systems actually receive data: multiple branches, multiple sources, files arriving on a schedule, and the same pipeline needing to run safely day after day without creating duplicates.
>
> **Every behavior described in this lesson was tested in a live Python environment before being written here** - including a real bug we found and fixed during testing, documented below so you learn from it too.

---

## Today's objective

By the end of today, you should be able to:

- Process **multiple incoming files** in a single pipeline run.
- Build a folder-based batch processing structure: incoming → processed → archive → rejected.
- Apply the **idempotent UPSERT pattern** from Wednesday's theory in real working code.
- Archive processed source files so they are never accidentally reprocessed.
- Verify a pipeline's idempotency by deliberately rerunning it and checking the results.
- Recognize and fix a real data validation bug - the same kind you will encounter in your own pipelines.

---

## 1. The batch processing folder structure

Real-world batch pipelines almost always use a folder-based pattern like this:

```text
data/
  ├── incoming/      → new files waiting to be processed land here
  ├── processed/      → the clean, loaded master dataset
  ├── archive/        → source files already processed, kept for record-keeping
  └── rejected/        → records that failed validation, timestamped per run
```

### Why this structure matters

| Folder | Purpose |
|---|---|
| `incoming/` | Acts as a "to-do list" - anything here has not been processed yet |
| `processed/` | The destination - our equivalent of a data warehouse table |
| `archive/` | Proof of what was already processed - prevents accidental reprocessing and supports audits |
| `rejected/` | Visibility into data quality issues - never silently deleted, exactly as we practiced on Tuesday |

> This mirrors exactly how cloud pipelines work - an S3 bucket with an `incoming/` prefix triggers a Lambda function, processed files move to `archive/`, and rejected records go to a dead-letter location. The folder names change, but the pattern is identical.

---

## 2. Set up the project

```bash
mkdir batch_etl_pipeline
cd batch_etl_pipeline
mkdir -p data/incoming data/processed data/archive data/rejected
pip install pandas
```

Your structure should look like:

```text
batch_etl_pipeline/
  ├── data/
  │     ├── incoming/
  │     ├── processed/
  │     ├── archive/
  │     └── rejected/
  └── batch_pipeline.py
```

---

## 3. Create the source files - simulating multiple branches

In a real company, different branches or systems each drop their own daily file. Let's simulate two branches submitting their orders for the same day.

**`data/incoming/branch_nairobi_2025-06-01.csv`:**
```csv
order_id,branch,customer_name,product,quantity,unit_price,order_date,status
N001,Nairobi,Alice Wanjiru,Laptop,1,85000,2025-06-01,delivered
N002,Nairobi,Brian Otieno,Headphones,2,3500,2025-06-01,delivered
N003,Nairobi,Carol Muthoni,Smartphone,1,45000,2025-06-01,DELIVERED
N004,Nairobi,,Mouse,1,1200,2025-06-01,shipped
N005,Nairobi,David Kimani,Desk,-1,35000,2025-06-01,pending
```

**`data/incoming/branch_mombasa_2025-06-01.csv`:**
```csv
order_id,branch,customer_name,product,quantity,unit_price,order_date,status
M001,Mombasa,Eve Achieng,Backpack,3,4500,2025-06-01,delivered
M002,Mombasa,Frank Mwangi,Laptop,1,85000,2025-06-01,delivered
M003,Mombasa,Grace Njeri,Notebook,5,150,2025-06-01,pending
```

### Spot the issues (same exercise as Tuesday, slightly harder)

| Row | Issue |
|---|---|
| N003 | Status is `DELIVERED` - inconsistent casing |
| N004 | `customer_name` is missing entirely |
| N005 | `quantity` is `-1` - invalid |

This time the issues are split across **two separate files** - our pipeline needs to combine them first, then validate.

---

## 4. Building the batch pipeline

### Step 1 - Imports and configuration

```python
import pandas as pd
import logging
import os
import glob
import shutil
from datetime import datetime

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)s | %(message)s"
)
logger = logging.getLogger("batch_etl_pipeline")

INCOMING_DIR = "data/incoming"
PROCESSED_FILE = "data/processed/orders_master.csv"
REJECTED_DIR = "data/rejected"
ARCHIVE_DIR = "data/archive"
```

>  **New here:** `glob` is a Python module used to find files matching a pattern (like `*.csv`) - essential when you don't know in advance how many files will be waiting in the incoming folder. `shutil` is used to move files between folders.

### Step 2 - Discover files to process

```python
def discover_batch_files(directory):
    """Find all CSV files waiting to be processed in the incoming directory."""
    pattern = os.path.join(directory, "*.csv")
    files = sorted(glob.glob(pattern))
    logger.info(f"Discovered {len(files)} file(s) to process: {[os.path.basename(f) for f in files]}")
    return files
```

This is new compared to Tuesday - instead of hardcoding one filename, we **discover** whatever files happen to be waiting. This is exactly how a real pipeline behaves: it doesn't know in advance how many branches submitted files today.

### Step 3 - Extract from multiple files at once

```python
def extract_batch(files):
    """Extract and combine all discovered files into a single DataFrame, tracking origin."""
    all_dfs = []
    for file_path in files:
        logger.info(f"Extracting {os.path.basename(file_path)}")
        df = pd.read_csv(file_path)
        df["source_file"] = os.path.basename(file_path)
        all_dfs.append(df)

    if not all_dfs:
        logger.warning("No files found to extract - returning empty DataFrame")
        return pd.DataFrame()

    combined = pd.concat(all_dfs, ignore_index=True)
    logger.info(f"Combined extraction: {len(combined)} total rows from {len(files)} file(s)")
    return combined
```

**What's new here:**
- We loop through every discovered file and read each one individually.
- We add a `source_file` column to every row - this is critical for **traceability**. If something looks wrong later, you can immediately tell which file it came from.
- `pd.concat(all_dfs, ignore_index=True)` stacks all the individual DataFrames into one combined DataFrame.

### Step 4 - Transform with a real bug fix included

```python
def transform_batch(df):
    """Clean, validate, and standardize the combined batch."""
    if df.empty:
        logger.warning("Transform received an empty DataFrame - skipping")
        return df, pd.DataFrame()

    logger.info("Starting batch transformation")
    original_count = len(df)

    # IMPORTANT: check for missing customer names BEFORE converting to string.
    # A NaN value does NOT reliably become the string "nan" after astype(str)
    # in every situation - never check for missing values AFTER casting to string.
    # Always check .isna() on the original column first.
    missing_customer = df["customer_name"].isna() | (df["customer_name"].astype(str).str.strip() == "")

    # Standardize text fields
    df["customer_name"] = df["customer_name"].astype(str).str.strip().str.title()
    df["status"] = df["status"].astype(str).str.strip().str.lower()
    df["branch"] = df["branch"].astype(str).str.strip().str.title()

    # Parse dates
    df["order_date"] = pd.to_datetime(df["order_date"], errors="coerce")

    # Separate good and bad rows
    rejected_mask = (
        missing_customer
        | df["quantity"].isna()
        | (df["quantity"] <= 0)
        | df["order_date"].isna()
    )
    rejected_df = df[rejected_mask].copy()
    clean_df = df[~rejected_mask].copy()

    logger.info(f"Validation: {len(clean_df)} valid rows, {len(rejected_df)} rejected rows")

    # Calculated column
    clean_df["subtotal"] = clean_df["quantity"] * clean_df["unit_price"]
    clean_df["processed_at"] = datetime.now().isoformat()

    logger.info(f"Transformation complete: {original_count} -> {len(clean_df)} clean rows")
    return clean_df, rejected_df
```

###  A real bug we found while testing this lesson

While preparing this lab, the **first version** of this validation logic checked for missing customer names like this:

```python
#  THE BUGGY VERSION - do not use this
rejected_mask = (
    df["customer_name"].isin(["Nan", "None", ""])   # checking AFTER casting to string
    | df["quantity"].isna()
    | (df["quantity"] <= 0)
)
```

When we ran this against our test data, **N004 - the row with a missing customer name - slipped through into the clean data instead of being rejected.** Here's why: converting a missing (`NaN`) value to a string does not reliably produce the exact string `"Nan"` that we were checking for, so the comparison silently failed to catch it.

**The fix:** check `.isna()` on the original column **before** any string conversion happens:

```python
#  THE FIXED VERSION
missing_customer = df["customer_name"].isna() | (df["customer_name"].astype(str).str.strip() == "")
```

> **This is exactly the kind of bug that slips into real pipelines.** It produced no error message - the pipeline ran "successfully" and the bad row simply ended up in the wrong place. The only way we caught it was by **checking the actual output row counts** against what we expected, rather than trusting that "no errors thrown" meant "the logic is correct." This is precisely why Wednesday's lesson emphasized validating row counts at every stage - it's not optional theory, it's how real bugs get caught.

### Step 5 - Load with the idempotent UPSERT pattern

This is where Wednesday's theory becomes real, working code.

```python
def load_batch(clean_df, rejected_df):
    """Idempotently load clean data into the master file, and save rejects."""
    if not clean_df.empty:
        if os.path.exists(PROCESSED_FILE):
            existing = pd.read_csv(PROCESSED_FILE)
            combined = pd.concat([existing, clean_df], ignore_index=True)
            # Idempotent upsert: keep latest version of each order_id
            before = len(combined)
            combined = combined.drop_duplicates(subset=["order_id"], keep="last")
            logger.info(f"Upsert: {before} rows combined -> {len(combined)} unique rows after dedup")
        else:
            combined = clean_df

        combined.to_csv(PROCESSED_FILE, index=False)
        logger.info(f"Loaded {len(clean_df)} new/updated rows. Master file now has {len(combined)} total rows")

    if not rejected_df.empty:
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        rejected_path = os.path.join(REJECTED_DIR, f"rejected_{timestamp}.csv")
        rejected_df.to_csv(rejected_path, index=False)
        logger.warning(f"Saved {len(rejected_df)} rejected rows to {rejected_path}")
```

**What's happening here, step by step:**
1. If a master file already exists, load it.
2. Combine the existing data with the newly cleaned batch.
3. **`drop_duplicates(subset=["order_id"], keep="last")`** - this is the UPSERT. If an `order_id` already existed, the **newest** version (from this run) replaces it. If it's a brand-new `order_id`, it's simply added.
4. Save the combined result back to the master file.
5. Rejected rows get their own **timestamped** file each run - so rejects from different days never overwrite each other.

### Step 6 - Archive processed files

```python
def archive_processed_files(files):
    """Move processed source files to the archive folder so they aren't reprocessed."""
    for file_path in files:
        filename = os.path.basename(file_path)
        destination = os.path.join(ARCHIVE_DIR, filename)
        shutil.move(file_path, destination)
        logger.info(f"Archived {filename}")
```

This moves each processed file out of `incoming/` and into `archive/`. The next time the pipeline runs, `discover_batch_files()` will find an empty (or only newly-arrived) `incoming/` folder - exactly as it should.

### Step 7 - Orchestrate the full pipeline

```python
def run_batch_pipeline():
    start_time = datetime.now()
    logger.info("=== BATCH ETL PIPELINE STARTED ===")

    try:
        files = discover_batch_files(INCOMING_DIR)

        if not files:
            logger.info("No new files to process. Exiting cleanly.")
            return

        raw_df = extract_batch(files)
        clean_df, rejected_df = transform_batch(raw_df)
        load_batch(clean_df, rejected_df)
        archive_processed_files(files)

        duration = (datetime.now() - start_time).total_seconds()
        logger.info(f"=== BATCH PIPELINE SUCCESS in {duration:.2f}s ===")

    except Exception as e:
        logger.error(f"Batch pipeline failed: {e}")
        raise


if __name__ == "__main__":
    run_batch_pipeline()
```

>  Notice the early exit: `if not files: ... return`. A well-behaved batch pipeline does nothing destructive when there is genuinely nothing new to process - it should not error, and it should not touch the master file.

---

## 5. Running the pipeline - four real test runs

We tested this exact pipeline four times in sequence to prove it behaves correctly. Follow along with the same scenarios.

### Run 1 - first batch (two branches, same day)

```bash
python batch_pipeline.py
```

**Actual output:**
```text
=== BATCH ETL PIPELINE STARTED ===
Discovered 2 file(s) to process: ['branch_mombasa_2025-06-01.csv', 'branch_nairobi_2025-06-01.csv']
Extracting branch_mombasa_2025-06-01.csv
Extracting branch_nairobi_2025-06-01.csv
Combined extraction: 8 total rows from 2 file(s)
Starting batch transformation
Validation: 6 valid rows, 2 rejected rows
Transformation complete: 8 -> 6 clean rows
Loaded 6 new/updated rows. Master file now has 6 total rows
Saved 2 rejected rows to data/rejected/rejected_20260628_112418.csv
Archived branch_mombasa_2025-06-01.csv
Archived branch_nairobi_2025-06-01.csv
=== BATCH PIPELINE SUCCESS in 0.01s ===
```

**Result:** 8 rows in across 2 files → 6 clean, 2 correctly rejected (N004 missing name, N005 negative quantity). Master file has 6 rows. Both source files moved to `archive/`.

### Run 2 - a new day's file arrives, including an UPDATE to an existing order

A new file arrives the next day. It includes order `N003` again - but this time with its status corrected from the inconsistent `DELIVERED` to a confirmed `delivered`, plus two brand-new orders.

```csv
order_id,branch,customer_name,product,quantity,unit_price,order_date,status
N003,Nairobi,Carol Muthoni,Smartphone,1,45000,2025-06-01,delivered
N006,Nairobi,Henry Odhiambo,Pens,10,50,2025-06-02,delivered
N007,Nairobi,Grace Njeri,Chair,1,18000,2025-06-02,shipped
```

**Actual output:**
```text
Discovered 1 file(s) to process: ['branch_nairobi_2025-06-02.csv']
Combined extraction: 3 total rows from 1 file(s)
Validation: 3 valid rows, 0 rejected rows
Upsert: 9 rows combined -> 8 unique rows after dedup
Loaded 3 new/updated rows. Master file now has 8 total rows
Archived branch_nairobi_2025-06-02.csv
=== BATCH PIPELINE SUCCESS in 0.01s ===
```

**Result:** 6 existing + 3 new = 9 combined rows, deduplicated to **8** - because `N003` already existed and got **updated**, not duplicated. We confirmed this directly:

```bash
grep -c "^N003" data/processed/orders_master.csv
# Output: 1
```

Exactly one `N003` row exists, and it now shows the corrected `delivered` status.

### Run 3 - rerun with nothing new (idempotency check #1)

```bash
python batch_pipeline.py
```

**Actual output:**
```text
Discovered 0 file(s) to process: []
No new files to process. Exiting cleanly.
```

**Result:** Master file row count confirmed unchanged at **8**. The pipeline correctly does nothing when there is nothing to do.

### Run 4 - accidental re-upload of an already-processed file (idempotency check #2)

This is the test that matters most. We copied `branch_nairobi_2025-06-01.csv` - a file already processed and archived - back into `incoming/`, simulating someone accidentally re-uploading it.

**Actual output:**
```text
Discovered 1 file(s) to process: ['branch_nairobi_2025-06-01.csv']
Combined extraction: 5 total rows from 1 file(s)
Validation: 3 valid rows, 2 rejected rows
Upsert: 11 rows combined -> 8 unique rows after dedup
Loaded 3 new/updated rows. Master file now has 8 total rows
Saved 2 rejected rows to data/rejected/rejected_20260628_112453.csv
Archived branch_nairobi_2025-06-01.csv
=== BATCH PIPELINE SUCCESS in 0.02s ===
```

**Result:** 8 existing + 3 reprocessed clean rows = 11 combined, deduplicated back down to exactly **8** - confirmed with:

```bash
grep -c "^N001" data/processed/orders_master.csv
# Output: 1
```

**No duplicates were created.** This is idempotency working exactly as Wednesday's theory described - reprocessing the same file twice produced the same correct result, not a doubled dataset.

---

## 6. How this connects to the whole week

| Concept from earlier this week | Where it appears today |
|---|---|
| ETL order (Monday) | `extract_batch()` → `transform_batch()` → `load_batch()`, in that order |
| Batch processing (Monday) | The entire pipeline processes a folder of files at once, not a continuous stream |
| Never silently drop bad data (Tuesday) | Rejected rows get their own timestamped file every run |
| Modular functions (Tuesday & Wednesday) | Each stage is its own tested, single-purpose function |
| Idempotency (Wednesday) | Proven directly via Run 3 and Run 4 - reruns do not create duplicates |
| UPSERT pattern (Wednesday) | `drop_duplicates(subset=["order_id"], keep="last")` |
| Validate at every stage (Wednesday) | Row counts logged before and after extraction, transformation, and loading |
| Schema/data quality bugs happen (Wednesday) | We found and fixed a real one together, live, in this lesson |

---

## 7. Try it yourself

1. **Add a third branch file** for `2025-06-03` with at least one new order and one data quality issue. Run the pipeline and confirm the master file and rejected file update correctly.
2. **Deliberately corrupt a date** (e.g. `not-a-date` in the `order_date` column) and confirm the row is correctly rejected rather than crashing the pipeline.
3. **Write a quick check script** that reads `orders_master.csv` and asserts there are no duplicate `order_id` values - a simple data quality test you can run after every pipeline execution.
4. **Simulate two files arriving with the *same* order_id but different data** (e.g. different quantities) - predict which one wins based on the UPSERT logic, then verify your prediction.

---

## 8. Key terms to know from today

| Term | Meaning |
|---|---|
| `glob` | A Python module for finding files that match a pattern, such as all `.csv` files in a folder |
| Batch discovery | The step where a pipeline finds all files waiting to be processed |
| Source traceability | Recording which file/source a row came from, to support debugging |
| `shutil.move()` | Moves a file from one location to another - used here to archive processed files |
| Master file/table | The destination dataset that accumulates clean records across multiple pipeline runs |
| Timestamped rejects | Saving rejected records to a uniquely named file per run, so they are never overwritten |
| Idempotency verification | Deliberately rerunning a pipeline to confirm it does not produce incorrect duplicate results |

---

## 9. Summary

> Today we built a complete batch ETL pipeline that processes **multiple incoming files at once**, validates and cleans the combined data, and loads it using an **idempotent UPSERT pattern** - exactly the best practices introduced in theory on Wednesday. We proved, through four real test runs, that the pipeline handles new data, updates to existing records, empty runs, and even accidental file re-uploads - all without creating duplicates or losing data.
>
> We also hit a real bug during testing - a missing-value check that silently failed - and fixed it the same way you will in your career: by checking actual output against expected output, not by trusting that a lack of errors means correct behavior. This habit, more than any single line of code, is what separates a reliable data engineer from one who gets surprised by their own pipeline in production.

---

## 10. What's coming next

| Day | Topic |
|---|---|
| Friday | Peer project - ETL pipeline challenge |
| Saturday (Lab) | Mini project - end-to-end batch pipeline with PostgreSQL |
| Week 4 | Workflow orchestration - introduction to Apache Airflow |

---

*Notes by LuxDevHQ | Month 1 - Foundations of Data Engineering | Week 3, Day 4*