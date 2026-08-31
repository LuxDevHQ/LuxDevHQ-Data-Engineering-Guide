# Batch Processing in Modern Data Engineering

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

