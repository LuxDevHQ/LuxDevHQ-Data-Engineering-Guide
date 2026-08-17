# Learning Notes

Follow the numbered folders from foundations to production-oriented topics. Each topic explains what it solves before introducing tools and terminology.

```text
Getting Started → Cloud & Linux → SQL → Python → ETL/ELT → Modeling & Warehousing
→ Airflow → Kafka → Spark → Data Lakes → Governance → CDC → Projects
```

| Topic | What you will learn | Prerequisites | Notes | Related practice/project |
|---|---|---|---|---|
| 1. Getting started | What Data Engineering is, how the course works, and which tools to install. | None | [Open topic](01-getting-started/) | [Cloud/PostgreSQL project](../projects/beginner/cloud-storage-and-postgresql/) |
| 2. Cloud and Linux | Why companies rent computing and storage, and how major cloud platforms map to data work. | Getting started | [Cloud fundamentals](02-cloud-and-linux/cloud-computing-fundamentals.md) | [Linux assignment](../exercises/assignments/linux-server-administration.md) |
| 3. SQL and databases | Store, retrieve, join, aggregate, and optimize relational data. | Getting started | [Core SQL](03-sql-and-databases/core-sql.md), [advanced SQL](03-sql-and-databases/advanced-sql.md), [topic folder](03-sql-and-databases/) | [SQL questions](../exercises/sql/sql-practice-questions.md) |
| 4. Python | Write programs, functions, generated test data, and web-scraping code. | Basic command line | [Python basics](04-python/01-python-basics.md), [topic folder](04-python/) | [Faker notebook](04-python/examples/faker-demo.ipynb) |
| 5. ETL and pipelines | Move data through extract, transform, and load stages safely in batch or streams. | SQL and Python | [ETL/ELT](05-etl-and-data-pipelines/01-etl-elt-workflows.md), [best practices](05-etl-and-data-pipelines/02-etl-best-practices.md) | [Simple CSV ETL](../projects/intermediate/simple-csv-etl/) |
| 6. Modeling and warehousing | Shape operational data for reliable, fast analytics using facts and dimensions. | SQL | [Dimensional modeling](06-data-modeling-and-warehousing/01-normalization-and-dimensional-modeling.md), [warehousing](06-data-modeling-and-warehousing/data-warehousing.md) | Kenya food-prices capstone (after later topics) |
| 7. Apache Airflow | Schedule, order, monitor, and retry pipeline tasks as DAGs. | Python and ETL | [Introduction](07-apache-airflow/01-apache-airflow-introduction.md), [setup](07-apache-airflow/02-apache-airflow-setup.md), [DAGs](07-apache-airflow/03-writing-airflow-dags.md) | DAG labs inside the DAG guide |
| 8. Apache Kafka | Move event streams reliably between independent systems using topics, producers, and consumers. | Data pipelines | [Introduction](08-apache-kafka/01-apache-kafka-introduction.md), [topic folder](08-apache-kafka/) | [Producer/consumer lab](../exercises/kafka/producer-consumer-lab.md) |
| 9. Spark and big data | Process data across workers with Spark DataFrames, SQL, and PySpark. | Python and SQL | [Spark guide](09-spark-and-big-data/apache-spark-and-pyspark.md) | Weather ETL assignment inside the guide |
| 10. Data lakes | Store raw and curated files at scale, and practice locally with MinIO. | ETL and cloud storage | [Data lakes](10-data-lakes/data-lakes.md), [MinIO](10-data-lakes/minio-setup.md) | [Kenya food-prices capstone](../projects/capstone/kenya-food-prices/) |
| 11. Governance and security | Make data discoverable, controlled, traceable, private, and compliant. | Foundations | [Governance](11-data-governance-and-security/01-data-governance-security-and-access-control.md), [GDPR/HIPAA](11-data-governance-and-security/03-gdpr-and-hipaa.md) | Governance requirements in both capstones |
| 12. Change data capture | Publish database changes without repeatedly copying every row. | Databases and Kafka | [CDC guide](12-change-data-capture/change-data-capture.md) | [Crypto pipeline capstone](../projects/capstone/crypto-market-pipeline/) |

## Choose what comes next

- Need more repetition? Open the [exercise index](../exercises/README.md).
- Ready to combine tools? Open the [project index](../projects/README.md).
- Following a live cohort? Use the [course roadmap](../README.md#course-roadmap); topic filenames intentionally no longer encode class days.
