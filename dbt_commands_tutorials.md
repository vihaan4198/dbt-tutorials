# **🚀 dbt Hands-on Lab: Self-Paced Assignments for Mastering dbt Commands**

This **step-by-step lab** will help you **learn, practice, and master essential dbt commands** through **real-world exercises**. Each section covers a **concept, command, assignment, expected output, and best practices**.

---

## **🔹 Lab Setup: Install and Configure dbt**
### **Step 1: Install dbt (Choose Your Database)**
- **PostgreSQL**:  
  ```sh
  pip install dbt-core dbt-postgres
  ```
- **Snowflake**:  
  ```sh
  pip install dbt-core dbt-snowflake
  ```

### **Step 2: Initialize a dbt Project**
```sh
dbt init my_dbt_project
cd my_dbt_project
```
- Configure `profiles.yml` (ensure correct database connection).

---

## **📌 Lab 1: `dbt debug` - Check Your Setup**
### **Objective:** Ensure dbt is correctly configured.
### **Command:**
```sh
dbt debug
```
### **Assignment:**
1. Run `dbt debug` and capture the output.
2. Identify errors and correct them in `profiles.yml`.
3. Run `dbt debug --target prod` to verify the `prod` environment.
4. Answer:
   - What happens if credentials are incorrect?
   - How can you check where `profiles.yml` is stored?

### **Expected Output:**
- ✅ `"All checks passed!"` if everything is correct.
- ❌ `"ERROR"` if there’s a misconfiguration.

### **Best Practice:** Always run `dbt debug` before running models.

---

## **📌 Lab 2: `dbt run` - Execute Models**
### **Objective:** Run dbt models and load data.
### **Command:**
```sh
dbt run
```
### **Assignment:**
1. Open `models/example/my_first_model.sql`, update SQL:
   ```sql
   SELECT 1 AS id, 'Alice' AS name
   ```
2. Run `dbt run`.
3. Run `dbt run --select my_first_model` to execute a specific model.
4. Check data in the database:
   ```sql
   SELECT * FROM my_first_model;
   ```
5. Answer:
   - What happens when you modify the SQL and re-run dbt?

### **Expected Output:**
- A new table/view appears in the database.

### **Best Practice:** Use `--select` to run only necessary models.

---

## **📌 Lab 3: `dbt compile` - Check Generated SQL**
### **Objective:** View compiled SQL before execution.
### **Command:**
```sh
dbt compile
```
### **Assignment:**
1. Run `dbt compile`.
2. Locate compiled SQL in `target/compiled/`.
3. Modify a model and observe how compiled SQL changes.
4. Answer:
   - How does `dbt compile` help in debugging?
   - What’s the difference between `dbt compile` and `dbt run`?

### **Expected Output:**
- SQL files generated in `target/compiled/`.

### **Best Practice:** Use before `dbt run` to validate queries.

---

## **📌 Lab 4: `dbt test` - Validate Data**
### **Objective:** Ensure data quality using tests.
### **Command:**
```sh
dbt test
```
### **Assignment:**
1. Define tests in `schema.yml`:
   ```yaml
   version: 2

   models:
     - name: my_first_model
       columns:
         - name: id
           tests:
             - unique
             - not_null
   ```
2. Run `dbt test`.
3. Insert duplicate/null values and re-run `dbt test`.
4. Answer:
   - What happens when a test fails?
   - How can you fix test failures?

### **Expected Output:**
- ✅ Tests pass or ❌ test failures with error messages.

### **Best Practice:** Use tests to catch data issues early.

---

## **📌 Lab 5: `dbt seed` - Load Static Data**
### **Objective:** Load CSV data into the database.
### **Command:**
```sh
dbt seed
```
### **Assignment:**
1. Create `seeds/customers.csv`:
   ```csv
   id,name,email
   1,Alice,alice@example.com
   2,Bob,bob@example.com
   ```
2. Run `dbt seed`.
3. Query the `customers` table in the database.
4. Modify `customers.csv`, re-run `dbt seed`, and observe changes.

### **Expected Output:**
- A new table `customers` is created.

### **Best Practice:** Use for small, static reference tables.

---

## **📌 Lab 6: `dbt snapshot` - Track Data Changes**
### **Objective:** Capture historical changes in data.
### **Command:**
```sh
dbt snapshot
```
### **Assignment:**
1. Create a snapshot model `snapshots/customer_snapshot.sql`:
   ```sql
   {% snapshot customer_snapshot %}
   {{ config(
       unique_key='id',
       strategy='timestamp',
       updated_at='updated_at'
   ) }}
   SELECT * FROM customers
   {% endsnapshot %}
   ```
2. Run `dbt snapshot`.
3. Update a row in `customers` and re-run `dbt snapshot`.
4. Answer:
   - How does dbt snapshot track changes?

### **Expected Output:**
- A new snapshot table storing historical versions of data.

### **Best Practice:** Use for tracking slowly changing dimensions (SCD).

---

## **📌 Lab 7: `dbt clean` - Remove Temporary Files**
### **Objective:** Clear compiled files and dependencies.
### **Command:**
```sh
dbt clean
```
### **Assignment:**
1. Run `dbt clean`.
2. Check if `target/` and `dbt_packages/` are removed.

### **Expected Output:**
- Temporary files deleted.

### **Best Practice:** Run before deploying to avoid outdated artifacts.

---

## **📌 Lab 8: `dbt deps` - Install Packages**
### **Objective:** Use dbt packages.
### **Command:**
```sh
dbt deps
```
### **Assignment:**
1. Add this to `packages.yml`:
   ```yaml
   packages:
     - package: dbt-labs/dbt_utils
       version: 0.8.0
   ```
2. Run `dbt deps`.
3. Use `dbt_utils.safe_cast()` in a model.
4. Answer:
   - What happens if the package version is incorrect?

### **Expected Output:**
- `dbt_utils` functions available in models.

### **Best Practice:** Use packages to **extend dbt functionality**.

---

## **📌 Lab 9: `dbt list` - Explore Models**
### **Objective:** List all models in a dbt project.
### **Command:**
```sh
dbt list
```
### **Assignment:**
1. Run `dbt list`.
2. Run `dbt list --resource-type model`.
3. Answer:
   - How do you list only staging models?

### **Expected Output:**
- A list of all models in the project.

### **Best Practice:** Use to **understand project structure**.

---

## **📌 Lab 10: `dbt docs generate & serve` - Interactive Documentation**
### **Objective:** Generate and view project documentation.
### **Command:**
```sh
dbt docs generate
dbt docs serve
```
### **Assignment:**
1. Run `dbt docs generate`.
2. Open `localhost:8080` in a browser.
3. Answer:
   - How does documentation improve collaboration?

### **Expected Output:**
- A **web-based dbt documentation UI**.

### **Best Practice:** Use for **better project visibility**.

---

## **🚀 Final Challenge: Create a Complete dbt Workflow**
### **Objective:** Combine all commands to build a **full dbt pipeline**:
1. `dbt debug`
2. `dbt seed`
3. `dbt run`
4. `dbt test`
5. `dbt snapshot`
6. `dbt docs generate & serve`

### **Expected Output:**
- A working **data pipeline** with staging, transformation, and historical tracking.

---
