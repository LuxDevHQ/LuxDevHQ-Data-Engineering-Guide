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
- [Kenya Open Data](https://opendata.go.ke/)  
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
- Tools: **Metabase, Superset, Power BI**, or **Google Data Studio**

#### **🔒 Data Governance**
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

