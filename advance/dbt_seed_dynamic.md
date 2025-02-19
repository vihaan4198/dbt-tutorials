# **🚀 Advanced Use Cases for `dbt seed`**
`dbt seed` is more than just loading CSV files into a database! Let's explore **advanced use cases** like **dynamic seeding, incremental seed updates, cross-database seeding, security controls, and performance optimization.** 🚀  

---

## **📌 1. Dynamic Seed File Loading Using Jinja**
Instead of hardcoding values in a CSV, use **Jinja templating** in dbt models to dynamically reference seed data.  

### **🔹 Example: Dynamic Date-Based Filtering**
**📂 models/recent_customers.sql**
```sql
SELECT * 
FROM {{ ref('customers') }}
WHERE signup_date >= dateadd('day', -30, current_date)
```
✅ This model **automatically filters** for customers who signed up in the last 30 days, using the `customers` seed table.

---

## **📌 2. Incremental Updates for Seed Data**
By default, `dbt seed` **overwrites** the seed table each time. But what if you want to only add **new records**?  

### **🔹 Solution: Load Seeds into a Staging Table & Merge Data**
Modify `dbt_project.yml` to store seeds in a **staging schema**:  
```yaml
seeds:
  my_dbt_project:
    +schema: staging_seeds
```
📂 **models/staged_customers.sql**
```sql
WITH latest_seed AS (
    SELECT * FROM {{ ref('customers') }}
)
SELECT c.*
FROM my_database.production.customers c
LEFT JOIN latest_seed s
ON c.id = s.id
WHERE s.id IS NULL
```
📌 **Execution Strategy:**  
1️⃣ `dbt seed` loads new seed data into `staging_seeds.customers`.  
2️⃣ The model filters out **existing** records from production.  
3️⃣ Only **new records** are processed and inserted.  

✅ **Use Case:** Helps prevent accidental **data duplication**.

---

## **📌 3. Multi-Database Seeding (Cross-Environment Seeding)**
Need to load the same seed data into **multiple databases** (e.g., Snowflake & Postgres)?  
📌 Define separate **seed configurations** in `dbt_project.yml`:  

```yaml
seeds:
  my_dbt_project:
    dev:
      +database: dev_database
    prod:
      +database: prod_database
```
Run the command:
```sh
dbt seed --target dev
```
✅ This loads the seed data into the `dev_database`, and switching to `prod` follows the same process.

---

## **📌 4. Secure Sensitive Seed Data**
If your seed files contain **sensitive information** (e.g., **customer emails, pricing data**), consider **masking or encrypting** them.

📂 **seeds/sensitive_data.csv**
```csv
id,name,email,salary
1,Alice,alice@example.com,50000
2,Bob,bob@example.com,60000
```
Modify `dbt_project.yml` to mask the `salary` column:
```yaml
seeds:
  my_dbt_project:
    sensitive_data:
      +column_types:
        salary: VARCHAR(10)
```
📂 **models/masked_sensitive_data.sql**
```sql
SELECT 
    id, 
    name, 
    email, 
    '*****' AS salary 
FROM {{ ref('sensitive_data') }}
```
✅ **Benefit:** Prevents **exposing sensitive data** in downstream queries.

---

## **📌 5. Loading Large Seed Files Efficiently**
If you have **millions of rows** in a CSV, `dbt seed` might **slow down performance**.  
**📌 Solution: Split Large Files & Use Parallel Processing**  

### **🔹 1. Split Large CSVs into Smaller Chunks**
Instead of one large `customers.csv` file, split it into:  
- `customers_1.csv` (1M rows)  
- `customers_2.csv` (1M rows)  

Modify `dbt_project.yml`:
```yaml
seeds:
  my_dbt_project:
    customers_1:
      +schema: seed_partitions
    customers_2:
      +schema: seed_partitions
```
📂 **models/final_customers.sql**
```sql
SELECT * FROM {{ ref('customers_1') }}
UNION ALL
SELECT * FROM {{ ref('customers_2') }}
```
✅ **Benefit:** Increases **performance & scalability**.

### **🔹 2. Use `dbt seed --threads` for Parallel Execution**
```sh
dbt seed --threads 4
```
✅ **Benefit:** Speeds up **large CSV file processing**.

---

## **📌 6. Automate Seed Updates in CI/CD**
Modify GitHub Actions to run `dbt seed` only if the seed files have changed:
```yaml
- name: Check for Seed File Changes
  run: |
    git diff --name-only HEAD^ HEAD | grep 'seeds/' || echo "No changes in seeds"
- name: Run dbt Seed (If Files Changed)
  if: success()
  run: dbt seed
```
✅ **Use Case:** Saves **execution time** by skipping `dbt seed` if no changes exist.

---

## **📌 7. Use `dbt seed` for Custom Audit Logs**
Instead of manually updating audit tables, use `dbt seed` to **store audit reference data**.

📂 **seeds/audit_events.csv**
```csv
event_id,description,criticality
1,User Login,Low
2,Data Export,High
3,Schema Change,Critical
```
📂 **models/audit_tracking.sql**
```sql
SELECT 
    user_id, 
    event_time, 
    audit.description, 
    audit.criticality
FROM my_database.activity_logs logs
LEFT JOIN {{ ref('audit_events') }} audit
ON logs.event_id = audit.event_id
```
✅ **Use Case:** Enables **automatic alerting** on **high-risk actions**.

---

## **📌 8. Extend `dbt seed` with Custom Python Scripts**
If you need **dynamic seed file generation**, use a Python script:

📂 **scripts/generate_seeds.py**
```python
import pandas as pd

data = {
    "id": range(1, 101),
    "name": [f"Customer_{i}" for i in range(1, 101)],
    "signup_date": pd.date_range("2024-01-01", periods=100).strftime("%Y-%m-%d"),
}

df = pd.DataFrame(data)
df.to_csv("seeds/generated_customers.csv", index=False)
```
Run before `dbt seed`:
```sh
python scripts/generate_seeds.py
dbt seed
```
✅ **Benefit:** Generates **dynamic lookup tables** instead of manually updating CSVs.

---

## **📌 Summary: Advanced `dbt seed` Techniques**
| Feature | Technique | Command |
|---------|-----------|---------|
| 🔄 **Dynamic Filtering** | Use Jinja in models | `{{ ref('customers') }}` |
| 🔄 **Incremental Seeding** | Merge new records instead of full refresh | `dbt seed --full-refresh` (Use with caution) |
| 🏢 **Multi-Database Seeding** | Store in different environments | `dbt seed --target dev` |
| 🔒 **Secure Data** | Mask sensitive columns | Use `column_types` in `dbt_project.yml` |
| 🚀 **Performance Optimization** | Split large files & parallel execution | `dbt seed --threads 4` |
| 🤖 **CI/CD Automation** | Run seed only if files changed | GitHub Actions workflow |
| 📜 **Audit Logs** | Store reference audit data | `SELECT * FROM {{ ref('audit_events') }}` |
| 🐍 **Python-Based Seeding** | Generate dynamic CSVs | `python scripts/generate_seeds.py` |

---

## **🚀 Final Thoughts**
With these **advanced techniques**, you can:  
✅ Optimize `dbt seed` for **performance & scalability**  
✅ Automate **incremental updates** & **CI/CD workflows**  
✅ Implement **cross-database & secure seeding strategies**  

