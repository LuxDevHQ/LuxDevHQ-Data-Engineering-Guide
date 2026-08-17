# Structured Query Language (SQL)  
**Course Manual – Version 1.2**  
© 2025

---

## Table of Contents
1. [Introduction](#introduction)  
2. [Basic Queries](#basic-queries)  
3. [Advanced Operators](#advanced-operators)  
4. [Expressions](#expressions)  
5. [Functions](#functions)  
6. [Multi-Table Queries](#multi-table-queries)  
7. [Queries Within Queries](#queries-within-queries)  
8. [Maintaining Tables](#maintaining-tables)  
9. [Defining Database Objects](#defining-database-objects)  
A. [Appendices](#appendices)

---

<a name="introduction"></a>
# 1. Introduction

### Course Objectives
By the end of the course you will be able to:
- **Query** relational databases
- **Maintain** relational databases
- **Define** relational databases

> “MAINTAIN – QUERY – RELATIONAL DATABASE – DEFINE”

---

### What is a Relational Database?
- **Tables** (unique names, ≤ 18 chars, start with letter).  
- **Columns** (unique within table).  
- **Rows** (identified by data values, not position).

---

### What is SQL?
- **Structured Query Language** – IBM (1974), ANSI (1986), ISO (1987).  
- **Six core statements**: `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `CREATE`, `DROP`.

| Category        | Statements          |
|-----------------|---------------------|
| **Query**       | `SELECT`            |
| **Maintenance** | `INSERT`, `UPDATE`, `DELETE` |
| **Definition**  | `CREATE`, `DROP`    |

---

<a name="basic-queries"></a>
# 2. Basic Queries

### 2.1 Selecting All Columns & Rows
```sql
SELECT *
FROM countries;
```

### 2.2 Selecting Specific Columns
```sql
SELECT title, job
FROM jobs;
```

### 2.3 Selecting Specific Rows
| Value Type | Format Example |
|------------|----------------|
| Numeric    | `123`, `-45.67` |
| String     | `'Canada'`      |
| Date       | `'1999-12-31'`  |

```sql
SELECT name, country
FROM persons
WHERE country = 'Canada';
```

### 2.4 Sorting Rows
```sql
SELECT country, area
FROM countries
ORDER BY area DESC;      -- largest first
```

Multiple columns:
```sql
ORDER BY language, pop DESC;
```

### 2.5 Eliminating Duplicate Rows
```sql
SELECT DISTINCT job
FROM persons;
```

---

<a name="advanced-operators"></a>
# 3. Advanced Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `LIKE`   | Pattern | `WHERE name LIKE 'Z%'` |
| `AND`    | Both true | `WHERE gnp < 3000 AND literacy < 40` |
| `BETWEEN`| Inclusive range | `WHERE pop BETWEEN 100000 AND 200000` |
| `OR`     | Either true | `WHERE country = 'USA' OR country = 'Canada'` |
| `IN`     | List match | `WHERE language IN ('English', 'French')` |
| `IS NULL`| Missing value | `WHERE gnp IS NULL` |
| `NOT`    | Negation | `WHERE NOT country = 'USA'` |
| `( )`    | Precedence | `WHERE (job='S' OR job='W') AND country='Italy'` |

---

<a name="expressions"></a>
# 4. Expressions

### 4.1 Arithmetic Expressions
```sql
SELECT country, pop/area AS density
FROM countries;
```

### 4.2 Expressions in WHERE / ORDER BY
```sql
SELECT *
FROM countries
WHERE pop/area > 1000
ORDER BY pop/area DESC;
```

### 4.3 Column Aliases (`AS`)
```sql
SELECT gnp*1000000/pop AS gpp
FROM countries
WHERE gpp > 20000
ORDER BY gpp DESC;
```

---

<a name="functions"></a>
# 5. Functions

### 5.1 Statistical Functions
| Function | Purpose |
|----------|---------|
| `COUNT(*)` | Rows |
| `COUNT(col)` | Non-NULL |
| `SUM(col)` | Total |
| `AVG(col)` | Average |
| `MIN(col)` / `MAX(col)` | Min / Max |

Grand totals:
```sql
SELECT AVG(pop) AS avg_pop
FROM countries
WHERE language = 'English';
```

### 5.2 Grouping
```sql
SELECT job, COUNT(*) AS total
FROM persons
GROUP BY job;
```

### 5.3 HAVING (post-group filter)
```sql
SELECT language, AVG(literacy) AS avg_lit
FROM countries
GROUP BY language
HAVING AVG(literacy) > 90;
```

---

<a name="multi-table-queries"></a>
# 6. Multi-Table Queries

### 6.1 Joins
```sql
SELECT p.name, j.title
FROM persons p
JOIN jobs j ON p.job = j.job;
```

### 6.2 Table Aliases
Short-hand:
```sql
SELECT c.country, a.budget
FROM countries c
JOIN armies a ON c.country = a.country;
```

### 6.3 Union
Combine results (distinct):
```sql
SELECT country FROM religions WHERE percent > 40
UNION
SELECT country FROM countries WHERE language = 'German';
```

---

<a name="queries-within-queries"></a>
# 7. Queries Within Queries (Subqueries)

| Type | Template | Example |
|------|----------|---------|
| **Single-valued** | `WHERE col = (SELECT …)` | `WHERE pop > (SELECT AVG(pop) FROM countries)` |
| **Multi-valued** | `WHERE col IN (SELECT …)` | `WHERE country IN (SELECT country FROM religions WHERE percent > 95)` |
| **Correlated** | Inner query references outer alias | `WHERE bdate = (SELECT MIN(bdate) FROM persons p2 WHERE p2.job = p1.job)` |

---

<a name="maintaining-tables"></a>
# 8. Maintaining Tables

| Action | Syntax | Example |
|--------|--------|---------|
| **Insert** | `INSERT INTO … VALUES …` | `INSERT INTO jobs(job,title) VALUES ('A','Author');` |
| **Update** | `UPDATE … SET … WHERE …` | `UPDATE persons SET job='E' WHERE person=500;` |
| **Delete** | `DELETE FROM … WHERE …` | `DELETE FROM persons WHERE person=500;` |
| **Transaction** | `COMMIT;` or `ROLLBACK;` | Undo or save all changes since last `COMMIT`. |

---

<a name="defining-database-objects"></a>
# 9. Defining Database Objects

### 9.1 Tables
```sql
CREATE TABLE theologians (
  name    CHAR(20) PRIMARY KEY,
  bdate   DATE,
  gender  CHAR(6) NOT NULL CHECK (gender IN ('Male','Female')),
  country CHAR(20) REFERENCES countries(country)
);
```

### 9.2 Indexes
```sql
CREATE INDEX idx_gender_job ON persons(gender, job);
DROP INDEX idx_gender_job;
```

### 9.3 Views
```sql
CREATE VIEW iv AS
SELECT * FROM persons WHERE country = 'Israel';

SELECT * FROM iv;
DROP VIEW iv;
```

---

<a name="appendices"></a>
# A. Appendices

### Exercise Database Schema
| Table   | Key Columns (sample) |
|---------|----------------------|
| **persons**   | person, name, bdate, gender, country, job |
| **countries** | country, pop, area, gnp, language, literacy |
| **armies**    | country, budget, troops, tanks, ships, planes |
| **jobs**      | job, title |
| **religions** | country, religion, percent |

> All monetary values in millions; population in people; area in sq mi; literacy in %.

### Answers to Selected Exercises
See the original PDF **pages 94-110** for the complete answer key.

---

## Syntax Summary Cheat-Sheet
```sql
SELECT [DISTINCT] columns|functions
FROM   table [alias] [, ...]
[WHERE  conditions]
[GROUP BY columns]
[HAVING aggregate_conditions]
[ORDER BY column|expr|position [ASC|DESC]];

INSERT INTO table(col1,...) VALUES(val1,...);
UPDATE table SET col=val [, ...] WHERE ...;
DELETE FROM table WHERE ...;
CREATE TABLE tbl (...);
DROP TABLE tbl;
COMMIT;
ROLLBACK;
```
