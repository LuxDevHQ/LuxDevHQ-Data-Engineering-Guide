# Week 1, Day 2 - Introduction to Cloud Computing (Azure, GCP, and AWS)

> **Month 1: Foundations of Data Engineering**
> **Week 1: Onboarding and Environment Setup**
> **Tuesday**

---

## What we covered yesterday (Day 1 recap)

On Day 1 we learned what data engineering is - the process of **collecting, moving, cleaning, storing, and preparing data** so that analysts, data scientists, AI systems, and businesses can use it.

We also learned that data engineers build **pipelines** that move data from sources like mobile apps, websites, payment systems, and databases into a central place — a data warehouse or data lake - where it can be analyzed and used.

> **Quick recap definition:**
> Data engineering is the work of building and maintaining the data infrastructure that allows organizations to collect, store, process, and use data effectively.

---

## Today's objective

By the end of today, you should be able to:

- Explain what cloud computing is in simple terms.
- Name the three major cloud platforms used in data engineering.
- Identify the main categories of cloud services used by data engineers.
- Compare AWS, Azure, and GCP side by side.
- Understand which cloud platform might be best to start learning first.

---

## 1. What is cloud computing?

**Cloud computing** means using computing resources - like servers, storage, and software - **over the internet**, instead of buying and managing your own physical hardware.

### Simple analogy

Think of electricity. You do not build your own power plant at home. You connect to the grid and pay for what you use.

Cloud computing works the same way:

- Instead of buying servers → you **rent** them online.
- Instead of managing hard drives → you **store** data on the cloud.
- Instead of hiring a team to maintain infrastructure → the **cloud provider** handles it.

You only pay for what you use, and you can scale up or down as needed.

### Why cloud computing matters in data engineering

As a data engineer, cloud computing allows you to:

- Store **large amounts of data** without buying physical drives.
- Build **data pipelines** that run automatically.
- Process **big data** that is too large for a single computer.
- Create **data warehouses** for reporting and analytics.
- Stream and process **real-time data** as it arrives.
- Run **databases** without managing servers manually.
- **Automate workflows** so pipelines run on schedule.
- **Secure and monitor** data systems with built-in tools.
- Support **analytics, dashboards, AI, and machine learning**.

---

## 2. The three major cloud platforms

The three platforms you will work with as a data engineer are:

| Platform | Full name | Owned by |
|---|---|---|
| **AWS** | Amazon Web Services | Amazon |
| **Azure** | Microsoft Azure | Microsoft |
| **GCP** | Google Cloud Platform | Google |

Each platform offers similar services but with different names, tools, and strengths. You will often see jobs requiring knowledge of at least one of these.

---

## 3. Main cloud services used in data engineering

Cloud platforms offer many services. As a data engineer, you will mainly work with the following categories.

---

### A. Data storage services

These are used to store raw, processed, and historical data. Think of these as huge online hard drives.

| Purpose | AWS | Azure | GCP |
|---|---|---|---|
| Object storage / data lake | Amazon S3 | Azure Data Lake Storage / Blob Storage | Google Cloud Storage |
| File storage | Amazon EFS | Azure Files | Filestore |
| Archive storage | S3 Glacier | Azure Archive Storage | Archive Storage |

**Real-life example:**

A logistics company collects delivery data every hour; CSV files, GPS logs, and JSON records from drivers' mobile apps.

A data engineer stores all of this raw data in:
- **Amazon S3** (on AWS), or
- **Azure Data Lake Storage** (on Azure), or
- **Google Cloud Storage** (on GCP)

These storage services are the foundation of what is called a **data lake** - a central place where raw data of all types is stored before it is cleaned and processed.

> **Key concept - Data lake vs data warehouse:**
> - A **data lake** stores raw, unprocessed data of any format (CSV, JSON, images, logs).
> - A **data warehouse** stores clean, structured data ready for analysis.
> - Data engineers build the pipelines that move data from the lake to the warehouse.

---

### B. Databases

Databases store structured data used by applications and business systems.

| Purpose | AWS | Azure | GCP |
|---|---|---|---|
| Managed relational database | Amazon RDS | Azure SQL Database | Cloud SQL |
| PostgreSQL | Amazon RDS for PostgreSQL / Aurora PostgreSQL | Azure Database for PostgreSQL | Cloud SQL for PostgreSQL / AlloyDB |
| MySQL | Amazon RDS for MySQL / Aurora MySQL | Azure Database for MySQL | Cloud SQL for MySQL |
| NoSQL database | DynamoDB | Cosmos DB | Firestore / Bigtable |

**Real-life example:**

A payment platform stores customer transactions in PostgreSQL. Instead of installing and managing PostgreSQL manually on a server, a data engineer uses a **managed database service**:

- **Amazon RDS for PostgreSQL** on AWS
- **Azure Database for PostgreSQL** on Azure
- **Cloud SQL for PostgreSQL** on GCP

Using a managed service means the cloud provider handles backups, scaling, updates, and availability automatically - saving the data engineer hours of maintenance work.

> **Relational vs NoSQL:**
> - **Relational databases** (like PostgreSQL and MySQL) store data in structured tables with rows and columns. Best for structured business data.
> - **NoSQL databases** (like DynamoDB, Cosmos DB, Firestore) store data in flexible formats like documents or key-value pairs. Best for unstructured or rapidly changing data.

---

### C. Data warehouses

A **data warehouse** stores clean, structured, and organized data specifically for reporting, dashboards, business intelligence, and analytics.

| Purpose | AWS | Azure | GCP |
|---|---|---|---|
| Cloud data warehouse | Amazon Redshift | Azure Synapse Analytics / Microsoft Fabric Warehouse | BigQuery |

**Real-life example:**

A supermarket chain wants to analyze:
- Daily sales per branch
- Which products sell the most
- Customer purchasing patterns
- Revenue trends over the last 12 months

The cleaned and processed data is loaded into:
- **Amazon Redshift** (AWS)
- **Azure Synapse Analytics** (Azure)
- **Google BigQuery** (GCP)

Business analysts then use this data to build dashboards and reports.

>  **BigQuery note:** BigQuery is Google Cloud's flagship data platform - it combines data warehousing, analytics, and AI capabilities in one service. It is widely used and considered one of the most powerful cloud data warehouses available today.

---

### D. Data integration and ETL/ELT services

These services help move data from different sources into storage or data warehouses.

**ETL** stands for **Extract, Transform, Load**.
**ELT** stands for **Extract, Load, Transform** - the transform step happens after loading into the warehouse.

| Purpose | AWS | Azure | GCP |
|---|---|---|---|
| ETL / data integration | AWS Glue | Azure Data Factory / Fabric Data Factory | Cloud Data Fusion |
| Workflow orchestration | AWS Step Functions / MWAA | Azure Data Factory pipelines | Cloud Composer |
| Transformation | AWS Glue / dbt | Data Factory / Synapse / dbt | Dataform / dbt |

**Real-life example:**

A data engineer builds a pipeline that:
1. **Extracts** data from a PostgreSQL database.
2. **Loads** it into cloud storage (S3, Data Lake, or Cloud Storage).
3. **Cleans and transforms** the data - removing duplicates, fixing formats, filling missing values.
4. **Loads** the clean data into a data warehouse.
5. **Refreshes** Power BI or Looker dashboards automatically.

On Azure, **Azure Data Factory** is the primary tool for building these pipelines. Microsoft's Fabric Data Factory is the next-generation version built for modern data integration workflows.

> **What is dbt?** dbt (data build tool) is a popular open-source tool used for transforming data inside a data warehouse using SQL. It works across AWS, Azure, and GCP and is widely used in modern data engineering.

---

### E. Big data processing services

These are used when data is **too large** to process on a single normal computer - we are talking about billions of records, terabytes, or petabytes of data.

| Purpose | AWS | Azure | GCP |
|---|---|---|---|
| Spark / Hadoop processing | Amazon EMR | Azure HDInsight / Azure Databricks | Dataproc |
| Managed Spark platform | AWS Glue / EMR / Databricks on AWS | Azure Databricks | Dataproc / Databricks on GCP |
| Serverless data processing | AWS Glue | Synapse Spark / Fabric Spark | Dataflow |

**Real-life example:**

A telecom company generates billions of call records every day. Processing this on one computer would take days. A data engineer uses distributed processing tools:

- **Amazon EMR** (AWS) - runs Apache Spark and Hadoop across many machines in parallel.
- **Azure Databricks** (Azure) - a collaborative Spark platform widely used in enterprise data engineering.
- **Google Dataproc** or **Google Dataflow** (GCP) - managed services for batch and streaming data processing at scale.

>  **What is Apache Spark?** Spark is an open-source big data processing engine. It processes data in parallel across many computers, making it extremely fast for large datasets. Azure Databricks, Amazon EMR, and Google Dataproc all run Spark.

---

### F. Real-time streaming services

Streaming services are used when data must be **processed immediately** as it arrives - not hours later, not minutes later, but in real time.

| Purpose | AWS | Azure | GCP |
|---|---|---|---|
| Real-time event streaming | Amazon Kinesis | Azure Event Hubs | Pub/Sub |
| Message queues | Amazon SQS | Azure Queue Storage / Service Bus | Cloud Tasks / Pub/Sub |
| Kafka-compatible streaming | Amazon MSK | Event Hubs for Kafka / HDInsight Kafka | Managed Service for Apache Kafka / Pub/Sub |

**Real-life example:**

A banking app needs to detect fraud in real time. Every time a customer makes a transaction, the event is sent to a streaming service immediately and checked against fraud patterns - all within milliseconds.

Other use cases for real-time streaming:
- Customer clicks on an e-commerce site
- Login attempts on a platform
- Live payment processing
- Stock price updates
- System log monitoring

**GCP real-time pipeline pattern:**

```text
Pub/Sub  →  Dataflow  →  BigQuery
```

- **Pub/Sub** receives streaming messages as they arrive.
- **Dataflow** processes and transforms them in real time.
- **BigQuery** stores the results for analytics.

---

### G. Analytics and business intelligence (BI)

These tools help business users analyze and visualize data through dashboards and reports.

| Purpose | AWS | Azure | GCP |
|---|---|---|---|
| BI dashboards | Amazon QuickSight | Power BI | Looker / Looker Studio |
| SQL analytics | Athena / Redshift | Synapse SQL / Fabric | BigQuery |
| Data exploration | Athena | Synapse / Fabric | BigQuery |

**Real-life example:**

A retail company's management team wants a live dashboard showing daily revenue, top-selling products, and regional sales performance. A data engineer connects the data warehouse to:

- **Power BI** (Azure) - most widely used in corporate and Microsoft environments.
- **Looker Studio** (GCP) - free and easy to use, connects to BigQuery and other sources.
- **Amazon QuickSight** (AWS) - native AWS BI tool.

> **Power BI is especially important** for this course because it connects directly with Azure, SQL databases, Excel, and many cloud data sources. You will use it in projects throughout the program.

---

### H. Machine learning and AI services

Data engineers do not always build machine learning models, but they **prepare and deliver the clean data** that data scientists and ML engineers need to train models.

| Purpose | AWS | Azure | GCP |
|---|---|---|---|
| Machine learning platform | Amazon SageMaker | Azure Machine Learning | Vertex AI |
| AI/ML data preparation | Glue / Redshift / S3 | Synapse / Fabric / Data Factory | BigQuery / Dataflow / Dataproc |
| Model deployment support | SageMaker endpoints | Azure ML endpoints | Vertex AI endpoints |

**Real-life example:**

A bank wants to build a credit scoring model that predicts whether a loan applicant is likely to default. A data engineer:
1. Collects transaction history, account data, and repayment records.
2. Cleans and transforms the data.
3. Loads it into the ML platform.

The data scientist then trains the model using:
- **Amazon SageMaker** (AWS)
- **Azure Machine Learning** (Azure)
- **Vertex AI** (GCP)

---

## 4. How a complete cloud data engineering architecture looks

A typical end-to-end cloud data engineering system follows this pattern:

```text
Data Sources
    ↓
Ingestion Layer
    ↓
Cloud Storage / Data Lake
    ↓
Processing / Transformation
    ↓
Data Warehouse
    ↓
Dashboards / Reports / AI Models
```

### Example architecture on AWS

```text
PostgreSQL / APIs / CSV files
        ↓
AWS Glue / Kinesis
        ↓
Amazon S3
        ↓
AWS Glue / EMR
        ↓
Amazon Redshift / Athena
        ↓
QuickSight / Power BI
```

### Example architecture on Azure

```text
SQL Server / APIs / Excel / Apps
        ↓
Azure Data Factory / Event Hubs
        ↓
Azure Data Lake Storage
        ↓
Azure Databricks / Synapse Spark
        ↓
Azure Synapse Analytics / Fabric Warehouse
        ↓
Power BI
```

### Example architecture on GCP

```text
Cloud SQL / APIs / Files / App Events
        ↓
Pub/Sub / Dataflow / Cloud Data Fusion
        ↓
Google Cloud Storage
        ↓
Dataflow / Dataproc
        ↓
BigQuery
        ↓
Looker / Looker Studio / Power BI
```

---

## 5. Full side-by-side comparison: AWS vs Azure vs GCP

| Data engineering need | AWS | Azure | GCP |
|---|---|---|---|
| Store raw files | S3 | Data Lake Storage | Cloud Storage |
| Managed PostgreSQL | RDS PostgreSQL | Azure Database for PostgreSQL | Cloud SQL PostgreSQL |
| Data warehouse | Redshift | Synapse / Fabric Warehouse | BigQuery |
| ETL pipelines | Glue | Data Factory | Data Fusion / Dataflow |
| Workflow orchestration | MWAA / Step Functions | Data Factory pipelines | Cloud Composer |
| Big data processing | EMR / Glue | Databricks / Synapse / HDInsight | Dataproc / Dataflow |
| Real-time streaming | Kinesis / MSK | Event Hubs | Pub/Sub |
| BI dashboards | QuickSight | Power BI | Looker / Looker Studio |
| ML platform | SageMaker | Azure Machine Learning | Vertex AI |
| Serverless functions | Lambda | Azure Functions | Cloud Functions |
| Containers | ECS / EKS | AKS | GKE |
| Monitoring | CloudWatch | Azure Monitor | Cloud Monitoring |
| Secrets management | Secrets Manager | Key Vault | Secret Manager |

---

## 6. Which cloud should you learn first?

### Option 1 - Start with Azure (recommended if you use Power BI, Excel, or SQL)

Azure is very beginner-friendly for people coming from a Microsoft background.

**Best for:** Power BI users, SQL Server users, corporate environments, Microsoft tool users.

**Key Azure tools to learn:**
1. Azure Data Lake Storage
2. Azure SQL Database
3. Azure Data Factory
4. Azure Synapse Analytics
5. Azure Databricks
6. Power BI
7. Azure Monitor
8. Azure Key Vault

---

### Option 2 - Start with AWS (recommended for broad global cloud exposure)

AWS is the most widely used cloud platform globally. More job listings mention AWS than any other provider.

**Key AWS tools to learn:**
1. Amazon S3
2. Amazon RDS
3. AWS Glue
4. Amazon Redshift
5. Amazon Athena
6. Amazon Kinesis
7. Amazon EMR
8. AWS Lambda
9. Amazon CloudWatch
10. AWS IAM

---

### Option 3: Start with GCP (recommended for strong analytics, BigQuery, and AI)

GCP is powerful for analytics and AI workloads. BigQuery is considered one of the best cloud data warehouses in the industry.

**Key GCP tools to learn:**
1. Google Cloud Storage
2. Cloud SQL
3. BigQuery
4. Dataflow
5. Pub/Sub
6. Dataproc
7. Cloud Composer
8. Dataform
9. Looker Studio
10. Vertex AI

---

## 7. Key terms to know from today

| Term | Meaning |
|---|---|
| Cloud computing | Using computing resources (servers, storage, tools) over the internet instead of physical hardware |
| AWS | Amazon Web Services - Amazon's cloud platform |
| Azure | Microsoft's cloud platform |
| GCP | Google Cloud Platform - Google's cloud platform |
| Data lake | A central store for raw, unprocessed data of any format |
| Data warehouse | A store for clean, structured data used for analytics and reporting |
| ETL | Extract, Transform, Load - moving and transforming data between systems |
| ELT | Extract, Load, Transform - load first, transform after |
| Managed service | A cloud service where the provider handles maintenance, backups, and scaling |
| Apache Spark | An open-source big data processing engine used to process huge datasets in parallel |
| Streaming | Processing data in real time as it arrives, rather than in batches |
| Pipeline | A system that moves and processes data automatically from one place to another |

---

## 8. Summary

> **Cloud computing in data engineering** means using online platforms like AWS, Azure, and GCP to store, move, process, clean, analyze, and secure data - without managing physical servers.
>
> A cloud data engineer uses services such as S3, Data Lake Storage, Cloud Storage, Glue, Data Factory, Dataflow, Redshift, Synapse, and BigQuery to build reliable data pipelines and analytics systems.

The three platforms offer the same core capabilities with different tool names. As a data engineer, you do not need to master all three at once - but you should understand how they map to each other and be comfortable working in at least one.

---

## 9. What's coming next

| Day | Topic |
|---|---|
| Wednesday | Data governance, security, compliance, and access control |
| Thursday | Introduction to SQL for data engineering and PostgreSQL setup |
| Friday | Peer project - environment setup challenges |
| Saturday (Lab) | Mini project - build a basic pipeline with PostgreSQL and Azure Blob Storage |

---

*Notes by LuxDevHQ | Month 1 - Foundations of Data Engineering | Week 1, Day 2*