# **🛠️ Hands-On Tutorial: Implementing Pre/Post Hooks for Logging and Auditing in dbt with Snowflake/Aurora**

## **📌 Objective**
- Learn how to use **Pre-hooks** and **Post-hooks** in dbt for logging and auditing.
- Create an **audit table** in **Snowflake** or **Aurora** to capture **model execution metadata** like row counts, execution times, etc.
- Implement **Pre-hooks** to capture row counts **before** execution and **Post-hooks** to capture them **after** execution.
- Automate logging actions using dbt hooks.

---

### **🔹 Prerequisites**
1. **dbt** (Core or Cloud)
2. **Snowflake/Aurora (PostgreSQL/MySQL)** setup and access credentials
3. dbt profile configured for Snowflake/Aurora connection

#### **Example `profiles.yml` (Snowflake Configuration)**  
```yaml
my_snowflake_dbt:
  target: dev
  outputs:
    dev:
      type: snowflake
      account: xyz.snowflakecomputing.com
      user: your_user
      password: your_password
      role: your_role
      warehouse: your_warehouse
      database: your_database
      schema: raw
      threads: 4
```

#### **Example `profiles.yml` (Aurora PostgreSQL Configuration)**  
```yaml
my_aurora_dbt:
  target: dev
  outputs:
    dev:
      type: postgres
      host: aurora-cluster.endpoint.rds.amazonaws.com
      user: your_user
      password: your_password
      port: 5432
      dbname: your_db
      schema: raw
      threads: 4
```

---

## **📌 Step 1: Create the Audit Log Table**

First, let's create an **audit log table** in Snowflake or Aurora. This table will store metadata about the dbt model execution, including the model name, row count, execution timestamp, and run ID.

#### **Create the Audit Log Table in Snowflake:**
```sql
CREATE OR REPLACE TABLE dbt_audit_log (
    model_name STRING,
    execution_time TIMESTAMP,
    row_count INT,
    run_id STRING
);
```

#### **Create the Audit Log Table in Aurora (PostgreSQL Example):**
```sql
CREATE TABLE dbt_audit_log (
    model_name VARCHAR(255),
    execution_time TIMESTAMP,
    row_count INT,
    run_id VARCHAR(255)
);
```

---

## **📌 Step 2: Create a Simple dbt Model**

Let’s create a simple **staging model** (`stg_customers.sql`) to fetch customer data from the source and perform logging actions using **Pre-hook** and **Post-hook**.

### **📂 models/staging/stg_customers.sql**
```sql
{{ config(
    materialized='table',
    pre_hook="""
        INSERT INTO dbt_audit_log (model_name, execution_time, row_count, run_id)
        SELECT '{{ this }} - BEFORE', CURRENT_TIMESTAMP, (SELECT COUNT(*) FROM {{ this }}), '{{ invocation_id }}';
    """,
    post_hook="""
        INSERT INTO dbt_audit_log (model_name, execution_time, row_count, run_id)
        SELECT '{{ this }} - AFTER', CURRENT_TIMESTAMP, (SELECT COUNT(*) FROM {{ this }}), '{{ invocation_id }}';
    """
) }}

SELECT id, name, email, last_updated
FROM raw.customers
```

### **Explanation of Hooks:**
- **Pre-hook:** Runs before dbt executes the model. It captures the row count of the table and logs the `BEFORE` state into the `dbt_audit_log`.
- **Post-hook:** Runs after the dbt model has been executed. It captures the row count of the model and logs the `AFTER` state into the same audit table.

- **`invocation_id`** helps track this execution across dbt runs.

---

## **📌 Step 3: Run dbt Model and Verify Logs**

Once the model is defined, let's run the dbt model and verify if the **Pre-hook** and **Post-hook** are functioning as expected.

1. **Run the dbt model:**
   ```sh
   dbt run --select stg_customers
   ```

2. **Verify the Audit Log Table:**
   - **In Snowflake:**
     ```sql
     SELECT * FROM dbt_audit_log;
     ```

   - **In Aurora (PostgreSQL):**
     ```sql
     SELECT * FROM dbt_audit_log;
     ```

#### **Expected Output:**
| model_name                | execution_time         | row_count | run_id           |
|---------------------------|------------------------|-----------|------------------|
| `stg_customers - BEFORE`   | 2025-02-19 15:00:00    | 1000      | <run_id_value>   |
| `stg_customers - AFTER`    | 2025-02-19 15:05:00    | 1000      | <run_id_value>   |

---

## **📌 Step 4: Modify Pre/Post-hooks for Capturing Specific Changes**

You can capture specific changes, such as **newly added rows** or **updates**, by modifying the hooks. For instance, you can compare the **before** and **after row counts** or capture the **changes** based on a timestamp or change data tracking column.

### **📂 Example: Track Changes with Timestamp**

Let’s enhance the Pre/Post hooks to specifically capture changes using a timestamp column.

1. **Change the Table Model to Include Updated Timestamps:**

    📂 **models/staging/stg_customers_with_changes.sql**
    ```sql
    {{ config(
        materialized='table',
        pre_hook="""
            INSERT INTO dbt_audit_log (model_name, execution_time, row_count, run_id)
            SELECT '{{ this }} - BEFORE', CURRENT_TIMESTAMP, (SELECT COUNT(*) FROM {{ this }}), '{{ invocation_id }}';
        """,
        post_hook="""
            INSERT INTO dbt_audit_log (model_name, execution_time, row_count, run_id)
            SELECT '{{ this }} - AFTER', CURRENT_TIMESTAMP, (SELECT COUNT(*) FROM {{ this }}), '{{ invocation_id }}';
        """
    ) }}

    SELECT id, name, email, last_updated
    FROM raw.customers
    WHERE last_updated > (SELECT COALESCE(MAX(last_updated), '1900-01-01') FROM {{ this }})
    ```

#### **Explanation of Updates:**
- **Post-hook** logs the row count **after** processing new/updated records.
- **Pre-hook** captures the **before** row count, helping track incremental changes.

---

## **📌 Step 5: Automate the Auditing and Schedule Runs (CI/CD)**

You can automate the execution of your dbt models using **dbt Cloud** or **CI/CD tools** (like GitHub Actions, GitLab CI). 

#### **In dbt Cloud:**  
1. **Set up scheduled runs** for the dbt models to track changes at regular intervals (e.g., every hour).  
2. **Monitor the audit log** to ensure that the pre/post hooks are being executed properly and new records are being captured.

---

## **📌 Step 6: Review and Adjust Hook Logic for Scaling**

### **🔹 Best Practices:**
1. **Avoid heavy logic in pre/post hooks**: Hooks should be lightweight and perform non-complex SQL operations. They should focus on logging and auditing tasks.
2. **Use `invocation_id`**: For tracking executions and differentiating between different runs.
3. **Monitor hook performance**: In high-volume systems, excessive pre/post hooks can slow down performance. Use them carefully.

---

## **🔥 Summary**

| Step | Description |
|------|-------------|
| **Step 1:** Create an Audit Table | Create an audit table to store metadata about dbt model executions (row counts, timestamps). |
| **Step 2:** Add Pre/Post-hooks | Implement pre/post hooks to log execution metadata before and after running dbt models. |
| **Step 3:** Run dbt Models | Execute dbt models and verify audit table logging. |
| **Step 4:** Enhance Pre/Post-hooks | Enhance hooks to capture more specific changes or modifications (e.g., based on timestamps). |
| **Step 5:** Automate and Scale | Automate your dbt runs using scheduling and monitor your audit logs over time. |

---
