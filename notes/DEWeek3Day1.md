# Week 3, Day 1 - Introduction to ETL/ELT Workflows

> **Month 1: Foundations of Data Engineering**
> **Week 3: Introduction to Data Pipelines**
> **Monday - Theory**

---

## What we covered in Week 2 (recap)

In Week 2 we went deep into SQL and data modeling:

| Day | Topic |
|---|---|
| Day 1 | Core SQL - SELECT, WHERE, JOIN, GROUP BY |
| Day 2 | Advanced SQL - subqueries, CTEs, window functions, views, stored procedures, recursive queries |
| Day 3 | Query optimization and execution plans |
| Day 4 | Data modeling - normalization, denormalization, and star schemas |

We learned that data engineers work in two worlds: **normalized OLTP source systems** and **denormalized OLAP data warehouses** - and that the work of moving data from one to the other is the data engineer's core job.

> **This week we name that work properly: ETL and ELT.** Today is theory - understanding the concepts deeply before we touch any code tomorrow.

---

## Today's objective

By the end of today, you should be able to:

- Explain what a data pipeline is.
- Define ETL and break down each of its three stages.
- Define ELT and explain how it differs from ETL.
- Compare ETL and ELT and know when to use each.
- Identify the components and tools involved in a real-world pipeline.
- Understand common pipeline patterns: batch vs streaming, full load vs incremental load.

---

## 1. What is a data pipeline?

A **data pipeline** is a series of automated steps that move data from one or more sources to a destination, applying processing along the way.

Think back to Day 1 of this entire course - we used the example of an online supermarket. A data engineer builds a system that collects order data, payment data, and customer data, cleans it, and stores it for analysis. That entire system **is** a data pipeline.

### Simple analogy

Think of a water treatment plant.

```text
Raw river water (source)
        ↓
Filtration (remove debris)
        ↓
Chemical treatment (purify)
        ↓
Storage tank (clean water ready for use)
        ↓
Pipes to homes (consumption)
```

A data pipeline works the same way:

```text
Raw data (source systems)
        ↓
Extraction (pull the data out)
        ↓
Transformation (clean, validate, reshape)
        ↓
Loading (store in the destination)
        ↓
Consumption (dashboards, reports, ML models)
```

ETL and ELT are simply two different **orders** in which these treatment steps happen.

---

## 2. What is ETL?

**ETL** stands for **Extract, Transform, Load**. It is the traditional approach to building data pipelines, where data is transformed **before** it is loaded into the destination system.

```text
SOURCE  →  EXTRACT  →  TRANSFORM  →  LOAD  →  DESTINATION
```

### Stage 1 - Extract

**Extraction** is the process of pulling data out of one or more source systems.

Sources can include:
- Operational databases (PostgreSQL, MySQL, SQL Server)
- APIs (payment gateways, social media, weather services)
- Flat files (CSV, JSON, Excel)
- SaaS applications (Salesforce, HubSpot, Google Analytics)
- Streaming systems (Kafka, Kinesis)
- Logs and sensor data

**Example:**
```text
Extract daily orders from the PostgreSQL orders table.
Extract customer events from a marketing API.
Extract daily sales CSV files uploaded by branch managers.
```

### Stage 2 - Transform

**Transformation** is the process of converting raw extracted data into a clean, structured, and usable format - **before** it reaches the destination.

Common transformation tasks:
- Cleaning - removing duplicates, fixing typos, handling NULLs.
- Standardizing - converting formats, currencies, units, date formats.
- Validating - checking data quality rules, rejecting bad records.
- Enriching - joining with reference data to add context.
- Aggregating - summarizing data into a more useful grain.
- Restructuring - converting normalized data into a star schema (as we learned on Day 4 last week).

**Example:**
```text
Remove duplicate customer records.
Convert all currency values to KES.
Join order data with customer data to add city and segment.
Aggregate transactions into daily totals.
```

### Stage 3 - Load

**Loading** is the process of writing the transformed data into its final destination - usually a data warehouse, data lake, or database.

**Example:**
```text
Load the cleaned, transformed orders into the analytics.fact_orders table
in the data warehouse.
```

### Why ETL transforms before loading

ETL was designed in an era when:
- Storage was expensive - you did not want to store raw, messy data.
- Compute for transformation happened on a **separate server**, not inside the warehouse.
- Data warehouses were not powerful enough to handle heavy transformation workloads themselves.

So transformation happened on a dedicated ETL server, and only clean data was loaded into the warehouse.

---

## 3. What is ELT?

**ELT** stands for **Extract, Load, Transform**. It is the modern approach, where raw data is loaded into the destination **first**, and transformation happens **after**, inside the destination system itself.

```text
SOURCE  →  EXTRACT  →  LOAD  →  TRANSFORM (inside the warehouse)  →  DESTINATION
```

### Why ELT emerged

Modern cloud data warehouses like **BigQuery**, **Snowflake**, **Redshift**, and **Synapse** are massively powerful and can run transformations directly using SQL - at a scale and speed that a separate ETL server cannot match.

ELT became possible and popular because:
- **Cloud storage is cheap** - you can store raw data without worrying about cost.
- **Cloud warehouses are powerful** - they can transform billions of rows directly using SQL.
- **Tools like dbt** allow transformation logic to live entirely as SQL inside the warehouse.
- **Raw data preservation** - keeping the original raw data means you can always re-transform it differently later without re-extracting from the source.

### ELT stages

#### Stage 1 - Extract
Same as ETL - pull data from source systems.

#### Stage 2 - Load
Raw, untransformed data is loaded directly into the destination (typically a data lake or a "raw" schema in the warehouse).

```text
Load raw orders.csv directly into raw.orders in BigQuery - no cleaning yet.
```

#### Stage 3 - Transform
Transformation happens **inside the warehouse**, using SQL - often with CTEs, views, and tools like dbt (which we learned about on Day 2 last week).

```sql
-- Transformation happens here, inside the warehouse, after loading
CREATE TABLE staging.clean_orders AS
SELECT
    order_id,
    customer_id,
    order_date,
    COALESCE(status, 'unknown')  AS status,
    ROUND(total_amount, 2)       AS total_amount
FROM raw.orders
WHERE total_amount > 0;
```

---

## 4. ETL vs ELT - side-by-side comparison

| | ETL | ELT |
|---|---|---|
| **Order of operations** | Extract → Transform → Load | Extract → Load → Transform |
| **Where transformation happens** | Separate ETL server/engine | Inside the destination warehouse |
| **Raw data preserved?** | Usually not - only clean data is kept | Yes - raw data is always available |
| **Speed of loading** | Slower - must transform before loading | Faster - load immediately, transform later |
| **Flexibility** | Less flexible - re-transforming means re-extracting | More flexible - re-transform anytime from raw data |
| **Best suited for** | Smaller datasets, legacy systems, strict compliance needs | Big data, cloud-native warehouses, modern analytics |
| **Common tools** | Informatica, Talend, SSIS, AWS Glue | dbt, BigQuery, Snowflake, Redshift, Synapse |
| **Storage cost** | Lower - only clean data stored | Higher - both raw and clean data stored |
| **Compute cost** | Separate transformation infrastructure | Warehouse compute used for transformation |

>  **Real-world note:** Most modern data engineering teams use **ELT** today, especially with cloud warehouses. However, ETL is still common in regulated industries (banking, healthcare) where raw, unprocessed sensitive data should never touch the warehouse - transformations like masking and anonymization must happen **before** loading.

---

## 5. Real-life ETL example

**Scenario:** An online supermarket wants daily sales reports.

```text
EXTRACT
  → Pull yesterday's orders from PostgreSQL
  → Pull payment confirmations from the payment gateway API

TRANSFORM (on a separate processing server)
  → Remove cancelled orders
  → Join orders with payment confirmations
  → Convert all amounts to KES
  → Aggregate into daily totals per branch
  → Validate: no negative amounts, no missing branch IDs

LOAD
  → Insert the clean, aggregated daily totals into
    analytics.daily_sales_summary in the warehouse
```

---

## 6. Real-life ELT example

**Scenario:** A fintech company wants to analyze millions of transaction events from multiple sources.

```text
EXTRACT
  → Pull transaction events from the production database
  → Pull events from the mobile app API
  → Pull events from a third-party fraud detection service

LOAD (immediately, no transformation yet)
  → Load all raw events into raw.transactions in BigQuery
  → Load all raw API responses into raw.app_events in BigQuery

TRANSFORM (inside BigQuery, using SQL / dbt)
  → Clean and deduplicate in a staging layer
  → Join transactions with fraud scores
  → Build the star schema: fact_transactions + dim_customer + dim_date
  → Create analytics views for the fraud and finance teams
```

>  Notice that in the ELT example, the raw data lands in BigQuery **exactly as it came from the source** - nothing is cleaned yet. This means if the fraud team later wants to apply a completely different transformation logic, they can re-run it against the same raw data without re-extracting anything.

---

## 7. Batch vs streaming pipelines

Pipelines can also be categorized by **how often** they run.

| | Batch processing | Streaming processing |
|---|---|---|
| **How it works** | Processes data in scheduled chunks (e.g. every hour, every night) | Processes data continuously as it arrives |
| **Latency** | Minutes to hours | Milliseconds to seconds |
| **Example** | Nightly sales report, monthly billing run | Fraud detection, live dashboards, stock prices |
| **Tools** | Airflow, AWS Glue, Azure Data Factory | Kafka, Kinesis, Pub/Sub, Spark Streaming |
| **Complexity** | Simpler to build and debug | More complex - requires handling out-of-order data, failures |
| **Cost** | Generally cheaper | Generally more expensive to run continuously |

>  **Most beginner and even many production pipelines are batch.** Streaming is reserved for use cases that genuinely need real-time data - fraud detection, live tracking, real-time personalization. Don't reach for streaming just because it sounds advanced; batch is often the right and simpler choice.

---

## 8. Full load vs incremental load

This connects directly to what we learned about incremental filtering in Week 2.

| | Full load | Incremental load |
|---|---|---|
| **What happens** | The entire dataset is extracted and reloaded every time | Only new or changed records since the last run are extracted |
| **Example** | Re-extract the entire customers table every night | Extract only customers created or updated since yesterday |
| **Pros** | Simple, no risk of missing data | Much faster, uses fewer resources |
| **Cons** | Slow and expensive on large tables | Requires tracking what has already been processed (a "watermark") |
| **When to use** | Small tables, reference data that rarely changes | Large transactional tables (orders, events, logs) |

```sql
-- Full load - re-extract everything
SELECT * FROM raw.customers;

-- Incremental load - only new/changed records since the last watermark
SELECT * FROM raw.orders
WHERE updated_at > '2025-06-16 23:59:59';   -- last pipeline run time
```

---

## 9. Components of a real-world pipeline

A production pipeline typically has these components, regardless of whether it follows ETL or ELT:

```text
1. Source connectors      → connect to databases, APIs, files
2. Orchestrator           → schedules and sequences pipeline steps
3. Extraction logic       → pulls data from sources
4. Staging area           → temporary storage for raw or in-progress data
5. Transformation logic   → cleans, validates, and reshapes data
6. Destination            → data warehouse, data lake, or database
7. Monitoring & alerting  → detects failures and data quality issues
8. Logging                → records what happened, when, and with what results
```

### Common tools mapped to each component

| Component | Example tools |
|---|---|
| Orchestrator | Apache Airflow, Azure Data Factory, Cloud Composer, Dagster |
| Extraction | Python scripts, AWS Glue, Fivetran, Airbyte |
| Staging | S3, Azure Data Lake, Google Cloud Storage |
| Transformation | dbt, SQL, Spark, AWS Glue |
| Destination | Redshift, Synapse, BigQuery, Snowflake |
| Monitoring | CloudWatch, Azure Monitor, Datadog |

>  This week we will build pipelines using **Python** for extraction and transformation logic - the most common language for writing custom pipeline code when off-the-shelf tools cannot fully handle a use case.

---

## 10. ETL/ELT best practices preview

We will go deeper into this on Wednesday, but here are the principles to start thinking about:

- **Idempotency** - running the pipeline twice should not create duplicate data or break anything.
- **Error handling** - pipelines should fail loudly and clearly, not silently produce bad data.
- **Logging** - every pipeline run should leave a record of what happened.
- **Data quality checks** - validate data before and after each stage.
- **Modularity** - break extraction, transformation, and loading into separate, testable functions.
- **Idempotent loads** - use `UPSERT` or `MERGE` patterns instead of blind `INSERT` to avoid duplicates on reruns.

---

## 11. Key terms to know from today

| Term | Meaning |
|---|---|
| Data pipeline | A series of automated steps that move and process data from source to destination |
| ETL | Extract, Transform, Load - transform data before loading it into the destination |
| ELT | Extract, Load, Transform - load raw data first, transform inside the destination |
| Extraction | Pulling data out of source systems |
| Transformation | Cleaning, validating, reshaping, and enriching data |
| Loading | Writing data into its final destination |
| Staging area | Temporary storage used to hold raw or in-progress data during a pipeline run |
| Batch processing | Processing data in scheduled chunks at intervals |
| Streaming processing | Processing data continuously as it arrives |
| Full load | Re-extracting and reloading an entire dataset every run |
| Incremental load | Extracting only new or changed records since the last run |
| Watermark | A reference point (usually a timestamp) used to determine what data is "new" in an incremental load |
| Orchestrator | A tool that schedules and sequences the steps of a pipeline |
| Idempotency | The property that running a process multiple times produces the same result as running it once |
| UPSERT / MERGE | A load pattern that inserts new records and updates existing ones, avoiding duplicates |

---

## 12. Summary

> A **data pipeline** moves data from source systems to a destination, applying processing along the way. **ETL** transforms data before loading it - the traditional approach, still used where raw sensitive data should never reach the warehouse. **ELT** loads raw data first and transforms it afterward inside the warehouse - the modern, cloud-native approach that takes advantage of powerful warehouse compute and preserves raw data for future re-processing.
>
> Pipelines can run as **batch** (scheduled at intervals) or **streaming** (continuous), and can use **full loads** or **incremental loads** depending on data volume and freshness requirements.
>
> Tomorrow we move from theory to practice - building our first real Python-based ETL pipeline for CSV data.

---

## 13. What's coming next

| Day | Topic |
|---|---|
| Tuesday (Lab) | Create a simple Python-based ETL pipeline for CSV data |
| Wednesday (Theory) | ETL concepts and best practices in depth |
| Thursday (Lab) | Build a Python ETL pipeline for batch data processing |

---

*Notes by LuxDevHQ | Month 1 - Foundations of Data Engineering | Week 3, Day 1*