# MinIO Setup Guide

### Step 1: Download MinIO Server Binary

```bash
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
sudo mv minio /usr/local/bin/
```

This downloads and installs the `minio` binary system-wide.

### Step 2: Create MinIO Data Directory

```bash
mkdir -p ~/minio-data
```

This is where your data lake files (CSV, JSON, Parquet, etc.) will be stored.

### Step 3: Run MinIO Server

```bash
export MINIO_ROOT_USER=minioadmin
export MINIO_ROOT_PASSWORD=minioadmin

minio server ~/minio-data --console-address ":9001"
```

- Web UI will be available at: http://localhost:9001
- API endpoint: http://localhost:9000
- Credentials:
  - Username: `minioadmin`
  - Password: `minioadmin`

### Step 4: Access MinIO Web Console

1. Go to: http://localhost:9001
2. Login with:
   - Username: `minioadmin`
   - Password: `minioadmin`
3. Create a bucket called `datalake`.

### Step 5: Upload Files

Click **"Buckets" → "datalake" → "Upload"**, and upload sample files like:
- `sales.csv`
- `products.csv`
- `customers.csv`

## Step 6: Access MinIO with Python

Install dependencies:

```bash
pip install pandas s3fs boto3
```

Python example:

```python
import pandas as pd

df = pd.read_csv(
    's3://datalake/sales.csv',
    storage_options={
        "key": "minioadmin",
        "secret": "minioadmin",
        "client_kwargs": {
            "endpoint_url": "http://localhost:9000"
        }
    }
)

print(df.head())
```

## Optional Tools to Add

| Tool | Purpose |
|------|---------|
| **Apache Spark** | To query data in MinIO as a Data Lake |
| **Airflow** | To orchestrate ETL jobs |
| **Streamlit** | To create dashboards using cleaned data |
| **Parquet/Feather** | For storing large processed data efficiently |

## Mini Data Lake Project Idea (No Docker Needed)

**Project Name:** "Mini Data Lake for Kenyan Retail Sales"

**Data Flow:**
1. **Raw Data** → Upload to MinIO in bucket `datalake`
2. **ETL** → Use Python to clean and join data
3. **Save Output** → Store clean output back to MinIO as `clean_data.parquet`
4. **Analytics** → Use Power BI, Jupyter, or Streamlit
