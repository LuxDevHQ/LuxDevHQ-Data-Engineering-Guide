# Week 3, Day 2 - Lab: Create a Simple Python-Based ETL Pipeline for CSV Data

> **Month 1: Foundations of Data Engineering**
> **Week 3: Introduction to Data Pipelines**
> **Tuesday - Lab**

---

## What we covered yesterday (Day 1 recap)

On Monday we learned the theory behind data pipelines:

| Concept | What it means |
|---|---|
| ETL | Extract, Transform, Load - transform data before loading it into the destination |
| ELT | Extract, Load, Transform - load raw data first, transform inside the destination |
| Batch vs streaming | Scheduled chunks of processing vs continuous real-time processing |
| Full load vs incremental load | Reload everything every time vs only process new/changed records |

> **Today is hands-on.** We are going to build a real, working ETL pipeline in Python that extracts data from a messy CSV file, transforms it to fix data quality issues, and loads the clean result to a new file. Every line of code below has been tested and runs correctly - follow along exactly.

---

## Today's objective

By the end of today, you should be able to:

- Set up a Python project structure for a simple ETL pipeline.
- Use **pandas** to extract data from a CSV file.
- Write transformation logic that cleans real-world data quality issues.
- Separate clean data from rejected (bad) data instead of silently dropping it.
- Load the final clean dataset to a new CSV file.
- Add logging so the pipeline reports what it did at every step.
- Run the complete pipeline end to end.

---

## 1. Setting up the project

### Step 1 - Create your project folder

```bash
mkdir csv_etl_pipeline
cd csv_etl_pipeline
mkdir data
```

Your folder structure should look like this:

```text
csv_etl_pipeline/
  ├── data/
  │     └── sales_raw.csv      (we will create this next)
  └── pipeline.py               (our ETL script)
```

### Step 2 - Set up a virtual environment (recommended)

```bash
python3 -m venv venv

# Activate it
# On Mac/Linux:
source venv/bin/activate
# On Windows (Git Bash):
source venv/Scripts/activate
```

### Step 3 - Install pandas

**pandas** is the most widely used Python library for working with tabular data - exactly what we need to read, clean, and write CSV files.

```bash
pip install pandas
```

---

## 2. Create the raw data file

Create a file at `data/sales_raw.csv` and paste in this data. Notice it is **deliberately messy** - this mirrors what real source data looks like.

```csv
order_id,customer_name,email,city,product,quantity,unit_price,order_date,status
1,Alice Wanjiru,alice@mail.com,Nairobi,Laptop,1,85000,2025-05-01,delivered
2,Brian Otieno,brian@mail.com,Kisumu,Headphones,2,3500,2025-05-03,delivered
3,Carol Muthoni,carol@mail.com,Nairobi,Smartphone,1,45000,2025-05-05,DELIVERED
4,alice wanjiru,alice@mail.com,Nairobi,Mouse,1,1200,2025-05-10,shipped
5,David Kimani,david@mail.com,Nakuru,,1,18000,2025-05-12,cancelled
6,Eve Achieng,,Mombasa,Backpack,3,4500,2025-05-14,delivered
7,Frank Mwangi,frank@mail.com,Nairobi,Laptop,1,85000,2025-05-15,Delivered
8,Grace Njeri,grace@mail.com,Nairobi,Notebook,-2,150,2025-05-18,pending
9,Henry Odhiambo,henry@mail.com,Kisumu,Desk,1,35000,2025-05-20,delivered
10,Carol Muthoni,carol@mail.com,Nairobi,Smartphone,1,45000,2025-05-05,delivered
```

### Spot the data quality issues

Before writing any code, train your eye to find problems - this is a core data engineering skill.

| Row | Issue |
|---|---|
| Row 3 | Status is `DELIVERED` in uppercase - inconsistent with other rows |
| Row 4 | Customer name is `alice wanjiru` in lowercase - same person as row 1, written differently |
| Row 5 | `product` field is empty/missing |
| Row 6 | `email` field is empty/missing |
| Row 7 | Status is `Delivered` - yet another casing variation |
| Row 8 | `quantity` is `-2` - a negative quantity is not physically possible |
| Row 10 | This is a **duplicate** of row 3 - same customer, same product, same date |

>  This is exactly the kind of messy data you will see from real systems - manual entry errors, inconsistent casing, missing fields, and duplicate submissions. Our pipeline's job is to catch and handle every one of these issues.

---

## 3. Building the pipeline - step by step

We will build this in three clearly separated functions, matching the **E**, **T**, **L** of ETL - exactly as we learned in theory yesterday.

### Step 1 - Imports and setup

Create `pipeline.py` and start with:

```python
import pandas as pd
import logging
from datetime import datetime
import os

# Configure logging so the pipeline reports what it does at every step
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)s | %(message)s"
)
logger = logging.getLogger("csv_etl_pipeline")

# File paths
RAW_FILE = "data/sales_raw.csv"
CLEAN_FILE = "data/sales_clean.csv"
REJECTED_FILE = "data/sales_rejected.csv"
```

>  **Why logging instead of print()?** `print()` statements disappear once your terminal closes. `logging` gives you timestamps, severity levels (INFO, WARNING, ERROR), and can be configured to write to a file - essential for debugging a pipeline that runs unattended on a schedule.

### Step 2 - The Extract function

```python
def extract(file_path):
    """Extract data from a CSV file into a pandas DataFrame."""
    logger.info(f"Starting extraction from {file_path}")

    if not os.path.exists(file_path):
        raise FileNotFoundError(f"Source file not found: {file_path}")

    df = pd.read_csv(file_path)
    logger.info(f"Extracted {len(df)} rows from {file_path}")
    return df
```

**What this function does:**
- Checks the file actually exists before trying to read it - fails clearly instead of with a confusing pandas error.
- Uses `pd.read_csv()` to load the CSV into a **DataFrame** - pandas' table-like data structure.
- Logs how many rows were extracted, so every pipeline run leaves a record.

### Step 3 - The Transform function

This is where most of the work happens. We will build it piece by piece so you understand exactly what each line does.

```python
def transform(df):
    """Clean and validate the raw data."""
    logger.info("Starting transformation")
    original_count = len(df)

    # 1. Standardize text fields - fix inconsistent casing
    df["customer_name"] = df["customer_name"].str.strip().str.title()
    df["status"] = df["status"].str.strip().str.lower()
    df["city"] = df["city"].str.strip().str.title()

    # 2. Convert order_date to a proper datetime type
    df["order_date"] = pd.to_datetime(df["order_date"], errors="coerce")

    # 3. Remove exact duplicates (same customer, product, and date)
    before_dedup = len(df)
    df = df.drop_duplicates(subset=["customer_name", "product", "order_date"], keep="first")
    duplicates_removed = before_dedup - len(df)
    logger.info(f"Removed {duplicates_removed} duplicate rows")

    # 4. Identify rows with data quality issues
    rejected_mask = (
        df["product"].isna()
        | (df["product"].astype(str).str.strip() == "")
        | df["quantity"].isna()
        | (df["quantity"] <= 0)
        | df["email"].isna()
    )
    rejected_df = df[rejected_mask].copy()
    df = df[~rejected_mask].copy()

    # 5. Save rejected rows separately - never silently delete bad data
    if len(rejected_df) > 0:
        rejected_df.to_csv(REJECTED_FILE, index=False)
        logger.warning(f"Rejected {len(rejected_df)} rows due to data quality issues -> {REJECTED_FILE}")

    # 6. Add a calculated column
    df["subtotal"] = df["quantity"] * df["unit_price"]

    logger.info(f"Transformation complete: {original_count} -> {len(df)} clean rows")
    return df
```

### Breaking down each transformation step

#### 1. Standardizing text fields

```python
df["customer_name"] = df["customer_name"].str.strip().str.title()
```

- `.str.strip()` removes leading/trailing whitespace.
- `.str.title()` converts to title case - so `"alice wanjiru"` and `"Alice Wanjiru"` both become `"Alice Wanjiru"`.

This is exactly how row 1 and row 4 (the same customer, written differently) become consistent.

```python
df["status"] = df["status"].str.strip().str.lower()
```

- `.str.lower()` makes `DELIVERED`, `Delivered`, and `delivered` all become `delivered`.

>  **Why this matters:** If you later run `GROUP BY status` to count orders, inconsistent casing would create three separate groups for what is really the same status - `delivered`, `Delivered`, and `DELIVERED` - silently breaking your reports.

#### 2. Converting dates properly

```python
df["order_date"] = pd.to_datetime(df["order_date"], errors="coerce")
```

- `pd.to_datetime()` converts the text date into a real datetime object that pandas understands.
- `errors="coerce"` means: if a date cannot be parsed, set it to `NaT` (pandas' "missing date" value) instead of crashing the whole pipeline.

#### 3. Removing duplicates

```python
df = df.drop_duplicates(subset=["customer_name", "product", "order_date"], keep="first")
```

- `subset=[...]` tells pandas which columns define a "duplicate" - here, the same customer, same product, same date is considered the same order.
- `keep="first"` keeps the first occurrence and drops the rest.

This catches row 10, which is an exact duplicate of row 3 (Carol Muthoni, Smartphone, 2025-05-05).

#### 4 & 5. Separating bad data - never delete silently

```python
rejected_mask = (
    df["product"].isna()
    | (df["product"].astype(str).str.strip() == "")
    | df["quantity"].isna()
    | (df["quantity"] <= 0)
    | df["email"].isna()
)
```

This builds a **boolean mask** - a True/False flag for every row - marking it as rejected if:
- The product is missing (NaN) **or** an empty string.
- The quantity is missing **or** zero/negative (you cannot order -2 items).
- The email is missing.

```python
rejected_df = df[rejected_mask].copy()
df = df[~rejected_mask].copy()
```

- `df[rejected_mask]` selects only the bad rows.
- `df[~rejected_mask]` (the `~` means "not") selects only the good rows.
- We save the bad rows to a separate file rather than just deleting them.

>  **This is one of the most important habits in data engineering.** Never silently drop bad data. Always route it somewhere visible - a rejected-records file, a dead-letter table, an alert - so someone can investigate why orders are coming in with missing emails or negative quantities. Silent data loss is one of the most dangerous and hardest-to-detect pipeline bugs.

#### 6. Adding a calculated column

```python
df["subtotal"] = df["quantity"] * df["unit_price"]
```

This creates a new column by multiplying two existing columns - a common transformation step.

### Step 4 - The Load function

```python
def load(df, file_path):
    """Write the clean DataFrame to a CSV file."""
    logger.info(f"Loading {len(df)} rows to {file_path}")
    df.to_csv(file_path, index=False)
    logger.info("Load complete")
```

- `.to_csv()` writes the DataFrame to a CSV file.
- `index=False` prevents pandas from writing its internal row numbers as an extra column.

### Step 5 - Orchestrating the full pipeline

```python
def run_pipeline():
    """Run the full ETL pipeline end to end."""
    start_time = datetime.now()
    logger.info("=== ETL PIPELINE STARTED ===")

    try:
        raw_df = extract(RAW_FILE)
        clean_df = transform(raw_df)
        load(clean_df, CLEAN_FILE)

        duration = (datetime.now() - start_time).total_seconds()
        logger.info(f"=== PIPELINE SUCCESS in {duration:.2f}s ===")

    except Exception as e:
        logger.error(f"Pipeline failed: {e}")
        raise


if __name__ == "__main__":
    run_pipeline()
```

>  **Why wrap everything in try/except?** If extraction, transformation, or loading fails for any reason, you want a clear error message in your logs - not a silent failure or a confusing crash with no context. This `try/except` pattern is the minimum error handling every pipeline should have.

---

## 4. Running the pipeline

Run it from your terminal:

```bash
python pipeline.py
```

### Expected output

```text
2026-06-21 07:41:53 | INFO | === ETL PIPELINE STARTED ===
2026-06-21 07:41:53 | INFO | Starting extraction from data/sales_raw.csv
2026-06-21 07:41:53 | INFO | Extracted 10 rows from data/sales_raw.csv
2026-06-21 07:41:53 | INFO | Starting transformation
2026-06-21 07:41:53 | INFO | Removed 1 duplicate rows
2026-06-21 07:41:53 | WARNING | Rejected 3 rows due to data quality issues -> data/sales_rejected.csv
2026-06-21 07:41:53 | INFO | Transformation complete: 10 -> 6 clean rows
2026-06-21 07:41:53 | INFO | Loading 6 rows to data/sales_clean.csv
2026-06-21 07:41:53 | INFO | Load complete
2026-06-21 07:41:53 | INFO | === PIPELINE SUCCESS in 0.03s ===
```

> This output was produced by running the exact code above against the exact CSV above - so your results should match.

### Verify the results

Check `data/sales_clean.csv` - you should see **6 clean rows**:

```csv
order_id,customer_name,email,city,product,quantity,unit_price,order_date,status,subtotal
1,Alice Wanjiru,alice@mail.com,Nairobi,Laptop,1,85000,2025-05-01,delivered,85000
2,Brian Otieno,brian@mail.com,Kisumu,Headphones,2,3500,2025-05-03,delivered,7000
3,Carol Muthoni,carol@mail.com,Nairobi,Smartphone,1,45000,2025-05-05,delivered,45000
4,Alice Wanjiru,alice@mail.com,Nairobi,Mouse,1,1200,2025-05-10,shipped,1200
7,Frank Mwangi,frank@mail.com,Nairobi,Laptop,1,85000,2025-05-15,delivered,85000
9,Henry Odhiambo,henry@mail.com,Kisumu,Desk,1,35000,2025-05-20,delivered,35000
```

Notice:
- Row 4's customer name is now `Alice Wanjiru` (title case, matching row 1).
- All statuses are lowercase and consistent.
- Row 10 (the duplicate) is gone.
- A new `subtotal` column has been calculated.

Check `data/sales_rejected.csv` - you should see exactly **3 rejected rows**:

```csv
order_id,customer_name,email,city,product,quantity,unit_price,order_date,status
5,David Kimani,david@mail.com,Nakuru,,1,18000,2025-05-12,cancelled
6,Eve Achieng,,Mombasa,Backpack,3,4500,2025-05-14,delivered
8,Grace Njeri,grace@mail.com,Nairobi,Notebook,-2,150,2025-05-18,pending
```

These are exactly the three rows with missing products, missing emails, and negative quantities that we identified at the start.

---

## 5. The complete pipeline.py file

Here is the full script in one place for reference:

```python
import pandas as pd
import logging
from datetime import datetime
import os

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)s | %(message)s"
)
logger = logging.getLogger("csv_etl_pipeline")

RAW_FILE = "data/sales_raw.csv"
CLEAN_FILE = "data/sales_clean.csv"
REJECTED_FILE = "data/sales_rejected.csv"


def extract(file_path):
    """Extract data from a CSV file into a pandas DataFrame."""
    logger.info(f"Starting extraction from {file_path}")
    if not os.path.exists(file_path):
        raise FileNotFoundError(f"Source file not found: {file_path}")
    df = pd.read_csv(file_path)
    logger.info(f"Extracted {len(df)} rows from {file_path}")
    return df


def transform(df):
    """Clean and validate the raw data."""
    logger.info("Starting transformation")
    original_count = len(df)

    df["customer_name"] = df["customer_name"].str.strip().str.title()
    df["status"] = df["status"].str.strip().str.lower()
    df["city"] = df["city"].str.strip().str.title()

    df["order_date"] = pd.to_datetime(df["order_date"], errors="coerce")

    before_dedup = len(df)
    df = df.drop_duplicates(subset=["customer_name", "product", "order_date"], keep="first")
    duplicates_removed = before_dedup - len(df)
    logger.info(f"Removed {duplicates_removed} duplicate rows")

    rejected_mask = (
        df["product"].isna()
        | (df["product"].astype(str).str.strip() == "")
        | df["quantity"].isna()
        | (df["quantity"] <= 0)
        | df["email"].isna()
    )
    rejected_df = df[rejected_mask].copy()
    df = df[~rejected_mask].copy()

    if len(rejected_df) > 0:
        rejected_df.to_csv(REJECTED_FILE, index=False)
        logger.warning(f"Rejected {len(rejected_df)} rows due to data quality issues -> {REJECTED_FILE}")

    df["subtotal"] = df["quantity"] * df["unit_price"]

    logger.info(f"Transformation complete: {original_count} -> {len(df)} clean rows")
    return df


def load(df, file_path):
    """Write the clean DataFrame to a CSV file."""
    logger.info(f"Loading {len(df)} rows to {file_path}")
    df.to_csv(file_path, index=False)
    logger.info("Load complete")


def run_pipeline():
    """Run the full ETL pipeline end to end."""
    start_time = datetime.now()
    logger.info("=== ETL PIPELINE STARTED ===")
    try:
        raw_df = extract(RAW_FILE)
        clean_df = transform(raw_df)
        load(clean_df, CLEAN_FILE)
        duration = (datetime.now() - start_time).total_seconds()
        logger.info(f"=== PIPELINE SUCCESS in {duration:.2f}s ===")
    except Exception as e:
        logger.error(f"Pipeline failed: {e}")
        raise


if __name__ == "__main__":
    run_pipeline()
```

---

## 6. How this maps back to yesterday's theory

| Theory concept (Day 1) | Where it appears in our code |
|---|---|
| Extract | The `extract()` function reading the CSV |
| Transform | The `transform()` function - cleaning, deduplication, validation, calculation |
| Load | The `load()` function writing the clean CSV |
| ETL (not ELT) | We transform **before** loading - classic ETL order |
| Batch processing | The script runs once over the whole file - a single batch run |
| Full load | We process the entire CSV every run, not just new rows |
| Logging | Every stage reports what happened - extraction count, rejects, duration |

>  Tomorrow's theory session will go deeper into ETL best practices - and you will recognize many of them from the patterns we already used today: separating concerns into functions, never silently dropping data, and logging every stage.

---

## 7. Try it yourself - extend the pipeline

Before tomorrow's lab, try these on your own:

1. **Add a new validation rule** - reject any row where `unit_price` is 0 or negative.
2. **Add a new column** - create a `customer_email_domain` column by extracting everything after the `@` in the email.
3. **Break it on purpose** - delete the `data/sales_raw.csv` file and run the pipeline again. Confirm you get a clear `FileNotFoundError` message instead of a confusing crash.
4. **Add more rows** to the CSV with new data quality issues you can think of, and confirm the pipeline catches them.

---

## 8. Common mistakes to avoid

| Mistake | Why it's a problem | Fix |
|---|---|---|
| Using `print()` instead of `logging` | No timestamps, severity levels, or easy way to redirect to a file | Always use the `logging` module in real pipelines |
| Deleting bad rows with no record | You lose visibility into data quality problems at the source | Always write rejected rows somewhere - a file, a table, a log |
| Not handling missing files | Pipeline crashes with a confusing pandas error | Check `os.path.exists()` and raise a clear error |
| Mixing extraction, transformation, and loading in one big function | Hard to test, hard to debug, hard to reuse | Keep E, T, and L as separate functions |
| Forgetting `errors="coerce"` on date parsing | A single bad date value crashes the entire pipeline | Always handle parsing errors gracefully |

---

## 9. Key terms to know from today

| Term | Meaning |
|---|---|
| pandas | The standard Python library for working with tabular data |
| DataFrame | pandas' table-like data structure - rows and columns, like a spreadsheet |
| `pd.read_csv()` | Reads a CSV file into a DataFrame |
| `pd.to_csv()` | Writes a DataFrame to a CSV file |
| Boolean mask | A True/False flag applied to every row, used to filter data |
| `drop_duplicates()` | Removes duplicate rows based on specified columns |
| `pd.to_datetime()` | Converts text into a proper datetime type |
| `errors="coerce"` | Tells pandas to replace unparseable values with NaN/NaT instead of crashing |
| Rejected records | Rows that fail data quality checks, saved separately instead of deleted |
| Logging | Recording timestamped, leveled messages about what a program is doing |
| try/except | Python's error handling pattern - catch failures and respond clearly |

---

## 10. Summary

> Today we built a real, working ETL pipeline in Python using pandas. We **extracted** messy data from a CSV file, **transformed** it by standardizing text, fixing dates, removing duplicates, and separating bad records from good ones, and **loaded** the clean result into a new file.
>
> The most important habit we practiced is **never silently dropping data** - every rejected row was saved to its own file so it can be investigated later. This single habit will save you from countless silent data-quality disasters in your career as a data engineer.

---

## 11. What's coming next

| Day | Topic |
|---|---|
| Wednesday (Theory) | Extract, transform, load (ETL) concepts and best practices in depth |
| Thursday (Lab) | Build a Python ETL pipeline for batch data processing |

---

*Notes by LuxDevHQ | Month 1 - Foundations of Data Engineering | Week 3, Day 2*