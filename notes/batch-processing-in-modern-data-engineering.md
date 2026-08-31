## Batch Processing in Modern Data Engineering

Batch processing is one of the most important concepts in data engineering because a large amount of enterprise data is still processed this way.

At its simplest:

> **Batch processing means collecting a group of data and processing that group together as a unit, rather than processing every individual record immediately when it arrives.**

Think of the word **batch** literally as a group.

Suppose a company generates 2 million sales transactions during the day. Instead of recalculating the company's analytics dashboard after every single transaction, the data engineering team might collect those transactions and process them every hour, every night, or at another defined interval.

```text
Transactions during the day

08:01   Sale
08:03   Sale
08:04   Sale
08:07   Sale
...
17:59   Sale
18:00   Sale

        ↓

Collect records

        ↓

Process them together

        ↓

Daily Sales Batch

        ↓

Data Warehouse

        ↓

Power BI / Tableau / Reports
```

AWS describes batch-oriented ETL as a common approach for moving and transforming data for analytical systems, while cloud services such as Google Dataflow now support both batch and streaming workloads. 


### **1. Batch Processing Starts With a Business Requirement.**

One common mistake is thinking:

> Batch processing means running something at midnight.

Not necessarily.

Batch processing is about processing a bounded collection of data together.

**The batch might contain:**
- All transactions from yesterday
- All customer records created this hour
- All files uploaded this morning
- All payments received this month
- All logs generated in the last 15 minutes
- All orders that have not yet been processed

A batch therefore has some boundary.

**For example:**
```text
start_time = 2026-08-30 00:00:00
end_time   = 2026-08-30 23:59:59
```

Your pipeline processes everything inside that boundary.

### **2. A Simple Real-World Example.**

Imagine a supermarket.

During Monday, several stores produce:

```text
Store A → 35,000 transactions
Store B → 28,000 transactions
Store C → 42,000 transactions
Store D → 19,000 transactions
```
All these transactions are saved in operational systems. At 1:00 AM on Tuesday, a pipeline starts.

```text
Store Databases
      │
      │ Extract Monday's transactions
      ▼
┌─────────────────────┐
│ 124,000 transactions│
└─────────────────────┘
          │
          ▼
      Validate
          │
          ▼
       Clean
          │
          ▼
     Transform
          │
          ▼
     Aggregate
          │
          ▼
  Data Warehouse
          │
          ▼
    BI Dashboards
```
The company may calculate:
- Total sales
- Sales per store
- Sales per product
- Average transaction value
- Top-selling products
- Payment-method distribution
- Inventory movements
- Profit margins

The business opens its dashboard the next morning and sees the previous day's results. That is a classic batch analytics pipeline.


3. Batch Processing vs Real-Time Processing

Consider a bank transaction.

A customer transfers:

KSh 10,000

The bank cannot normally tell the customer:

"We will update your account balance tonight."

The transaction must usually be processed immediately.

That is closer to real-time transactional processing.

However, calculations such as:

Total transactions yesterday
Average transaction size
Branch performance
Customer segmentation
Daily settlement reports
Risk reports
Regulatory reports

may be processed in batches.

The same organization can therefore use both batch and real-time processing.

                    BANKING SYSTEM

                         │
            ┌────────────┴────────────┐
            │                         │
            ▼                         ▼

       Real-Time                  Batch
       Processing               Processing

            │                         │
     Customer balance           Daily reports
     Payment approval           Reconciliation
     Fraud alert                Aggregations
     Card authorization         Data warehouse
4. Batch Processing vs Streaming

The major distinction students should understand is how data is processed.

Batch Processing
Data arrives

○ ○ ○ ○ ○ ○ ○ ○ ○ ○

       ↓

Wait / accumulate

       ↓

○ ○ ○ ○ ○ ○ ○ ○ ○ ○
        BATCH

       ↓

Process together
Streaming Processing
Data arrives

○ → process
○ → process
○ → process
○ → process
○ → process

Streaming systems continuously consume and process arriving records.

A useful comparison is:

Characteristic	Batch Processing	Streaming Processing
Processing	Groups of records	Continuously arriving records
Latency	Seconds, minutes, or hours	Milliseconds to seconds
Typical trigger	Schedule or data availability	Event arrival
Complexity	Usually lower	Usually higher
Example	Daily revenue report	Live fraud detection
Common tools	Airflow, Spark, dbt, SQL	Kafka, Flink, Spark Streaming
Historical reprocessing	Relatively straightforward	Often more complicated
Infrastructure	Can run only when required	Often continuously running

### **5. Batch Processing Does Not Mean Slow Processing.**

Batch processing does not necessarily mean processing data once per day.

**A batch could run:**
```text
Once every month
Once every day
Once every hour
Every 15 minutes
Every 5 minutes
```

**For example:**

```text
10:00 → process records from 09:55–10:00
10:05 → process records from 10:00–10:05
10:10 → process records from 10:05–10:10
```

These are still batches.

The difference is simply the size of the processing interval.

### **6. Micro-Batch Processing.**

Instead of processing one large daily batch:

```text
24 hours
──────────────────────────
              ↓
         ONE BATCH
```
you can process many smaller batches.

```text
09:00–09:05
09:05–09:10
09:10–09:15
09:15–09:20
```
This is known as micro-batching. Micro-batching provides lower latency while maintaining many batch-processing characteristics.

Therefore, it is incorrect to think:
```text
Batch = Old
Streaming = Modern
```
Modern systems frequently use both batch and streaming architectures. 

### **7. What Happens Inside a Batch Pipeline?**

A typical modern data engineering batch pipeline has several stages.

```text
SOURCE
  │
  ▼
EXTRACT
  │
  ▼
VALIDATE
  │
  ▼
TRANSFORM
  │
  ▼
LOAD
  │
  ▼
QUALITY CHECK
  │
  ▼
REPORT / SERVE
```

These stages often form an ETL or ELT pipeline.

### **8. Stage 1: Extract.**

The first step is retrieving data from one or more sources.

**Sources might include:**

- PostgreSQL
- Oracle
- MySQL
- SAP
- Salesforce
- REST APIs
- CSV files
- Excel files
- JSON files
- Amazon S3
- Google Cloud Storage
- Azure Blob Storage

**For example:**
```sql
SELECT *
FROM transactions
WHERE transaction_date = '2026-08-30';
```

The result becomes the batch that needs to be processed.


### **9. Stage 2: Validate.**

Before transforming the data, a production pipeline should verify that the expected data has actually arrived.

**Questions might include:**
- Did we receive the expected file?
- Is the file empty?
- Are all required columns available?
- Did the row count suddenly decrease?
- Are timestamps valid?
- Are transaction IDs missing?
- Are there duplicate records?

**For example:**
```text
Expected:
1,500,000 transactions

Received:
23,000 transactions
```

This may indicate that something failed upstream. A good pipeline should normally stop rather than publish incomplete or incorrect data.

### **10. Stage 3: Transform.**

Raw data is rarely immediately useful.

Suppose the raw record looks like:
```text
customer_id = " C001 "
amount      = "1,500"
date        = "31/08/2026"
country     = "ke"
```
Transformation could produce:
```text
customer_id = C001
amount      = 1500.00
date        = 2026-08-31
country     = KE
```
Other transformations might include:
- Removing duplicates
- Joining customer information
- Calculating taxes
- Converting currencies
- Categorizing products
- Standardizing timestamps
- Calculating profit
- Creating customer segments
- Handling missing values

### **11. Stage 4: Aggregate.**

Large raw datasets are often summarized.

Imagine a company has:
```text
50,000,000 transactions
```

Management may not need to inspect every individual transaction.

Instead, the pipeline might calculate:

```sql
SELECT
    transaction_date,
    branch,
    SUM(amount) AS total_sales,
    COUNT(*) AS transaction_count
FROM transactions
GROUP BY
    transaction_date,
    branch;
```
This transforms millions of detailed records into useful analytical information.


### **12. Stage 5: Load.**

The processed data may then be loaded into:
- A data warehouse
- A data lake
- A lakehouse
- An operational database
- An analytics database
- A feature store
- A reporting database

For example:
```
Oracle
   │
   ▼
Airflow
   │
   ▼
Spark
   │
   ▼
Amazon S3
   │
   ▼
Snowflake
   │
   ▼
Power BI
```

### **13. Stage 6: Data Quality Checks.**

After loading, the pipeline may perform additional quality checks.

**For example:**

- No duplicate transaction IDs
- No negative quantities
- Required columns are populated
- Row count is greater than zero
- Warehouse totals match source totals
- Timestamps are valid
- Foreign-key relationships are valid

For example:

```text
Source Total    = KSh 89,500,000
Warehouse Total = KSh 64,200,000
```
That difference may indicate a serious pipeline problem.

### **14. Stage 7: Publish the Data.**

Finally, successfully processed data can be made available to downstream users and systems.
```
Pipeline Successful
        ↓
Warehouse Tables Updated
        ↓
Power BI Refresh
        ↓
Management Dashboard
```

If the process fails:
```
Pipeline Failed
        ↓
Send Alert
        ↓
Data Engineer Investigates
```

### **15. Where Batch Processing Is Used in Modern Data Engineering**

Batch processing is used across almost every major industry.

**Some of the most common areas include:**
- Data warehouses
- Business intelligence
- Banking and financial reconciliation
- Payroll
- Billing
- Machine learning
- Log processing
- Data lakes
- Regulatory reporting
- Historical data processing
- Reporting systems
- Database migration
- Backup processing

### **16. Data Warehouses.**

One of the largest uses of batch processing is loading analytical data warehouses.
```text
CRM ─────────┐
ERP ─────────┤
Website ─────┤
Payments ────┼── ETL / ELT ──► Data Warehouse
Mobile App ──┤
Files ───────┤
Third-party ─┘
```
**The data warehouse might be:**

- Snowflake
- BigQuery
- Amazon Redshift
- Databricks
- PostgreSQL
- Oracle
- SQL Server

**Batch pipelines may update tables such as:**
```text
fact_sales
fact_transactions
dim_customer
dim_product
dim_branch
```






