# ❄️ Snowflake Architecture & Query Lifecycle

This document explains Snowflake’s architecture and provides a step‑by‑step walkthrough of what happens when a user runs a query in the Snowflake worksheet.

---

## 🏛️ Snowflake Architecture Overview

![Snowflake_Architecture.png](https://raw.githubusercontent.com/naga-vamsi-001/Images/main/snowflake/Snowflake_Architecture.png)

Snowflake’s architecture is divided into **three layers** inside a Virtual Private Cloud (VPC):

1. **Cloud Services Layer**  
2. **Query Processing Layer**  
3. **Database Storage Layer**  

---

### 1️⃣ Cloud Services Layer
This is the **control plane** — the brain of Snowflake that manages and coordinates all operations.

- **Authentication & Access Control** → Manages users, roles, and privileges.  
- **Infrastructure Manager** → Handles provisioning and orchestration of compute/storage.  
- **Optimizer** → Generates the most efficient execution plan for SQL queries.  
- **Metadata Manager** → Stores table structures, statistics, and query history.  
- **Security** → End-to-end encryption, auditing, compliance controls.  

🔑 Think of this as Snowflake’s **management & intelligence layer**.

---

### 2️⃣ Query Processing Layer
This is the **compute plane**, powered by **Virtual Warehouses**.

- **Virtual Warehouse** = a compute cluster dedicated to query execution.  
- Multiple warehouses can run independently, all accessing the same shared data.  
- Each warehouse can scale up/down or suspend when not in use.  
- Warehouses can be assigned to different workloads (ETL jobs, BI dashboards, ML training).  

🔑 This layer ensures **performance, scalability, and workload isolation**.

---

### 3️⃣ Database Storage Layer
This is the **storage plane**, fully managed by Snowflake.

- Data is stored in a **compressed, columnar format**.  
- Physically resides in cloud object storage (AWS S3, Azure Blob, or GCP Storage).  
- Automatically partitioned, optimized, and encrypted by Snowflake.  
- Users don’t manage files — only tables and schemas.  

🔑 This layer ensures **durability, security, and transparency**.

---

## ⚙️ Snowflake Query Lifecycle (Worksheet → Results)

Below is the **step‑by‑step process** when a user logs in and runs a query in Snowflake.

---

### 🔐 1) Login & Session Creation
- **User logs in** via unique Snowflake account URL.  
- **Authentication & Access Control** verifies identity (password, SSO, MFA).  
- **Security** ensures TLS encryption and audit logging.  
- A **session** is created with role, warehouse, and database context.

---

### 🧩 2) Worksheet Context
- Worksheet attaches to session.  
- **RBAC** enforces role-based access to objects and warehouses.  
- Context (`USE ROLE`, `USE WAREHOUSE`, `USE DATABASE`) is set.

---

### 📨 3) Query Submitted
- SQL is parsed & validated.  
- **Metadata Manager** resolves objects and fetches stats.  
- **RBAC/Security** checks SELECT/USAGE privileges.  

---

### ⚙️ 4) Warehouse Provisioning
- **Infrastructure Manager** checks warehouse state:  
  - Suspended → auto‑resume.  
  - High load → auto‑scale extra clusters (if multi‑cluster enabled).  
- Compute billing starts only while running.

---

### 🧠 5) Optimization
- **Optimizer** builds the physical execution plan.  
- Uses metadata & stats for:  
  - Join order & type.  
  - Predicate pushdown.  
  - Micro‑partition pruning.  

---

### 🚀 6) Execution
- **Virtual Warehouse** executes the plan.  
- Reads required micro‑partitions from **Database Storage**.  
- Data is decrypted and scanned in a columnar/vectorized manner.  
- Intermediate results shuffled across nodes for joins/aggregations.

---

### ⚡ 7) Caching
- **Result Cache (Cloud Services):** same query, same role → instant results.  
- **Metadata Cache:** speeds up planning.  
- **Data Cache (Warehouse):** hot partitions stay in local SSD/RAM.  

---

### 🗄️ 8) Storage Layer
- Data in **columnar micro‑partitions** in cloud storage.  
- Automatic clustering + pruning skip irrelevant data.  
- **Time Travel** can serve past versions if requested.  
- **Fail‑safe** (7 days) available for recovery.

---

### 📤 9) Results & Observability
- Results streamed back to worksheet.  
- **Query Profile** available (execution graph, pruning stats).  
- Usage & billing recorded.

---

### 💤 10) Autosuspend
- After idle timeout, **Infrastructure Manager** suspends the warehouse.  
- Billing stops until next auto‑resume.

---

## ✅ TL;DR
1. **Cloud Services** authenticate, authorize, plan, and manage resources.  
2. **Virtual Warehouse** executes the optimized plan.  
3. **Database Storage** serves secure, optimized data.  
4. **Caches & autosuspend** ensure performance and cost efficiency.

---
