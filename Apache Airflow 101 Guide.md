# Apache Airflow Setup & Introduction (Multi-Component Mode)

## Understanding Workflow Orchestration

**Workflow orchestration** is the automated coordination and management of complex data workflows. Think of it as a conductor directing an orchestra - it ensures that different data processing tasks run in the correct order, at the right time, and handles failures gracefully.

**Why Apache Airflow?**
- **Dependency Management**: Automatically handles task dependencies
- **Scheduling**: Run workflows on time-based or event-based triggers
- **Monitoring**: Visual interface to track job progress and failures
- **Scalability**: Handles complex workflows with hundreds of tasks
- **Flexibility**: Python-based, extensible with custom operators

## Airflow Architecture Overview (Multi-Component Setup)

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Web Server │    │  Scheduler  │    │  Executor   │
│   (Port     │    │ (Triggers   │    │(Runs Tasks) │
│    8080)    │    │  DAGs)      │    │             │
└─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │
       └───────────────────┼───────────────────┘
                           │
                    ┌─────────────┐
                    │ Metadata DB │
                    │  (PostgreSQL)   │
                    └─────────────┘
```

## Step-by-Step Installation Guide (Multi-Component Mode)

### Setup with Separate Web Server and Scheduler

```bash
# 1. Create a new directory for your Airflow project
mkdir airflow-tutorial
cd airflow-tutorial

# 2. Create and activate virtual environment
python -m venv airflow-env
source airflow-env/bin/activate  # On Windows: airflow-env\Scripts\activate

# 3. Set Airflow home directory
export AIRFLOW_HOME=$(pwd)/airflow  # On Windows: set AIRFLOW_HOME=%cd%\airflow

# 4. Install Airflow (using constraints for compatibility)
pip install apache-airflow==2.8.0 --constraint https://raw.githubusercontent.com/apache/airflow/constraints-2.8.0/constraints-3.9.txt

# 5. Initialize the database
airflow db init

# 6. Create an admin user (replace with your details)
airflow users create \
    --username admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com \
    --password admin123
```

### Running Web Server and Scheduler Separately

You'll need **two terminal windows** for this setup:

#### Terminal 1: Start the Web Server
```bash
# Activate virtual environment
source airflow-env/bin/activate  # On Windows: airflow-env\Scripts\activate

# Set Airflow home
export AIRFLOW_HOME=$(pwd)/airflow  # On Windows: set AIRFLOW_HOME=%cd%\airflow

# Start the web server
airflow webserver --port 8080
```

#### Terminal 2: Start the Scheduler
```bash
# Open a new terminal window/tab
cd airflow-tutorial

# Activate virtual environment
source airflow-env/bin/activate  # On Windows: airflow-env\Scripts\activate

# Set Airflow home
export AIRFLOW_HOME=$(pwd)/airflow  # On Windows: set AIRFLOW_HOME=%cd%\airflow

# Start the scheduler
airflow scheduler
```

## Why Use Separate Components?

### Benefits of Multi-Component Setup:
- **Production-Ready**: Mirrors production deployment patterns
- **Resource Management**: Each component can be scaled independently
- **Monitoring**: Easier to monitor individual component performance
- **Debugging**: Separate logs for web server and scheduler
- **High Availability**: Can run multiple instances of each component

### Component Responsibilities:

**Web Server:**
- Serves the Airflow UI
- Handles user authentication
- Provides REST API endpoints
- Displays DAG visualization and monitoring

**Scheduler:**
- Monitors DAG files for changes
- Triggers task execution based on schedule
- Manages task dependencies
- Handles task retries and failures

## Accessing the Airflow UI

1. **Ensure both components are running**:
   - Web server should show: `Serving on http://0.0.0.0:8080`
   - Scheduler should show: `Starting the scheduler`

2. **Open your browser** and navigate to: `http://localhost:8080`

3. **Login credentials**:
   - Username: `admin`
   - Password: `admin123`

## Exploring the Airflow UI

### 1. DAGs View (Main Dashboard)
- **What you'll see**: List of all available DAGs
- **Key elements**:
  - DAG name and description
  - Recent runs (green = success, red = failed)
  - Schedule interval
  - Last run date
  - Toggle to pause/unpause DAGs

### 2. Tree View
- **Purpose**: Shows DAG runs over time
- **How to access**: Click on any DAG → Tree View tab
- **What it shows**: Task instances arranged by execution date

### 3. Graph View
- **Purpose**: Visual representation of DAG structure
- **Shows**: Task dependencies and current status
- **Color coding**:
  - Green: Success
  - Red: Failed
  - Yellow: Running
  - Light Blue: Queued
  - Gray: Not started

### 4. Code View
- **Purpose**: Shows the Python code that defines the DAG
- **Useful for**: Understanding DAG logic and debugging

### 5. Gantt Chart
- **Purpose**: Shows task execution timeline
- **Useful for**: Identifying bottlenecks and optimizing performance

## Key Concepts Explained Simply

### DAG (Directed Acyclic Graph)
A workflow definition - like a recipe that tells Airflow what tasks to run and in what order.

### Tasks
Individual units of work (like "download file", "process data", "send email").

### Operators
Templates for tasks (PythonOperator, BashOperator, EmailOperator, etc.).

### Task Instance
A specific execution of a task for a particular DAG run.

## Monitoring Your Setup

### Checking Component Status:

**Web Server Logs:**
- Look for: `Serving on http://0.0.0.0:8080`
- No error messages about port conflicts

**Scheduler Logs:**
- Look for: `Starting the scheduler`
- Regular DAG processing messages
- No database connection errors

### Health Check Commands:
```bash
# Check if web server is responding
curl http://localhost:8080/health

# List DAGs (requires both components running)
airflow dags list

# Check scheduler status
airflow jobs check --job-type SchedulerJob
```

## Assignment Solutions

### Part 1: Why Airflow is Useful in Data Engineering

Apache Airflow is essential in data engineering because it provides automated workflow orchestration that eliminates manual intervention in complex data pipelines. Unlike traditional cron jobs or script-based scheduling, Airflow offers dependency management, failure handling, and retry mechanisms that ensure data workflows run reliably at scale. Its Python-based approach allows data engineers to define workflows as code, making them version-controlled, testable, and maintainable, while the web UI provides real-time monitoring and debugging capabilities that are crucial for managing production data pipelines. The separation of the web server and scheduler components allows for better resource allocation and mirrors production deployment patterns used in enterprise environments.

### Part 2: Screenshot Documentation

**Expected Screenshot Elements:**
- Airflow UI header with "Apache Airflow" logo
- Navigation menu (DAGs, Browse, Admin, etc.)
- DAGs list showing example DAGs
- Status indicators (green/red circles)
- URL showing `localhost:8080`
- Both terminal windows showing web server and scheduler running

**Troubleshooting Common Issues:**

1. **Port 8080 already in use**: 
   ```bash
   # Kill process using port 8080
   sudo lsof -t -i:8080 | xargs sudo kill -9
   # Or use a different port
   airflow webserver --port 8081
   ```

2. **Scheduler not picking up DAGs**: 
   - Ensure scheduler is running
   - Check DAG file syntax
   - Verify DAG is not paused

3. **Database lock errors**: 
   - Stop all Airflow processes
   - Delete `airflow.db` file
   - Run `airflow db init` again

4. **Web server can't connect to database**: 
   - Ensure scheduler is running (it initializes the database)
   - Check file permissions on `airflow.db`

## Success Checklist ✅

- [ ] Airflow installed in virtual environment
- [ ] Database initialized successfully
- [ ] Admin user created
- [ ] Web server running on port 8080
- [ ] Scheduler running in separate terminal
- [ ] Can access http://localhost:8080
- [ ] Can login with admin credentials
- [ ] Can see example DAGs in the interface
- [ ] Both components showing healthy status in logs
- [ ] Can navigate between different views (Tree, Graph, Code)
- [ ] Screenshot saved showing successful multi-component setup

## Stopping Airflow Properly

To stop Airflow cleanly:
1. **Stop the scheduler**: `Ctrl+C` in the scheduler terminal
2. **Stop the web server**: `Ctrl+C` in the web server terminal
3. **Deactivate virtual environment**: `deactivate`

## Next Steps Preview
Tomorrow we'll create our first custom DAG and understand how to define tasks and dependencies using this multi-component setup!
