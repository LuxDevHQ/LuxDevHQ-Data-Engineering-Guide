# Apache Spark & PySpark: Learning Guide

## Understanding Apache Spark

Apache Spark is an open-source distributed computing framework designed for processing large datasets across clusters of computers. Spark maintains data in memory between operations, which significantly reduces the time spent reading from and writing to disk storage. This in-memory processing capability makes Spark substantially faster than traditional disk-based processing systems.

The framework provides a unified computing engine that handles multiple types of data processing workloads including batch processing (handling large chunks of data at once), stream processing (handling data as it arrives continuously), machine learning, and graph computation. Spark can scale from running on your laptop to running across thousands of computers in a data center.

## Core Spark Architecture

Understanding Spark's architecture helps you comprehend how your data processing actually happens behind the scenes. The Spark runtime consists of several key components that work together to execute distributed computations.

**Driver Program**: The main application process that runs your Spark code. The driver creates the SparkContext, converts your program into tasks, and coordinates the execution across the cluster.

**Cluster Manager**: The external service responsible for acquiring resources and allocating them to Spark applications. Common cluster managers include YARN, Mesos, and Kubernetes.

**Executors**: Worker processes that run on cluster nodes. Executors execute tasks assigned by the driver and store data for caching operations.

**Tasks**: Individual units of work that executors perform on data partitions.

**Partitions**: Logical divisions of your data that enable parallel processing across multiple executor cores.

## Essential Spark Concepts

**Transformations** are operations that create new datasets from existing ones. The key insight here is that transformations follow lazy evaluation, meaning Spark doesn't actually do the work immediately. Instead, it builds up a plan of what you want to do.

Common transformations include filtering data (keeping only rows that meet certain criteria), mapping data (applying a function to transform each row), grouping data by certain columns, and joining different datasets together.

**Actions** are operations that actually trigger Spark to execute all those planned transformations and give you results. Actions either return results to your driver program or save data to external storage.

**RDDs (Resilient Distributed Datasets)** represent Spark's most fundamental way of thinking about data. RDDs are immutable (they never change once created), distributed across your cluster, and can be processed in parallel. They maintain lineage information, which means Spark remembers how each RDD was created so it can recreate it if something goes wrong.

**DataFrames** are a higher-level way to work with data that's more familiar if you've used databases or tools like Excel. DataFrames organize data into named columns and provide powerful optimization through Spark's Catalyst optimizer, which automatically makes your queries run faster.

**Spark SQL** lets you write familiar SQL queries against your DataFrames. This means you can use SELECT, WHERE, GROUP BY, and other SQL statements you might already know, while still getting all the benefits of distributed processing.



## Practical Example: Understanding the Basics

Let's walk through a comprehensive example that demonstrates how these concepts work together. I'll explain each part as we go, so you can see both what the code does and why we're doing it that way.

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, avg, count, max, min, sum as spark_sum
from pyspark.sql.types import StructType, StructField, StringType, IntegerType, FloatType

# Initialize Spark session - this creates your SparkContext
# Think of this as starting up your distributed computing engine
spark = SparkSession.builder \
    .appName("SparkFundamentals") \
    .config("spark.sql.adaptive.enabled", "true") \
    .getOrCreate()
```

This first section creates our Spark session, which is our entry point to all Spark functionality. The appName helps you identify your application when multiple Spark jobs are running. The config setting enables adaptive query execution, which helps Spark automatically optimize your queries as they run.

```python
# Create sample data representing employee records
# In real scenarios, this data would come from files, databases, or APIs
employee_data = [
    ("E001", "Alice Johnson", "Engineering", "Senior", 85000, 5, 92.5, "2019-03-15"),
    ("E002", "Bob Smith", "Marketing", "Manager", 72000, 3, 88.2, "2021-07-20"),
    ("E003", "Carol Davis", "Engineering", "Junior", 65000, 2, 85.7, "2022-01-10"),
    ("E004", "David Wilson", "Sales", "Senior", 78000, 4, 90.1, "2020-05-18"),
    ("E005", "Eva Brown", "Engineering", "Manager", 95000, 6, 94.3, "2018-11-22"),
    ("E006", "Frank Miller", "Marketing", "Junior", 58000, 1, 82.4, "2023-02-14"),
    ("E007", "Grace Lee", "Sales", "Senior", 82000, 4, 91.8, "2020-08-30"),
    ("E008", "Henry Taylor", "Engineering", "Senior", 88000, 5, 93.1, "2019-06-12"),
    ("E009", "Iris Chen", "Marketing", "Manager", 75000, 3, 89.6, "2021-10-05"),
    ("E010", "Jack Anderson", "Sales", "Junior", 62000, 2, 84.9, "2022-09-28")
]

# Define schema explicitly for data quality and performance
# This tells Spark exactly what type of data to expect in each column
employee_schema = StructType([
    StructField("employee_id", StringType(), False),     # False means this field cannot be null
    StructField("name", StringType(), False),
    StructField("department", StringType(), False),
    StructField("level", StringType(), False),
    StructField("salary", IntegerType(), False),
    StructField("years_experience", IntegerType(), False),
    StructField("performance_score", FloatType(), False),
    StructField("hire_date", StringType(), False)
])
```

Here we're creating sample data and defining its structure. In real applications, your data would typically come from files, databases, or streaming sources. The schema definition is important because it tells Spark exactly what to expect, which enables better performance and catches data quality issues early.

```python
# Create DataFrame - this represents your distributed dataset
# Even though our data is small, Spark treats it as if it could be distributed across many machines
df = spark.createDataFrame(employee_data, employee_schema)

# Basic transformations and actions
print("Dataset Overview:")
df.show()  # Action: displays data - this actually executes and shows results
print(f"Total employees: {df.count()}")  # Action: counts rows - another execution
```

This is where we create our DataFrame, which is Spark's way of representing structured data. Even though our example uses small data that fits in memory, Spark handles it the same way it would handle terabytes of data spread across hundreds of machines.

The show() and count() operations are actions, which means they trigger Spark to actually process the data and return results. Up until this point, Spark was just planning what to do.

```python
# Transformation: filter high performers
# This creates a new DataFrame but doesn't execute yet (lazy evaluation)
high_performers = df.filter(col("performance_score") > 90.0)
print(f"High performers (>90 score): {high_performers.count()}")  # Now it executes
```

This demonstrates the difference between transformations and actions. The filter operation creates a new DataFrame that represents "employees with performance scores above 90," but Spark doesn't actually do the filtering until we call count(), which is an action.

```python
# Transformation and action: departmental analysis
# This shows how to perform aggregations - very common in data processing
dept_analysis = df.groupBy("department").agg(
    avg("salary").alias("avg_salary"),              # Calculate average salary per department
    count("*").alias("employee_count"),             # Count employees per department
    avg("performance_score").alias("avg_performance"), # Average performance per department
    max("years_experience").alias("max_experience")   # Maximum experience per department
)

print("\nDepartmental Analysis:")
dept_analysis.show()  # Action: triggers execution of the entire aggregation
```

This section shows aggregation, which is one of the most common patterns in data processing. We're grouping employees by department and calculating various statistics for each group. The alias() method gives friendly names to our calculated columns.

```python
# Using Spark SQL - same functionality, different syntax
# Some people prefer SQL syntax for complex queries
df.createOrReplaceTempView("employees")  # Creates a temporary SQL table

print("\nSenior Employee Analysis (SQL):")
spark.sql("""
    SELECT department,
           COUNT(*) as senior_count,
           AVG(salary) as avg_senior_salary,
           AVG(performance_score) as avg_senior_performance
    FROM employees 
    WHERE level = 'Senior'
    GROUP BY department
    ORDER BY avg_senior_salary DESC
""").show()
```

This demonstrates that you can use SQL syntax to accomplish the same data processing tasks. Some people find SQL more intuitive for complex queries, especially when joining multiple tables or doing complex filtering and aggregation.

```python
# Demonstrate caching for performance
# This tells Spark to keep this DataFrame in memory for faster access
df.cache()  # Keeps data in memory for faster subsequent operations
```

Caching is a performance optimization technique. When you cache a DataFrame, Spark stores it in memory across your cluster, so subsequent operations on that DataFrame don't need to recompute it from the original data source.

## Understanding the Concepts in Action

Let's trace through what happens when you run this code to understand how the concepts work together:

When you create the SparkSession, you're establishing your connection to Spark's distributed computing capabilities. Even if you're running on a single machine, Spark still uses the same distributed architecture internally.

When you create the DataFrame, Spark doesn't immediately load or process the data. Instead, it creates a logical representation of what the data looks like and how it's structured. This is part of Spark's lazy evaluation strategy.

When you call transformations like filter() or groupBy(), Spark adds these operations to its execution plan but still doesn't do any actual work. It's building up a recipe for how to process your data when the time comes.

When you call an action like show() or count(), Spark finally executes the entire chain of transformations. It looks at all the operations you've requested, optimizes the execution plan, and then processes the data across your cluster.

The caching operation tells Spark to store the results in memory after the first computation, so if you perform additional operations on the same DataFrame, it can reuse the cached data instead of recomputing everything from scratch.



---

# Weather Data ETL Assignment

## Assignment Overview

Build a data pipeline that extracts weather data from the OpenWeatherMap API, processes it using Apache Spark, and visualizes the results in Grafana.

## Requirements

Extract weather data for at least 10 cities using the OpenWeatherMap API. Select any 10 columns from the API response that you find interesting or relevant.

Transform the data using PySpark to prepare it for visualization. Apply data cleaning, type conversions, or calculations as needed.

Store the processed data in any format that allows you to visualize it in Grafana.

Create visualizations in Grafana that demonstrate your data processing results. Include screenshots of your panels.



## Deliverables

1. **GitHub Repository**: Create a public repository containing your Python scripts and any configuration files
2. **Technical Article**: Write an article on Dev.to or Medium explaining your project, including:
   - Overview of your approach
   - Code explanations
   - Screenshots of your Grafana panels
   - Any challenges you encountered and how you solved them
