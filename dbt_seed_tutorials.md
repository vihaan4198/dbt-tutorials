# **🚀 dbt Seed: A Complete Hands-On Tutorial**  

## **📌 What is `dbt seed`?**  
`dbt seed` allows you to **load static CSV files** into your database as tables. It is useful for:  
✅ **Reference Data** (e.g., country codes, categories)  
✅ **Small Lookup Tables** (e.g., product types, region mappings)  
✅ **Initial Test Data** for development  

---

## **📌 Step 1: Setup Your dbt Project**
If you haven’t initialized a dbt project yet, do so with:  
```sh
dbt init my_dbt_project
cd my_dbt_project
```
Ensure your **profiles.yml** is configured correctly for Snowflake, PostgreSQL, or BigQuery.

---

## **📌 Step 2: Create a `seeds/` Directory & CSV Files**
Inside your dbt project, navigate to the `seeds/` directory:  
```sh
mkdir seeds
```
Create a file **seeds/customers.csv** with the following content:  

```csv
id,name,email,signup_date
1,Alice,alice@example.com,2024-01-10
2,Bob,bob@example.com,2023-12-15
3,Charlie,charlie@example.com,2024-02-05
4,Diana,diana@example.com,2023-11-20
5,Eve,eve@example.com,2024-01-30
```
✅ This file contains **customer reference data**.

---

## **📌 Step 3: Load CSV Data into Database**
Run the following command to load the seed file:  
```sh
dbt seed
```
✅ **What Happens?**  
- dbt **creates a table** in your database using this CSV data.  
- Table name: **customers**  
- Data gets **inserted automatically**.  

🛠 **Verify the Table in Snowflake/PostgreSQL**  
```sql
SELECT * FROM customers;
```
---

## **📌 Step 4: Control Seed Behavior Using `dbt_project.yml`**
By default, `dbt seed` **overwrites** the table each time you run it.  
You can change this behavior by modifying `dbt_project.yml`:

```yaml
seeds:
  my_dbt_project:
    +schema: reference_data  # Saves in a separate schema
    +quote_columns: false  # Prevents quoting column names
    +full_refresh: false  # Prevents overwriting data
```
Now, `dbt seed` will store the data in the **reference_data** schema.

Run:  
```sh
dbt seed --full-refresh
```
✅ This **overwrites** the seed data.

---

## **📌 Step 5: Use Seed Data in dbt Models**
You can **reference seed data** in dbt models:

📍 Create a model `models/customer_summary.sql`  
```sql
SELECT 
    id, 
    name, 
    email, 
    signup_date,
    CASE 
        WHEN signup_date > '2024-01-01' THEN 'New'
        ELSE 'Existing'
    END AS customer_status
FROM {{ ref('customers') }}
```
Run:  
```sh
dbt run
```
✅ This model **uses the seed table** and categorizes customers.

---

## **📌 Step 6: Run Tests on Seed Data**
Define tests in `schema.yml`:
```yaml
version: 2

seeds:
  - name: customers
    description: "Customer reference data"
    columns:
      - name: id
        tests:
          - unique
          - not_null
      - name: email
        tests:
          - not_null
```
Run tests:  
```sh
dbt test
```
✅ Ensures **data integrity** in seed tables.

---

## **📌 Step 7: Select & Filter Specific Seeds**
Run a specific seed file:  
```sh
dbt seed --select customers
```
Run all seeds except `customers`:  
```sh
dbt seed --exclude customers
```
✅ **Use Cases:** Load only **certain datasets**.

---

## **📌 Step 8: Automate `dbt seed` in CI/CD**
Modify GitHub Actions to **load seeds before running dbt models**:
```yaml
- name: Run dbt Seed
  run: dbt seed
```
✅ Ensures **reference data is always available** in production.

---

## **📌 Best Practices for `dbt seed`**
1️⃣ **Use Small, Static Data** → Avoid large datasets.  
2️⃣ **Store in a Separate Schema** → Keep reference data organized.  
3️⃣ **Use `dbt test` on Seed Data** → Ensure data integrity.  
4️⃣ **Automate `dbt seed` in CI/CD** → Keep lookup data updated.  
5️⃣ **Use `--full-refresh` Cautiously** → Prevent unnecessary overwrites.  

---

## **🎯 Summary**
| Feature               | Command                           | Description |
|-----------------------|----------------------------------|-------------|
| Load all seed files  | `dbt seed`                       | Creates tables from CSVs |
| Run specific seed    | `dbt seed --select customers`    | Loads only the `customers` seed |
| Refresh seed data    | `dbt seed --full-refresh`        | Overwrites existing seed tables |
| Test seed integrity  | `dbt test --select customers`    | Runs data tests on seed tables |
| Use in models        | `SELECT * FROM {{ ref('customers') }}` | References seed data in dbt models |

