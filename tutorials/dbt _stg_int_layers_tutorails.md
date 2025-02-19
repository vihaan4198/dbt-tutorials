### **Understanding Staging (`stg_`) vs. Transformation (`int_`) in dbt with Examples**  
In **dbt (Data Build Tool)**, the separation between **staging (`stg_`) and transformation (`int_`) models is a best practice for modular, scalable, and maintainable SQL-based data transformations.  

### **💡 Key Differences Between `stg_` and `int_` Models**  

| Feature         | **Staging (`stg_`)** | **Intermediate/Transformation (`int_`)** |
|---------------|-----------------|-----------------|
| **Purpose** | Cleans and standardizes raw source data | Applies business logic and transformations |
| **Data Source** | Extracted from raw tables via `sources.yml` | Uses `stg_` models as inputs |
| **Operations** | Renaming columns, casting, deduplication | Joins, aggregations, business rules |
| **Naming Convention** | `stg_<source>_<table>` | `int_<domain>_<business_logic>` |
| **Granularity** | 1:1 with raw source tables | Aggregated or transformed data |

---
**Pre-requisite (setup in Snowflake)**
Create a new schema in Snowflake (any database) and create these tables

Here’s how you can create the **source tables** in **Snowflake** along with **sample insert statements** for your dbt project.

---

## **1️⃣ Create Source Tables in Snowflake (`customers` & `orders`)**
Run the following **DDL (Data Definition Language) statements** in Snowflake:
CREATE SCHEMA DBT_RAW_DB.ECOMMERCE;

### **📍 Create `customers` Table**
```sql
CREATE TABLE DBT_RAW_DB.ECOMMERCE.customers (
    id INT PRIMARY KEY,
    first_name STRING,
    last_name STRING,
    email STRING,
    created_at TIMESTAMP
);
```

### **📍 Insert Sample Data into `customers`**
```sql
INSERT INTO DBT_RAW_DB.ECOMMERCE.customers (id, first_name, last_name, email, created_at) VALUES
(1, 'John', 'Doe', 'john.doe@example.com', '2024-02-01 10:15:00'),
(2, 'Jane', 'Smith', 'jane.smith@example.com', '2024-02-02 11:30:00'),
(3, 'Alice', 'Johnson', 'alice.johnson@example.com', '2024-02-03 09:45:00'),
(4, 'Bob', 'Brown', 'bob.brown@example.com', '2024-02-04 14:20:00'),
(5, 'Charlie', 'Davis', 'charlie.davis@example.com', '2024-02-05 13:10:00'),
(6, 'David', 'Miller', 'david.miller@example.com', '2024-02-06 16:50:00'),
(7, 'Eve', 'Wilson', 'eve.wilson@example.com', '2024-02-07 08:30:00'),
(8, 'Frank', 'Moore', 'frank.moore@example.com', '2024-02-08 18:05:00'),
(9, 'Grace', 'Lee', 'grace.lee@example.com', '2024-02-09 12:25:00'),
(10, 'Henry', 'White', 'henry.white@example.com', '2024-02-10 19:40:00');
```

---

### **📍 Create `orders` Table**
```sql
CREATE TABLE DBT_RAW_DB.ECOMMERCE.orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10,2),
    FOREIGN KEY (customer_id) REFERENCES DBT_RAW_DB.ECOMMERCE.customers(id)
);
```

### **📍 Insert Sample Data into `orders`**
```sql
INSERT INTO DBT_RAW_DB.ECOMMERCE.orders (order_id, customer_id, order_date, total_amount) VALUES
(101, 1, '2024-02-05', 100.50),
(102, 2, '2024-02-06', 250.75),
(103, 3, '2024-02-07', 175.00),
(104, 1, '2024-02-08', 80.00),
(105, 5, '2024-02-09', 90.25),
(106, 6, '2024-02-10', 300.00),
(107, 7, '2024-02-11', 450.50),
(108, 3, '2024-02-12', 120.00),
(109, 9, '2024-02-13', 200.75),
(110, 10, '2024-02-14', 150.00);
```

---

## **Next Steps**


## **1️⃣ Example: Staging (`stg_`) Model**
The **staging model** ensures that raw data from `sources.yml` is cleaned, renamed, and formatted correctly.

📍 **`models/staging/stg_customers.sql`**  
```sql
-- Cleans the raw source data from the database
WITH source_data AS (
    SELECT 
        id AS customer_id, 
        first_name, 
        last_name, 
        email, 
        created_at::timestamp AS created_date
    FROM {{ source('ecommerce', 'customers') }}  -- Fetching from `sources.yml`
)
SELECT * FROM source_data;
```

📍 **`models/staging/stg_orders.sql`**  
```sql
WITH source_data AS (
    SELECT 
        order_id, 
        customer_id, 
        order_date::date AS order_date, 
        total_amount::numeric(10,2) AS total_amount
    FROM {{ source('ecommerce', 'orders') }}
)
SELECT * FROM source_data;
```

📍 **`models/staging/sources.yml`**  
```yaml
version: 2

sources:
  - name: ecommerce 
    database: dbt_raw_db
    schema: public
    tables:
      - name: customers
      - name: orders
```
---

## **2️⃣ Example: Transformation (`int_`) Model**
The **transformation (intermediate) model** applies business logic, joins, filters, and aggregations.

📍 **`models/intermediate/int_customer_orders.sql`**  
```sql
-- Aggregate order count and total spent per customer
WITH customer_orders AS (
    SELECT 
        o.customer_id,
        COUNT(o.order_id) AS total_orders,
        SUM(o.total_amount) AS total_spent
    FROM {{ ref('stg_orders') }} o
    JOIN {{ ref('stg_customers') }} c ON o.customer_id = c.customer_id
    GROUP BY o.customer_id
)
SELECT * FROM customer_orders;
```

---

## **3️⃣ How to Run These Models in dbt**
### **Step 1: Initialize dbt**
```sh
dbt init my_dbt_project
cd my_dbt_project
```

### **Step 2: Create Staging and Transformation Models**
Place the staging models inside `models/staging/` and transformation models inside `models/intermediate/`.

### **Step 3: Run dbt**
Run the staging models:
```sh
dbt run --select stg_customers stg_orders
```
Run the transformation model:
```sh
dbt run --select int_customer_orders
```

---

## **4️⃣ Why Use This Approach?**
✅ **Modular & Scalable** – Separating `stg_` and `int_` allows easy debugging.  
✅ **Consistent Naming Convention** – Makes it easier to organize and maintain models.  
✅ **Faster Debugging** – If there's an issue, you can debug staging first before transformations.  

** What if you want to create (materialised) target objects in seperate schema** 

Follow these steps:

To **store `stg_` and `int_` objects in separate schemas** in **Snowflake using dbt**, you need to update the `dbt_project.yml` and configure schema settings in `profiles.yml`. Here's how you can do it:

---

## **1️⃣ Modify `dbt_project.yml` to Use Different Schemas**
You can dynamically configure schemas for **staging (`stg_`)** and **intermediate (`int_`)** models using **custom schema configuration**.

📍 **Update `dbt_project.yml`**  
```yaml
name: my_dbt_project
version: 1.0.0
config-version: 2

profile: my_snowflake_profile

models:
  my_dbt_project:
    staging:
      +schema: staging_schema  # Store staging models in this schema
      +materialized: view  # Typically, staging models are views
    intermediate:
      +schema: intermediate_schema  # Store intermediate models in this schema
      +materialized: table  # Usually, intermediate models are tables
```
- **`staging_schema`** → Stores **stg_** models  
- **`intermediate_schema`** → Stores **int_** models  


## **2️⃣ Modify Staging (`stg_`) and Intermediate (`int_`) Models**
You don’t need to modify the SQL files inside `models/staging/` and `models/intermediate/`, as dbt will automatically place them in the correct schemas based on `dbt_project.yml`.

However, **for clarity**, you can specify schema overrides at the model level.

📍 **Modify `stg_customers.sql`** (Optional)
```sql
{{ config(schema='staging_schema') }}  -- Explicitly set schema

WITH source_data AS (
    SELECT 
        id AS customer_id, 
        first_name, 
        last_name, 
        email, 
        created_at::timestamp AS created_date
    FROM {{ source('ecommerce', 'customers') }}
)
SELECT * FROM source_data;
```

📍 **Modify `int_customer_orders.sql`** (Optional)
```sql
{{ config(schema='intermediate_schema') }}  -- Explicitly set schema

WITH customer_orders AS (
    SELECT 
        o.customer_id,
        COUNT(o.order_id) AS total_orders,
        SUM(o.total_amount) AS total_spent
    FROM {{ ref('stg_orders') }} o
    JOIN {{ ref('stg_customers') }} c ON o.customer_id = c.customer_id
    GROUP BY o.customer_id
)
SELECT * FROM customer_orders;
```

---

## ** Run dbt & Verify Schema Placement**
Now, run your dbt models and check if they are stored in their respective schemas.

```sh
dbt run --select stg_customers stg_orders
```
This will place `stg_customers` and `stg_orders` in `staging_schema`.

```sh
dbt run --select int_customer_orders
```
This will place `int_customer_orders` in `intermediate_schema`.

To confirm, run the following query in Snowflake:
```sql
SHOW TABLES IN staging_schema;
SHOW TABLES IN intermediate_schema;
```

---

## **5️⃣ Summary of Schema Separation**
| Model Type | Schema Name | Example Models |
|------------|-------------|---------------|
| **Staging (`stg_`)** | `staging_schema` | `stg_customers`, `stg_orders` |
| **Intermediate (`int_`)** | `intermediate_schema` | `int_customer_orders` |

Now, your **staging models** and **intermediate models** are stored in separate schemas, improving organization and maintainability. 🚀

