# Projects

Projects turn notes into portfolio-ready practice. Start with one beginner project, then combine Python and ETL at intermediate level before attempting a capstone.

## Beginner

| Project | Difficulty | Technologies | What you build |
|---|---|---|---|
| [Cloud storage and PostgreSQL](beginner/cloud-storage-and-postgresql/) | Beginner | AWS or Azure, PostgreSQL, DBeaver | A cloud file-to-database workflow and documented SQL connection. |
| [Aiven PostgreSQL](beginner/aiven-postgresql/) | Beginner | Aiven, PostgreSQL, DBeaver | A managed database, sample table, and database-client connection. |

## Intermediate

| Project | Difficulty | Technologies | What you build |
|---|---|---|---|
| [Simple CSV ETL](intermediate/simple-csv-etl/) | Intermediate | Python, pandas, CSV | A logged pipeline that cleans data, separates rejected rows, and writes valid output. |
| [Batch file ETL](intermediate/batch-file-etl/) | Intermediate | Python, pandas, SQLite | An idempotent multi-file pipeline with archiving and upserts. |

## Advanced

No standalone advanced project was present during the migration. Complete an intermediate project, then use a capstone's optional streaming and production extensions rather than starting from an empty decorative template.

## Capstone

| Project | Difficulty | Technologies | What you build |
|---|---|---|---|
| [Kenya food prices](capstone/kenya-food-prices/) | Capstone | Airflow, pandas or Spark, Kafka (optional), PostgreSQL/lake, dashboard | A governed batch platform, with an optional near-real-time path, for food-price and inflation analysis. |
| [Crypto market pipeline](capstone/crypto-market-pipeline/) | Capstone | Airflow, PostgreSQL, Kafka, Spark, Cassandra | A production-style pipeline for ingesting, processing, and serving live crypto-market data. |

Each migrated project retains its original assignment detail. The project README also contains a standardized student guide covering purpose, prerequisites, architecture, tasks, expected output, and extensions.
