# Week 1, Day 3 - Data Governance, Security, Compliance, and Access Control

> **Month 1: Foundations of Data Engineering**
> **Week 1: Onboarding and Environment Setup**
> **Wednesday**

---

## What we covered yesterday (Day 2 recap)

On Day 2 we learned about **cloud computing** and the three major cloud platforms used in data engineering ; **AWS**, **Azure**, and **GCP**.

We learned that cloud platforms provide services for storage, databases, data warehouses, ETL pipelines, big data processing, real-time streaming, BI dashboards, and machine learning - and that data engineers use these services to build reliable data pipelines without managing physical servers.

> **Quick recap definition:**
> Cloud computing in data engineering means using online platforms like AWS, Azure, and GCP to store, move, process, clean, analyze, and secure data without managing physical servers.

---

## Today's objective

By the end of today, you should be able to:

- Explain what data governance is and why it matters.
- Describe the key principles of data security in cloud data engineering.
- Understand compliance frameworks like GDPR and HIPAA and what they mean for data engineers.
- Explain what access control is and how it works in cloud environments.
- Map governance, security, and compliance concepts to tools on AWS, Azure, and GCP.

---

## 1. What is data governance?

**Data governance** is the set of rules, policies, processes, and standards that control how data is collected, stored, managed, used, and protected within an organization.

In simple terms - data governance answers the questions:

- Who **owns** this data?
- Who is **allowed to see** it?
- How long should we **keep** it?
- How do we make sure it is **accurate**?
- How do we know **where it came from**?
- What happens if it is **misused or lost**?

### Simple analogy

Think of a hospital. Patient records contain highly sensitive information. The hospital needs rules about:

- Who can access a patient's records (only the treating doctor, not everyone).
- Where records are stored and for how long.
- What happens if records are leaked or lost.
- How records are audited and tracked.

Data governance in a company works exactly the same way - but for all the data the organization collects and uses.

### Why data governance matters for data engineers

As a data engineer, you are the person **building the systems that move and store data**. That means:

- If data is lost, it likely happened in a pipeline **you built**.
- If the wrong person accessed sensitive data, it likely came through a system **you designed**.
- If a company is fined for a compliance violation, the data infrastructure **you manage** may be involved.

Data governance is not just a management concern - it is a **core part of your job**.

---

## 2. The four pillars of data governance

| Pillar | What it means |
|---|---|
| **Data quality** | Ensuring data is accurate, complete, consistent, and up to date |
| **Data cataloguing** | Keeping a record of what data exists, where it lives, and what it means |
| **Data lineage** | Tracking where data came from, how it moved, and how it was transformed |
| **Data stewardship** | Assigning people who are responsible for maintaining data standards |

### Real-life example

An e-commerce company collects customer orders from its website, mobile app, and partner stores.

Without governance:
- The same customer may appear three times with slightly different names.
- Order amounts may be in different currencies with no conversion applied.
- Nobody knows which version of a product price is correct.
- An analyst builds a revenue report that is completely wrong.

With governance:
- There is one master customer record (a **single source of truth**).
- Currency conversion rules are documented and applied consistently.
- Every change to data is tracked - who changed it, when, and why.
- Reports are accurate and trusted by the business.

---

## 3. Data cataloguing and lineage tools

Data engineers use cataloguing and lineage tools to document and track data across cloud systems.

| Purpose | AWS | Azure | GCP |
|---|---|---|---|
| Data catalogue | AWS Glue Data Catalog | Microsoft Purview | Dataplex |
| Data lineage | AWS Glue / Glue Data Catalog | Microsoft Purview | Dataplex / BigQuery lineage |
| Metadata management | AWS Glue Data Catalog | Microsoft Purview | Data Catalog (now Dataplex) |

### Real-life example

A data engineer at a bank builds a pipeline that:
1. Pulls customer transaction data from PostgreSQL.
2. Transforms it in Azure Databricks.
3. Loads it into Azure Synapse Analytics.
4. Feeds it into a Power BI dashboard.

Using **Microsoft Purview**, the data engineer can show:
- Where the data originated (PostgreSQL).
- Every transformation it went through.
- Who accessed it at each stage.
- What the final dashboard is showing and where those numbers came from.

This full picture is called **data lineage** ; and it is critical for debugging, auditing, and compliance.

---

## 4. What is data security?

**Data security** is the practice of protecting data from unauthorized access, corruption, theft, or loss.

In cloud data engineering, data security covers:

- **Encryption** - scrambling data so it cannot be read without a key.
- **Access control** - making sure only the right people and systems can access data.
- **Network security** - protecting the paths data travels through.
- **Secrets management** - safely storing passwords, API keys, and credentials.
- **Monitoring and alerting** - detecting suspicious activity in real time.
- **Backup and recovery** - ensuring data can be restored if something goes wrong.

### Simple analogy

Think of your data as cash in a bank vault:

- **Encryption** = the vault door that locks everything inside.
- **Access control** = the list of people who are allowed inside the vault.
- **Network security** = the security cameras and guards outside.
- **Secrets management** = the safe where the vault combination is kept.
- **Monitoring** = the alarm system that alerts you if someone tries to break in.
- **Backup** = the copy of your money held in a second secure location.

---

## 5. Encryption - protecting data at rest and in transit

There are two states where data needs to be encrypted:

| State | What it means | Example |
|---|---|---|
| **At rest** | Data stored in a file, database, or data warehouse | CSV files in S3, rows in a database table |
| **In transit** | Data moving between systems over a network | Data being sent from an API to a pipeline |

All three cloud platforms encrypt data by default, but data engineers must verify and configure this correctly.

| Purpose | AWS | Azure | GCP |
|---|---|---|---|
| Encryption at rest | S3 SSE / KMS | Azure Storage Service Encryption | Google-managed encryption / CMEK |
| Encryption in transit | TLS / SSL | TLS / SSL | TLS / SSL |
| Key management | AWS KMS | Azure Key Vault | Cloud KMS |

> **Key concept - KMS:** KMS stands for Key Management Service. It is used to create and manage the encryption keys that protect your data. Think of it as the place where the vault combinations are stored and controlled.

---

## 6. Access control - who can see and do what?

**Access control** is the process of defining and enforcing who is allowed to access which data, systems, and resources - and what actions they are allowed to perform.

### The principle of least privilege

The most important rule in access control is the **principle of least privilege**:

> Give every person and system only the minimum access they need to do their job , nothing more.

**Example:**
- A junior data analyst should be able to **read** data from the warehouse - but not **delete** or **modify** it.
- An ETL pipeline should be able to **write** data to a storage bucket - but not access the production database.
- A BI dashboard tool should be able to **query** the data warehouse - but not modify its schema.

### Types of access control

| Type | What it means | Example |
|---|---|---|
| **RBAC** (Role-Based Access Control) | Access is granted based on a user's role | All data analysts get read-only access |
| **ABAC** (Attribute-Based Access Control) | Access is granted based on attributes like department, location, or data sensitivity | Only Nairobi branch staff can access Nairobi sales data |
| **DAC** (Discretionary Access Control) | The data owner decides who gets access | A team lead grants access to their folder |

### Access control tools across cloud platforms

| Purpose | AWS | Azure | GCP |
|---|---|---|---|
| Identity and access management | AWS IAM | Azure Active Directory (Entra ID) | Google Cloud IAM |
| Role assignment | IAM Roles | Azure RBAC | IAM Roles |
| Service accounts | IAM Roles for services | Managed Identities | Service Accounts |
| Secrets storage | AWS Secrets Manager | Azure Key Vault | Secret Manager |

### Real-life example

A data engineering team has three types of users:

| Person | Role | Access needed |
|---|---|---|
| Data engineer | Builds pipelines | Read/write to storage, run ETL jobs |
| Data analyst | Builds reports | Read-only access to the data warehouse |
| ETL pipeline (automated) | Moves data | Write to storage, read from source database |

On **Azure**, a data engineer would:
1. Create roles in **Azure Active Directory (Entra ID)** for each type of user.
2. Assign the appropriate permissions to each role.
3. Use a **Managed Identity** for the ETL pipeline so it authenticates automatically without storing a password.
4. Store any API keys or database passwords in **Azure Key Vault**.

---

## 7. What is compliance?

**Compliance** means following the laws, regulations, and industry standards that govern how data must be collected, stored, used, and protected.

Compliance is not optional. Organizations that violate data compliance laws can face:
- Heavy financial fines.
- Legal action.
- Reputational damage.
- Loss of operating licenses.

As a data engineer, you need to understand compliance requirements because **the systems you build must be designed to meet them**.

---

## 8. Key compliance frameworks

### GDPR - General Data Protection Regulation

| Item | Detail |
|---|---|
| **Region** | European Union (applies to any company handling EU citizens' data) |
| **Introduced** | May 2018 |
| **Governs** | How personal data of EU citizens is collected, stored, and used |
| **Key rights** | Right to access, right to erasure ("right to be forgotten"), right to data portability |
| **Fines** | Up to €20 million or 4% of global annual revenue , whichever is higher |

**What GDPR means for data engineers:**
- You must know exactly **where personal data is stored** across all systems.
- You must be able to **delete a specific person's data** on request - across every table, pipeline, and storage bucket it has touched.
- You must ensure personal data is **not stored longer than necessary**.
- You must be able to **report a data breach within 72 hours**.

---

### HIPAA - Health Insurance Portability and Accountability Act

| Item | Detail |
|---|---|
| **Region** | United States |
| **Governs** | Protected Health Information (PHI) - patient medical records and health data |
| **Key rules** | Data must be encrypted, access must be logged, breaches must be reported |
| **Fines** | Up to $1.9 million per violation per year |

**What HIPAA means for data engineers:**
- All health data must be **encrypted at rest and in transit**.
- Every access to patient data must be **logged and auditable**.
- Only **authorized personnel** may access PHI.
- Data must be stored in **HIPAA-compliant cloud environments** (all three major clouds offer HIPAA-compliant configurations).

---

### Other compliance frameworks to know

| Framework | Region / Industry | What it covers |
|---|---|---|
| **PCI-DSS** | Global - payments industry | Security standards for handling credit card data |
| **SOC 2** | Global - technology companies | Security, availability, and confidentiality of systems |
| **ISO 27001** | Global | International standard for information security management |
| **PDPA** | Kenya and East Africa | Personal Data Protection Act - Kenya's data protection law |

>  **PDPA - Kenya's data protection law:** As a data engineer working in Kenya or East Africa, the **Data Protection Act 2019 (Kenya)** is directly relevant to your work. It mirrors many GDPR principles and is enforced by the **Office of the Data Protection Commissioner (ODPC)**. Any system you build that handles Kenyan citizens' personal data must comply with this law.

---

## 9. Compliance tools across cloud platforms

| Purpose | AWS | Azure | GCP |
|---|---|---|---|
| Compliance management | AWS Artifact / Security Hub | Microsoft Defender for Cloud / Purview Compliance | Security Command Center |
| Audit logging | AWS CloudTrail | Azure Monitor / Activity Log | Cloud Audit Logs |
| Policy enforcement | AWS Config | Azure Policy | Organization Policy Service |
| Data classification | AWS Macie | Microsoft Purview | Cloud DLP (Data Loss Prevention) |
| Threat detection | Amazon GuardDuty | Microsoft Defender for Cloud | Security Command Center |

### Real-life example

A fintech startup in Nairobi processes payments for customers across Kenya and the UK. Their data systems must comply with:
- **Kenya's Data Protection Act** for Kenyan customers.
- **GDPR** for UK customers.
- **PCI-DSS** for payment card data.

Their data engineer on Azure would:
1. Use **Microsoft Purview** to classify and track all personal and payment data.
2. Use **Azure Policy** to enforce rules like "all storage accounts must have encryption enabled."
3. Use **Azure Monitor and Activity Logs** to create an audit trail of every access event.
4. Use **Azure Key Vault** to store all encryption keys and secrets.
5. Use **Microsoft Defender for Cloud** to detect threats and misconfigurations.

---

## 10. Monitoring and auditing

Governance and security are only effective if you can **see what is happening** in your systems at all times.

**Monitoring** means tracking the health and activity of your data systems in real time.
**Auditing** means keeping a detailed, tamper-proof record of who did what, when, and on which data.

| Purpose | AWS | Azure | GCP |
|---|---|---|---|
| Activity monitoring | Amazon CloudWatch | Azure Monitor | Cloud Monitoring |
| Audit trail / logs | AWS CloudTrail | Azure Activity Log | Cloud Audit Logs |
| Anomaly / threat detection | Amazon GuardDuty | Microsoft Defender for Cloud | Security Command Center |
| Alerts and notifications | CloudWatch Alarms | Azure Alerts | Cloud Monitoring Alerts |

### Real-life example

A data engineer at a bank sets up an alert that fires whenever:
- Someone tries to access the production database outside of business hours.
- A pipeline job fails three times in a row.
- A large volume of data is exported from the data warehouse unexpectedly.

These alerts are configured using **CloudWatch** (AWS), **Azure Monitor** (Azure), or **Cloud Monitoring** (GCP) - and they notify the engineer immediately so they can investigate.

---

## 11. How governance, security, and compliance connect

These three concepts work together and cannot be separated:

```text
Data Governance
    → defines the rules and policies for your data

Data Security
    → implements the technical controls that enforce those rules

Compliance
    → ensures your rules and controls meet legal and regulatory requirements
```

**Simple way to remember it:**

- **Governance** = the rulebook.
- **Security** = the locks and guards.
- **Compliance** = the inspector who checks that your locks meet the law.

---

## 12. Key terms to know from today

| Term | Meaning |
|---|---|
| Data governance | The rules, policies, and processes that control how data is managed and used |
| Data quality | Ensuring data is accurate, complete, and consistent |
| Data lineage | A record of where data came from and how it was transformed |
| Data catalogue | A directory of what data exists, where it lives, and what it means |
| Encryption at rest | Encrypting data while it is stored |
| Encryption in transit | Encrypting data while it is moving between systems |
| KMS | Key Management Service — manages encryption keys |
| Access control | Rules that define who can access what data and what they can do with it |
| Principle of least privilege | Give every user or system only the minimum access needed |
| RBAC | Role-Based Access Control — access granted based on a user's role |
| IAM | Identity and Access Management — the system that manages user identities and permissions |
| GDPR | EU regulation governing personal data of EU citizens |
| HIPAA | US regulation governing patient health data |
| PDPA | Kenya's Data Protection Act 2019 |
| PCI-DSS | Security standard for payment card data |
| Audit log | A tamper-proof record of who accessed or changed data and when |
| Data stewardship | Assigning responsibility for maintaining data quality and standards |
| Single source of truth | One authoritative, trusted version of a piece of data |

---

## 13. Summary

> **Data governance** is the set of rules and policies that control how data is managed, used, and protected.
>
> **Data security** implements technical controls - encryption, access control, secrets management, and monitoring — to enforce those rules.
>
> **Compliance** ensures that your governance and security practices meet legal and regulatory requirements like GDPR, HIPAA, PCI-DSS, and Kenya's Data Protection Act.
>
> As a data engineer, these are not abstract concepts - they shape how you design pipelines, configure cloud services, manage access, and handle sensitive data every single day.

---

## 14. What's coming next

| Day | Topic |
|---|---|
| Thursday | Introduction to SQL for data engineering and PostgreSQL setup |
| Friday | Peer project - environment setup challenges |
| Saturday (Lab) | Mini project - build a basic pipeline with PostgreSQL and Azure Blob Storage |

---

*Notes by LuxDevHQ | Month 1 - Foundations of Data Engineering | Week 1, Day 3*