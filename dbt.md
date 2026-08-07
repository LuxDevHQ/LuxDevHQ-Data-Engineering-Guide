# Introduction to Data Build Tool (dbt)

## What is dbt?

**dbt (Data Build Tool)** is an open-source data transformation framework that enables data teams to transform raw data into clean, reliable, and analytics-ready datasets using SQL and Jinja.

Unlike data ingestion tools, dbt **does not move data** between systems. Instead, it transforms data that already exists in your data warehouse or database as part of the **Transform** step in an ELT pipeline.

With dbt, developers write SQL `SELECT` statements that define transformations, while dbt handles dependency management, execution order, testing, documentation, and lineage.



## Why Use dbt?

As data projects grow, managing SQL transformations manually becomes increasingly difficult.

Common challenges include:

- Knowing which transformations should run first
- Avoiding duplicated SQL logic
- Maintaining consistent business rules
- Testing data quality
- Documenting transformations
- Collaborating across teams

dbt addresses these challenges by bringing software engineering practices into analytics engineering.

### Key Benefits

- **Modular Development** – Break transformations into reusable SQL models.
- **Dependency Management** – Automatically builds models in the correct order.
- **Data Quality Testing** – Validate data with built-in tests.
- **Documentation** – Generate interactive project documentation.
- **Version Control** – Integrates seamlessly with Git.
- **Data Lineage** – Visualize relationships between models.



# dbt Core vs dbt Cloud

## dbt Core

dbt Core is the open-source command-line version of dbt.

It allows developers to:

- Build transformation pipelines
- Run SQL models
- Execute tests
- Generate documentation
- Integrate with CI/CD pipelines

Common commands include:

```bash
dbt run
dbt test
dbt docs generate
dbt docs serve
```



## dbt Cloud

dbt Cloud is the managed version of dbt.

In addition to everything available in dbt Core, it provides:

- Web-based IDE
- Job scheduling
- Team collaboration
- Deployment management
- Hosted documentation
- Git integration

Internally, dbt Cloud still uses dbt Core to execute transformations.



# Key Concepts in dbt

## Models

A **model** is a SQL file that defines a transformation.

Each model typically creates either a table or a view inside your database.

Example:

```sql
SELECT
    customer_id,
    SUM(total_amount) AS total_sales
FROM orders
GROUP BY customer_id
```

Models should perform a single responsibility and remain easy to understand.


## Sources

A **source** represents raw data that already exists in your database.

Instead of hardcoding table names, dbt recommends defining sources in YAML.

Example:

```yaml
sources:
  - name: raw
    schema: raw

    tables:
      - name: orders
```

You can then reference the source using:

```sql
SELECT *
FROM {{ source('raw', 'orders') }}
```

Using sources improves documentation, testing, and lineage.


## References (`ref()`)

One of dbt's most important features is `ref()`.

Instead of writing:

```sql
SELECT *
FROM customer_orders
```

Use:

```sql
SELECT *
FROM {{ ref('customer_orders') }}
```

dbt automatically:

- Determines execution order
- Tracks dependencies
- Resolves object names
- Builds lineage graphs


## Materializations

Materializations determine how dbt stores the result of a model.

### Table

Creates a physical table.

```sql
{{ config(materialized='table') }}
```

### View

Creates a database view.

```sql
{{ config(materialized='view') }}
```

### Incremental

Processes only new or changed data instead of rebuilding the entire table.

```sql
{{ config(materialized='incremental') }}
```

Choosing the appropriate materialization helps balance performance and storage requirements.


## Tests

dbt provides built-in testing for validating data quality.

Example:

```yaml
columns:
  - name: customer_id
    tests:
      - not_null
      - unique
```

Another example:

```yaml
columns:
  - name: order_status
    tests:
      - accepted_values:
          values:
            - Pending
            - Completed
            - Cancelled
```

Run tests using:

```bash
dbt test
```

Testing helps identify data quality issues before they affect downstream analytics.



## Documentation

dbt automatically generates project documentation from SQL models and YAML files.

Generate documentation using:

```bash
dbt docs generate
dbt docs serve
```

The generated documentation includes:

- Models
- Sources
- Column descriptions
- Tests
- Data lineage
- Dependencies

This makes it easier for teams to understand and maintain transformation pipelines.



## Jinja

dbt combines SQL with the **Jinja** templating language.

Jinja enables dynamic SQL while keeping code clean and reusable.

Example:

```sql
{{ config(materialized='table') }}
```

Another example:

```sql
SELECT *
FROM {{ ref('customers') }}
```

Jinja also supports:

- Variables
- Loops
- Conditional statements
- Macros

allowing developers to reduce repetitive SQL.


# Typical dbt Workflow

A typical transformation workflow looks like this:

```text
Raw Data
    │
    ▼
Source
    │
    ▼
Staging Models
    │
    ▼
Intermediate Models
    │
    ▼
Mart / Analytics Models
    │
    ▼
Dashboards & Reports
```

Each downstream model builds upon previous models using `ref()`, allowing dbt to automatically determine execution order.



# Useful dbt Commands

Validate project configuration:

```bash
dbt debug
```

Compile SQL without executing it:

```bash
dbt compile
```

Run all models:

```bash
dbt run
```

Run selected models:

```bash
dbt run --select staging
```

Execute tests:

```bash
dbt test
```

Generate documentation:

```bash
dbt docs generate
```

Serve documentation locally:

```bash
dbt docs serve
```



# Best Practices

When working with dbt, consider the following best practices:

- Build small, focused models.
- Keep staging models lightweight.
- Reuse models with `ref()` instead of duplicating SQL.
- Define raw data using `source()`.
- Add tests early.
- Document models and columns.
- Use Git for version control.
- Organize projects using layered architecture (staging, intermediate, marts).

