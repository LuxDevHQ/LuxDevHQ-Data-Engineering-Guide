# Day 2: Writing Your First DAGs - Complete Beginner Guide

## 📖 Table of Contents
1. [What is a DAG?](#what-is-a-dag)
2. [Airflow Architecture](#airflow-architecture)
3. [DAG Components Deep Dive](#dag-components-deep-dive)
4. [Writing Your First DAGs](#writing-your-first-dags)
5. [Hands-On Labs](#hands-on-labs)
6. [Assignment](#assignment)
7. [Best Practices](#best-practices)
8. [Troubleshooting](#troubleshooting)

---

## 🎯 What You'll Learn Today

By the end of this lesson, you'll understand:
- What DAGs are and why they're important
- How Airflow architecture works
- How to create DAGs from scratch
- Core operators and their usage
- Task dependencies and scheduling
- Error handling and monitoring

---

## 📚 What is a DAG?

### DAG Definition
**DAG** stands for **Directed Acyclic Graph**. Let's break this down:

- **Directed**: Tasks have a specific direction/order (A → B → C)
- **Acyclic**: No circular dependencies (A can't depend on C if C depends on A)
- **Graph**: A collection of nodes (tasks) connected by edges (dependencies)

### Real-World Analogy
Think of a DAG like a **cooking recipe**:

```
🥘 Making Pasta Recipe (DAG)
├── 1. Boil Water (Task)
├── 2. Add Salt (Task) 
├── 3. Add Pasta (Task)
├── 4. Make Sauce (Task) - Can happen in parallel
├── 5. Drain Pasta (Task)
└── 6. Combine & Serve (Task)
```

**Dependencies**:
- You can't add pasta before boiling water
- You can make sauce while pasta cooks (parallel tasks)
- You must drain pasta before combining

### Why DAGs in Data Engineering?

```python
# Example: Daily Sales Report DAG
"""
📊 Daily Sales Report Workflow
├── Extract data from database
├── Clean and validate data
├── Calculate metrics
├── Generate report
├── Send email to stakeholders
└── Archive data
"""
```

**Benefits**:
- **Reliability**: Automatic retries and error handling
- **Scalability**: Parallel processing of independent tasks
- **Monitoring**: Visual tracking of task status
- **Scheduling**: Automatic execution at specified times

---

## 🏗️ Airflow Architecture

### Core Components Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    AIRFLOW ECOSYSTEM                        │
├─────────────────────────────────────────────────────────────┤
│  Web Server  │  Scheduler  │  Executor  │  Metadata DB     │
│  (UI/API)    │  (Planner)  │  (Runner)  │  (Storage)       │
└─────────────────────────────────────────────────────────────┘
```

### 1. **Web Server** 🌐
- **Purpose**: Provides the web interface (UI) and REST API
- **What it does**: 
  - Display DAG status and logs
  - Allow manual task triggering
  - Show task history and metrics
  - User authentication and permissions

```python
# How you interact with it:
# - Navigate to http://localhost:8080
# - View DAG graphs and logs
# - Trigger DAGs manually
# - Monitor task execution
```

### 2. **Scheduler** ⏰
- **Purpose**: The "brain" that decides when and what to run
- **What it does**:
  - Reads DAG files from the DAGs folder
  - Determines which tasks are ready to run
  - Submits tasks to the executor
  - Handles scheduling intervals

```python
# Scheduler workflow:
# 1. Scan DAGs folder every 30 seconds
# 2. Parse DAG files for changes
# 3. Check if tasks are ready to run
# 4. Submit eligible tasks to executor
```

### 3. **Executor** 🚀
- **Purpose**: Actually runs the tasks
- **Types**:
  - **SequentialExecutor**: One task at a time (development)
  - **LocalExecutor**: Multiple tasks on single machine
  - **CeleryExecutor**: Distributed execution across multiple machines
  - **KubernetesExecutor**: Runs tasks in Kubernetes pods

```python
# Executor comparison:
executors = {
    'Sequential': 'One task at a time, simple setup',
    'Local': 'Multiple tasks, single machine',
    'Celery': 'Multiple machines, queue-based',
    'Kubernetes': 'Container-based, highly scalable'
}
```

### 4. **Metadata Database** 🗄️
- **Purpose**: Stores all Airflow information
- **What it stores**:
  - DAG definitions and schedules
  - Task instances and their states
  - User information and permissions
  - Connection details and variables

```python
# Database stores:
# - DAG runs: When DAGs executed
# - Task instances: Individual task executions
# - Variables: Shared configuration
# - Connections: Database/API credentials
```

### 5. **Worker Nodes** 👷
- **Purpose**: Machines that execute tasks
- **Components**:
  - Airflow worker process
  - Task execution environment
  - Local logs and temporary files

### Architecture Flow Diagram

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   DAG File  │───▶│  Scheduler  │───▶│  Executor   │
│ (Python)    │    │ (Parsing)   │    │ (Queuing)   │
└─────────────┘    └─────────────┘    └─────────────┘
                                             │
                                             ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Web Server │◀───│ Metadata DB │◀───│   Worker    │
│    (UI)     │    │  (Storage)  │    │ (Execution) │
└─────────────┘    └─────────────┘    └─────────────┘
```

---

## 🧩 DAG Components Deep Dive

### 1. **DAG Object** - The Container

```python
from airflow import DAG
from datetime import datetime, timedelta

# The DAG is like a project folder
dag = DAG(
    dag_id='my_data_pipeline',           # Unique identifier
    description='Daily data processing',  # Human-readable description
    start_date=datetime(2024, 1, 1),     # When DAG becomes active
    schedule_interval='@daily',          # How often to run
    catchup=False,                       # Don't run for past dates
    max_active_runs=1,                   # Only one run at a time
    tags=['data', 'daily', 'production'] # Categories for organization
)
```

### 2. **Default Arguments** - Global Rules

```python
# These settings apply to ALL tasks in the DAG
default_args = {
    'owner': 'data_team',
    'depends_on_past': False,        # Don't wait for previous DAG runs
    'start_date': datetime(2024, 1, 1),
    'email_on_failure': True,
    'email_on_retry': False,
    'retries': 2,                    # Try 2 times if task fails
    'retry_delay': timedelta(minutes=5),
    'execution_timeout': timedelta(hours=1),  # Kill task after 1 hour
    'sla': timedelta(hours=2),       # Service Level Agreement
    'email': ['admin@company.com']
}
```

### 3. **Schedule Intervals** - When DAGs Run

```python
# Common scheduling patterns:
schedule_patterns = {
    # Presets
    '@once': 'Run only once',
    '@daily': 'Run once per day at midnight',
    '@hourly': 'Run once per hour',
    '@weekly': 'Run once per week',
    '@monthly': 'Run once per month',
    '@yearly': 'Run once per year',
    
    # Cron expressions
    '0 2 * * *': 'Daily at 2:00 AM',
    '0 14 * * MON': 'Every Monday at 2:00 PM',
    '0 0 1 * *': 'First day of every month',
    '*/15 * * * *': 'Every 15 minutes',
    
    # Timedelta objects
    timedelta(hours=6): 'Every 6 hours',
    timedelta(days=1): 'Every 24 hours',
    
    # Manual only
    None: 'Only manual triggers'
}
```

### 4. **Operators** - The Work Units

#### **PythonOperator** - Run Python Functions
```python
from airflow.operators.python import PythonOperator

def my_python_function(ds, **context):
    """
    ds: execution date as string
    context: dictionary with Airflow context
    """
    print(f"Executing for date: {ds}")
    return "Task completed successfully"

python_task = PythonOperator(
    task_id='run_python_code',
    python_callable=my_python_function,
    dag=dag
)
```

#### **BashOperator** - Run Shell Commands
```python
from airflow.operators.bash import BashOperator

bash_task = BashOperator(
    task_id='run_bash_command',
    bash_command='echo "Hello from Bash!" && date',
    dag=dag
)
```

#### **Other Common Operators**
```python
# Email operator
from airflow.operators.email import EmailOperator

# SQL operators
from airflow.operators.sql import SQLExecuteQueryOperator

# File operators
from airflow.operators.filesystem import FileOperator

# HTTP operators
from airflow.operators.http import HttpOperator
```

### 5. **Task Dependencies** - Execution Order

```python
# Method 1: Bitshift operators (>>)
task_a >> task_b >> task_c

# Method 2: Set downstream/upstream
task_a.set_downstream(task_b)
task_b.set_upstream(task_a)

# Method 3: Lists for parallel execution
task_a >> [task_b, task_c] >> task_d

# Method 4: Complex dependencies
start_task >> preprocessing_task
preprocessing_task >> [validate_data, clean_data]
[validate_data, clean_data] >> merge_task
merge_task >> end_task
```

### 6. **Task States** - Understanding Execution

```python
task_states = {
    'success': '✅ Task completed successfully',
    'running': '🔄 Task is currently executing',
    'failed': '❌ Task failed and won\'t retry',
    'retry': '🔄 Task failed but will retry',
    'skipped': '⏭️ Task was skipped',
    'up_for_retry': '⏳ Task waiting to retry',
    'up_for_reschedule': '⏳ Task waiting to be rescheduled',
    'queued': '📋 Task waiting in queue',
    'scheduled': '📅 Task scheduled to run'
}
```

---

## 🛠️ Writing Your First DAGs

### Basic DAG Structure Template

```python
# File: template_dag.py
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator
from datetime import datetime, timedelta

# Step 1: Define your Python functions
def my_function():
    # Your business logic here
    pass

# Step 2: Set default arguments
default_args = {
    'owner': 'your_name',
    'depends_on_past': False,
    'start_date': datetime(2024, 1, 1),
    'email_on_failure': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5)
}

# Step 3: Create DAG
dag = DAG(
    'template_dag',
    default_args=default_args,
    description='Template for new DAGs',
    schedule_interval='@daily',
    catchup=False
)

# Step 4: Define tasks
task1 = PythonOperator(
    task_id='python_task',
    python_callable=my_function,
    dag=dag
)

task2 = BashOperator(
    task_id='bash_task',
    bash_command='echo "Hello World"',
    dag=dag
)

# Step 5: Set dependencies
task1 >> task2
```

---

## 🧪 Hands-On Labs

### 🛠️ Lab 1: Your First DAG - Print Today's Date

**Objective**: Create a simple DAG that prints today's date using BashOperator

**File**: `date_printer_dag.py`

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime, timedelta

# Default arguments
default_args = {
    'owner': 'beginner',
    'depends_on_past': False,
    'start_date': datetime(2024, 1, 1),
    'email_on_failure': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5)
}

# Create DAG
dag = DAG(
    'date_printer_dag',
    default_args=default_args,
    description='Print current date and time',
    schedule_interval='@daily',
    catchup=False,
    tags=['beginner', 'bash', 'utility']
)

# Task 1: Print current date
print_date = BashOperator(
    task_id='print_current_date',
    bash_command='date',
    dag=dag
)

# Task 2: Print formatted date
print_formatted_date = BashOperator(
    task_id='print_formatted_date',
    bash_command='date "+Today is %A, %B %d, %Y"',
    dag=dag
)

# Task 3: Print system info
print_system_info = BashOperator(
    task_id='print_system_info',
    bash_command='echo "Running on: $(hostname)" && echo "User: $(whoami)"',
    dag=dag
)

# Set dependencies
print_date >> print_formatted_date >> print_system_info
```

**What This DAG Does**:
1. Prints the current date and time
2. Prints a formatted, human-readable date
3. Shows system information
4. Runs daily at midnight

---

### 🛠️ Lab 2: Python Function DAG - Hello Airflow!

**Objective**: Create a DAG that runs custom Python functions

**File**: `hello_airflow_dag.py`

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta
import logging

# Custom Python functions
def greet_airflow():
    """Simple greeting function"""
    message = "Hello, Airflow! Welcome to data orchestration!"
    print(message)
    logging.info(message)
    return message

def get_current_time():
    """Get and format current time"""
    current_time = datetime.now()
    formatted_time = current_time.strftime("%Y-%m-%d %H:%M:%S")
    
    print(f"Current time: {formatted_time}")
    print(f"Day of week: {current_time.strftime('%A')}")
    print(f"Week number: {current_time.isocalendar()[1]}")
    
    return {
        'timestamp': formatted_time,
        'day_of_week': current_time.strftime('%A'),
        'week_number': current_time.isocalendar()[1]
    }

def process_data(**context):
    """Process data using context"""
    # Access execution date
    execution_date = context['ds']
    dag_run = context['dag_run']
    
    print(f"Processing data for: {execution_date}")
    print(f"DAG run ID: {dag_run.run_id}")
    
    # Simulate data processing
    data = {
        'records_processed': 1000,
        'execution_date': execution_date,
        'status': 'success'
    }
    
    print(f"Processed {data['records_processed']} records")
    return data

# Default arguments
default_args = {
    'owner': 'beginner',
    'depends_on_past': False,
    'start_date': datetime(2024, 1, 1),
    'email_on_failure': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5)
}

# Create DAG
dag = DAG(
    'hello_airflow_dag',
    default_args=default_args,
    description='Python functions demonstration',
    schedule_interval='@daily',
    catchup=False,
    tags=['beginner', 'python', 'demo']
)

# Task 1: Greeting
greeting_task = PythonOperator(
    task_id='greet_airflow',
    python_callable=greet_airflow,
    dag=dag
)

# Task 2: Time processing
time_task = PythonOperator(
    task_id='get_current_time',
    python_callable=get_current_time,
    dag=dag
)

# Task 3: Data processing with context
data_task = PythonOperator(
    task_id='process_data',
    python_callable=process_data,
    provide_context=True,  # Pass Airflow context to function
    dag=dag
)

# Set dependencies
greeting_task >> time_task >> data_task
```

**What This DAG Does**:
1. Greets Airflow with a welcome message
2. Gets and formats current time information
3. Processes data using Airflow context
4. Demonstrates logging and return values

---

### 🛠️ Lab 3: Multi-Task DAG with Dependencies

**Objective**: Create a complex DAG with multiple tasks and parallel execution

**File**: `data_pipeline_dag.py`

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator
from airflow.operators.dummy import DummyOperator
from datetime import datetime, timedelta
import random

# Simulation functions
def extract_data():
    """Simulate data extraction"""
    print("🔍 Extracting data from source systems...")
    
    # Simulate different data sources
    sources = ['database', 'api', 'file_system']
    records = random.randint(5000, 15000)
    
    for source in sources:
        print(f"   - Extracting from {source}")
    
    print(f"✅ Extracted {records} records")
    return {'records_extracted': records, 'sources': sources}

def validate_data():
    """Simulate data validation"""
    print("✅ Validating data quality...")
    
    # Simulate validation checks
    checks = ['null_check', 'format_check', 'range_check', 'duplicate_check']
    
    for check in checks:
        print(f"   - Running {check}")
    
    # Simulate occasional validation issues
    if random.random() < 0.1:  # 10% chance of validation error
        raise ValueError("Data validation failed!")
    
    print("✅ Data validation passed")
    return {'validation_status': 'passed', 'checks_run': checks}

def transform_data():
    """Simulate data transformation"""
    print("🔄 Transforming data...")
    
    transformations = [
        'standardize_formats',
        'calculate_aggregates',
        'apply_business_rules',
        'enrich_with_external_data'
    ]
    
    for transform in transformations:
        print(f"   - Applying {transform}")
    
    processed_records = random.randint(4500, 14000)
    print(f"✅ Transformed {processed_records} records")
    
    return {'records_transformed': processed_records}

def load_data():
    """Simulate data loading"""
    print("📦 Loading data to destination...")
    
    destinations = ['data_warehouse', 'analytics_db', 'reporting_system']
    
    for dest in destinations:
        print(f"   - Loading to {dest}")
    
    print("✅ Data loading completed")
    return {'load_status': 'completed', 'destinations': destinations}

def generate_report():
    """Generate summary report"""
    print("📊 Generating data pipeline report...")
    
    # Simulate report generation
    report_sections = [
        'executive_summary',
        'data_quality_metrics',
        'performance_statistics',
        'error_analysis'
    ]
    
    for section in report_sections:
        print(f"   - Generating {section}")
    
    print("✅ Report generated successfully")
    return {'report_status': 'generated', 'sections': report_sections}

# Default arguments
default_args = {
    'owner': 'data_team',
    'depends_on_past': False,
    'start_date': datetime(2024, 1, 1),
    'email_on_failure': True,
    'email_on_retry': False,
    'retries': 2,
    'retry_delay': timedelta(minutes=5),
    'email': ['data-team@company.com']
}

# Create DAG
dag = DAG(
    'data_pipeline_dag',
    default_args=default_args,
    description='Complete data pipeline with ETL process',
    schedule_interval='0 2 * * *',  # Daily at 2 AM
    catchup=False,
    max_active_runs=1,
    tags=['etl', 'data_pipeline', 'production']
)

# Start point
start_pipeline = DummyOperator(
    task_id='start_pipeline',
    dag=dag
)

# Data extraction
extract_task = PythonOperator(
    task_id='extract_data',
    python_callable=extract_data,
    dag=dag
)

# Data validation (parallel with transformation prep)
validate_task = PythonOperator(
    task_id='validate_data',
    python_callable=validate_data,
    dag=dag
)

# System health check (parallel with validation)
health_check = BashOperator(
    task_id='system_health_check',
    bash_command='''
    echo "🔍 Checking system health..."
    echo "   - CPU usage: $(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | sed 's/%us,//')"
    echo "   - Memory usage: $(free -h | awk '/^Mem:/ {print $3 "/" $2}')"
    echo "   - Disk space: $(df -h / | awk 'NR==2 {print $5}')"
    echo "✅ System health check completed"
    ''',
    dag=dag
)

# Data transformation
transform_task = PythonOperator(
    task_id='transform_data',
    python_callable=transform_data,
    dag=dag
)

# Data loading
load_task = PythonOperator(
    task_id='load_data',
    python_callable=load_data,
    dag=dag
)

# Report generation
report_task = PythonOperator(
    task_id='generate_report',
    python_callable=generate_report,
    dag=dag
)

# Cleanup
cleanup_task = BashOperator(
    task_id='cleanup_temp_files',
    bash_command='''
    echo "🧹 Cleaning up temporary files..."
    echo "   - Removing temp data files"
    echo "   - Clearing cache"
    echo "   - Archiving logs"
    echo "✅ Cleanup completed"
    ''',
    dag=dag
)

# End point
end_pipeline = DummyOperator(
    task_id='end_pipeline',
    dag=dag
)

# Set up dependencies
start_pipeline >> extract_task
extract_task >> [validate_task, health_check]
[validate_task, health_check] >> transform_task
transform_task >> load_task
load_task >> report_task
report_task >> cleanup_task
cleanup_task >> end_pipeline
```

**What This DAG Does**:
1. **Start**: Initiates the pipeline
2. **Extract**: Pulls data from multiple sources
3. **Validate & Health Check**: Runs in parallel to check data quality and system health
4. **Transform**: Processes and transforms the data
5. **Load**: Loads data to destination systems
6. **Report**: Generates pipeline summary
7. **Cleanup**: Removes temporary files
8. **End**: Completes the pipeline

---
