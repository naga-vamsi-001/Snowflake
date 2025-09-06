# ❄️ Snowflake Loading & Stages

This document continues from **snowflake_arch_and_lifecycle.md** and covers topics such as warehouse requirements, databases, schemas, bulk loading, staging, and integrations.

---

## 🏗️ 1. Warehouse Requirement
- **Most queries need a warehouse** (compute) → SELECT, INSERT, UPDATE, DELETE, MERGE, COPY INTO.
- **Some queries don’t need a warehouse**:
  - Metadata queries: `SHOW TABLES`, `DESCRIBE TABLE`.
  - Cached queries: results from Result Cache.
  - DDL for metadata objects: `CREATE DATABASE`, `CREATE SCHEMA`, `CREATE ROLE`.

✅ Rule: **DDL can be done without warehouse; DML requires a warehouse.**

---

## 🗄️ 2. Database Creation
- Creating a database is a **metadata operation**.
- Does **not** need a warehouse.
- Example:
  ```sql
  CREATE DATABASE sales_db;
  ```
- Same for schemas, roles, warehouses.

---

## 📂 3. Default Schemas in a Database
When you create a database, Snowflake automatically provides:
1. **INFORMATION_SCHEMA**
   - A built-in, read-only catalog with metadata views (TABLES, COLUMNS, VIEWS).
   - Exists in every database.
   - Shows metadata only for that database.
2. **PUBLIC**
   - A user schema created by default for convenience.
   - Normal schema, can be dropped or ignored.
   - Used when no schema is explicitly defined.

---

## 📦 4. Bulk Load vs Normal Load
- **Normal load** = inserting row-by-row or small sets with `INSERT`.
- **Bulk load** = loading large files via `COPY INTO` (CSV, JSON, Parquet, ORC, Avro).
- Bulk load is parallelized across warehouse nodes.
- **Why “bulk”?** → moves batches of files (MBs, GBs, TBs) efficiently.

---

## 🔄 5. Bulk Load Types
1. **Internal Bulk Load**
   - Files uploaded to Snowflake-managed storage (internal stage).
   - Types: user stage, table stage, named stage.
   - Upload via `PUT` command from SnowSQL.

2. **External Bulk Load**
   - Files remain in cloud storage (S3, Blob, GCS).
   - Snowflake reads them directly via an external stage.
   - Requires credentials or storage integration.

---

## 📥 6. COPY INTO from Cloud Storage
- `COPY INTO` is the command to load data from stages into tables.
- Works the same for internal and external stages.
- Example (S3):
  ```sql
  COPY INTO my_table
  FROM @my_s3_stage
  FILE_FORMAT = (TYPE=CSV FIELD_OPTIONALLY_ENCLOSED_BY='"' SKIP_HEADER=1);
  ```
- Requires a running warehouse.

---

## 🏷️ 7. Types of Stages
### Internal Stages (Snowflake-managed)
- **User Stage** → `@~`
- **Table Stage** → `@%table_name`
- **Named Stage** → `@stage_name`

### External Stages (Cloud storage)
- Amazon S3 → `@s3_stage`
- Azure Blob → `@azure_stage`
- Google Cloud Storage → `@gcs_stage`

---

## 🤔 8. Why Staging is Required
- Ensures **parallelism** → files can be split and read by many nodes.
- Provides **reliability** → files are checkpointed before load.
- Supports **different file formats** → CSV, JSON, Parquet, ORC.
- Enables **cloud-native integration** → direct access to S3, Blob, GCS.

👉 Direct load into a table without staging is **not supported**.

---

## 💻 9. SnowSQL vs UI
- Snowflake UI (Snowsight) is not designed for heavy file uploads.
- For local-to-Snowflake loads:
  - Use SnowSQL with `PUT` command to move files into an internal stage.
- For large production loads:
  - Use **external stages** (S3, Blob, GCS) or **Snowpipe**.

---

## 🔐 10. External Stage vs Storage Integration
- **External Stage with credentials**
  - Embeds AWS keys or SAS tokens in stage definition.
  - Less secure; requires manual rotation.

- **External Stage with Storage Integration**
  - Uses IAM Role (AWS), Service Principal (Azure), or Service Account (GCP).
  - No keys in Snowflake.
  - Preferred for production and compliance.

---

## ⚡ 11. Auto Loading
- **External Stage** = manual pull (you run `COPY INTO` or `REFRESH`).
- **Snowpipe** = event-driven auto-ingestion when files arrive in S3/Blob/GCS.
- Cloud notifications (SNS/SQS, Event Grid, Pub/Sub) trigger loads.

---

## 📤 12. COPY INTO Behavior
- Regardless of stage type (internal or external), load command is:
  ```sql
  COPY INTO <table> FROM @<stage>;
  ```
- Only difference:  
  - Internal stage → files uploaded via `PUT`.  
  - External stage → files already exist in S3/Blob/GCS.

---

# ✅ Summary
- **Warehouse needed** for DML, not for DDL.  
- **Databases auto-create INFORMATION_SCHEMA & PUBLIC schemas.**  
- **Bulk load (COPY INTO)** is the standard way to ingest large files.  
- **Stages** (internal or external) are required before loading.  
- **SnowSQL** bridges local → internal stage; external stages connect to cloud storage.  
- **Storage Integration** is the secure, recommended method.  
- **Snowpipe** automates continuous ingestion; external stage alone does not.
