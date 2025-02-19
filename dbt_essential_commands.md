# **🚀 Essential dbt Commands: Use Cases, Examples, and Best Practices**  

This guide covers the **most important dbt commands** along with **practical use cases, examples, and special cases** to help you efficiently work with dbt in your **data transformation projects**.

---

## **🔹 1️⃣ `dbt debug`**
### **✔️ Use Case**  
Checks if your dbt project is correctly configured, including database connection and credentials.

### **📌 Basic Usage**
```sh
dbt debug
```
✅ **Expected Output:**  
- `"All checks passed!"` (If everything is set up correctly)  
- Error messages if there's a misconfiguration (e.g., incorrect credentials, missing `profiles.yml`).

### **📌 Special Case: Debug with Specific Target**
```sh
dbt debug --target prod
```
✅ **Use Case:** If you have multiple targets (e.g., `dev`, `prod`), this ensures you're testing the correct environment.

### **🔹 Best Practices**
- Run `dbt debug` **before running dbt models** to ensure everything is set up correctly.
- Use `--config-dir` to check where dbt is looking for `profiles.yml`.

---

## **🔹 2️⃣ `dbt run`**
### **✔️ Use Case**  
Executes all models in your dbt project and materializes them in the database.

### **📌 Basic Usage**
```sh
dbt run
```
✅ Runs all models and creates **tables, views, and incremental models** based on their configurations.

### **📌 Special Case: Run a Specific Model**
```sh
dbt run --select mart_sales_data
```
✅ **Use Case:** When you only want to execute one model.

### **📌 Special Case: Run All Models in a Specific Folder**
```sh
dbt run --select models/staging/
```
✅ Runs only the models inside `models/staging/`.

### **📌 Special Case: Run Incremental Models Only**
```sh
dbt run --select state:modified+ --defer
```
✅ **Use Case:** Runs only modified models.

### **🔹 Best Practices**
- Always test in **dev** before running in **prod**.
- If using **incremental models**, ensure the `unique_key` is correctly set to avoid duplicates.

---

## **🔹 3️⃣ `dbt compile`**
### **✔️ Use Case**  
Compiles SQL queries without executing them, allowing you to inspect the generated SQL.

### **📌 Basic Usage**
```sh
dbt compile
```
✅ Stores the compiled SQL in `target/compiled/`.

### **📌 Special Case: Compile a Specific Model**
```sh
dbt compile --select stg_orders
```
✅ **Use Case:** When you want to check the compiled SQL of a specific model.

### **🔹 Best Practices**
- Use `dbt compile` before running models to **validate syntax** without modifying data.
- Review compiled queries in `target/compiled/` to debug complex transformations.

---

## **🔹 4️⃣ `dbt test`**
### **✔️ Use Case**  
Runs tests on your models to check for **null values, uniqueness, and data integrity**.

### **📌 Basic Usage**
```sh
dbt test
```
✅ Runs all tests defined in `tests/` and `schema.yml`.

### **📌 Special Case: Test a Single Model**
```sh
dbt test --select stg_customers
```
✅ Only runs tests for the `stg_customers` model.

### **📌 Example: Add Tests to `schema.yml`**
```yaml
version: 2

models:
  - name: stg_customers
    columns:
      - name: customer_id
        tests:
          - unique
          - not_null
```
✅ **Best Practice:** Define tests in `schema.yml` to **automate data quality checks**.

---

## **🔹 5️⃣ `dbt seed`**
### **✔️ Use Case**  
Loads CSV files from `seeds/` into the database as tables.

### **📌 Example**
1️⃣ **Create a CSV file** in `seeds/` (e.g., `customers.csv`):
```csv
id,first_name,last_name,email
1,Alice,Brown,alice@example.com
2,Bob,Smith,bob@example.com
```
2️⃣ **Run the seed command**:
```sh
dbt seed
```
✅ The CSV file is loaded into the database as a table.

### **📌 Special Case: Run a Specific Seed File**
```sh
dbt seed --select customers
```
✅ Loads only `customers.csv`.

### **🔹 Best Practices**
- Use **seeds for lookup tables** (e.g., mapping tables, country codes).
- Keep seed files **small** to avoid performance issues.

---

## **🔹 6️⃣ `dbt snapshot`**
### **✔️ Use Case**  
Tracks historical changes in data for Slowly Changing Dimensions (SCD Type 2).

### **📌 Example: Define a Snapshot Model**
📍 Create `snapshots/customer_snapshots.sql`:
```sql
{% snapshot customer_snapshot %}

{{ config(
    target_schema='snapshots',
    unique_key='customer_id',
    strategy='timestamp',
    updated_at='updated_at'
) }}

SELECT * FROM {{ source('raw', 'customers') }}

{% endsnapshot %}
```
Run the snapshot command:
```sh
dbt snapshot
```
✅ Captures historical changes in `snapshots.customer_snapshots`.

---

## **🔹 7️⃣ `dbt clean`**
### **✔️ Use Case**  
Deletes temporary files (`target/` and `dbt_packages/`).

### **📌 Basic Usage**
```sh
dbt clean
```
✅ Clears compiled SQL and temporary cache files.

### **🔹 Best Practices**
- Run `dbt clean` before deploying to **remove outdated files**.

---

## **🔹 8️⃣ `dbt deps`**
### **✔️ Use Case**  
Installs **dbt packages** from `packages.yml` (like dependencies in Python).

### **📌 Example**
📍 Define dependencies in `packages.yml`:
```yaml
packages:
  - package: dbt-labs/dbt_utils
    version: 0.8.0
```
Run:
```sh
dbt deps
```
✅ Installs `dbt_utils` package.

---

## **🔹 9️⃣ `dbt list`**
### **✔️ Use Case**  
Lists models, tests, and sources in a dbt project.

### **📌 Example**
```sh
dbt list
```
✅ Returns a list of all models.

### **📌 Special Case: List Only Staging Models**
```sh
dbt list --resource-type model --selector staging
```
✅ Lists all models in the `staging` folder.

---

## **🔹 🔟 `dbt docs generate & serve`**
### **✔️ Use Case**  
Generates and serves **interactive documentation** for your dbt project.

### **📌 Generate Documentation**
```sh
dbt docs generate
```
✅ Creates `index.html` inside the `target/` folder.

### **📌 Serve Documentation Locally**
```sh
dbt docs serve
```
✅ Opens a **web-based interactive dbt documentation UI**.

---

## **🎯 Special Case Scenarios & Best Practices**
| Scenario | Command | Best Practice |
|----------|--------|--------------|
| **Debugging Connection Issues** | `dbt debug --target prod` | Run `dbt debug` before `dbt run` to avoid connection issues. |
| **Running Models in Parallel** | `dbt run --threads 8` | Use higher thread count for faster execution. |
| **Refreshing Incremental Models** | `dbt run --full-refresh --select my_model` | Use `--full-refresh` to rebuild incremental tables. |
| **Running Tests for Modified Models** | `dbt test --select state:modified+` | Run tests only for modified models in a CI/CD pipeline. |

---

## **✅ Final Thoughts**
- **Use `dbt run` with `--select`** to run only what’s necessary.
- **Enable tests (`dbt test`)** for **data integrity**.
- **Use `dbt docs generate`** to maintain good documentation.
- **Use `dbt snapshot`** to track historical changes.

---
