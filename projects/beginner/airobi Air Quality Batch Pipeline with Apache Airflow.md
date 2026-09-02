### **Data Engineering Project: Nairobi Air Quality Batch Pipeline with Apache Airflow**
### **3-Day Class Schedule**

#### **Project Scenario**

You have joined a data engineering team responsible for building an environmental data platform for Nairobi.

The analytics team wants to monitor air-quality conditions across the city using data published by **OpenAQ**.

At the moment, analysts manually retrieve data from the OpenAQ API whenever they need to perform analysis. This approach is unreliable, repetitive, and difficult to scale.

Your task is to design and implement an automated **batch data pipeline using Apache Airflow**.

The pipeline should periodically retrieve air-quality measurements from monitoring stations around Nairobi, validate the data, and load it into PostgreSQL for downstream analytics.

---

#### **The Engineering Problem**

Build an Airflow pipeline that performs the following workflow:

```text
OpenAQ API
     |
     v
Discover Nairobi Locations
     |
     v
Discover Sensors
     |
     v
Extract Measurements
     |
     v
Validate Data
     |
     v
PostgreSQL
```

The pipeline must be capable of running repeatedly without creating duplicate data.

Airflow should be responsible for **orchestration**. Airflow is not the database, transformation engine, or API. Its job is to coordinate when and how the different pieces of the pipeline execute.

---

#### **Project Objective**

By the end of the three days, you should have a working Apache Airflow DAG that:

1. Retrieves air-quality monitoring locations around Nairobi.
2. Discovers the sensors available at each location.
3. Retrieves recent pollutant measurements from those sensors.
4. Validates incoming measurements.
5. Loads clean records into PostgreSQL.
6. Prevents duplicate records when the pipeline is rerun.
7. Handles API and database failures appropriately.
8. Executes automatically on a schedule.
9. Demonstrates dynamic task mapping.
10. Can be monitored and debugged using the Airflow UI.

---

#### **Important Concept: This Is a Batch Pipeline**

The OpenAQ API continuously receives new measurements, but your Airflow pipeline will **not continuously listen for events**.

Instead, Airflow will periodically ask:

> What data is available since the previous pipeline run?

Each DAG run represents one discrete batch-processing cycle.

```text
01:00 DAG Run                    02:00 DAG Run
    ↓                                 ↓
Collect available measurements   Collect next measurements
    ↓                                 ↓
Validate                         Validate
    ↓                                 ↓
Load                              Load
    ↓                                 ↓
Complete                         Complete
```

You should be able to explain why this differs from a streaming architecture using technologies such as Kafka.

---
---

#### **Day 1 — Investigate, Design, and Set Up Infrastructure**

**Goal for today:** understand the data source, design the pipeline on paper, and have Airflow + PostgreSQL running locally with a target table created. No DAG code yet.

##### **Data Source**

You will use the **OpenAQ API v3**.

Base endpoint:

```text
https://api.openaq.org/v3
```

You will need an OpenAQ API key.

Your API key must **never be hardcoded inside your Python files or committed to GitHub**. Store credentials using an Airflow Connection (you'll set this up in the infrastructure step below).

##### **Phase 1 — API Investigation**

Before creating your DAG, investigate the API manually.

You need to understand the relationship between:

```text
Nairobi
   ↓
Locations
   ↓
Sensors
   ↓
Measurements
```

Start by retrieving locations around Nairobi using approximately:

```text
Latitude:  -1.286389
Longitude: 36.817223
Radius:    25 km
```

Then investigate:

```text
GET /v3/locations
```

For one location investigate:

```text
GET /v3/locations/{location_id}/sensors
```

For one sensor investigate:

```text
GET /v3/sensors/{sensor_id}/measurements
```

###### **Investigation Questions**

Before writing your DAG, answer:

* What is a location?
* What is a sensor?
* What is a measurement?
* Can one location contain several sensors?
* What pollutants are available?
* How is measurement time represented?
* What units are used?
* How does pagination work?
* What information exists inside the API `meta` object?
* What happens when an invalid API key is supplied?
* What HTTP status code is returned?

##### **Phase 2 — Design the Pipeline**

Before implementing anything, draw your proposed DAG.

A possible conceptual workflow is:

```text
extract_locations
        |
        v
extract_sensors
        |
        v
extract_measurements
        |
        v
validate_measurements
        |
        v
load_to_postgres
```

However, there is an important complication.

Nairobi may have many locations. Each location may have many sensors. Therefore:

```text
                 Location A
               /    |     \
              /     |      \
          PM2.5    PM10    NO2

Nairobi →       Location B
               /        \
             PM2.5       PM10

                 Location C
                    |
                   NO2
```

You should investigate how **Dynamic Task Mapping** can allow Airflow to create tasks at runtime rather than hardcoding every location and sensor.

##### **Phase 3 — Infrastructure**

Run the project using Docker Compose.

Your environment should contain at minimum:

```text
Apache Airflow
│
├── API Server / Web UI
├── Scheduler
├── DAG Processor
├── Airflow Metadata Database
│
└── Worker / Executor components

PostgreSQL Warehouse
│
└── Air Quality Data

OpenAQ API
```

The PostgreSQL warehouse used for air-quality measurements should be logically separate from Airflow's metadata database.

Set up your Airflow Connections now, while infrastructure is fresh in your mind:
- One Connection for your PostgreSQL warehouse
- One Connection storing your OpenAQ API key

##### **Phase 4 — Database Design**

Design a PostgreSQL table capable of storing measurements.

At minimum, think about storing:

```text
location_id
location_name
sensor_id
parameter
unit
value
measurement_timestamp
loaded_timestamp
```

Do not immediately create an auto-incrementing ID and assume the problem is solved. Answer this question first:

> What uniquely identifies one pollutant measurement?

Your table design must support **idempotency**. Running the same batch multiple times should not produce duplicated measurements. You may use PostgreSQL constraints and `ON CONFLICT` behaviour to achieve this.

###### **Day 1 Checkpoint**

Before you leave today, you should have:
- [ ] Answered all API investigation questions
- [ ] A sketched/drawn pipeline diagram, including where dynamic task mapping applies
- [ ] Airflow and a separate PostgreSQL warehouse running via Docker Compose
- [ ] An Airflow Connection storing your OpenAQ API key (not hardcoded anywhere)
- [ ] Your `raw_air_quality` table created, with a primary key that enforces idempotency

---
---

#### **Day 2 — Build and Run the DAG**

**Goal for today:** implement every task in the pipeline, load real Nairobi data into PostgreSQL, and prove the pipeline is idempotent.

##### **Phase 5 — Build the DAG**

Implement the following logical tasks.

###### **`extract_locations`**

Retrieve available monitoring locations around Nairobi.

Expected output:

```python
[
    {
        "id": 123,
        "name": "Station A"
    },
    ...
]
```

###### **`extract_sensors`**

For every discovered location, retrieve its available sensors.

A location may have sensors measuring pollutants such as:

```text
PM2.5
PM10
NO2
CO
O3
SO2
```

Your implementation must not assume every location contains the same sensors.

###### **`extract_measurements`**

Retrieve measurements from every relevant sensor.

Your implementation should consider:

```text
pagination
HTTP errors
timeouts
rate limits
empty responses
```

###### **`validate_measurements`**

Validate the extracted data before loading it.

At minimum, handle:

```text
null measurements
missing sensor IDs
missing timestamps
impossible values
duplicate records
```

Do not silently discard invalid data without recording how many records were rejected. Your Airflow logs should show something similar to:

```text
Records extracted: 1250
Records valid:     1237
Records rejected:    13
```

###### **`load_to_postgres`**

Load valid measurements into PostgreSQL.

The loading strategy must be **idempotent**. You should be able to safely rerun the same DAG without corrupting the warehouse or creating duplicate measurements.

##### **Phase 6 — Airflow Requirements**

Your DAG must demonstrate the following concepts:

```text
DAG
Tasks
TaskFlow API
Dependencies
Dynamic Task Mapping
Airflow Connections
Hooks
Retries
Retry Delay
Scheduling
Catchup
XCom
Task Logs
Mapped Task Instances
Idempotency
```

You should be able to explain where each concept appears in your implementation.

##### **Run It**

Trigger the DAG manually. Watch the Graph view — `extract_sensors` and `extract_measurements` should fan out into multiple mapped task instances. Open a mapped task instance's logs to confirm what it actually fetched.

##### **Mandatory Idempotency Test**

This test must be demonstrated during presentation.

Before rerunning the pipeline:

```sql
SELECT COUNT(*)
FROM raw_air_quality;
```

Record the result.

Trigger the DAG again.

Then run:

```sql
SELECT COUNT(*)
FROM raw_air_quality;
```

Explain the difference.

Simply saying:

> "The DAG ran successfully."

is not enough. You must prove that the pipeline can safely be rerun.

##### **Data Quality Verification**

After loading the data, demonstrate queries such as:

```sql
SELECT
    location_name,
    parameter,
    COUNT(*) AS measurements,
    MIN(measured_at_utc),
    MAX(measured_at_utc)
FROM raw_air_quality
GROUP BY location_name, parameter
ORDER BY location_name, parameter;
```

Your results should contain actual monitoring stations and measurements retrieved from OpenAQ.

###### **Day 2 Checkpoint**

Before you leave today, you should have:
- [ ] All five tasks implemented and chained correctly
- [ ] A successful DAG run showing mapped task instances in the Graph view
- [ ] Validation logs showing extracted/valid/rejected record counts
- [ ] Real Nairobi station data visible in `raw_air_quality`
- [ ] Completed and documented the idempotency test (row counts before/after rerun)

---
---

#### **Day 3 — Break It, Schedule It, and Present It**

**Goal for today:** deliberately fail the pipeline in controlled ways, configure scheduling, finish documentation, and prepare your presentation.

##### **Phase 7 — Failure Engineering**

A production data engineer does not only demonstrate successful pipelines. You must deliberately break your pipeline.

Test at least the following failures.

###### **Scenario 1 — Invalid API Key**

Change or remove the API credentials.

Observe:

```text
task failure
HTTP error
Airflow logs
retry behaviour
final DAG state
```

###### **Scenario 2 — PostgreSQL Unavailable**

Stop the warehouse container and trigger the pipeline. What happens?

###### **Scenario 3 — Invalid Measurement**

Pass a null or negative pollutant measurement through your validation logic. Does the pipeline detect it?

###### **Scenario 4 — Rerun**

Trigger the same pipeline again. Does PostgreSQL receive duplicate rows?

Document what you observe for each scenario.

##### **Phase 8 — Scheduling**

Configure your DAG to run automatically.

Start with:

```text
@daily
```

Then answer:

> What would change if this pipeline were scheduled hourly?

Consider:

```text
API volume
number of measurements
duplicate handling
time windows
database growth
rate limits
pipeline runtime
```

##### **README Questions**

Your README should answer:

**Architecture** — Why did you choose this task structure?

**Batch Processing** — Why is this a batch pipeline rather than a streaming pipeline?

**Dynamic Task Mapping** — Why is `.expand()` useful in this problem?

**Connections** — Why shouldn't the OpenAQ API key be stored directly inside the DAG?

**Idempotency** — What would happen if the DAG ran twice without duplicate protection?

**Reliability** — What failures could happen while communicating with OpenAQ?

**Data Quality** — What makes an air-quality measurement invalid?

**Scalability** — What problems might appear if you expanded the pipeline from Nairobi to every monitoring station in Africa?

##### **Required Project Structure**

Your GitHub repository should be organized cleanly.

```text
nairobi-air-quality-airflow/
│
├── dags/
│   └── nairobi_air_quality.py
│
├── sql/
│   └── create_tables.sql
│
├── config/
│
├── logs/
│
├── tests/
│
├── docker-compose.yaml
├── requirements.txt
├── .env.example
├── .gitignore
└── README.md
```

Never commit:

```text
API keys
passwords
.env
Airflow secrets
database credentials
```

##### **Required Deliverables**

Each student/team must submit:

1. GitHub repository containing the complete project.
2. Working Docker Compose environment.
3. Airflow DAG.
4. PostgreSQL table creation script.
5. README explaining how to run the project.
6. Architecture diagram.
7. Screenshot of a successful DAG run.
8. Screenshot showing Dynamic Task Mapping.
9. SQL query demonstrating loaded data.
10. Evidence of the idempotency test.
11. Example of a failed task and explanation of how it was diagnosed.
12. Short technical write-up explaining the design decisions.

##### **Presentation**

Each team/student should be prepared to demonstrate the project live.

Your presentation should show:

```text
1. Architecture
2. OpenAQ API
3. Airflow DAG
4. Dynamic mapped tasks
5. Task logs
6. PostgreSQL data
7. Idempotency test
8. Failure and retry behaviour
```

Do not spend the presentation reading Python code line by line. Explain your **engineering decisions**.

###### **Day 3 Checkpoint**

Before presenting, you should have:
- [ ] Documented all four failure scenarios with what you observed
- [ ] A working `@daily` schedule, plus a written answer on what hourly scheduling would change
- [ ] README fully answering all eight questions
- [ ] Clean repository structure with no committed secrets
- [ ] All 12 deliverables ready
- [ ] A presentation ready that explains decisions, not just code

---
---

#### **Assessment Rubric**

| Area                           | Weight |
| ------------------------------ | -----: |
| Airflow DAG design             |    20% |
| Extraction and API handling    |    15% |
| Dynamic Task Mapping           |    10% |
| PostgreSQL/database design     |    10% |
| Idempotency                    |    15% |
| Data validation                |    10% |
| Failure handling and retries   |    10% |
| Documentation and architecture |     5% |
| Presentation and explanation   |     5% |

**Total: 100%**

---

#### **Stretch Challenges (if time remains after Day 3)**

###### **Challenge 1 — Data Mart**

Create:

```text
raw_air_quality
        ↓
stg_air_quality
        ↓
daily_air_quality_summary
```

Calculate daily averages by pollutant and station.

###### **Challenge 2 — dbt**

Introduce dbt for transformations and data-quality tests.

###### **Challenge 3 — Alerting**

Generate an alert when PM2.5 exceeds a defined threshold.

###### **Challenge 4 — Analytics Dashboard**

Connect Grafana, Power BI, or another visualization tool to PostgreSQL. Visualize:

```text
PM2.5 over time
PM10 over time
pollution by station
daily averages
highest pollution locations
```

###### **Challenge 5 — Incremental Extraction**

Instead of repeatedly requesting all available measurements, determine the last successful measurement timestamp and request only the required time window.

###### **Challenge 6 — Expand Beyond Nairobi**

Make the pipeline configurable:

```text
CITY=Nairobi
LATITUDE=-1.286389
LONGITUDE=36.817223
RADIUS=25000
```

Then demonstrate that the same DAG design can be reused for another city.

---

#### **Final Engineering Question**

At the end of the project you should be able to answer:

> If this DAG fails at 2:00 AM, can I safely rerun it at 8:00 AM without manually cleaning the database first?

If the answer is **yes**, and you can explain why, you have understood one of the most important principles in production data engineering:

**Reliable pipelines are designed to be safely rerunnable.**
