# LuxDevHQ Data Engineering Curriculum (6 Months, Extendable to 7)

## Program Design Principles
- **Total baseline duration:** 24 weeks (about 6 months)
  - **Weeks 1–16:** Instructor-led training
  - **Weeks 17–24:** Internship (real-world projects)
- **Optional extension:** up to **Weeks 25–28** (remedial, specialization, interview readiness) based on learner pace.
- **Learning philosophy:** 70% hands-on labs/projects, 20% guided instruction, 10% assessments/reflection.
- **Portfolio-first:** Every learner ships production-style artifacts (pipelines, dashboards, orchestration DAGs, dbt models, CI/CD, cloud deployment).

---

## Industry Alignment Snapshot (2026-ready)
This curriculum aligns with modern data engineering expectations: SQL + Python depth, cloud-native pipelines, batch + streaming systems, orchestration, testing, observability, and governance. It also follows current platform directions from Microsoft Learn (Power BI/Azure data engineer role expectations), Apache Airflow/Kafka/Spark documentation, Docker guidance, and dbt/Grafana practices.

---

## Week-by-Week Curriculum

> **Week 1 remains exactly as currently designed at LuxDevHQ** (Foundations are assumed complete and unchanged).

---

### Week 2 — Power BI I: Data Preparation & Core Visuals
**Learning objectives**
- Connect Power BI to CSV, Excel, and SQL sources.
- Clean and shape data using Power Query.
- Build first dashboard with meaningful KPIs.

**Key concepts**
- BI workflow, Power Query, relationships, star schema basics.
- Visual encoding, chart selection, dashboard usability.

**Hands-on project (Kenyan context)**
- **County Health Dashboard:** Analyze outpatient visits and medicine stock by county.

**Tools/technologies**
- Power BI Desktop, Excel/CSV, SQL Server/PostgreSQL (read-only).

---

### Week 3 — Power BI II: Data Storytelling & Publishing
**Learning objectives**
- Create interactive reports and drill-through pages.
- Build DAX measures for business logic.
- Publish and share reports responsibly.

**Key concepts**
- DAX basics, row/filter context (intro), RLS, storytelling with data.
- Report usability, stakeholder communication.

**Hands-on project (African context)**
- **Mobile Money Insights:** Transaction trends by region, customer segment, and anomaly flags.

**Tools/technologies**
- Power BI Service, DAX, workspace sharing.

---

### Week 4 — SQL Fundamentals for Data Engineers
**Learning objectives**
- Write robust SQL queries for analysis and engineering.
- Use joins, aggregations, CTEs, and window functions.

**Key concepts**
- SELECT pipeline, joins, GROUP BY/HAVING, CASE, CTEs.
- Query quality: readability, correctness, reproducibility.

**Hands-on project**
- **Retail Orders Mart:** Build daily sales and customer activity queries.

**Tools/technologies**
- PostgreSQL, DBeaver/pgAdmin, Git.

---

### Week 5 — Advanced SQL & Data Modeling
**Learning objectives**
- Design analytical schemas for reporting and pipeline outputs.
- Optimize queries and reason about performance.

**Key concepts**
- Normalization vs denormalization, star/snowflake schemas.
- Indexing, partitions (intro), execution plans, SCD concepts.

**Hands-on project (Kenyan context)**
- **SACCO Analytics Warehouse Model:** Members, loans, repayments, branch dimensions.

**Tools/technologies**
- PostgreSQL, SQL execution plans, dbdiagram.

---

### Week 6 — Python for Data Engineering I
**Learning objectives**
- Build Python scripts for ingestion and data cleanup.
- Work safely with files, APIs, and structured data.

**Key concepts**
- Python fundamentals for DE: functions, modules, virtual envs.
- Pandas basics, JSON/CSV handling, API requests.

**Hands-on project**
- **Public Data Ingestion Script:** Pull and standardize open data from government/API sources.

**Tools/technologies**
- Python, Pandas, Requests, Jupyter/VS Code.

---

### Week 7 — Python for Data Engineering II (Automation & Quality)
**Learning objectives**
- Create maintainable ETL scripts.
- Add tests, logging, configs, and CLI behavior.

**Key concepts**
- Project structure, argparse/typer, logging, exception strategy.
- Data validation (Great Expectations/Pandera intro), unit tests.

**Hands-on project**
- **Automated Data Cleaner CLI:** Scheduled script with logs and validation report.

**Tools/technologies**
- Python, pytest, Great Expectations/Pandera, pre-commit.

---

### Week 8 — ETL/ELT Concepts & Pipeline Design
**Learning objectives**
- Design reliable pipelines from source to warehouse.
- Choose ETL vs ELT patterns based on constraints.

**Key concepts**
- Batch ingestion, CDC intro, idempotency, incremental loads.
- Medallion/layered architecture (bronze/silver/gold).

**Hands-on project (African context)**
- **NGO Donations Pipeline:** ingest, clean, transform, and load trusted reporting tables.

**Tools/technologies**
- Python SQL pipeline scripts, PostgreSQL, object storage (MinIO/S3-compatible).

---

### Week 9 — Data Warehousing & Lakehouse Foundations (Missing Critical Topic)
**Learning objectives**
- Build modern analytical storage architecture.
- Compare warehouse, lake, lakehouse patterns.

**Key concepts**
- Kimball basics, data marts, partition strategy.
- Lakehouse table formats concept (Delta/Iceberg overview).

**Hands-on project**
- **E-commerce Data Mart Design:** Sales, inventory, and customer behavior marts.

**Tools/technologies**
- PostgreSQL/BigQuery/Synapse (choose one primary), Parquet.

---

### Week 10 — Cloud for Data Engineers (Missing Critical Topic)
**Learning objectives**
- Deploy storage and pipeline components on cloud.
- Apply IAM, secrets management, and basic cost control.

**Key concepts**
- Core cloud services: compute, storage, IAM, networking basics.
- Managed data services (Azure-first with AWS/GCP comparison).

**Hands-on project (Kenyan context)**
- **County Revenue Cloud Landing Zone:** Secure ingestion bucket + warehouse schema.

**Tools/technologies**
- Azure (recommended), optionally AWS/GCP mapping, Terraform intro.

---

### Week 11 — Containerization with Docker
**Learning objectives**
- Package Python/SQL pipeline apps as containers.
- Run local multi-service data stack with Compose.

**Key concepts**
- Dockerfile, images, containers, networks, volumes.
- Environment parity across dev/test/prod.

**Hands-on project**
- **Containerized ETL App:** ETL service + Postgres + admin UI via Docker Compose.

**Tools/technologies**
- Docker, Docker Compose, Docker Hub/GHCR.

---

### Week 12 — Workflow Orchestration with Apache Airflow
**Learning objectives**
- Author, schedule, and monitor DAGs.
- Build retry-safe, production-aware workflows.

**Key concepts**
- DAG/task dependencies, sensors, hooks/operators.
- Backfills, SLAs, retries, idempotent tasks, environment separation.

**Hands-on project**
- **Daily Data Pipeline DAG:** ingestion → validation → transform → publish.

**Tools/technologies**
- Apache Airflow, Dockerized Airflow, SQL/Python operators.

---

### Week 13 — Batch Data Processing at Scale
**Learning objectives**
- Build reliable large-volume batch jobs.
- Optimize processing windows and throughput.

**Key concepts**
- Partitioned processing, checkpointing, job scheduling.
- Late-arriving data and backfill strategy.

**Hands-on project (African context)**
- **Utility Billing Batch System:** monthly billing and reconciliation from meter files.

**Tools/technologies**
- Python + SQL batch jobs, Airflow scheduling, cloud/object storage.

---

### Week 14 — Real-Time Data Processing & Streaming Fundamentals
**Learning objectives**
- Understand streaming architecture and event-time processing.
- Build near-real-time analytics pipeline.

**Key concepts**
- Event streams, windows, watermarking, exactly-once vs at-least-once.
- Stream processing use cases and operational trade-offs.

**Hands-on project**
- **Ride/Delivery Tracking Stream:** compute live KPIs from events.

**Tools/technologies**
- Spark Structured Streaming (intro) or Kafka Streams conceptually.

---

### Week 15 — Messaging Systems with Apache Kafka
**Learning objectives**
- Produce/consume events and design robust topics.
- Integrate Kafka with downstream processing/storage.

**Key concepts**
- Brokers, topics/partitions, consumer groups, retention.
- Schema management intro, dead-letter queues, replay.

**Hands-on project (Kenyan context)**
- **M-Pesa-like Event Bus Simulation:** transaction events with fraud signal stream.

**Tools/technologies**
- Apache Kafka, Kafka UI, Python producers/consumers.

---

### Week 16 — Data Transformation with dbt + Monitoring with Grafana
**Learning objectives**
- Build tested transformation layers with dbt.
- Monitor pipeline/system health and data freshness.

**Key concepts**
- dbt models, tests, documentation, lineage, snapshots.
- Metrics dashboards, alerts, SLO basics, runbook practice.

**Hands-on project**
- **Analytics Engineering Layer:** transform raw data to business marts + Grafana alerts.

**Tools/technologies**
- dbt Core/Cloud, PostgreSQL/warehouse, Grafana, Prometheus/Loki basics.

---

## Internship Phase (Weeks 17–24)

### Week 17 — Internship Onboarding & Problem Framing
**Learning objectives:** translate business needs into data requirements.

**Key concepts:** requirement elicitation, KPI contracts, data contracts.

**Project:** team charter + architecture proposal + sprint backlog.

**Tools:** Jira/Trello, GitHub Projects, Miro, RFC template.

---

### Week 18 — Solution Architecture & Environment Setup
**Learning objectives:** provision environments and choose architectural pattern.

**Key concepts:** dev/stage/prod, secrets management, IAM roles.

**Project:** deploy base stack (ingestion + storage + orchestration).

**Tools:** Cloud platform, Terraform (guided), Docker, Airflow.

---

### Week 19 — Data Ingestion & Quality Gates
**Learning objectives:** implement ingestion from at least 2 heterogeneous sources.

**Key concepts:** schema evolution, validation checks, quarantine pattern.

**Project:** source-to-bronze pipelines with quality scorecard.

**Tools:** Python, Airflow, Great Expectations/Pandera, SQL.

---

### Week 20 — Transformation & Modeling for Analytics
**Learning objectives:** model silver/gold layers for business analytics.

**Key concepts:** dimensional modeling, incremental transformations, dbt tests.

**Project:** curated business marts and semantic metrics layer.

**Tools:** dbt, SQL warehouse, documentation site generation.

---

### Week 21 — Streaming/Message-Driven Extension
**Learning objectives:** add one real-time component to an existing pipeline.

**Key concepts:** topic design, stream enrichment, failure handling.

**Project:** Kafka-driven near-real-time operational dashboard.

**Tools:** Kafka, Spark Structured Streaming or lightweight consumers.

---

### Week 22 — Observability, Reliability, and Incident Response
**Learning objectives:** monitor and troubleshoot pipelines under failures.

**Key concepts:** metrics/logs/traces basics, alert fatigue reduction, on-call playbooks.

**Project:** define SLOs, alerts, runbooks, and execute game-day drill.

**Tools:** Grafana, Prometheus/Loki, Airflow monitoring.

---

### Week 23 — CI/CD, Testing, and Production Deployment (Missing Critical Topic)
**Learning objectives:** automate quality checks and deployment.

**Key concepts:** CI pipelines, artifact versioning, migration workflows, rollback.

**Project:** GitHub Actions/GitLab CI for tests + deploy DAG/dbt pipeline.

**Tools:** GitHub Actions/GitLab CI, pytest, pre-commit, Docker registry.

---

### Week 24 — Capstone Delivery, Portfolio & Career Readiness
**Learning objectives:** present a production-like project and justify trade-offs.

**Key concepts:** system design storytelling, documentation, interview preparation.

**Project:** capstone demo day + technical defense + portfolio publication.

**Tools:** GitHub portfolio, architecture diagrams, slide deck, demo environment.

---

## Optional Extension (Weeks 25–28, only if needed)

### Week 25 — Remedial SQL/Python/Modeling Clinics
Targeted support by diagnostic gaps.

### Week 26 — Cloud Specialization Track
Choose Azure/AWS/GCP deepening labs.

### Week 27 — Advanced Spark & Performance Optimization
Tuning, joins, partitioning, cost/performance trade-offs.

### Week 28 — Interview Sprint + Certification Support
Mock interviews, CV refinement, optional cert prep (e.g., Power BI, cloud DE tracks).

---

## Missing-but-Critical Topics Added (and Why)
1. **Data Warehousing & Lakehouse Architecture** — foundational for real analytics platforms.
2. **Cloud Data Engineering** — most DE roles are cloud-first.
3. **CI/CD for Data Pipelines** — modern teams require automated testing and deployment.
4. **Data Quality, Governance, Security** — trust, compliance, and production safety.
5. **Observability & Incident Response** — job-readiness for operating live systems.
6. **Portfolio + Career Engineering** — improves hiring outcomes.

---

## Assessment & Portfolio Framework
- **Weekly check-ins:** lab completion + mini-quizzes + code review.
- **End-of-module assessments:** practical build challenge (not theory-only).
- **Portfolio artifacts expected by Week 24:**
  - 1 BI dashboard (Power BI)
  - 2 SQL case studies
  - 2 Python ETL automation projects
  - 1 Dockerized data app
  - 1 Airflow orchestrated pipeline
  - 1 Kafka/Spark streaming mini-system
  - 1 dbt project with tests/docs/lineage
  - 1 monitoring stack with alerts (Grafana)
  - 1 capstone repository with CI/CD and architecture docs

---

## Role Outcomes (Graduate Readiness)
By completion, learners should be prepared for:
- Junior Data Engineer
- Analytics Engineer (entry level)
- BI/Data Pipeline Developer
- Data Operations Engineer (entry level)

They will be able to design, build, test, deploy, and monitor end-to-end data systems aligned with current industry workflows.
