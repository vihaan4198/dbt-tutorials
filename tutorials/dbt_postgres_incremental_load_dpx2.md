
**DBT Customer 360 project part 2** 


1. **DBT Snapshots** → Track historical changes for slowly changing dimensions (SCD Type 2).
2. **DBT Incremental Models** → Optimize performance by updating only new or changed data.

---

## **🔹 Step 1: Implement DBT Snapshots for Historical Tracking**
### **Why Snapshots?**
Snapshots **track changes** in your data over time. This is useful for:
- **Customer Profile Changes** (e.g., name, email updates)
- **Order Status Changes** (e.g., pending → shipped → delivered)
- **Payments Status Changes** (e.g., pending → completed → refunded)

### **Create Snapshot Configuration**
1. Create a `snapshots/` folder in your DBT project:
   ```sh
   mkdir snapshots
   ```

2. Define a **customer snapshot** (`snapshots/customer_snapshot.sql`):
```sql
{% snapshot customer_snapshot %}

{{ config(
    target_schema='snapshots',
    unique_key='customer_id',
    strategy='timestamp',
    updated_at='created_at'  -- Change this to 'updated_at' if available
) }}

SELECT 
    customer_id, 
    first_name, 
    last_name, 
    email, 
    phone, 
    created_at
FROM {{ source('raw_data', 'customers') }}

{% endsnapshot %}
```

3. Define a **order status snapshot** (`snapshots/order_snapshot.sql`):
```sql
{% snapshot order_snapshot %}

{{ config(
    target_schema='snapshots',
    unique_key='order_id',
    strategy='timestamp',
    updated_at='order_date'
) }}

SELECT 
    order_id, 
    customer_id, 
    status, 
    order_date, 
    total_amount
FROM {{ source('raw_data', 'orders') }}

{% endsnapshot %}
```

4. Run the snapshot command:
   ```sh
   dbt snapshot
   ```
   This will create **historical records** for each update.

---

## **🔹 Step 2: Optimize Performance with DBT Incremental Models**
### **Why Incremental Models?**
- Instead of reprocessing the **entire dataset**, only **new or updated records** are processed.
- Reduces **query execution time**.
- Useful for **large tables** like orders, payments, and interactions.

### **Example: Incremental Model for Orders**
Create `models/intermediate/int_customer_orders.sql`:
```sql
{{ config(
    materialized='incremental',
    unique_key='order_id'
) }}

WITH new_orders AS (
    SELECT 
        order_id,
        customer_id,
        order_date,
        status,
        total_amount
    FROM {{ ref('stg_orders') }}
    
    {% if is_incremental() %}
    -- Only fetch new or updated records
    WHERE order_date > (SELECT MAX(order_date) FROM {{ this }})
    {% endif %}
)

SELECT * FROM new_orders;
```

**Key Points:**
- The `is_incremental()` function ensures **only new or changed orders** are processed.
- The `WHERE order_date > (SELECT MAX(order_date) FROM {{ this }})` condition ensures that **only new orders** are fetched.

### **Example: Incremental Model for Payments**
Create `models/intermediate/int_payments.sql`:
```sql
{{ config(
    materialized='incremental',
    unique_key='payment_id'
) }}

WITH new_payments AS (
    SELECT 
        payment_id,
        customer_id,
        order_id,
        payment_method,
        payment_status,
        payment_amount,
        payment_date
    FROM {{ ref('stg_payments') }}

    {% if is_incremental() %}
    WHERE payment_date > (SELECT MAX(payment_date) FROM {{ this }})
    {% endif %}
)

SELECT * FROM new_payments;
```

### **Run Incremental Models**
1. **First-time Run (full refresh)**
   ```sh
   dbt run --full-refresh
   ```
2. **Subsequent Runs (only new data)**
   ```sh
   dbt run
   ```

---

## **🔹 Step 3: Automate & Schedule DBT Runs**
1. **Schedule DBT Snapshots & Incremental Models in Airflow or DBT Cloud**
   - Run **snapshots** daily:  
     ```sh
     dbt snapshot
     ```
   - Run **incremental models** every few hours:  
     ```sh
     dbt run
     ```
   
2. **Set up DBT Jobs in DBT Cloud**
   - Create a **job** for **snapshots** (run daily at midnight).
   - Create a **job** for **incremental models** (run every 3-6 hours).

---

## **🔹 Step 4: Expose Data to BI Tools (Looker, Tableau)**
1. **Create Exposures for BI**
   Add `models/mart/exposures.yml`:
   ```yaml
   version: 2

   exposures:
     - name: customer_360_dashboard
       type: dashboard
       maturity: high
       owner:
         name: Data Team
         email: data_team@example.com
       depends_on:
         - ref('mart_customer_360')
   ```

2. **Run the following to update docs & exposures**
   ```sh
   dbt docs generate
   dbt docs serve
   ```

---

## **🚀 Final Summary**
✅ **DBT Snapshots** → Track historical changes for **customer profiles & order status**  
✅ **DBT Incremental Models** → Improve performance for **orders, payments, interactions**  
✅ **Automation & BI Integration** → Schedule jobs & expose data for **Tableau, Looker**  
