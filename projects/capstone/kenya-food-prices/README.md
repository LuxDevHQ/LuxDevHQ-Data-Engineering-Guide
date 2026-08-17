### **CH02-2025-Data Engineering Capstone Project: Build a Data Platform for Analyzing Kenya’s Food Prices and Inflation Trends.**

**Domain**: Public Data || Economics || Agriculture.

**Data Availability**: Easy – Publicly available from official government sources  

---
#### **🧩 Project Brief**

Your team has been contracted by a government think tank to build a **data platform** that tracks food prices across Kenyan counties, detects inflation patterns, and generates insights for consumers, farmers, and policymakers.

You'll pull **real data** from public sources (see below), clean and model it, and build both **batch and near-real-time** pipelines for analysis and visualization.

---

### **🗃️ Suggested Data Sources**
- 🇰🇪 [Kenya National Bureau of Statistics (KNBS)](https://www.knbs.or.ke/)
  - Monthly food price reports (PDF/Excel)
  - CPI & inflation datasets  
- [World Bank Open Data – Kenya](https://data.worldbank.org/country/kenya)
- [FAOSTAT – Food & Agriculture Data](https://www.fao.org/faostat/)
- [Kenya Open Data](https://kenya.opendataforafrica.org/)  
  - County-level data on market prices, commodities, population, etc.

---

### **🔧 Requirements**

#### **🏗️ Batch Pipeline**
- Ingest food price data (monthly or weekly) from KNBS or Open Data portal.
- Clean and normalize pricing formats (handle missing values, different currencies/units).
- Use **Airflow** to automate downloads and ETL processes with **PySpark** or **Pandas**.
- Create **fact/dimension tables** with a star schema (e.g., product, county, time).

#### **🌐 Optional Web Scraping Add-on**
- Scrape data from a public market pricing site or KNBS portal (if available).
- Use **BeautifulSoup** or **Selenium** (optional, only if permitted).

#### **📡 Streaming Component (Optional but Impressive)**
- Simulate daily market price updates using a **Kafka producer** (e.g., tomatoes in Nairobi).
- Consume and store using **Spark Streaming → Delta Lake/S3/PostgreSQL**.

#### **📊 Visualization & Dashboarding**
- Build an analytics dashboard with:
  - Price changes over time
  - Inflation heatmaps by county
  - Product comparison across regions
- Tool: **Grafana or Power BI**

#### **Data Governance**
- Add metadata to tag data sources, update frequency, and validation steps.
- Track data lineage through Airflow logs or a simple metadata table.

---

### **📁 Deliverables**
- GitHub repo with pipeline code, Airflow DAGs, and documentation
- Final dashboard (hosted or screenshots)
- README with architecture diagram and data model
- Presentation deck with insights and demo

---

### **🧠 Learning Outcomes**
- Automate real-world data collection and transformation  
- Practice ETL, data modeling, and basic analytics  
- Work with government and open datasets  
- Communicate insights through dashboards and presentations  


---

## Student Project Guide

### Why This Project Matters
Reliable public-price data can help analysts and policymakers understand regional inflation while demonstrating an end-to-end portfolio workflow.

### Learning Objectives
Design ingestion, quality checks, dimensional models, orchestration, analytical outputs, governance metadata, and clear technical documentation.

### Architecture

```text
Public data / permitted web source → Airflow → pandas or Spark → lake/PostgreSQL
                                                    ↘ dashboard
Optional simulated events → Kafka → Spark Streaming ────────┘
```

### Tools Used
**Airflow** schedules work; **pandas or Spark** cleans data; **PostgreSQL or object storage** persists it; **Kafka** optionally carries simulated events; **Grafana or Power BI** communicates results.

### Prerequisites
Complete the SQL, ETL, modeling, Airflow, Spark, lake, and governance notes in the [learning index](../../../notes/README.md). Confirm each source's current availability, terms, and schema before implementation.

### Setup and Project Tasks
1. Choose and profile one permitted source.
2. Draw the architecture and star schema before coding.
3. Build and test a small batch path, then automate it.
4. Add validation, metadata, and the dashboard.
5. Attempt streaming only after the batch path is reproducible.

### Expected Output
A reproducible repository containing the deliverables listed above, quality evidence, safe configuration examples, and a dashboard answering defined questions.

### Challenges / Extensions
Add the optional permitted scraper or simulated stream, track source versions, measure freshness, or deploy monitoring.
