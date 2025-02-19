# **🚀 Understanding Pre-hooks & Post-hooks in dbt**  

## **📌 What are Pre-hooks and Post-hooks in dbt?**  
🔹 **Pre-hooks:** SQL statements that run **before** a model executes.  
🔹 **Post-hooks:** SQL statements that run **after** a model executes.  

✅ Used for **logging, auditing, row count validation, or automating tasks**.  
✅ Works in **all dbt materializations**: `table`, `view`, `incremental`, `ephemeral`.  

---

## **📌 Use Cases: Why Use Pre/Post Hooks?**  

### **1️⃣ Auditing & Logging Execution Metadata**
🔹 Log **execution time, row counts, and model runs** in an audit table.  
🔹 Helps with **data quality monitoring**.  

### **2️⃣ Capturing Row Counts & Changes (CDC)**
🔹 Store **before and after row counts** to detect **changes**.  
🔹 Useful for **incremental models, snapshot tracking, and debugging**.  

### **3️⃣ Managing Permissions & Grants**
🔹 Automate **GRANT SELECT** after model creation.  
🔹 Ensure correct **RBAC permissions** in Snowflake, Redshift, etc.  

### **4️⃣ Sending Notifications**
🔹 Notify **Slack, Datadog, PagerDuty** when a dbt model runs.  

---

## **📌 Example 1: Logging Model Execution in an Audit Table**
### **🔹 Create an Audit Log Table**
```sql
CREATE TABLE IF NOT EXISTS dbt_audit_log (
    model_name STRING,
    execution_time TIMESTAMP,
    row_count INT,
    run_id STRING
);
```

### **🔹 Add a Post-hook to Log Execution**
📂 **models/customers.sql**
```sql
{{ config(
    materialized='table',
    post_hook="
        INSERT INTO dbt_audit_log (model_name, execution_time, row_count, run_id)
        SELECT '{{ this }}', CURRENT_TIMESTAMP, (SELECT COUNT(*) FROM {{ this }}), '{{ invocation_id }}'
    "
) }}

SELECT * FROM raw.customers
```
✅ **Now, every time the model runs, an audit log is saved!**  

---

## **📌 Example 2: Capturing Row Changes in Aurora/Snowflake**
### **🔹 Pre-hook to Capture Row Count Before Execution**
```sql
{{ config(
    pre_hook="
        INSERT INTO dbt_audit_log (model_name, execution_time, row_count, run_id)
        SELECT '{{ this }} - BEFORE', CURRENT_TIMESTAMP, (SELECT COUNT(*) FROM {{ this }}), '{{ invocation_id }}'
    ",
    post_hook="
        INSERT INTO dbt_audit_log (model_name, execution_time, row_count, run_id)
        SELECT '{{ this }} - AFTER', CURRENT_TIMESTAMP, (SELECT COUNT(*) FROM {{ this }}), '{{ invocation_id }}'
    "
) }}

SELECT * FROM raw.orders
```
✅ **Now, we log both the BEFORE and AFTER row counts!**  

---

## **📌 Example 3: Automating Grants in Snowflake**
### **🔹 Post-hook to Assign Permissions**
```sql
{{ config(
    materialized='table',
    post_hook="GRANT SELECT ON {{ this }} TO ROLE analyst"
) }}

SELECT * FROM raw.transactions
```
✅ **After the table is created, SELECT permissions are granted to analysts!**  

---

## **📌 Example 4: Sending Alerts on Model Completion**
### **🔹 Post-hook to Send a Slack Notification**
```sql
{{ config(
    post_hook="
        CALL notify_slack('dbt_model {{ this }} completed successfully!')
    "
) }}

SELECT * FROM raw.sales
```
✅ **Triggers a Slack message when the model runs successfully!**  

---

## **🚀 Summary of Pre/Post Hook Use Cases**
| Use Case | Hook Type | Example |
|----------|----------|---------|
| **Logging Execution Time** | `post-hook` | Save timestamps & row counts |
| **Auditing Row Changes (CDC)** | `pre-hook` & `post-hook` | Capture BEFORE/AFTER row counts |
| **Automating Grants in Snowflake** | `post-hook` | `GRANT SELECT` after table creation |
| **Sending Notifications** | `post-hook` | Call external APIs like Slack/PagerDuty |

---

## **💡 Best Practices**
✅ **Use Hooks for Logging & Automation, Not Business Logic**  
✅ **Minimize Complexity in Hooks** (Keep them lightweight)  
✅ **Use `invocation_id` for Tracking Runs**  
✅ **Avoid Side Effects in Pre-hooks (e.g., Deletes, Updates)**  

---

