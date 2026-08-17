## Data Warehouse Naming Conventions

Data warehouse naming standards ensure consistency, clarity, and manageability within your data environment. These standards establish guidelines for naming various components of your data warehouse, including both the structural elements that organize your data mart and the individual data records and attributes contained within it.

This document proposes a set of base conventions to be used as a template for data warehousing naming conventions, thus ensuring standards are enforced from the outset of a data engineering project.



## Table of Contents
- [General Principles](#general-principles)
- [Table Naming Conventions](#table-naming-conventions)
  - [Bronze Rules](#bronze-rules)
  - [Silver Rules](#silver-rules)
  - [Gold Rules](#gold-rules)
- [Column Naming Conventions](#column-naming-conventions)
  - [Surrogate Keys](#surrogate-keys)
  - [Technical Columns](#technical-columns)
- [Stored Procedure](#stored-procedure)


## General Principles

- **Naming Conventions**: Use `snake_case`, with lowercase letters and underscores (`_`) to separate words.
- **Language**: Use English for all names.
- **Avoid Reserved Words**: Do not use SQL reserved words as object names. (eg; `Order`, `date`, `select`, `table`)
- **Be Descriptive**: Names should be self-explanatory, avoiding abbreviations unless they are widely understood (eg `id`, `qty`, `amt`).
- **Consistency**: Apply the same naming pattern uniformly across all layers and schemas.

## Table Naming Conventions
The data warehouse is organized into three layers — Bronze, Silver, and Gold, following the Medallion Architecture. Each layer has distinct naming rules that reflect its purpose.

 ```
Source Systems  > Bronze (Raw)  >  Silver (Cleansed)  >  Gold (Business-Ready)
     CRM        >  crm_customer  >  crm_customer        >  dim_customers
     ERP        >  erp_orders    >  erp_orders          >  fact_sales
```

### Bronze Rules
The Bronze layer stores raw, unprocessed data ingested directly from source systems. Table names must mirror the source system and entity exactly; no renaming, no transformation.

- All names must start with the source system name, and table names must match their original names without renaming.
- Format: `<sourcesystem>_<entity>`
  - `<sourcesystem>`: Name of the source system (e.g., crm, erp).
  - `<entity>`: Exact table name from the source system.
    

**Example**:
```
-- Source System: CRM
-- Original Source Table Name: "Customer_Info"

CREATE TABLE bronze.crm_customer_info (
    -- Original columns kept exactly as they appear in source
    
    id INT,             
    firstname VARCHAR,        
    lastname VARCHAR,
    created at TIMESTAMP      
  
);
```

### Silver Rules
The Silver layer cleanses, standardises, and lightly transforms data from Bronze. Table names still reference the originating source system and entity, but column names and data types are normalized according to enterprise standards.

- All names must start with the source system name, and table names must match their original names without renaming.
- Format: `<sourcesystem>_<entity>`
  - `<sourcesystem>`: Name of the source system (e.g., crm, erp).
  - `<entity>`: Exact table name from the source system.

**Example**:
```
-- SOURCE (Bronze): bronze.crm_customer_info

CREATE TABLE silver.dim_customer AS
SELECT 
    id AS customer_id,                         
    firstname AS first_name,
    lastname AS last_name,
     created_at AS created_at
FROM bronze.crm_customer_info;
              
```

### Gold Rules

The Gold layer contains business ready, aggregated, and modelled data following the Kimball dimensional modelling approach. Tables are named using a category prefix that describes their role in the dimensional model, followed by a meaningful, business aligned entity name.

- All names must use meaningful, business-aligned names for tables, starting with the category prefix.
- Format: `<category>_<entity>`
  - `<category>`: Describes the role of the table, such as `dim` (dimension) or `fact` (fact table).
  - `<entity>`: Descriptive name of the table, aligned with the business domain (e.g., customers, products, sales).

**Example - Dimension Table**:

In Kimball modelling, dimension tables carry descriptive attributes about business entities. Each row is uniquely identified by a surrogate key (not the natural/business key from the source system).

```
-- Gold Dimension: dim_customers
-- Joins cleansed CRM data; assigns surrogate key for warehouse integrity.
-- Kimball Type 1 SCD: overwrites existing records on change (no history kept).

CREATE TABLE gold.dim_customers AS
SELECT
    ROW_NUMBER() OVER (ORDER BY c.customer_id)  AS customer_key,   -- Surrogate key
    c.customer_id AS customer_id,                                -- Natural/business key
    c.first_name,
    c.last_name,
    c.created_at
FROM silver.crm_customer_info c
WHERE c.customer_id IS NOT NULL;
```

**Example - Fact Table**:

Fact tables record measurable business events eg sales. They store numeric measures and link to dimension tables via their surrogate keys.
```
-- Gold Fact: fact_sales
-- Joins Silver crm order data with Gold dimension surrogate keys.


CREATE TABLE gold.fact_sales AS
SELECT
    ROW_NUMBER() OVER (ORDER BY o.customer_id)  AS sales_key, --surrogate key
    dc.customer_key,
    o.customer_id,
    o.total_amount,
    o.quantity_ordered
FROM silver.crm_customer_info o
LEFT JOIN gold.dim_customers dc ON o.customer_id = dc.customer_id;
```

### Glossary of Category Patterns

| Pattern  | Meaning             | Example(s)                  |
|----------|---------------------|-----------------------------|
| `dim_`   | Dimension table      | `dim_customer`, `dim_product` |
| `fact_`  | Fact table           | `fact_sales` , `fact_orders`               |
| `agg_`   | Aggregated table     | `agg_customers`, `agg_sales_monthly` |

## Column Naming Conventions

### Surrogate Keys
In the Kimball dimensional model, surrogate keys are system generated integer identifiers that uniquely identify each row in a dimension table. They are independent of the source system natural keys, which ensures stability even when source data changes.

- All primary keys in dimension tables must use the suffix `_key`.
- Format: `<table_name>_key`
  - `<table_name>`: Refers to the name of the table or entity the key belongs to.
  - `_key`: A suffix indicating that this column is a surrogate key.

**Example - Surrogate key generation in a dimension table**:
```
CREATE TABLE gold.dim_customers AS
SELECT
    ROW_NUMBER() OVER (ORDER BY customer_id)  AS customer_key, -- surrogate Key
    customer_id,                                             -- natural key from source
    first_name,
    last_name,
    created_at
FROM silver.crm_customer_info
WHERE customer_id IS NOT NULL;
```

### Technical Columns
Technical columns are system generated metadata columns used for auditing, lineage tracking, and pipeline debugging. They must be clearly distinguishable from business columns using the dwh_ prefix.

- All technical columns must start with the prefix `dwh_`, followed by a descriptive name indicating the column's purpose.
- Format: `dwh_<column_name>`
  - `dwh`: Prefix exclusively for system-generated metadata.
  - `<column_name>`: Descriptive name indicating the column's purpose.

**Example**:
- `dwh_load_date` → System-generated column used to store the date when the record was loaded.
- `dwh_update_date` → Date when the record was last updated.
- `dwh_source_system` → Name of the source system the record originated from.
- `dwh_batch_id` → Identifier for the pipeline batch run that loaded the record.

**Example -  Technical columns on a Bronze table**:
```
CREATE TABLE bronze.crm_customer_info (
    id              INT,
    firstname       VARCHAR(100),
    lastname        VARCHAR(100),
    email           VARCHAR(255),

    -- Technical columns
    dwh_load_date       TIMESTAMP   DEFAULT CURRENT_TIMESTAMP,  -- When this record was ingested
    dwh_source_system   VARCHAR(50) DEFAULT 'crm',              -- Origin system identifier
    dwh_batch_id        VARCHAR(100)                            -- Pipeline run reference
);
```

## Stored Procedure

Stored procedures responsible for loading data into each warehouse layer follow a consistent naming pattern to make pipeline orchestration intuitive and auditable.

- All stored procedures used for loading data must follow the naming pattern:
  - `load_<layer>`
  - `<layer>`: Represents the layer being loaded, such as `bronze`, `silver`, or `gold`.

**Examples**:
- `load_bronze` → Stored procedure for loading data into the Bronze layer.
- `load_silver` → Stored procedure for loading data into the Silver layer.
- `load_gold` → Stored procedure for loading data into the Gold layer.

**Example - Bronze Load Procedure**
```  
CREATE OR REPLACE PROCEDURE load_bronze()
AS $$
BEGIN
    TRUNCATE TABLE bronze.crm_customer_info;

    INSERT INTO bronze.crm_customer_info (id, firstname, lastname, created_at)
    SELECT id, firstname, lastname, created_at
    FROM staging.crm_customer_info_raw;

END;
$$;
```
