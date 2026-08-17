# **Introduction to Data Modeling**

### **What is a Model?**

A **model** is a structure or dimension representing data.

### **What is Data Modeling?**

The process of designing how data will be **organized**, **stored**, and **accessed**.

* It provides a **visual representation** (tables, rows, columns).
* It acts as a **blueprint** for database design.

### **Purpose of Data Modeling**

* It ensures **accuracy**, **consistency**, and **integrity** of data.
* It optimizes **database performance**.
* It facilitates **communication** between technical and business stakeholders.

---

# **Types of Data Modeling**

## 1. Conceptual Data Modeling

* This represents a high-level overview of business/domain data without going into details.
* No technical details, no attributes or data types.
* It focuses on **entities** and their relationships.
* Example (Hospital Domain):

  * Entities: Patient, Doctor, Appointment.

---

## 2. Logical Data Modeling

* This model describes **data elements and relationships** in detail (without considering physical storage).
* It defines **attributes** for each entity.
* Example:

  * Doctor: doctor\_id, name, specialization.
  * Appointment: start\_time, end\_time, doctor\_id (FK).

---

## 3. Physical Data Modeling

* This model defines **how data is physically stored** in a database.
* It includes:

  * **Data types** (e.g., INT, VARCHAR).
  * **Constraints** (e.g., PRIMARY KEY, FOREIGN KEY).
  * **Indexes** for performance.

Example Schema:

### Doctor Table

* doctor\_id (INT) – Primary Key
* doctor\_name (VARCHAR)

### Customers Table

* customer\_id (INT) – Primary Key
* name (VARCHAR)
* age (INT)
* email (VARCHAR)
* DOB (DATE)
* phone (VARCHAR)

### Account Table

* account\_id (INT) – Primary Key
* balance (INT)
* dr (INT) (Debit)
* cr (INT) (Credit)
* acc\_type (VARCHAR)
* customer\_id (INT) – Foreign Key

### Branch Table

* branch\_id (INT) – Primary Key
* location (VARCHAR)

---

## 4. Entity-Relationship Data Modeling

* **Entity**: An object or concept representing data (e.g., Patient, Doctor, Appointment).
* **Attributes**: Properties of an entity (e.g., patient\_id, doctor\_name).
* **ERD (Entity-Relationship Diagram)**: Visual diagram showing entities and relationships.

### Relationship Types

| Type         | Example                                                                        |
| ------------ | ------------------------------------------------------------------------------ |
| One-to-One   | A Doctor has one doctor\_id, and each doctor\_id belongs to one Doctor.        |
| One-to-Many  | A Doctor has many Patients, but a Patient has only one primary Doctor.         |
| Many-to-Many | A Patient can have many Appointments, and a Doctor can have many Appointments. |

Detailed Examples:

* **One-to-Many (1\:M):**

  * "A Doctor has many Patients, but a Patient has only one primary Doctor."
  * Implementation: Patients table has a foreign key (doctor\_id).

* **Many-to-Many (M\:N):**

  * "A Patient can book many Appointments, and a Doctor can handle many Appointments."
  * Implementation: A junction table (Appointment) with patient\_id and doctor\_id as foreign keys.

* **One-to-One (1:1):**

  * "A Doctor has exactly one unique doctor\_id."
  * Implementation: doctor\_id is both a primary key and unique.

---

## 5. Dimensional Data Modeling

* Dimensional Data Modeling is primarily used in **data warehouses** for analytical purposes.
* it organizes data into **fact tables** and **dimension tables**.

### Key Components

| Component       | Description                               | Example                           |
| --------------- | ----------------------------------------- | --------------------------------- |
| Fact Table      | Numerical/measurable data (metrics/KPIs). | sales\_amount, quantity\_sold     |
| Dimension Table | Descriptive context for facts.            | customer\_name, product\_category |

Definitions:

* **Dimensions**: Describe business entities (name, age, product category).
* **Measures**: Quantitative facts (e.g., number of products sold).

---

# **Summary**

* **Conceptual**: High-level overview (what data is important?).
* **Logical**: Detailed attributes and relationships (how is data related?).
* **Physical**: Technical implementation (how is data stored?).
* **ER Modeling**: Graphical view of entities and relationships.
* **Dimensional**: Optimized for analytics (facts and dimensions).

