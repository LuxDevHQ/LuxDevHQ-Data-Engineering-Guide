# Crypto Market Data Engineering Capstone Project

## Project Overview

In this capstone, you will design and implement a production-style distributed data pipeline that ingests, processes, and stores real-time cryptocurrency market data using modern data engineering tools.

---

## Data Sources APIs

You are required to use at least one of the following data providers:

### 1. Binance API

Primary source of crypto market data:

- Live price ticks
- Trading pairs
- Order book data
- Historical OHLCV data

Official docs: https://binance-docs.github.io/apidocs/

---

### 2. Kraken API Optional but Recommended

Alternative or additional data source:

- Real-time trade data
- OHLC candlestick data
- Asset pairs
- Market depth

Official docs: https://docs.kraken.com/

---

## System Architecture Required Components

You will build the following end-to-end pipeline:

---

## 1. Data Ingestion Layer

Use API calls from Binance or Kraken and schedule ingestion using Airflow.

### Responsibilities

- Fetch data from APIs
- Validate incoming data
- Store raw data in PostgreSQL

---

## 2. Staging Database

Use PostgreSQL as the staging database.

### Responsibilities

- Store raw ingested data
- Act as a buffer before streaming
- Maintain structured tables such as:
  - trades
  - prices
  - symbols
  - timestamps

---

## 3. Streaming Layer

Use Kafka as the streaming layer.

Supporting components may include:

- Schema Registry for data validation and schema evolution
- Control Center or Kafka UI for monitoring Kafka streams

### Responsibilities

- Stream data from PostgreSQL to Kafka topics
- Enable real-time data movement across the pipeline

---

## 4. Processing Layer

Use Apache Spark for distributed stream processing.

### Responsibilities

- Consume Kafka streams
- Perform transformations such as:
  - Minute or hourly price averages
  - Volatility calculations
  - Trade volume analysis
  - Optional anomaly detection

---

## 5. Serving Layer

Use Cassandra as the serving database.

### Responsibilities

- Store processed streaming outputs
- Support fast query access for dashboards and analytics

---

## Environment Variables and Security

All secrets must be stored in a `.env` file.

### Example `.env`

```bash
BINANCE_API_KEY=your_key_here
BINANCE_SECRET_KEY=your_secret_here

KRAKEN_API_KEY=your_key_here
KRAKEN_SECRET_KEY=your_secret_here

POSTGRES_USER=admin
POSTGRES_PASSWORD=strongpassword
POSTGRES_DB=crypto_pipeline
```

---

## GitHub Safety

You must include a `.gitignore` file to avoid leaking sensitive files.

### Required `.gitignore`

```gitignore
# Environment variables
.env

# Python cache
__pycache__/
*.pyc

# Jupyter checkpoints
.ipynb_checkpoints/

# Logs
logs/
*.log

# Airflow
airflow.db
airflow.cfg
logs/

# OS files
.DS_Store
Thumbs.db

# Virtual environments
venv/
env/
```

---

## System Requirements

Your pipeline must include:

- Batch and streaming hybrid architecture
- Airflow for orchestration
- Kafka for real-time streaming
- Spark for distributed processing
- PostgreSQL for raw storage
- Cassandra for processed storage

---

## Expected Outputs

By the end of the project, your system should:

- Ingest live crypto data from Binance or Kraken
- Stream data through Kafka in real time
- Process streaming data with Spark
- Store raw and processed data appropriately
- Be fully reproducible from code

---

## Submission Requirements

You must submit:

- GitHub repository that is clean and organized
- `README.md` with:
  - Architecture diagram
  - Setup instructions
  - Environment setup guide
- `.env.example` file with no real secrets
- Proper `.gitignore`
- Airflow DAGs
- Spark jobs
- Kafka configurations, if applicable

---

## Evaluation Criteria

Your project will be evaluated based on:

- System design correctness
- Streaming pipeline implementation
- Code quality and modularity
- Security practices using `.env` and `.gitignore`
- Scalability thinking
- Documentation clarity

---

## Bonus Features Optional

You may add any of the following bonus features:

- Dockerize the full pipeline
- Add monitoring dashboards
- Compare multiple exchanges, for example Binance vs Kraken
- Add real-time anomaly detection
- Build an API layer for querying Cassandra data

---

## Student Project Guide

### Why This Project Matters
Market events are a practical way to learn how ingestion, streaming, distributed processing, and serving systems work together under changing load.

### Learning Objectives
Design event schemas, orchestrate ingestion, stream with Kafka, aggregate with Spark, serve results from Cassandra, and operate secrets safely.

### Architecture

```text
Binance/Kraken API → Airflow ingestion → PostgreSQL staging → Kafka
                                                        ↓
Dashboard/API ← Cassandra serving ← Spark streaming processing
```

### Tools Used
**Airflow** schedules API ingestion; **PostgreSQL** stages raw records; **Kafka** transports events; **Spark** performs distributed transformations; **Cassandra** serves high-volume query patterns.

### Prerequisites
Complete the Airflow, Kafka, Spark, SQL, and ETL sections in the [learning index](../../../notes/README.md). Verify current API terms, regional availability, authentication, and rate limits before use.

### Setup and Project Tasks
1. Write a small, rate-limited API client and safe `.env.example`.
2. Define raw tables and versioned event schemas.
3. Prove PostgreSQL-to-Kafka flow with a small sample.
4. Add Spark transformations and Cassandra tables designed for dashboard queries.
5. Add orchestration, failure tests, observability, and reproducibility documentation.

### Expected Output
All outputs and submission items listed above, plus evidence of a controlled restart without lost or duplicated analytical results.

### Challenges / Extensions
Choose from the bonus features above; add dead-letter handling, data-quality alerts, checkpoint recovery tests, or infrastructure as code.
