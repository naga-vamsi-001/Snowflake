# ❄️ Snowpipe – Continuous Data Ingestion with Pipeline Flows (Fixed GitHub Diagrams)

Snowpipe enables **continuous, automated loading of data** into Snowflake from cloud storage like **AWS S3** or **Azure Blob Storage**.
It listens for file arrival events and triggers a `COPY INTO` operation automatically — eliminating the need for manual loads.

---

## 🔍 What is Snowpipe?

Snowpipe is a **serverless ingestion service** that:

* Connects to cloud storage (S3 / Blob / GCS).
* Automatically detects new files (via event notifications).
* Loads data into Snowflake tables using `COPY INTO`.

✅ **In short:**
Snowpipe = *event-driven data loader* → from **cloud storage → stage → target table**.

---

# ☁️ Snowpipe in Azure (Blob Storage)

## 🔹 Step 1. Create Azure Queue

1. In Azure Portal → go to **Storage Account**.
2. Navigate to **Queues** → click **+ Queue** → give it a name → **Create**.

   * This queue will receive event notifications when new blobs are created.

---

## 🔹 Step 2. Create Azure Event Grid Subscription

1. Go to **Storage Account → Events → + Event Subscription**.
2. Fill details:

   * **Event Schema** → *Event Grid Schema*
   * **Source Resource** → Default
   * **System Topic Name** → choose any name
   * **Event Types** → select **Blob Created**
   * **Endpoint Type** → *Storage Queue*
   * **Endpoint** → choose your **Storage Account + Queue**
3. Click **Create**.

👉 This ensures that whenever a file lands in your Blob container, a message is pushed to the Queue.

---

## 🔹 Step 3. Create Notification Integration in Snowflake

```sql
CREATE NOTIFICATION INTEGRATION azure_notif_int
  ENABLED = TRUE
  TYPE = QUEUE
  NOTIFICATION_PROVIDER = AZURE_STORAGE_QUEUE
  AZURE_STORAGE_QUEUE_PRIMARY_URI = 'https://<storage-account>.queue.core.windows.net/<queue-name>'
  AZURE_TENANT_ID = '<your-tenant-id>';
```

---

## 🔹 Step 4. Approve Access (Consent)

```sql
DESC NOTIFICATION INTEGRATION azure_notif_int;
```

This returns:

* **CONSENT_URL** → Open in browser to approve Snowflake access.
* **AZURE_MULTI_TENANT_APP_NAME** → used for IAM role assignment.

---

## 🔹 Step 5. Modify Azure Access Control (IAM)

1. Go to **Storage Account → Access Control (IAM)**.
2. Click **+ Add Role Assignment**.
3. Choose **Role**: *Storage Queue Data Contributor*.
4. **Assign access to:** *User, group, or service principal*.
5. Add **Member** = *Azure_Multi_Tenant_App_Name*.
6. Save changes.

---

## 🔹 Step 6. Create Snowpipe

If no external stage exists, create one:

```sql
CREATE OR REPLACE STAGE ext_blob_stage
  URL='azure://<storage-account>.blob.core.windows.net/<container-name>/'
  STORAGE_INTEGRATION = azure_storage_integration
  FILE_FORMAT = (TYPE = 'CSV' SKIP_HEADER = 1 FIELD_OPTIONALLY_ENCLOSED_BY='"');
```

Then create the Snowpipe:

```sql
CREATE OR REPLACE PIPE azure_sales_pipe
  AUTO_INGEST = TRUE
  INTEGRATION = azure_notif_int
AS
  COPY INTO sales
  FROM @ext_blob_stage
  FILE_FORMAT = (TYPE = 'CSV' SKIP_HEADER = 1 FIELD_OPTIONALLY_ENCLOSED_BY='"');
```

---

## 🔹 Step 7. (Optional) Load Existing Files

```sql
ALTER PIPE azure_sales_pipe REFRESH;
```

---

# ☁️ Snowpipe in AWS (S3)

## 🔹 Step 1. Create SQS Queue

1. In AWS Console → **SQS** → *Create Queue*.
2. Choose **Standard Queue**.
3. Copy the **ARN**.

---

## 🔹 Step 2. Configure S3 Event Notification

1. Go to **S3 Bucket → Properties → Event Notifications → Create event notification**.
2. Choose:

   * **Event type:** *All object create events*
   * **Destination:** *SQS queue*
   * **Queue:** select the one you created
3. Save.

---

## 🔹 Step 3. Create Notification Integration in Snowflake

```sql
CREATE NOTIFICATION INTEGRATION aws_notif_int
  ENABLED = TRUE
  TYPE = QUEUE
  NOTIFICATION_PROVIDER = AWS_SNS
  AWS_SQS_ARN = 'arn:aws:sqs:us-east-1:123456789012:myqueue'
  AWS_SNS_TOPIC_ARN = 'arn:aws:sns:us-east-1:123456789012:mysnstopic'
  AWS_SNS_ROLE_ARN = 'arn:aws:iam::123456789012:role/my-snowflake-role';
```

---

## 🔹 Step 4. Create External Stage

```sql
CREATE OR REPLACE STAGE ext_s3_stage
  URL='s3://my-bucket/data/'
  STORAGE_INTEGRATION = aws_storage_integration
  FILE_FORMAT = (TYPE = 'CSV' SKIP_HEADER = 1 FIELD_OPTIONALLY_ENCLOSED_BY='"');
```

---

## 🔹 Step 5. Create Snowpipe

```sql
CREATE OR REPLACE PIPE aws_sales_pipe
  AUTO_INGEST = TRUE
  INTEGRATION = aws_notif_int
AS
  COPY INTO sales
  FROM @ext_s3_stage
  FILE_FORMAT = (TYPE = 'CSV' SKIP_HEADER = 1 FIELD_OPTIONALLY_ENCLOSED_BY='"');
```

---

## 🔹 Step 6. Load Existing Files

```sql
ALTER PIPE aws_sales_pipe REFRESH;
```

---

# 🧩 Snowpipe Summary

| Component                        | Purpose                                          |
| -------------------------------- | ------------------------------------------------ |
| **Event Grid / S3 Notification** | Detects new files and triggers an event.         |
| **Queue (SQS / Azure Queue)**    | Receives notification messages.                  |
| **Notification Integration**     | Snowflake connection to the queue.               |
| **Pipe**                         | Defines the COPY operation and target table.     |
| **Stage**                        | Location of source files (internal or external). |
| **Snowpipe**                     | Automates loading new files continuously.        |

---

# 🧭 Snowflake Data Flow Diagrams

## 🔹 Bulk Load Flow (S3 / Blob → Stage → COPY INTO → Table)

```mermaid
flowchart LR
    DL["Data Lake: S3 / Blob / GCS"] --> STAGE["Stage (Internal or External)"]
    STAGE -->|COPY INTO table| WH["Virtual Warehouse"]
    WH --> TBL["Target Table"]
    TBL --> STG["Snowflake Managed Storage"]
    WH -->|Load Status| ACC["Account Usage / Load History"]
```

---

## 🔹 Snowpipe Auto-Ingest Flow (Event-Driven)

```mermaid
flowchart LR
    subgraph Cloud["Cloud Storage"]
      A["New File in S3 or Blob"] --> EVT["Event (S3 Notification or Event Grid)"]
      EVT --> Q["Queue (SQS or Azure Queue)"]
    end

    Q --> NI["Notification Integration"]
    NI --> PIPE["Snowpipe (AUTO_INGEST = TRUE)"]
    PIPE -->|COPY INTO table| WH["Virtual Warehouse"]
    WH --> TBL["Target Table"]
    TBL --> STG["Snowflake Managed Storage"]
    PIPE --> MON["Load History / Pipe Status"]
```

---

# ✅ Best Practices

* Use **AUTO_INGEST = TRUE** only when event notifications are configured.
* Use **STORAGE INTEGRATION** for secure access (avoid embedding credentials).
* Use **ALTER PIPE REFRESH** only for backfills (not frequent loads).
* Monitor Snowpipe activity with:

  ```sql
  SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.LOAD_HISTORY WHERE PIPE_NAME='AZURE_SALES_PIPE';
  ```

---

# 🚀 Summary

Snowpipe = *continuous, event-driven data ingestion*:

* **Azure:** Event Grid → Queue → Notification Integration → Pipe → Table
* **AWS:** S3 Event → SQS/SNS → Notification Integration → Pipe → Table

Once configured, every new file landing in your cloud storage is **automatically loaded into Snowflake** in near real-time.