# 🚀 **Handling Change Data Capture (CDC) in Aurora using dbt**  

### 🎯 **Goal**  
Implement **Change Data Capture (CDC) in Amazon Aurora** using only **dbt**, avoiding tools like Debezium or AWS DMS. This tutorial will:  
✅ Capture **INSERT, UPDATE, DELETE** changes from Aurora  
✅ Implement **Incremental CDC Models** in **dbt**  
✅ Maintain **History and Snapshot** tables  
✅ Use **dbt Tests & Audits** to validate CDC  

---

## **📌 1. Setup: Aurora + dbt**
### **🔹 Prerequisites**  
1️⃣ **Amazon Aurora MySQL/PostgreSQL** (Existing or New Database)  
2️⃣ **dbt Core Installed** (Locally or in dbt Cloud)  
3️⃣ **Python & dbt Dependencies Installed**  
4️⃣ **Profiles.yml Configured for Aurora Connection**  

**Example `profiles.yml` (Postgres Example)**
```yaml
my_aurora_dbt:
  target: dev
  outputs:
    dev:
      type: postgres
      host: aurora-cluster.endpoint.rds.amazonaws.com
      user: myuser
      password: mypassword
      port: 5432
      dbname: mydb
      schema: raw
      threads: 4
```

---

## **📌 2. Create Source Table in Aurora**
We assume a table **customers** where changes happen.

```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255),
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```
✅ **Note:** The `last_updated` column is required for CDC.

---

## **📌 3. Capture Changes Using dbt Incremental Models**
### **🔹 Step 1: Create Source Definition in dbt**
📂 **models/staging/stg_customers.sql**
```sql
WITH source AS (
    SELECT
        id,
        name,
        email,
        last_updated
    FROM {{ source('aurora', 'customers') }}
)
SELECT * FROM source
```
✅ **This allows dbt to recognize the table as a source.**  

---

### **🔹 Step 2: Create Incremental CDC Model in dbt**
📂 **models/incremental/int_customers_cdc.sql**
```sql
{{ config(
    materialized='incremental',
    unique_key='id'
) }}

WITH source AS (
    SELECT * FROM {{ ref('stg_customers') }}
)
{% if is_incremental() %}
-- Only fetch new or updated records
SELECT *
FROM source
WHERE last_updated > (SELECT MAX(last_updated) FROM {{ this }})
{% else %}
-- Full load for initial run
SELECT * FROM source
{% endif %}
```
✅ **This model ensures:**  
🔹 **Only new/updated records** are processed  
🔹 Maintains a **historical record** of changes  
🔹 Uses `is_incremental()` to fetch only changed data  

---

## **📌 4. Track Deleted Records in CDC**
### **🔹 Step 1: Add a Deleted Records Table**
```sql
CREATE TABLE deleted_customers (
    id INT PRIMARY KEY,
    deleted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### **🔹 Step 2: Modify CDC Model to Detect Deletes**
📂 **models/incremental/int_customers_cdc_with_deletes.sql**
```sql
{{ config(
    materialized='incremental',
    unique_key='id'
) }}

WITH source AS (
    SELECT * FROM {{ ref('stg_customers') }}
),
deleted AS (
    SELECT id FROM deleted_customers
)
{% if is_incremental() %}
SELECT * FROM source
WHERE last_updated > (SELECT MAX(last_updated) FROM {{ this }})
AND id NOT IN (SELECT id FROM deleted)
{% else %}
SELECT * FROM source
WHERE id NOT IN (SELECT id FROM deleted)
{% endif %}
```
✅ **Now, if a row is deleted, it is automatically removed from the CDC model.**  

---

## **📌 5. Automate CDC Execution in dbt**
### **🔹 Run CDC Incrementally**
```sh
dbt run --select int_customers_cdc
```

### **🔹 Schedule CDC Runs**
In **dbt Cloud**, schedule this job to run every **5 minutes** for real-time CDC.

---

## **📌 6. Add dbt Tests for CDC Validation**
📂 **tests/cdc_test.sql**
```yaml
version: 2
models:
  - name: int_customers_cdc
    tests:
      - not_null:
          column_name: id
      - unique:
          column_name: id
```
Run tests:
```sh
dbt test --select int_customers_cdc
```
✅ Ensures data **integrity and correctness.**

---

## **📌 7. CDC Auditing: Maintain a Snapshot Table**
If you want a **full history of changes**, create a **snapshot model**.

### **🔹 Step 1: Enable Snapshots in dbt**
📂 **snapshots/customers_snapshot.sql**
```sql
{% snapshot customers_snapshot %}
    {{
        config(
            target_schema='snapshots',
            unique_key='id',
            strategy='timestamp',
            updated_at='last_updated'
        )
    }}
    SELECT * FROM {{ ref('stg_customers') }}
{% endsnapshot %}
```
### **🔹 Step 2: Run Snapshots**
```sh
dbt snapshot
```
✅ **Keeps a full history of row changes in Aurora!**  

---

## **🚀 Final Summary**
| Step | Action | dbt Feature |
|------|--------|------------|
| 1️⃣ **Enable CDC** | Use `last_updated` timestamp | Aurora Table |
| 2️⃣ **Extract Data** | Create Staging Model | `stg_customers.sql` |
| 3️⃣ **Incremental Processing** | Use `is_incremental()` | `int_customers_cdc.sql` |
| 4️⃣ **Track Deletes** | Maintain a deleted records table | `deleted_customers` |
| 5️⃣ **Run Incremental Updates** | `dbt run --select int_customers_cdc` | Scheduled Jobs |
| 6️⃣ **Validate Data** | Add `dbt tests` for CDC | `dbt test` |
| 7️⃣ **Audit with Snapshots** | Track historical changes | `dbt snapshot` |

---

