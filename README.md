# LuxDevHQ Data Engineering Course Guide

A beginner-friendly path through Data Engineering concepts, guided practice, and real-world projects from LuxDevHQ. Data Engineering is the work of collecting, cleaning, moving, storing, and serving trustworthy data so that analysts, applications, and businesses can use it.

## Start here

1. Read [Introduction to Data Engineering](notes/01-getting-started/introduction-to-data-engineering.md).
2. Follow the [learning notes in order](notes/README.md).
3. Reinforce each topic with [short exercises](exercises/README.md).
4. Build the [beginner projects](projects/README.md#beginner), then progress toward a capstone.
5. Use the course roadmap below to match the topic-based folders to your class week.

## Learning path

```text
Foundations → Cloud & Linux → SQL → Python → ETL/ELT → Data Modeling
→ Airflow → Kafka → Spark → Data Lakes → Governance → CDC → Capstones
```

## Repository structure

| Path | Purpose |
|---|---|
| [`notes/`](notes/README.md) | Concepts, tutorials, setup instructions, examples, and references in recommended study order. |
| [`exercises/`](exercises/README.md) | Focused practice and classroom assignments. |
| [`projects/`](projects/README.md) | Practical builds grouped by beginner, intermediate, advanced, and capstone level. |
| [`datasets/`](datasets/README.md) | Shared sample inputs that are not owned by one project. |
| [`RESTRUCTURE_SUMMARY.md`](RESTRUCTURE_SUMMARY.md) | Migration map, preservation notes, and instructor-review items. |

## Course roadmap

This table preserves the **17-week / four-month sequence from the previous repository README** while linking to the reorganized topics. Detailed day-by-day material remains inside the linked notes. A separate [6-to-7-month curriculum](notes/01-getting-started/curriculum-6-to-7-months.md) also exists; the two schedules differ and require an instructor decision rather than silent consolidation.

| Course week | Existing curriculum topic | Start with | Practice or build |
|---|---|---|---|
| 1 | Onboarding and environment setup | [Getting started](notes/01-getting-started/) and [cloud fundamentals](notes/02-cloud-and-linux/cloud-computing-fundamentals.md) | [Cloud storage and PostgreSQL](projects/beginner/cloud-storage-and-postgresql/) |
| 2 | Power BI for data visualization | See curriculum schedule; dedicated notes are not yet present | Instructor review |
| 3 | SQL essentials for Data Engineering | [Core SQL](notes/03-sql-and-databases/core-sql.md) | [SQL practice](exercises/sql/sql-practice-questions.md) |
| 4 | Introduction to data pipelines | [ETL/ELT workflows](notes/05-etl-and-data-pipelines/01-etl-elt-workflows.md) | [Simple CSV ETL](projects/intermediate/simple-csv-etl/) |
| 5 | Introduction to Apache Airflow | [Airflow introduction](notes/07-apache-airflow/01-apache-airflow-introduction.md) | [Writing DAGs](notes/07-apache-airflow/03-writing-airflow-dags.md) |
| 6 | Data warehousing and data lakes | [Data warehousing](notes/06-data-modeling-and-warehousing/data-warehousing.md) | [MinIO setup](notes/10-data-lakes/minio-setup.md) |
| 7 | Data governance and security | [Governance and access control](notes/11-data-governance-and-security/01-data-governance-security-and-access-control.md) | [GDPR and HIPAA](notes/11-data-governance-and-security/03-gdpr-and-hipaa.md) |
| 8 | Real-time data processing with Kafka | [Kafka introduction](notes/08-apache-kafka/01-apache-kafka-introduction.md) | [Producer/consumer lab](exercises/kafka/producer-consumer-lab.md) |
| 9 | Batch vs. stream processing | [Batch and stream processing](notes/05-etl-and-data-pipelines/batch-and-stream-processing.md) | [Batch file ETL](projects/intermediate/batch-file-etl/) |
| 10 | Machine learning integration in pipelines | See curriculum schedule; dedicated notes are not yet present | Instructor review |
| 11 | Spark and PySpark for big data | [Spark and PySpark](notes/09-spark-and-big-data/apache-spark-and-pyspark.md) | Weather ETL assignment in the same guide |
| 12 | Advanced Apache Airflow techniques | [DAG guide](notes/07-apache-airflow/03-writing-airflow-dags.md) | DAG hands-on labs in the same guide |
| 13 | Data lakes and Delta Lake | [Data lakes](notes/10-data-lakes/data-lakes.md) | [MinIO setup](notes/10-data-lakes/minio-setup.md) |
| 14 | Batch data pipeline development | [ETL best practices](notes/05-etl-and-data-pipelines/02-etl-best-practices.md) | [Batch file ETL](projects/intermediate/batch-file-etl/) |
| 15 | Real-time data pipeline development | [Kafka notes](notes/08-apache-kafka/) | [Kafka lab](exercises/kafka/producer-consumer-lab.md) |
| 16 | Final project integration | Review prerequisite notes | [Capstone projects](projects/README.md#capstone) |
| 17 | Capstone project presentation | Review project deliverables | Present a completed capstone |

## Who this is for

This guide is for LuxDevHQ students, including learners changing careers or starting without a technical background. Begin at the first note; unfamiliar tools are introduced before they are combined in projects.

## How to use this repository

- Study one topic at a time and run examples rather than only reading them.
- Complete small exercises before attempting a project.
- Keep secrets out of Git, use current official installation documentation when a pinned version has aged, and ask an instructor when a review warning appears.
- Fork or clone the repository, create a focused branch, and keep your project work in its own repository unless an instructor says otherwise.

## Contributing

Use lowercase kebab-case filenames, preserve existing educational detail, test commands and relative links, and explain curriculum changes. Never commit credentials, generated environments, or large unlicensed datasets.
