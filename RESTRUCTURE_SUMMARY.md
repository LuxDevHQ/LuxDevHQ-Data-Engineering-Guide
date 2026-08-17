# Repository Restructure Summary

## Purpose

The repository was reorganized around a topic-first learning path. Existing educational text was moved with Git-aware renames and retained; project guides and navigation were added around it. No useful lesson or project was intentionally deleted.

## Folders created

- `notes/01-getting-started/` through `notes/12-change-data-capture/` provide the recommended topic order.
- `projects/beginner/`, `projects/intermediate/`, `projects/advanced/`, and `projects/capstone/` separate build difficulty. No empty decorative project directory is tracked.
- `exercises/sql/`, `exercises/kafka/`, and `exercises/assignments/` separate focused practice from projects.
- `datasets/` documents shared sample inputs.

## Migration map

| Previous path | New path | Reason |
|---|---|---|
| `Day1-WeekOneDayOneGude.md` | `notes/01-getting-started/introduction-to-data-engineering-legacy.md` | Preserve the complementary earlier introduction and correct unclear spelling. |
| `notes/DEWeek1Day1.md` | `notes/01-getting-started/introduction-to-data-engineering.md` | Give the current introduction a topic-based name. |
| `ToolsAndTechnologiesInstallation Guide.md` | `notes/01-getting-started/tools-installation.md` | Group setup guidance. |
| `LuxDevHQ_Data_Engineering_Curriculum_6_to_7_Months.md` | `notes/01-getting-started/curriculum-6-to-7-months.md` | Preserve the alternate curriculum for instructor review. |
| `notes/DEWeek1Day2.md` | `notes/02-cloud-and-linux/cloud-computing-fundamentals.md` | Use a descriptive current-lesson name. |
| `Day2-intro_to_CloudComputing.md` | `notes/02-cloud-and-linux/cloud-computing-detailed-guide.md` | Retain its additional detailed coverage rather than merge it. |
| `IntroductiontoCloudComputing.md` | `notes/02-cloud-and-linux/cloud-computing-session-plan.md` | Identify it as complementary session material. |
| `notes/DEWeek1Day4.md` | `notes/03-sql-and-databases/postgresql-setup-and-sql-introduction.md` | Describe the actual lesson. |
| `notes/DEWeek2Day1.md` | `notes/03-sql-and-databases/core-sql.md` | Topic-based name. |
| `notes/DEWeek2Day2.md` | `notes/03-sql-and-databases/advanced-sql.md` | Topic-based name. |
| `notes/DEWeek2Day3.md` | `notes/03-sql-and-databases/postgresql-query-optimization.md` | Distinguish PostgreSQL coverage from the MySQL reference. |
| `SQL-Manual.md` | `notes/03-sql-and-databases/sql-manual.md` | Consistent lowercase kebab-case. |
| `SQL_FOR_ENGINEERING.md` | `notes/03-sql-and-databases/sql-for-data-engineering-reference.md` | Identify the comprehensive reference role. |
| `MySQLQueryExecutionPlans.md` | `notes/03-sql-and-databases/mysql-query-execution-plans.md` | Consistent name and database-specific placement. |
| `Day3-WeekOneDayThreeClass.md` | `notes/03-sql-and-databases/postgresql-classroom-quickstart.md` | Its content is primarily a short PostgreSQL exercise; governance is covered elsewhere. |
| `PYTHON/chapter_1.md` | `notes/04-python/01-python-basics.md` | Ordered descriptive chapter. |
| `PYTHON/chapter_2.md` | `notes/04-python/02-variables-expressions-and-statements.md` | Ordered descriptive chapter. |
| `PYTHON/chapter_3.md` | `notes/04-python/03-python-functions.md` | Ordered descriptive chapter. |
| `Data Engineering Python Faker Demo.ipynb` | `notes/04-python/examples/faker-demo.ipynb` | Keep the notebook beside Python notes. |
| `scrapping.md` | `notes/04-python/web-scraping-with-beautiful-soup.md` | Correct spelling and describe the library. |
| `notes/DEWeek3Day1.md` | `notes/05-etl-and-data-pipelines/01-etl-elt-workflows.md` | Ordered theory note. |
| `notes/DEWeek3Day3.md` | `notes/05-etl-and-data-pipelines/02-etl-best-practices.md` | Ordered deeper theory note. |
| `ETL/ETL-ELT.md` | `notes/05-etl-and-data-pipelines/etl-elt-reference.md` | Retain complementary reference material. |
| `DATA-PROCESSING.md` | `notes/05-etl-and-data-pipelines/batch-and-stream-processing.md` | Descriptive topic name. |
| `notes/DEWeek2Day4.md` | `notes/06-data-modeling-and-warehousing/01-normalization-and-dimensional-modeling.md` | Preserve course lesson and clarify scope. |
| `CoreConceptsDataModeling.md` | `notes/06-data-modeling-and-warehousing/data-modeling-core-concepts.md` | Retain complementary fundamentals. |
| `DATA-MODELING.md` | `notes/06-data-modeling-and-warehousing/data-modeling-introduction.md` | Retain its distinct introductory examples. |
| `data_warehouse.md` | `notes/06-data-modeling-and-warehousing/data-warehousing.md` | Consistent naming. |
| `Data_Warehouse_Naming_Conventions.md` | `notes/06-data-modeling-and-warehousing/data-warehouse-naming-conventions.md` | Consistent naming. |
| `notes/DEWeek4Day1.md` | `notes/07-apache-airflow/01-apache-airflow-introduction.md` | Current ordered introduction. |
| `Apache Airflow 101 Guide.md` | `notes/07-apache-airflow/02-apache-airflow-setup.md` | Setup-focused descriptive name. |
| `Airflow-Part 1.md` | `notes/07-apache-airflow/03-writing-airflow-dags.md` | DAG-focused descriptive name. |
| `Apache Airflow Operators Guide.md` | `notes/07-apache-airflow/04-airflow-operators-reference.md` | Reference-focused descriptive name. |
| `introduction-to-Kafka.md` | `notes/08-apache-kafka/01-apache-kafka-introduction.md` | Ordered introduction. |
| `Apache Kafka 101: Apache Kafka for Data Engineering Guide.md` | `notes/08-apache-kafka/02-kafka-cli-cheat-sheet.md` | Describe command-heavy content. |
| `Apache Kafka 102: Apache Kafka for Data Engineering Guide.md` | `notes/08-apache-kafka/03-kafka-concepts-and-commands.md` | Preserve expanded complementary material. |
| `Data Engineer Apache Kafka Producers.md` | `notes/08-apache-kafka/04-python-kafka-producer-example.md` | Identify the code example. |
| `Apache Spark.md` | `notes/09-spark-and-big-data/apache-spark-and-pyspark.md` | Group big-data learning. |
| `data_lake.md` | `notes/10-data-lakes/data-lakes.md` | Consistent naming. |
| `minio_setup.md` | `notes/10-data-lakes/minio-setup.md` | Keep local object-storage practice beside lake concepts. |
| `notes/DEWeek1Day3.md` | `notes/11-data-governance-and-security/01-data-governance-security-and-access-control.md` | Current ordered governance lesson. |
| `Data_Governance_Frameworks.md` | `notes/11-data-governance-and-security/02-data-governance-frameworks.md` | Preserve deeper framework coverage. |
| `GDPR & HIPAA Compliance Guide.md` | `notes/11-data-governance-and-security/03-gdpr-and-hipaa.md` | Consistent descriptive name. |
| `Data Governance Frameworks and Data Security Principles.md` | `notes/11-data-governance-and-security/incomplete-governance-placeholder.md` | Keep and explicitly flag the one-byte incomplete file. |
| `Change Data Capture.md` | `notes/12-change-data-capture/change-data-capture.md` | Ordered topic placement. |
| `SQL-PRACTICE-QUESTIONS.md` | `exercises/sql/sql-practice-questions.md` | Separate focused practice from notes. |
| `Tuesday-Kafka-Lab.md` | `exercises/kafka/producer-consumer-lab.md` | Small single-topic lab, not a multi-tool project. |
| `Assignments/Cohort6 Week 1.md` | `exercises/assignments/linux-server-administration.md` | Descriptive assignment placement. |
| `WeekOneProject.md` | `projects/beginner/cloud-storage-and-postgresql/README.md` | Give the project its own directory and standard guide. |
| `AivenProjectVersionWeekOneProject.md` | `projects/beginner/aiven-postgresql/README.md` | Keep the materially different managed-service variant. |
| `notes/DEWeek3Day2.md` | `projects/intermediate/simple-csv-etl/README.md` | Substantial runnable lab is better treated as a project. |
| `notes/DEWeek3Day4.md` | `projects/intermediate/batch-file-etl/README.md` | Substantial multi-file, database-backed lab is a project. |
| `CH02-2025-DE-Capstone-Project.md` | `projects/capstone/kenya-food-prices/README.md` | Portfolio-scale capstone directory. |
| `Cohort6CapstoneProjet.md` | `projects/capstone/crypto-market-pipeline/README.md` | Correct spelling through a descriptive capstone directory. |
| `samplejson.json` | `datasets/sample-kafka-event.json` | Standalone Kafka-like sample has no current single owner; document globally. |

## Duplicate and overlapping material

Nothing was consolidated or deleted. Cloud, data-modeling, governance, SQL, Airflow, and Kafka files overlap, but inspection showed differences in depth, database/tool version, examples, or classroom purpose. They remain complementary chapters with clearer names. The two beginner cloud/database projects also remain separate because one is provider-neutral and one specifically uses Aiven.

## Content requiring instructor review

1. **Conflicting schedules:** the former root README described 17 weeks/four months, while `curriculum-6-to-7-months.md` describes 6 months extendable to 7 and begins its visible week-by-week section at Week 2. Both are preserved.
2. **Empty file:** `incomplete-governance-placeholder.md` contains only a newline. It is retained and flagged rather than populated or deleted.
3. **Historical credentials:** the Linux assignment published a root server address and password. They were replaced with instructor-provided placeholders; the original credentials must be considered compromised and rotated.
4. **Version-sensitive setup:** Kafka lab commands pin Kafka 3.6.1 and ZooKeeper; Airflow notes cover both older multi-component commands and Airflow 3 syntax. Instructors should validate the intended supported versions.
5. **Aiven wording:** the historical project describes managed object-storage behavior that may depend on Aiven's current products and plan. Students are told to verify it before spending money.
6. **Curriculum gaps:** the retained roadmap names Power BI and machine-learning weeks, but dedicated repository notes were not found.
7. **External data/APIs:** capstone sources, terms, schemas, regional availability, and rate limits can change and must be checked at project start.

## Navigation and link repairs

The root README now links to every current course area; directory indexes connect notes, exercises, and projects; project prerequisites link back to notes. Relative links changed by the migration were updated in the new navigation. An automated local Markdown target check was run after the move (see commit/PR test record); external URLs were not treated as repository paths.

## Preservation statement

All 49 pre-existing tracked educational/data files remain represented after the migration. No useful content was intentionally removed. Additions are navigation, safety context, and standardized project guidance; the only replacement within existing material was removal of exposed server credentials.
