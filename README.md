## **LuxDevHQ Data Engineering Course Outline**

This comprehensive course spans **4 months plus 1 extra week** (17 weeks total) and equips learners with expertise in Python, SQL, Azure, AWS, Apache Airflow, Kafka, Spark, and more.
- **Learning Days**: Monday to Thursday (theory and practice).
- **Friday**: Job shadowing or peer projects.
- **Saturday**: Hands-on lab sessions and project-based learning.

---
## Table of Contents

1. [Week 1](#week-1-Onboarding-and-Environment-Setup)
2. [Week 2](#week-2-Power-BI-for-Data-Visualization)
3. [Week 3](#week-3-SQL-Essentials-for-Data-Engineering)
4. [Week 4](#week-4-Introduction-to-Data-Pipelines)
5. [Week 5](#week-5-Introduction-to-Apache-Airflow)
6. [Week 6](#week-6-Data-Warehousing-and-Data-Lakes)
7. [Week 7](#week-7-Data-Governance-and-Security)
8. [Week 8](#week-8-Real-Time-Data-Processing-with-Kafka)
9. [Week 9](#week-9-Batch-vs-Stream-Processing)
10. [Week 10](#week-10-Machine-Learning-Integration-in-Data-Pipelines)
11. [Week 11](#week-11-Spark-and-PySpark-for-Big-Data)
12. [Week 12](#week-12-Advanced-Apache-Airflow-Techniques)
13. [Week 13](#week-13-Data-Lakes-and-Delta-Lake)
14. [Week 14](#week-14-Batch-Data-Pipeline-Development)
15. [Week 15](#week-15-Real-Time-Data-Pipeline-Development)
16. [Week 16](#week-16-Final-Project-Integration)
17. [Week 17](#week-17-capstone-project-presentation)



   ---

### **Month 1: Foundations of Data Engineering**

#### Week 1: Onboarding and Environment Setup
- **Monday**:  
  - Onboarding, course overview, career pathways, tools introduction.
  - Notes: [Week 1, Day 1](notes/DEWeek1Day1.md)
- **Tuesday**:  
  - Introduction to cloud computing (Azure, GCP, and AWS). 
  - Notes: [Week 1, Day 2](notes/DEWeek1Day2.md) 
- **Wednesday**:  
  - Data governance, security, compliance, and access control.
  - Notes: [Week 1, Day 3](notes/DEWeek1Day3.md)  
- **Thursday**:  
  - Introduction to SQL for data engineering and PostgreSQL setup. 
  - Notes: [Week 1, Day 4](notes/DEWeek1Day4.md) 
- **Friday**:  
  - **Peer Project**: Environment setup challenges.  
- **Saturday (Lab)**:  
  - **Mini Project**: Build a basic pipeline with PostgreSQL and Azure Blob Storage.  

---

#### Week 2: Power BI for Data Visualization

##### **Week 2 - Day 1: Introduction to Power BI and Power Query Editor (Data Transformation)**

**Overview & Setup**
- What is Power BI? (Comparison with Excel & Tableau)
- Power BI Components: Desktop, Service, Mobile
- Installing Power BI Desktop
- Power BI Desktop Interface Overview

**Getting Data**
- Importing Data from Excel, CSV, Web
- Understanding Data Types and Field Formatting

**Power Query Editor**
- Opening Power Query Editor
- Removing Rows, Columns, and Duplicates
- Changing Data Types, Renaming Columns
- Splitting & Merging Columns
- Using "Replace Values"
- Applied Steps, Reordering, and Removing Steps

**Combining Queries**
- **Merge Queries** (SQL-style joins)
- **Append Queries** (Union of datasets)

##### **Week 2 - Day 2: DAX Basics – Measures, Calculated Columns & Aggregations**

**Introduction to DAX**
- What is DAX (Data Analysis Expressions)?
- Syntax Rules: =, (), [], Table & Column references

**Calculated Columns vs Measures**
- When to use each
- Creating new fields using Calculated Columns
- Building Measures for aggregations

**Common DAX Functions**
- **Aggregation**: SUM(), AVERAGE(), COUNT(), COUNTROWS(), DISTINCTCOUNT()
- **Logical**: IF(), SWITCH(), AND(), OR()
- **Date**: TODAY(), NOW(), YEAR(), MONTH(), DATEDIFF()
- **Text**: CONCATENATE(), LEFT(), RIGHT(), LEN()

**Practical Use Cases**
- Total Revenue, Profit Margin, % Growth
- IF(Sales > 50000, "High", "Low")
- Number of Orders per Customer

##### **Week 2 - Day 3: Data Modeling, Relationships & Joins**

**Data Modeling Concepts**
- What is a Data Model?
- Star Schema vs Snowflake Schema
- Importance of Fact and Dimension tables

**Relationships in Power BI**
- One-to-Many and Many-to-One relationships
- Creating & Managing Relationships in Model View
- Active vs Inactive Relationships

**Data Joins**
- Relationship-based vs Merge Query joins
- Cardinality (One-to-One, One-to-Many)
- Cross filter direction

**Modeling Best Practices**
- Hiding unnecessary columns
- Using Lookup Tables
- Creating Role-Playing Dimensions (e.g. Order Date vs Delivery Date)

##### **Week 2 - Day 4: Visualizations, Charts & Dashboards**

**Basic Visuals**
- Bar Chart, Column Chart, Line Chart
- Pie & Donut Charts
- Card, KPI
- Table & Matrix

**Advanced Visuals**
- Tree Map, Funnel, Gauge
- Maps: Filled Map, Shape Map, ArcGIS Map
- Custom Visuals (via AppSource)

**Interactivity**
- Visual-Level, Page-Level, Report-Level Filters
- Slicers (Text, Date, Dropdowns)
- Drill-down & Drill-through
- Tooltips, Bookmarks, Buttons

**Dashboard Building**
- Designing a complete report page
- Adding Titles, Backgrounds, Logos, Images
- Aligning and Formatting Visuals
- Creating Navigation Buttons

**Publishing & Sharing**
- Publishing to Power BI Service
- Sharing Reports & Dashboards
- Setting Scheduled Refresh

---

#### Week 3: SQL Essentials for Data Engineering
- **Monday**:  
  - Core SQL concepts (`SELECT`, `WHERE`, `JOIN`, `GROUP BY`).
  - Notes: [Week 2, Day 1](notes/DEWeek2Day1.md)  
- **Tuesday**:  
  - Advanced SQL techniques: recursive queries, window functions, Views, Stored Procedures, Subqueries and CTEs.  
  - Notes: [Week 2, Day 2](notes/DEWeek2Day2.md)
- **Wednesday**:  
  - Query optimization and execution plans.  
  - Notes: [Week 2, Day 3](notes/DEWeek2Day3.md)
- **Thursday**:  
  - Data modeling: normalization, denormalization, and star schemas. 
  - Notes: [Week 2 Day 4](notes/DEWeek2Day4.md) 
- **Friday**:  
  - **Job Shadowing**: Observe senior engineers writing and optimizing SQL queries.  
- **Saturday (Lab)**:  
  - **Mini Project**: Create a star schema and analyze data using SQL.  

---

#### Week 4: Introduction to Data Pipelines
- **Monday**:  
  - Theory: Introduction to ETL/ELT workflows.
  - Notes: [Week 3 Day 1](notes/DEWeek3Day1.md)  
- **Tuesday**:  
  - Lab: Create a simple Python-based ETL pipeline for CSV data. 
  - Notes: [Week 3 Day 2](notes/DEWeek3Day2.md) 
- **Wednesday**:  
  - Theory: Extract, transform, load (ETL) concepts and best practices.  
  - Notes: [Week 3 Day 3](notes/DEWeek3Day3.md)
- **Thursday**:  
  - Lab: Build a Python ETL pipeline for batch data processing.
  - Notes: [Week 3 Day 4](notes/DEWeek3Day4.md)  
- **Friday**:  
  - **Peer Project**: Collaborate to design a basic ETL workflow.  
- **Saturday (Lab)**:  
  - **Mini Project**: Develop a simple ETL pipeline to process sales data.  

---

### **Month 2: Intermediate Tools and Concepts**

#### Week 5: Introduction to Apache Airflow
- **Monday**:  
  - Theory: Introduction to Apache Airflow, DAGs, and scheduling. 
  - Notes : [Week 4 Day 1](notes/DEWeek4Day1.md) 
- **Tuesday**:  
  - Lab: Set up Apache Airflow and create a basic DAG.  
- **Wednesday**:  
  - Theory: DAG best practices and scheduling in Airflow.  
- **Thursday**:  
  - Lab: Integrate Airflow with PostgreSQL and Azure Blob Storage.  
- **Friday**:  
  - **Job Shadowing**: Observe real-world Airflow pipelines.  
- **Saturday (Lab)**:  
  - **Mini Project**: Automate an ETL pipeline with Airflow for batch data processing.  

---

#### Week 6: Data Warehousing and Data Lakes
- **Monday**:  
  - Theory: Introduction to data warehousing (OLAP vs. OLTP, partitioning, clustering).  
- **Tuesday**:  
  - Lab: Work with Amazon Redshift and Snowflake for data warehousing.  
- **Wednesday**:  
  - Theory: Data lakes and Lakehouse architecture.  
- **Thursday**:  
  - Lab: Set up Delta Lake for raw and curated data.  
- **Friday**:  
  - **Peer Project**: Implement a data warehouse model and data lake for sales data.  
- **Saturday (Lab)**:  
  - **Mini Project**: Design and implement a basic Lakehouse architecture.  

---

#### Week 7: Data Governance and Security
- **Monday**:  
  - Theory: Data governance frameworks and data security principles.  
- **Tuesday**:  
  - Lab: Use AWS Lake Formation for access control and security enforcement.  
- **Wednesday**:  
  - Theory: Managing sensitive data and compliance (GDPR, HIPAA).  
- **Thursday**:  
  - Lab: Implement security policies in S3 and Azure Blob Storage.  
- **Friday**:  
  - **Job Shadowing**: Observe senior engineers applying governance policies.  
- **Saturday (Lab)**:  
  - **Mini Project**: Secure data in the cloud using AWS and Azure.  

---

#### Week 8: Real-Time Data Processing with Kafka
- **Monday**:  
  - Theory: - [Introduction to Apache Kafka for real-time data streaming](/introduction-to-Kafka.md)
- **Tuesday**:  
  - Lab: [Set up a Kafka producer and consumer.](/Tuesday-Kafka-Lab.md)
- **Wednesday**:  
  - Theory: Kafka topics, partitions, and message brokers.  
- **Thursday**:  
  - Lab: Integrate Kafka with PostgreSQL for real-time updates.  
- **Friday**:  
  - **Peer Project**: Build a real-time Kafka pipeline for transactional data.  
- **Saturday (Lab)**:  
  - **Mini Project**: Create a pipeline to stream e-commerce data with Kafka.
    
[Apache Kafka 101](./Apache%20Kafka%20101%3A%20Apache%20Kafka%20for%20Data%20Engineering%20Guide.md)

[Apache Kafka 102](/Apache%20Kafka%20102%3A%20Apache%20Kafka%20for%20Data%20Engineering%20Guide.md)



---

### **Month 3: Advanced Data Engineering**

#### Week 9: Batch vs. Stream Processing
- **Monday**:  
  - Theory: Introduction to batch vs. stream processing.  
- **Tuesday**:  
  - Lab: Batch processing with PySpark.  
- **Wednesday**:  
  - Theory: Combining batch and stream processing workflows.  
- **Thursday**:  
  - Lab: Real-time processing with Apache Flink and Spark Streaming.  
- **Friday**:  
  - **Job Shadowing**: Observe a real-time processing pipeline.  
- **Saturday (Lab)**:  
  - **Mini Project**: Build a hybrid pipeline combining batch and real-time processing.  

---

#### Week 10: Machine Learning Integration in Data Pipelines
- **Monday**:  
  - Theory: Overview of ML workflows in data engineering.  
- **Tuesday**:  
  - Lab: Preprocess data for machine learning using Pandas and PySpark.  
- **Wednesday**:  
  - Theory: Feature engineering and automated feature extraction.  
- **Thursday**:  
  - Lab: Automate feature extraction using Apache Airflow.  
- **Friday**:  
  - **Peer Project**: Build a simple pipeline that integrates ML models.  
- **Saturday (Lab)**:  
  - **Mini Project**: Build an ML-powered recommendation system in a pipeline.  

---

#### Week 11: Spark and PySpark for Big Data
- **Monday**:  
  - Theory: Introduction to Apache Spark for big data processing.  
- **Tuesday**:  
  - Lab: Set up Spark and PySpark for data analysis.  
- **Wednesday**:  
  - Theory: Spark RDDs, DataFrames, Performance Optimization and SQL.  
- **Thursday**:  
  - Lab: Analyze large datasets using Spark SQL.  
- **Friday**:  
  - **Peer Project**: Build a PySpark pipeline for large-scale data processing.  
- **Saturday (Lab)**:  
  - **Mini Project**: Analyze big data sets with Spark and PySpark.  

---

#### Week 12: Advanced Apache Airflow Techniques
- **Monday**:  
  - Theory: Advanced Airflow features (XCom, task dependencies).  
- **Tuesday**:  
  - Lab: Implement dynamic DAGs and task dependencies in Airflow.  
- **Wednesday**:  
  - Theory: Airflow scheduling, monitoring, and error handling.  
- **Thursday**:  
  - Lab: Create complex DAGs for multi-step ETL pipelines.  
- **Friday**:  
  - **Job Shadowing**: Observe advanced Airflow pipeline implementations.  
- **Saturday (Lab)**:  
  - **Mini Project**: Design an advanced Airflow DAG for complex data workflows.  

---

### **Month 4: Capstone Projects**

#### Week 13: Data Lakes and Delta Lake
- **Monday**:  
  - Theory: Data lakes, Lakehouses, and Delta Lake architecture.  
- **Tuesday**:  
  - Lab: Set up Delta Lake on AWS for data storage and management.  
- **Wednesday**:  
  - Theory: Managing schema evolution in Delta Lake.  
- **Thursday**:  
  - Lab: Implement batch and real-time data loading to Delta Lake.  
- **Friday**:  
  - **Peer Project**: Design a Lakehouse architecture for an e-commerce platform.  
- **Saturday (Lab)**:  
  - **Mini Project**: Implement a scalable Delta Lake architecture.  

---

#### Week 14: Batch Data Pipeline Development
- **Monday to Thursday**:  
  - **Design and Implementation**:  
    - Build an end-to-end batch data pipeline for e-commerce sales analytics.  
  - **Tools**: PySpark, SQL, PostgreSQL, Airflow, S3.  
- **Friday**:  
  - **Peer Review**: Present progress and receive feedback.  
- **Saturday (Lab)**:  
  - **Project Milestone**: Finalize and present batch pipeline results.  

---

#### Week 15: Real-Time Data Pipeline Development
- **Monday to Thursday**:  
  - **Design and Implementation**:  
    - Build an end-to-end real-time data pipeline for IoT sensor monitoring.  
  - **Tools**: Kafka, Spark Streaming, Flink, S3.  
- **Friday**:  
  - **Peer Review**: Present progress and receive feedback.  
- **Saturday (Lab)**:  
  - **Project Milestone**: Finalize and present real-time pipeline results.  

---

#### Week 16: Final Project Integration
- **Monday to Thursday**:  
  - **Design and Implementation**:  
    - Integrate both batch and real-time pipelines for a comprehensive end-to-end solution.  
  - **Tools**: Kafka, PySpark, Airflow, Delta Lake, PostgreSQL, and S3.  
- **Friday**:  
  - **Job Shadowing**: Observe senior engineers integrating complex pipelines.  
- **Saturday (Lab)**:  
  - **Project Milestone**: Showcase integrated solution for review.  

---

### **Extra Week**

#### Week 17: Capstone Project Presentation
- **Monday to Thursday**:  
  - Final Presentation Preparation:  
    - Polish, test, and document the final project.  
- **Friday**:  
  - **Peer Review**: Present final projects to peers and receive feedback.  
- **Saturday (Lab)**:  
  - **Capstone Presentation**: Showcase completed capstone projects to industry professionals and instructors.