# Snowflake – Learning Notes

Snowflake is a **cloud-native Data Warehouse-as-a-Service (DWaaS)** that provides scalable storage and compute for analytics workloads. Unlike traditional warehouses, Snowflake is fully managed (**SaaS**) and runs on **AWS, Azure, or GCP**.

---

## 1. What is Snowflake?
- **Cloud Data Warehouse**: Fully managed, serverless platform for storing and analyzing structured & semi-structured data.
- **SaaS**: No need to manage hardware, patching, or upgrades — Snowflake handles it automatically.
- **Separation of Compute & Storage**: Storage (data) and compute (virtual warehouses) scale independently.

---

## 2. Key Features
- **Elastic Scaling** – Scale compute clusters up/down on demand.
- **Multi-Cluster Architecture** – Handle concurrent workloads without contention.
- **Semi-Structured Data Support** – Query JSON, Avro, ORC, and Parquet with SQL.
- **Time Travel** – Query historical versions of data (1–90 days).
- **Fail-safe** – 7-day disaster recovery beyond time travel.
- **Secure Data Sharing** – Share live data across accounts/partners without copying.

---

## 3. OLTP vs OLAP
- **OLTP (Online Transaction Processing)**  
  - Day-to-day transactions (banking app, insurance policy updates).  
  - Short, frequent queries.  
  - Example: Insert a new insurance claim record.  

- **OLAP (Online Analytical Processing)**  
  - Historical + analytical queries for insights.  
  - Complex aggregations, joins, trend analysis.  
  - Example: Total claim payout trends by state for last 10 years.  

👉 Snowflake is **OLAP**.

---

## 4. Snowflake Login & Accounts
- Each Snowflake account has a **unique login URL**:
  ```
  https://<account_identifier>.<region>.<cloud>.snowflakecomputing.com
  ```
- Example (AWS, us-east-1):
  ```
  https://xy12345.us-east-1.aws.snowflakecomputing.com
  ```
- By looking at the URL, you know:
  - **Cloud provider** (AWS/Azure/GCP)  
  - **Region** (us-east-1, eastus2, etc.)  
  - **Account identifier**  

---

## 5. Data Storage Options
- **Internal (Managed)**: Data fully stored inside Snowflake.  
- **External Tables**: Point Snowflake at files in S3/ADLS/GCS (schema-on-read, requires metadata refresh).  
- **Iceberg Tables**: Open table format with **ACID, schema evolution, and time travel** on data lakes.  

---

## 6. Time Travel & Versioning
- Every table change (INSERT/UPDATE/DELETE) creates a version.
- Undrop table:
  ```
  UNDROP TABLE policy_info;
  ```
- Retention:
  - Standard: 1 day
  - Enterprise: Up to 90 days
- Fail-safe: Extra 7 days (Snowflake-managed only).

---

## 7. Delta vs Iceberg (Table Formats)
- **Delta Lake (Databricks)**  
  - Parquet + `_delta_log/` transaction log  
  - Best inside Databricks ecosystem  

- **Iceberg (Apache)**  
  - Open standard, Parquet/ORC/Avro + metadata + snapshots  
  - Strong multi-engine support (Snowflake, Spark, Trino, Athena)  

👉 Snowflake supports **Iceberg tables**, not Delta.

---

## 8. Key Concepts in Snowflake
- **Database** → Logical grouping of schemas.
- **Schema** → Logical grouping of tables, views, etc.
- **Table Types**:
  - Permanent (default, full features + fail-safe).
  - Temporary (session-based).
  - Transient (no fail-safe, cheaper).
- **Virtual Warehouse** → Compute cluster for query execution.
- **Stage** → Location for loading/unloading files (internal or external).
- **Role-Based Access Control (RBAC)** → Security & permissions model.

---

## 9. Typical Use Cases
- Centralized Data Warehouse
- BI & Analytics (Power BI, Tableau, Looker)
- Data Lakehouse (via Iceberg)
- ELT Processing (transform inside Snowflake)
- Secure Data Sharing across orgs

---

## 10. Why Snowflake?
- SaaS (no infra mgmt)
- Cross-cloud availability
- Pay-per-second compute
- Easy scaling for big data analytics
- Built-in versioning & recovery

---

# ✅ Summary
Snowflake is a **modern OLAP cloud data warehouse** that removes the burden of infrastructure and lets teams focus on **data, governance, and analytics**. With features like **time travel**, **separation of storage/compute**, and **open format support (Iceberg)**, it is widely used for building enterprise data platforms.

---
