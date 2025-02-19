# **🚀 Hands-on dbt Materialization Demo in Snowflake**  

This step-by-step guide will help you **set up dbt Core on Windows**, connect it to **Snowflake**, and implement different **materializations (Table, View, Incremental, Ephemeral)** in a **real dbt project**.

---

# **🔹 Step 1: Install & Setup dbt Core on Windows**
## **📌 Install Python & dbt Core**
1️⃣ Install **Python** (if not installed):  
   - Download from [Python.org](https://www.python.org/downloads/)  
   - Install with **"Add to PATH" enabled**  
2️⃣ Install **dbt Core & dbt Snowflake Adapter**  
   ```sh
   pip install dbt-core dbt-snowflake
   ```
3️⃣ Verify installation:
   ```sh
   dbt --version
   ```
   ✅ Output should show installed versions of dbt and dbt-snowflake.

---

# **🔹 Step 2: Connect dbt to Snowflake**
## **📌 Configure `profiles.yml`**
📍 Create a **profile configuration file** at:
```sh
C:\Users\YourUser\.dbt\profiles.yml
```
Add the following config:
```yaml
my_snowflake_profile:
  target: dev
  outputs:
    dev:
      type: snowflake
      account: "your_snowflake_account"  # Example: abc123.us-east-1
      user: "your_username"
      password: "your_password"
      role: "YOUR_ROLE"
      warehouse: "YOUR_WAREHOUSE"
      database: "YOUR_DATABASE"
      schema: "dbt_dev"
      threads: 4
      client_session_keep_alive: False
```
📌 **Test connection**  
```sh
dbt debug
```
✅ Should return **"All checks passed!"**

---

# **🔹 Step 3: Create a dbt Project**
Run:
```sh
dbt init dbt_snowflake_demo
cd dbt_snowflake_demo
```
Modify **`dbt_project.yml`**:
```yaml
name: dbt_snowflake_demo
version: "1.0"
config-version: 2
profile: my_snowflake_profile

models:
  dbt_snowflake_demo:
    staging:
      +schema: staging
      +materialized: view  # Default for staging
    marts:
      +schema: marts
      +materialized: table  # Default for marts
    intermediate:
      +schema: intermediate
      +materialized: ephemeral  # Default for intermediate models
```

---

# **🔹 Step 4: Create Source Tables in Snowflake**
Run these SQL commands in Snowflake:

```sql
CREATE OR REPLACE TABLE raw.public.customers (
    id INT PRIMARY KEY,
    first_name STRING,
    last_name STRING,
    email STRING,
    created_at TIMESTAMP
);

INSERT INTO raw.public.customers VALUES
(1, 'Alice', 'Brown', 'alice@example.com', '2024-02-01 10:00:00'),
(2, 'Bob', 'Smith', 'bob@example.com', '2024-02-02 11:00:00');
```

```sql
CREATE OR REPLACE TABLE raw.public.orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10,2),
    FOREIGN KEY (customer_id) REFERENCES raw.public.customers(id)
);

INSERT INTO raw.public.orders VALUES
(101, 1, '2024-02-05', 100.50),
(102, 2, '2024-02-06', 250.75);
```

---

# **🔹 Step 5: Define dbt Sources**
📍 Create **`models/staging/sources.yml`**  
```yaml
version: 2

sources:
  - name: raw
    database: raw
    schema: public
    tables:
      - name: customers
      - name: orders
```

---

# **🔹 Step 6: Create dbt Models with Different Materializations**
## **1️⃣ Staging Model (View)**
📍 **`models/staging/stg_customers.sql`**
```sql
{{ config(materialized='view') }}

SELECT 
    id AS customer_id, 
    first_name, 
    last_name, 
    email, 
    created_at
FROM {{ source('raw', 'customers') }}
```
✅ Creates a **VIEW** in **staging schema**.

## **2️⃣ Intermediate Model (Ephemeral)**
📍 **`models/intermediate/int_customer_orders.sql`**
```sql
{{ config(materialized='ephemeral') }}

SELECT 
    o.customer_id,
    COUNT(o.order_id) AS total_orders,
    SUM(o.total_amount) AS total_spent
FROM {{ ref('stg_customers') }} c
JOIN {{ ref('stg_orders') }} o ON c.customer_id = o.customer_id
GROUP BY o.customer_id
```
✅ **No table or view is created**, query is **embedded in downstream models**.

## **3️⃣ Marts Model (Table)**
📍 **`models/marts/mart_customer_orders.sql`**
```sql
{{ config(materialized='table') }}

SELECT 
    customer_id, 
    total_orders, 
    total_spent
FROM {{ ref('int_customer_orders') }}
```
✅ **Creates a physical table** in **marts schema**.

## **4️⃣ Incremental Model (Only New Data)**
📍 **`models/marts/mart_new_orders.sql`**
```sql
{{ config(
    materialized='incremental',
    unique_key='order_id'
) }}

SELECT 
    order_id, 
    customer_id, 
    order_date, 
    total_amount
FROM {{ ref('stg_orders') }}
{% if is_incremental() %}
WHERE order_date > (SELECT MAX(order_date) FROM {{ this }})  -- Only new data
{% endif %}
```
✅ **First run:** Creates full table  
✅ **Subsequent runs:** Only inserts new records  

---

# **🔹 Step 7: Run and Test Models**
## **📌 Run All Models**
```sh
dbt run
```
✅ Check Snowflake:
```sql
SHOW TABLES IN staging;
SHOW TABLES IN marts;
```
## **📌 Run Specific Models**
```sh
dbt run --select stg_customers
dbt run --select mart_customer_orders
```

## **📌 Check Compiled SQL**
```sh
dbt compile --select mart_customer_orders
```

## **📌 Test Models**
```sh
dbt test --select stg_customers
```
Add **tests** in `sources.yml`:
```yaml
columns:
  - name: customer_id
    tests:
      - unique
      - not_null
```

---

# **🔹 Summary**
| **Materialization** | **Schema** | **Example Model** | **Stored in DB?** |
|------------------|---------|----------------|-------------|
| **View** | staging | `stg_customers` | ❌ No (always recomputed) |
| **Ephemeral** | intermediate | `int_customer_orders` | ❌ No (only embedded) |
| **Table** | marts | `mart_customer_orders` | ✅ Yes (physical table) |
| **Incremental** | marts | `mart_new_orders` | ✅ Yes (only new records added) |

---
