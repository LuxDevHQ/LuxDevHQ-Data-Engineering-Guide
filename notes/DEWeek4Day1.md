# Week 4, Day 1 - Introduction to Apache Airflow, DAGs, and Scheduling

> **Month 1: Foundations of Data Engineering**
> **Week 4: Introduction to Apache Airflow**
> **Monday - Theory**

---

## What we covered in Week 3 (recap)

| Day | Topic |
|---|---|
| Monday (Theory) | Introduction to ETL/ELT workflows - the difference, batch vs streaming, full vs incremental |
| Tuesday (Lab) | Built a simple Python ETL pipeline for CSV data using pandas |
| Wednesday (Theory) | ETL best practices - idempotency, UPSERT, retry logic, layered validation |
| Thursday (Lab) | Built a batch ETL pipeline - multiple files, folder structure, archiving, proven idempotency |

In Week 3 we built pipelines that run by executing a Python script manually. But in production, pipelines need to:
- Run **automatically** on a schedule - every hour, every day, every Monday at 6am.
- Run tasks **in a specific order** - extract first, then transform, then load.
- **Retry** automatically if a step fails.
- Show you **what succeeded and what failed** without you having to read log files.
- Let you **re-run a specific failed step** without rerunning the whole pipeline.

> **This is exactly what Apache Airflow was built to do.** Today we learn what it is, how it works, and why it is the most widely used workflow orchestration tool in data engineering today.

---

## Today's objective

By the end of today, you should be able to:

- Explain what Apache Airflow is and what problem it solves.
- Understand the Airflow 3 architecture and its core components.
- Define a DAG and explain what makes it a Directed Acyclic Graph.
- Describe the key concepts: DAGs, Tasks, Operators, and Dependencies.
- Understand how Airflow schedules DAGs using cron expressions and timetables.
- Know what is new and different in Airflow 3 compared to older versions.

---

## 1. What is Apache Airflow?

**Apache Airflow** is an open-source platform for programmatically authoring, scheduling, and monitoring workflows.

In simple terms - Airflow is a tool that:
- Lets you define your data pipeline as **Python code**.
- Runs that pipeline **automatically on a schedule**.
- Shows you the **status of every step** in a visual dashboard.
- **Retries** failed steps automatically.
- Keeps a full **history** of every pipeline run.

### Simple analogy

Think of Airflow as a very smart, automated **operations manager** for your data pipelines.

Without Airflow, you would need to:
- Manually run your Python ETL scripts at the right time.
- Check whether each step succeeded or failed.
- Re-run failed steps yourself.
- Remember which pipelines depend on which other pipelines.
- Track when the last successful run happened.

With Airflow, all of this is automated, tracked, and visible in a dashboard.

### Why Airflow is the industry standard

<cite index="13-1">Airflow has over 30 million monthly downloads and is used by 80,000 organizations worldwide.</cite> You will encounter it in virtually every data engineering role. It is used by companies of all sizes - from startups to large enterprises - to orchestrate everything from simple nightly ETL pipelines to complex machine learning workflows.

---

## 2. Airflow 3 - the current version (important)

> **Critical note for these notes:** The current stable version of Apache Airflow is **3.2.2** (released May 2026). <cite index="22-1">Airflow 2.x reached end-of-life in April 2026.</cite> These notes are written entirely for **Airflow 3** - which has a different import structure, architecture, and syntax from Airflow 2. Most tutorials you find on the internet are still written for Airflow 2 - be careful when following external resources.

### What changed in Airflow 3

<cite index="13-1">Airflow 3.0, released in April 2025, is the biggest release in Airflow's history.</cite> The key changes that affect you as a beginner:

| What changed | Airflow 2 | Airflow 3 |
|---|---|---|
| **Import location** | `from airflow import DAG` | `from airflow.sdk import dag, task` |
| **DAG definition style** | Class-based `with DAG(...)` block | Decorator-based `@dag` function |
| **Task definition** | Operator classes (still supported) | `@task` decorator via TaskFlow API |
| **DAG versioning** | No versioning - latest code always used | Every run tied to a specific DAG version |
| **Datasets (Airflow 2)** | `Dataset` for event-driven scheduling | Renamed to **Assets** in Airflow 3 |
| **UI** | Flask/FAB-based | Fully rebuilt in React - much faster |
| **Architecture** | All components access the database directly | API-driven - tasks communicate via Task Execution API |
| **Security** | Task code could access the metadata DB | <cite index="19-1">Direct metadata database access from task code is now restricted</cite> |

---

## 3. Core concepts - the vocabulary of Airflow

### DAG - Directed Acyclic Graph

A **DAG** is the central concept in Airflow. It is a collection of tasks organized to show their relationships and dependencies.

Breaking down the name:
- **Directed** - tasks flow in one direction (A runs before B, B runs before C).
- **Acyclic** - there are no cycles - a task cannot eventually depend on itself.
- **Graph** - a set of nodes (tasks) connected by edges (dependencies).

```text
                   ┌─────────────┐
                   │  extract()  │
                   └──────┬──────┘
                          │
                   ┌──────▼──────┐
                   │ transform() │
                   └──────┬──────┘
                          │
                   ┌──────▼──────┐
                   │   load()    │
                   └─────────────┘
```

This is a simple linear DAG - extract runs first, then transform, then load. But DAGs can also have parallel branches:

```text
          ┌─────────────┐
          │  extract()  │
          └──────┬──────┘
                 │
        ┌────────┴────────┐
        │                 │
┌───────▼──────┐  ┌───────▼──────┐
│ transform_A()│  │ transform_B()│
└───────┬──────┘  └───────┬──────┘
        │                 │
        └────────┬────────┘
                 │
          ┌──────▼──────┐
          │   load()    │
          └─────────────┘
```

Here, `transform_A()` and `transform_B()` run in **parallel** after extraction completes, and both must finish before loading begins.

> **Why "Acyclic" matters:** If a cycle existed - if `load()` somehow triggered `extract()` again - the pipeline would run forever. The "acyclic" constraint ensures every pipeline has a clear start and a clear end.

### Tasks

A **task** is a single unit of work within a DAG. Each task does one specific thing:
- Extract data from a database.
- Transform a DataFrame.
- Load a file to cloud storage.
- Send a notification email.
- Run a SQL query.

### Operators

An **operator** is the class that defines what a task does. Airflow has many built-in operators:

| Operator | What it does |
|---|---|
| `PythonOperator` | Runs a Python function |
| `BashOperator` | Runs a bash command |
| `PostgresOperator` | Runs a SQL query against PostgreSQL |
| `S3ToRedshiftOperator` | Moves data from S3 to Redshift |
| `EmailOperator` | Sends an email |
| `HttpOperator` | Makes an HTTP request to an API |

In Airflow 3, the **TaskFlow API** (`@task` decorator) is the recommended way to define Python tasks - it is cleaner and more Pythonic than using `PythonOperator` directly. We will use this throughout the labs.

### Task Dependencies

Dependencies define the **order** in which tasks run. In Airflow 3, they are defined using the `>>` operator (called the "bitshift" operator):

```python
extract >> transform >> load
```

This means: run `extract` first, then `transform`, then `load`.

For parallel tasks:

```python
extract >> [transform_a, transform_b] >> load
```

This means: run `extract` first, then run both `transform_a` and `transform_b` in parallel, then run `load` once both transforms are done.

### Task Instances

A **task instance** is a specific run of a task at a specific point in time. When Airflow runs your DAG on `2025-06-01`, it creates a task instance for each task in that run. The next day's run creates a new set of task instances.

This means you can look back at a specific date's run, see which tasks succeeded or failed, and re-run only the failed ones - without re-running tasks that already succeeded.

### DAG Runs

A **DAG run** is a single execution of a complete DAG. Each DAG run has:
- A **run ID** - a unique identifier.
- A **logical date** (previously called "execution date") - the time period the run represents.
- A **state** - running, success, failed, or queued.

>  **DAG Versioning in Airflow 3:** <cite index="18-1">In Airflow 3, every DAG run is tied to a specific version of the DAG code. In Airflow 2, if you updated a DAG while it was running, tasks could execute under different versions of the code - causing silent bugs. Airflow 3 solves this: once a DAG run starts, it runs to completion based on the version that existed when it started.</cite>

---

## 4. The Airflow 3 architecture

<cite index="17-1">Airflow 3 represents a fundamental redesign focusing on modularity and decoupling, with a shift to a client-server architecture.</cite> Here are the core components:

```text
┌─────────────────────────────────────────────────────────┐
│                    Airflow 3 Architecture                │
│                                                         │
│  ┌──────────┐    ┌──────────┐    ┌──────────────────┐  │
│  │Webserver │    │Scheduler │    │   DAG Processor  │  │
│  │(React UI)│    │          │    │                  │  │
│  └────┬─────┘    └────┬─────┘    └────────┬─────────┘  │
│       │               │                   │             │
│       └───────────────┼───────────────────┘             │
│                       │                                 │
│              ┌────────▼────────┐                        │
│              │   API Server    │                        │
│              │  (Task Exec API)│                        │
│              └────────┬────────┘                        │
│                       │                                 │
│              ┌────────▼────────┐                        │
│              │Metadata Database│                        │
│              │  (PostgreSQL)   │                        │
│              └─────────────────┘                        │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │                    Executor                      │  │
│  │  (Local / Celery / Kubernetes / Edge)            │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### Component breakdown

| Component | What it does |
|---|---|
| **Webserver** | The React-based UI - shows DAGs, task statuses, logs, and run history |
| **Scheduler** | Reads DAG files, creates DAG runs on schedule, triggers tasks to the executor |
| **DAG Processor** | Parses DAG files separately from the scheduler - a key Airflow 3 architectural improvement |
| **API Server** | The Task Execution API - all communication between tasks and Airflow now goes through here |
| **Metadata Database** | Stores DAG definitions, task states, run history, connections, and variables (PostgreSQL recommended) |
| **Executor** | The component that actually runs tasks - Local (single machine), Celery (distributed), or Kubernetes |

> 💡 **Why the API Server matters (Airflow 3 change):** In Airflow 2, task code could directly access and modify the metadata database - a significant security risk. <cite index="19-1">In Airflow 3, all runtime interactions (state transitions, XComs, and resource fetching) are handled through the dedicated Task Execution API. Task code can no longer directly import and use Airflow database sessions or models.</cite> This makes Airflow 3 significantly more secure and scalable.

---

## 5. DAG files - how pipelines are defined

In Airflow, a pipeline is defined as a **Python file** in a `dags/` folder. Airflow watches this folder and automatically picks up any new or changed DAG files.

### Airflow 3 DAG syntax - using the SDK

<cite index="22-1">In Airflow 3, DAG and task imports come from `airflow.sdk`</cite> - the new unified SDK surface. Here is a minimal DAG:

```python
from airflow.sdk import dag, task
from datetime import datetime


@dag(
    dag_id="my_first_dag",
    start_date=datetime(2025, 1, 1),
    schedule="@daily",
    catchup=False,
    tags=["example", "week4"]
)
def my_first_dag():

    @task
    def extract():
        print("Extracting data...")
        return {"rows": 100}

    @task
    def transform(data: dict):
        print(f"Transforming {data['rows']} rows...")
        return {"clean_rows": data["rows"] - 5}

    @task
    def load(data: dict):
        print(f"Loading {data['clean_rows']} clean rows...")

    # Define dependencies using >> operator
    raw_data = extract()
    clean_data = transform(raw_data)
    load(clean_data)


# Instantiate the DAG
my_first_dag()
```

> Notice that `extract()` returns data that gets passed directly to `transform()`. This is how the **TaskFlow API** passes data between tasks - it uses **XCom** (cross-communication) under the hood automatically. No need to manually push and pull XComs like in Airflow 2.

### Key DAG parameters

| Parameter | What it does | Example |
|---|---|---|
| `dag_id` | Unique name for the DAG - must be unique across all DAGs | `"daily_sales_etl"` |
| `start_date` | The date from which Airflow starts scheduling runs | `datetime(2025, 1, 1)` |
| `schedule` | How often to run - cron expression or preset | `"@daily"`, `"0 6 * * *"` |
| `catchup` | Whether to backfill missed runs since `start_date` | `False` (recommended for most pipelines) |
| `retries` | How many times to retry a failed task automatically | `3` |
| `retry_delay` | How long to wait between retries | `timedelta(minutes=5)` |
| `tags` | Labels for organizing DAGs in the UI | `["etl", "sales"]` |

---

## 6. Scheduling - telling Airflow when to run

### Cron expressions

Airflow uses **cron expressions** to define schedules. A cron expression has five fields:

```text
┌──────────── minute (0-59)
│  ┌─────────── hour (0-23)
│  │  ┌──────────── day of month (1-31)
│  │  │  ┌─────────── month (1-12)
│  │  │  │  ┌──────────── day of week (0-6, Sunday=0)
│  │  │  │  │
*  *  *  *  *
```

### Common schedule examples

| Schedule | Cron expression | Meaning |
|---|---|---|
| Every minute | `"* * * * *"` | Run every minute |
| Every hour | `"0 * * * *"` | Run at the start of every hour |
| Daily at midnight | `"0 0 * * *"` | Run at 12:00 AM every day |
| Daily at 6am | `"0 6 * * *"` | Run at 6:00 AM every day |
| Every Monday at 7am | `"0 7 * * 1"` | Run at 7:00 AM every Monday |
| First day of month | `"0 0 1 * *"` | Run at midnight on the 1st of every month |
| Every 15 minutes | `"*/15 * * * *"` | Run every 15 minutes |

### Preset schedules (easier to read)

Airflow provides preset names as shortcuts:

| Preset | Equivalent cron | Meaning |
|---|---|---|
| `"@once"` | (runs once) | Run only once |
| `"@hourly"` | `"0 * * * *"` | Every hour |
| `"@daily"` | `"0 0 * * *"` | Every day at midnight |
| `"@weekly"` | `"0 0 * * 0"` | Every Sunday at midnight |
| `"@monthly"` | `"0 0 1 * *"` | First day of every month |
| `None` | (no schedule) | Only run when triggered manually |

>  **`catchup=False` is almost always what you want.** If your DAG's `start_date` is 3 months ago and `catchup=True` (the default), Airflow will try to run the DAG for every missed interval since `start_date` - potentially triggering hundreds of unexpected runs. Always set `catchup=False` unless you specifically need to backfill historical data.

### The logical date

When Airflow runs a `@daily` DAG, each run represents a **24-hour window**. The `logical_date` (what Airflow 2 called `execution_date`) is the **start** of that window - not when the run was triggered.

For example, a `@daily` DAG running on `2025-06-02` has a `logical_date` of `2025-06-01` - it represents "the data for June 1st."

This matters for pipelines that process "yesterday's data" - you use the logical date to know which day's data to extract.

---

## 7. XCom - passing data between tasks

**XCom** (Cross-Communication) is Airflow's mechanism for tasks to share small pieces of data with each other.

In the TaskFlow API (Airflow 3), XCom is handled automatically when a task returns a value - you don't need to push/pull manually:

```python
@task
def extract():
    return {"record_count": 1000}   # This is automatically pushed to XCom

@task
def transform(data: dict):          # This is automatically pulled from XCom
    print(f"Processing {data['record_count']} records")
```

>  **XCom is for small data only** - things like counts, status flags, file paths, or configuration values. Never use XCom to pass a large DataFrame or a file's contents. For large data, write to cloud storage (S3, ADLS, GCS) in one task and pass only the **file path** via XCom to the next task.

---

## 8. Assets - event-driven scheduling (Airflow 3)

In Airflow 2, there was a feature called **Datasets** that let one DAG trigger another when data was produced. <cite index="22-1">In Airflow 3, Datasets have been renamed to **Assets** and significantly expanded</cite> to support true event-driven scheduling.

```python
from airflow.sdk import dag, task, Asset

# Define an asset - a logical representation of a data artifact
sales_data = Asset("s3://my-bucket/sales/daily/")

@dag(schedule="@daily")
def producer_dag():
    @task(outlets=[sales_data])     # This task "produces" the asset
    def load_to_s3():
        # load data to S3
        pass
    load_to_s3()

@dag(schedule=[sales_data])         # This DAG triggers when sales_data is updated
def consumer_dag():
    @task
    def process_sales():
        pass
    process_sales()
```

>  **Assets are a powerful pattern for data-driven pipelines.** Instead of scheduling the second DAG to run "every day at 6:05 AM, assuming the first DAG finished," you simply say "run when the data is ready." This eliminates fragile timing-based dependencies between pipelines.

---

## 9. Task states - understanding the Airflow UI

Every task instance in Airflow has a state. Understanding these states is essential for monitoring and debugging:

| State | Color in UI | Meaning |
|---|---|---|
| `queued` | Grey | Task is waiting to be picked up by the executor |
| `running` | Light green | Task is currently executing |
| `success` | Dark green | Task completed successfully |
| `failed` | Red | Task failed - Airflow will retry if retries are configured |
| `up_for_retry` | Orange | Task failed and is waiting to retry |
| `skipped` | Pink | Task was skipped (due to a branch condition) |
| `upstream_failed` | Orange/red | A task this task depends on failed - this task won't run |

---

## 10. How Airflow connects to the rest of our pipeline work

```text
Week 3 (what we built)               Week 4 (what Airflow adds)
──────────────────────                ──────────────────────────
Python script run manually      →     DAG run automatically on schedule
Functions called in sequence    →     Tasks with defined dependencies
No retry on failure             →     Automatic retry with configurable delay
Check logs manually             →     Visual dashboard showing all task states
Re-run the whole script         →     Re-run only the failed task instance
No run history                  →     Full history of every run, every task
```

Airflow does not replace the Python ETL code we wrote in Week 3 - it **wraps and orchestrates** it. The `extract()`, `transform()`, and `load()` functions we wrote become individual Airflow tasks inside a DAG.

---

## 11. Key terms to know from today

| Term | Meaning |
|---|---|
| Apache Airflow | An open-source platform for authoring, scheduling, and monitoring workflows |
| DAG | Directed Acyclic Graph - a pipeline defined as a collection of tasks and their dependencies |
| Task | A single unit of work within a DAG |
| Operator | The class that defines what a task does |
| TaskFlow API | The `@task` decorator approach for defining Python tasks in Airflow 3 |
| Task instance | A specific execution of a task at a specific point in time |
| DAG run | A single execution of a complete DAG |
| Scheduler | The Airflow component that triggers DAG runs on schedule |
| Executor | The component that actually runs tasks (Local, Celery, Kubernetes) |
| Metadata database | PostgreSQL database that stores all Airflow state, history, and configuration |
| XCom | Cross-communication - Airflow's mechanism for passing small data between tasks |
| Cron expression | A string that defines when a schedule should trigger |
| Logical date | The time period a DAG run represents (not when it was triggered) |
| `catchup` | Whether Airflow should backfill missed runs since `start_date` |
| Asset | An Airflow 3 concept representing a data artifact - used for event-driven scheduling |
| DAG versioning | Airflow 3 feature that ties every run to a specific version of the DAG code |
| `airflow.sdk` | The unified import location for DAG and task definitions in Airflow 3 |

---

## 12. Summary

> **Apache Airflow** is the industry-standard tool for orchestrating data pipelines. It lets you define workflows as Python code (DAGs), run them automatically on a schedule, and monitor every task's status in a visual dashboard.
>
> The current version is **Airflow 3.2.2**. Airflow 3 is a significant redesign from Airflow 2 - with a new SDK-based import structure (`from airflow.sdk import dag, task`), DAG versioning, Assets for event-driven scheduling, a rebuilt React UI, and a more secure API-driven architecture. Airflow 2 reached end-of-life in April 2026 - all code in these notes uses Airflow 3 syntax.
>
> Tomorrow we install Airflow 3 and build our first real working DAG.

---

## 13. What's coming next

| Day | Topic |
|---|---|
| Tuesday (Lab) | Set up Apache Airflow 3 and create a basic DAG |
| Wednesday (Theory) | DAG best practices and scheduling in Airflow |
| Thursday (Lab) | Integrate Airflow with PostgreSQL and Azure Blob Storage |
| Friday | Job shadowing - observe real-world Airflow pipelines |
| Saturday (Mini Project) | Automate an ETL pipeline with Airflow for batch data processing |

---

*Notes by LuxDevHQ | Month 1 - Foundations of Data Engineering | Week 4, Day 1*