
# **Tutorial: Incremental Load Design for PostgreSQL using dbt**

## **📌 What is Incremental Loading?**
Incremental loading refers to the process of only loading **new or modified data** into your database rather than reloading the entire dataset. This is particularly useful when dealing with large datasets that are updated periodically.

In **dbt**, the **incremental materialization** allows you to **add new or updated rows** to an existing table, rather than reprocessing all records.

---

## **🔹 1️⃣ Setup dbt with PostgreSQL**
Follow these steps to set up **dbt with PostgreSQL**.

### **Install dbt & PostgreSQL Adapter:**
1. Install dbt and the PostgreSQL adapter using **pip**:
   ```sh
   pip install dbt-core dbt-postgres
   ```
2. Verify installation:
   ```sh
   dbt --version
   ```

---

## **🔹 2️⃣ Configure `profiles.yml` for PostgreSQL**
In the **`profiles.yml`** file, add a connection configuration for PostgreSQL. The file is usually located in **`~/.dbt/profiles.yml`**.

### **Example `profiles.yml` for PostgreSQL:**
```yaml
my_postgres_profile:
  target: dev
  outputs:
    dev:
      type: postgres
      host: "localhost"  # Your PostgreSQL host
      user: "your_user"
      password: "your_password"
      port: 5432
      dbname: "your_database"
      schema: "public"  # or a specific schema you want to use
      threads: 4
      client_session_keep_alive: False
```

---

## **🔹 3️⃣ Set Up dbt Project**
1. Create a new **dbt project**:
   ```sh
   dbt init dbt_postgres_demo
   cd dbt_postgres_demo
   ```
2. Open **`dbt_project.yml`** and configure the project:
   ```yaml
   name: dbt_postgres_demo
   version: "1.0"
   config-version: 2
   profile: my_postgres_profile

   models:
     dbt_postgres_demo:
       staging:
         +schema: staging
         +materialized: view
       marts:
         +schema: marts
         +materialized: table
       intermediate:
         +schema: intermediate
         +materialized: ephemeral
   ```

---

## **🔹 4️⃣ Create Example Tables in PostgreSQL**
Create example **source tables** in PostgreSQL to use for our dbt models.

### **Example SQL to Create Source Tables:**

```sql
CREATE TABLE raw.customers (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    created_at TIMESTAMP
);

INSERT INTO raw.customers (first_name, last_name, email, created_at) 
VALUES 
('Alice', 'Brown', 'alice@example.com', '2024-01-01'),
('Bob', 'Smith', 'bob@example.com', '2024-01-02');
```

```sql
CREATE TABLE raw.orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10,2),
    FOREIGN KEY (customer_id) REFERENCES raw.customers(id)
);

INSERT INTO raw.orders (customer_id, order_date, total_amount) 
VALUES 
(1, '2024-02-01', 100.50),
(2, '2024-02-02', 150.75);
```

---

## **🔹 5️⃣ Define dbt Sources**
Create **`models/staging/sources.yml`** to define the source tables.

### **Example `sources.yml`:**
```yaml
version: 2

sources:
  - name: raw
    database: your_database
    schema: raw
    tables:
      - name: customers
      - name: orders
```

---

## **🔹 6️⃣ Create dbt Models with Incremental Load**

### **1️⃣ Staging Model (View)**
```sql
-- `models/staging/stg_customers.sql`
{{ config(materialized='view') }}

SELECT 
    id AS customer_id, 
    first_name, 
    last_name, 
    email, 
    created_at
FROM {{ source('raw', 'customers') }}
```

### **2️⃣ Incremental Model (Orders)**
In this example, we will implement an **incremental model** to only load new or updated orders.

```sql
-- `models/marts/incremental_orders.sql`
{{ config(
    materialized='incremental',
    unique_key='order_id'  -- Ensures no duplicate orders are inserted
) }}

SELECT 
    order_id, 
    customer_id, 
    order_date, 
    total_amount
FROM {{ ref('stg_orders') }}
{% if is_incremental() %}
  -- Only load new or updated records
  WHERE order_date > (SELECT MAX(order_date) FROM {{ this }})
{% endif %}
```
In this case:
- On **first run**, it loads all data.
- On **subsequent runs**, it only loads records with a **`order_date`** greater than the last one in the table.

---

## **🔹 7️⃣ Running the Models**

### **Run the dbt Models:**
Execute all the models to populate your staging and incremental tables:

```sh
dbt run
```

### **Run a Specific Incremental Model:**
If you want to run only the incremental load model (e.g., `incremental_orders`):

```sh
dbt run --select incremental_orders
```

### **Check the Data in PostgreSQL:**
After running the models, query the **incremental table** in PostgreSQL to verify the data:

```sql
SELECT * FROM marts.incremental_orders;
```

---

## **🔹 8️⃣ Best Practices for Incremental Loading**

### **Best Practices for Incremental Loading in dbt:**
1. **Use a `unique_key`:**  
   Always specify a `unique_key` in incremental models to ensure no duplicate records are inserted.

2. **Proper Filtering for Incremental Loads:**  
   Ensure the filter (e.g., `WHERE order_date > ...`) only selects new or changed data since the last run.

3. **Monitor Incremental Logic:**  
   Ensure your incremental logic properly handles late-arriving data or reprocessing of records when necessary.

4. **Test Incremental Loads:**  
   Regularly test incremental models with data updates to make sure no data is missed or duplicated.

---

## **🔹 9️⃣ Summary of Incremental Loading Concepts**
| **Materialization** | **Key Features** | **Use Case** |
|---------------------|------------------|--------------|
| **Incremental** | Only inserts new or changed data | Large datasets where full refresh is expensive. |

---

