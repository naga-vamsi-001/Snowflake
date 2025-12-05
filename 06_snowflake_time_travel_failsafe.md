# ❄️ Snowflake Time Travel & Fail-Safe

## ⏳ Time Travel in Snowflake

Time Travel allows you to **query, clone, and restore data from the past** within a defined retention period.

### ✅ What Time Travel Allows
- Query past data versions  
- Clone tables/schemas/databases at a past timestamp  
- Restore tables to older versions  
- Undrop tables, schemas, or databases  

### 🔍 Query Examples
```sql
SELECT * FROM orders AT (TIMESTAMP => '2024-01-10 12:00:00');
SELECT * FROM orders BEFORE (OFFSET => 60*5);  -- 5 minutes ago
```

### 🔍 Clone Example (Zero Copy)
```sql
CREATE OR REPLACE SCHEMA labwork_schema
CLONE labwork
AT (OFFSET => 600);
```
Here `600` = **600 seconds = 10 minutes** → clone the schema as it existed **10 minutes ago**.

---

## 🧊 Zero-Copy Clone Explained

Cloning a table, schema, or database **does not copy data physically**.

- Uses existing micro-partitions  
- Only new changes after clone consume storage  
- Fast & cost-efficient  
- Perfect for testing, debugging, or creating safe sandboxes  

---

## 🔥 Fail-Safe in Snowflake

Fail-Safe is a **7‑day period after Time Travel ends** where Snowflake internally retains data for **disaster recovery only**.

### ❗ Important Facts
- ❌ You *cannot* query Fail-Safe  
- ❌ You *cannot* restore directly from Fail-Safe  
- ❌ Fail-Safe is *not* user-accessible  
- ✔ Only **Snowflake Support** can access Fail-Safe data  
- ✔ Applies **only to Permanent tables**

### 🔁 Data Lifecycle
```
Active Data → Time Travel → Fail-Safe → Permanent Deletion
```

---

## 📌 Fail-Safe Support by Table Type

| Table Type | Time Travel | Fail-Safe | Description |
|------------|-------------|-----------|-------------|
| Permanent | ✔ | ✔ 7 days | Full protection |
| Transient | ✔ (1 day max) | ❌ | Lower cost, staging |
| Temporary | ❌ | ❌ | Session-only |
| External | ❌ | ❌ | Data in cloud storage |
| Iceberg | Limited | ❌ | Depends on mode |

---

## 🧠 Summary

- **Time Travel** = Recover, restore, clone old data versions  
- **Offset** = Number of seconds from current timestamp  
- **Clone** = Zero-copy snapshot of any object at any point in Time Travel window  
- **Fail-Safe** = 7 days extra internal protection (Snowflake-managed)  
- Fail-Safe is **not** for users; only Snowflake can use it

