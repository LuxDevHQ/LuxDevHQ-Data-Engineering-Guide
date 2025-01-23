# Project: Setting Up Aiven Cloud Storage and Connecting a PostgreSQL Database Using DBeaver

### **Objective**:
The goal of this project is to set up a **managed PostgreSQL database** on **Aiven**, use their storage options, and connect to it using **DBeaver** to manage and query the data.

---

### **Steps to Complete the Project**:

#### **1. Set Up Aiven Account and Create PostgreSQL Service**
- **Create an Aiven account** if you don’t already have one: [Aiven Sign Up](https://aiven.io/).
- **Create a PostgreSQL service** on Aiven:
  - Log into the Aiven console.
  - Click **Create Service**.
  - Select **PostgreSQL** from the list of available services.
  - Choose the cloud provider (AWS, Google Cloud, etc.) and the region.
  - Configure your service and choose the storage options provided by Aiven.
  - Aiven will manage your PostgreSQL setup automatically, including backups, monitoring, and scaling.
- **Obtain connection details**:
  - Once the PostgreSQL service is created, note down the **hostname**, **port**, **username**, and **password**.
  - You'll use this information to connect from DBeaver.

#### **2. Set Up Cloud Storage on Aiven**
- **Aiven provides object storage** integrated with their managed services, so you don't need to manually set up AWS S3 or Azure Blob Storage. You can upload data directly to Aiven storage or integrate with other services.
- **Upload a sample file** to Aiven storage:
  - Go to the **Aiven dashboard** and look for storage options related to your service.
  - Upload a file (like a CSV) that you want to interact with using your PostgreSQL database.
- **Alternatively**, you can use an external storage service (e.g., AWS S3 or Azure Blob) to interact with Aiven if required. However, Aiven's managed storage service should work well for this project.

#### **3. Install and Configure DBeaver**
- **Download and install DBeaver** (SQL client for PostgreSQL):
  - Go to [DBeaver's official site](https://dbeaver.io/) and download the version suitable for your OS.
- **Connect to Aiven PostgreSQL service**:
  - Open DBeaver.
  - Create a **new connection**:
    - Select **PostgreSQL**.
    - Fill in the connection details (hostname, port, username, password, and database name from Aiven).
- Test the connection and make sure it's successful.

#### **4. Set Up PostgreSQL Database and Create Tables**
- Once connected via DBeaver, you can **create a new database** or use the existing one.
  - Example: Create a simple `products` table:
    ```sql
    CREATE TABLE products (
        id SERIAL PRIMARY KEY,
        name VARCHAR(100),
        price DECIMAL
    );
    ```
- **Insert some data** into the `products` table:
    ```sql
    INSERT INTO products (name, price) 
    VALUES ('Laptop', 1000), ('Smartphone', 700);
    ```
- **Query the data**:
    ```sql
    SELECT * FROM products;
    ```

#### **5. (Optional) Integrate Aiven Storage with PostgreSQL**
- If you're using Aiven's managed storage, you can perform the following operations:
- **Download data from Aiven storage** (if required), using Aiven's integration options or by connecting to storage buckets.
- **Load data into PostgreSQL** (if you’ve uploaded a CSV):
  - You can use `COPY` commands in PostgreSQL or perform an import directly through DBeaver’s **Import Data** option.
  - Example `COPY` command:
    ```sql
    COPY products FROM '/path/to/your/file.csv' DELIMITER ',' CSV HEADER;
    ```

#### **6. Perform Data Operations Using DBeaver**
- Use DBeaver to interact with the **PostgreSQL database**.
  - **CRUD Operations**: Create, read, update, and delete data.
  - **Querying**: Run SQL queries and get results directly in DBeaver.
  - **Database Management**: Create new tables, define schemas, and more.

---

### **Deliverables**:
1. **Screenshots** of the Aiven dashboard with the PostgreSQL service and storage bucket setup.
2. **SQL scripts** for creating and inserting data into the `products` table.
3. **Python script** (optional) for uploading files to Aiven storage (if applicable).
4. **Connection details** and queries executed via DBeaver.
5. A brief report documenting the steps taken, cloud setup, and any challenges faced.

---

### **Skills Gained**:
- Configuring and using **Aiven's managed PostgreSQL**.
- **Uploading data** to managed cloud storage.
- Using **DBeaver** to connect and query PostgreSQL.
- Integrating **cloud storage** with your database system.
- Performing **ETL operations** (optional if data is being uploaded).

---

### **Why Use Aiven for this Project?**
- **Managed PostgreSQL**: Aiven handles your PostgreSQL installation, backups, scaling, and monitoring, so you can focus on the data engineering tasks.
- **Storage Integration**: Easily manage cloud storage for your data and avoid manual setups of services like AWS S3 or Azure Blob.
- **Simplified Setup**: Aiven offers a streamlined, unified experience for cloud services, databases, and storage.

This updated version with **Aiven** simplifies your cloud storage and database setup while still providing the core hands-on experience in managing cloud databases and interacting with them via DBeaver.
