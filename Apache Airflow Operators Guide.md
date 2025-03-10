### **1. Bash & Python Operators**

```python
def function_name():
  pass
```

- `BashOperator` - Executes a bash command.
- `PythonOperator` - Runs a Python function.
- `BranchPythonOperator` - Executes one of multiple Python functions based on logic.

### **2. SQL & Database Operators**
- `MySqlOperator` - Executes SQL queries in MySQL.
- `PostgresOperator` - Executes SQL queries in PostgreSQL.
- `SqliteOperator` - Executes SQL queries in SQLite.
- `MSSqlOperator` - Executes SQL queries in MS SQL Server.
- `OracleOperator` - Executes SQL queries in Oracle.
- `SnowflakeOperator` - Executes SQL queries in Snowflake.
- `BigQueryOperator` - Runs queries in Google BigQuery.
- `RedshiftSQLOperator` - Runs SQL commands in Amazon Redshift.

### **3. File Transfer & Storage Operators**
- `S3FileTransformOperator` - Processes files stored in Amazon S3.
- `S3ToRedshiftOperator` - Loads data from S3 to Redshift.
- `GCSToBigQueryOperator` - Loads files from Google Cloud Storage to BigQuery.
- `FTPOperator` - Transfers files via FTP.
- `FileSensor` - Waits for a file to appear in a directory.

### **4. Data Processing & ETL Operators**
- `SparkSubmitOperator` - Submits a Spark job.
- `DataProcPySparkOperator` - Runs PySpark jobs on Google Dataproc.
- `HiveOperator` - Runs Hive queries.
- `DruidOperator` - Submits queries to Apache Druid.
- `PrestoOperator` - Runs Presto SQL queries.

### **5. AWS Operators**
- `S3ToSnowflakeOperator` - Loads S3 data into Snowflake.
- `DynamoDBToS3Operator` - Copies DynamoDB data to S3.
- `EMRCreateJobFlowOperator` - Starts an EMR cluster.
- `LambdaInvokeFunctionOperator` - Calls an AWS Lambda function.

### **6. Google Cloud Operators**
- `BigQueryCheckOperator` - Checks data in BigQuery.
- `DataflowTemplateOperator` - Runs a Google Cloud Dataflow job.
- `GCSCreateBucketOperator` - Creates a Google Cloud Storage bucket.

### **7. Kubernetes & Docker Operators**
- `KubernetesPodOperator` - Runs a task inside a Kubernetes pod.
- `DockerOperator` - Runs a Docker container.

### **8. Email & Notification Operators**
- `EmailOperator` - Sends an email.
- `SlackAPIPostOperator` - Sends messages to Slack.

### **9. Sensors (Wait for Events)**
- `HttpSensor` - Waits for an HTTP endpoint response.
- `S3KeySensor` - Waits for an object to appear in S3.
- `HdfsSensor` - Waits for a file to appear in HDFS.
- `ExternalTaskSensor` - Waits for another DAG task to complete.

### **10. Miscellaneous Operators**
- `DummyOperator` - A placeholder for dependencies.
- `HttpOperator` - Calls an HTTP endpoint.
