 The project will follow a **three-layered approach**:  

1. **Stage Layer** → Raw tables formatted and standardized  
2. **Intermediate Layer** → Transformed tables with business logic  
3. **Mart Layer** → Aggregated KPI tables for analytics  

---

## 🔹 **Step 1: Set Up Your DBT Project**
1. **Install DBT for PostgreSQL**  
   ```sh
   pip install dbt-postgres
   ```
2. **Initialize a DBT Project**  
   ```sh
   dbt init customer_360_project
   cd customer_360_project
   ```
3. **Configure `profiles.yml`** with your Aurora connection.

---

## 🔹 **Step 2: Define DBT Models**

### **1️⃣ Stage Layer (Staging Models)**
Purpose:  
- Clean raw data (standardize column names, remove duplicates, handle nulls).  
- Use **source definitions** to track raw tables.

#### **Define Sources (`sources.yml`)**
Create `models/staging/stg_sources.yml`:
```yaml
version: 2

sources:
  - name: raw_data
    schema: raw_data
    description: "Raw customer 360 tables from Aurora DB"
    tables:
      - name: customers
      - name: orders
      - name: order_items
      - name: payments
      - name: customer_interactions
      - name: website_activity
```

#### **Staging Models**
Create `models/staging/stg_customers.sql`:
```sql
WITH source AS (
    SELECT * FROM {{ source('raw_data', 'customers') }}
),
renamed AS (
    SELECT
        customer_id,
        LOWER(email) AS email,
        first_name,
        last_name,
        phone,
        date_of_birth,
        created_at
    FROM source
)
SELECT * FROM renamed;
```

Repeat for **orders, payments, and interactions** (e.g., `stg_orders.sql`, `stg_payments.sql`).

---

### **2️⃣ Intermediate Layer (Transformations & Relationships)**
Purpose:  
- Build business rules  
- Join tables for a unified view  
- Handle missing/invalid data  

#### **Example: Customer Orders Summary (`int_customer_orders.sql`)**
```sql
WITH customers AS (
    SELECT * FROM {{ ref('stg_customers') }}
),
orders AS (
    SELECT * FROM {{ ref('stg_orders') }}
),
order_items AS (
    SELECT * FROM {{ ref('stg_order_items') }}
),
joined AS (
    SELECT 
        o.customer_id,
        COUNT(o.order_id) AS total_orders,
        SUM(oi.price * oi.quantity) AS total_spent
    FROM orders o
    LEFT JOIN order_items oi ON o.order_id = oi.order_id
    GROUP BY o.customer_id
)
SELECT 
    c.customer_id, 
    c.first_name, 
    c.last_name, 
    c.email, 
    j.total_orders, 
    j.total_spent
FROM customers c
LEFT JOIN joined j ON c.customer_id = j.customer_id;
```

---

### **3️⃣ Mart Layer (KPI Aggregations & Analytics)**
Purpose:  
- Create final **business-ready** tables for dashboards.  
- Aggregate **KPIs** (e.g., customer lifetime value, retention rates).  

#### **Example: Customer 360 View (`mart_customer_360.sql`)**
```sql
WITH customer_orders AS (
    SELECT * FROM {{ ref('int_customer_orders') }}
),
payments AS (
    SELECT 
        customer_id, 
        SUM(payment_amount) AS total_payments
    FROM {{ ref('stg_payments') }}
    WHERE payment_status = 'completed'
    GROUP BY customer_id
),
interactions AS (
    SELECT 
        customer_id, 
        COUNT(interaction_id) AS total_interactions
    FROM {{ ref('stg_customer_interactions') }}
    GROUP BY customer_id
)
SELECT 
    co.customer_id,
    co.first_name,
    co.last_name,
    co.email,
    co.total_orders,
    co.total_spent,
    p.total_payments,
    i.total_interactions
FROM customer_orders co
LEFT JOIN payments p ON co.customer_id = p.customer_id
LEFT JOIN interactions i ON co.customer_id = i.customer_id;
```

---

## 🔹 **Step 3: Run & Test Your DBT Models**
1. **Run Models**  
   ```sh
   dbt run
   ```
2. **Test Data Integrity**  
   Add `tests/schema_tests.yml`:
   ```yaml
   version: 2
   models:
     - name: stg_customers
       tests:
         - unique:
             column_name: "customer_id"
         - not_null:
             column_name: "email"

     - name: mart_customer_360
       tests:
         - not_null:
             column_name: "total_orders"
   ```
   Run Tests:  
   ```sh
   dbt test
   ```
3. **Build Docs & Explore Lineage**  
   ```sh
   dbt docs generate
   dbt docs serve
   ```

---

## 🔹 **Final Outcome**
- **Staging Layer** (`stg_`) → Cleaned raw data  
- **Intermediate Layer** (`int_`) → Transformed data for analysis  
- **Mart Layer** (`mart_`) → **Business KPIs & dashboards**

---

