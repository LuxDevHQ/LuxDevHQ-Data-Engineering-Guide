
## Project Overview

In this capstone, you will design and implement a production-style distributed data pipeline that ingests, processes, and stores real-time cryptocurrency market data using modern data engineering tools.

---

### ** Data Sources (APIs)**

You are required to use at least one of the following data providers:

#### **1. Binance API**
:contentReference[oaicite:0]{index=0}

Primary source of crypto market data:
- Live price ticks
- Trading pairs
- Order book data
- Historical OHLCV data

Official docs: https://binance-docs.github.io/apidocs/

---

#### **2. Kraken API (OPTIONAL BUT RECOMMENDED)**
:contentReference[oaicite:1]{index=1}

Alternative or additional data source:
- Real-time trade data
- OHLC candlestick data
- Asset pairs
- Market depth

Official docs: https://docs.kraken.com/

---

### **System Architecture (Required Components)**

You will build the following end-to-end pipeline:

---

### **1. Data Ingestion Layer**

Use API calls from Binance or Kraken and schedule ingestion using:

:contentReference[oaicite:2]{index=2}

Responsibilities:
- Fetch data from APIs
- Validate incoming data
- Store raw data in PostgreSQL

---

### **2. Staging Database**

:contentReference[oaicite:3]{index=3}

Responsibilities:
- Store raw ingested data
- Act as a buffer before streaming
- Maintain structured tables:
  - trades
  - prices
  - symbols
  - timestamps

---

### **3. Streaming Layer**

:contentReference[oaicite:4]{index=4}  
:contentReference[oaicite:5]{index=5}  

Supporting Components:
- Schema Registry (data validation & evolution)
- Control Center (monitoring Kafka streams)

Responsibilities:
- Stream data from PostgreSQL to Kafka topics
- Enable real-time data movement

---

### **4. Processing Layer**

:contentReference[oaicite:6]{index=6}

Responsibilities:
- Consume Kafka streams
- Perform transformations:
  - Aggregations (minute/hour averages)
  - Volatility calculations
  - Trade volume analysis
  - Optional anomaly detection

---

### **5. Serving Layer**

:contentReference[oaicite:7]{index=7}

Responsibilities:
- Store processed streaming outputs
- Support fast query access for dashboards and analytics

---

### **Environment Variables & Security (VERY IMPORTANT)**

All secrets MUST be stored in a `.env` file.

#### Example `.env`
```bash
BINANCE_API_KEY=your_key_here
BINANCE_SECRET_KEY=your_secret_here

KRAKEN_API_KEY=your_key_here
KRAKEN_SECRET_KEY=your_secret_here

POSTGRES_USER=admin
POSTGRES_PASSWORD=strongpassword
POSTGRES_DB=crypto_pipeline
```


GitHub Safety (.gitignore REQUIRED)

You MUST include a .gitignore file to avoid leaking sensitive files.

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

 ### System Requirements

Your pipeline must include:

Batch + Streaming hybrid architecture
Airflow for orchestration
Kafka for real-time streaming
Spark for distributed processing
PostgreSQL for raw storage
Cassandra for processed storage
📊 Expected Outputs

By the end of the project, your system should:

Ingest live crypto data from Binance or Kraken
Stream data through Kafka in real time
Process streaming data with Spark
Store raw and processed data appropriately
Be fully reproducible from code
🏁 Submission Requirements

You must submit:

GitHub repository (clean and organized)
README.md with:
Architecture diagram
Setup instructions
Environment setup guide
.env.example file (NO real secrets)
Proper .gitignore
Airflow DAGs
Spark jobs
Kafka configs (if applicable)
🎯 Evaluation Criteria
System design correctness
Streaming pipeline implementation
Code quality & modularity
Security practices (.env + gitignore)
Scalability thinking
Documentation clarity
🚀 Bonus Features (Optional)
Dockerize the full pipeline
Add monitoring dashboards
Multi-exchange comparison (Binance vs Kraken)
Real-time anomaly detection
Build API layer for Cassandra queries
