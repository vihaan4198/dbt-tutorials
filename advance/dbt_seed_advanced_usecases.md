# 🚀 **Custom Advanced dbt Seed Tutorial**  
**🔥 Goal:** Build a **scalable, automated, and optimized seeding process** for **real-world enterprise projects** using dbt & Snowflake/Postgres.  
**🔍 Key Focus Areas:**  
✅ **Incremental Seed Updates** (Avoid overwriting all data)  
✅ **Multi-Environment Seeding** (Different databases for dev/stage/prod)  
✅ **Automated Seed Updates in CI/CD** (GitHub Actions integration)  
✅ **Dynamic Seed Data Generation** (Using Python & Jinja)  
✅ **Secure & Mask Sensitive Data**  

---

## **🛠 1. Project Setup**
### **📌 Step 1: Initialize dbt Project**
If you haven’t initialized a dbt project, run:  
```sh
dbt init my_dbt_project
cd my_dbt_project
```
Ensure your **profiles.yml** is configured for Snowflake/Postgres.

---

## **📌 2. Load Incremental Seeds (Avoid Overwrites)**  
By default, `dbt seed` **overwrites** the table each time. Instead, we'll **only insert new records.**

### **🔹 Step 1: Create a Seed File**
📂 **seeds/customers.csv**
```csv
id,name,email,signup_date
1,Alice,alice@example.com,2024-01-10
2,Bob,bob@example.com,2023-12-15
3,Charlie,charlie@example.com,2024-02-05
```
### **🔹 Step 2: Modify `dbt_project.yml`**
```yaml
seeds:
  my_dbt_project:
    customers:
      +schema: staging_seeds
```
This stores the `customers` seed table in **staging_seeds** instead of the default schema.

### **🔹 Step 3: Implement Incremental Logic**
📂 **models/stg_customers.sql**
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
**🔹 Execution Strategy:**  
1️⃣ `dbt seed` loads new seed data into `staging_seeds.customers`.  
2️⃣ The model filters out **existing** records from production.  
3️⃣ **Only new records** are inserted into the final table.  

✅ **Benefit:** **Avoids overwriting historical data!**  

---

## **📌 3. Multi-Environment Seeding**
Load the same seed data into **multiple environments** (e.g., **Snowflake dev, stage, prod**).  
Modify `dbt_project.yml`:  

```yaml
seeds:
  my_dbt_project:
    dev:
      +database: dev_db
      +schema: seed_dev
    stage:
      +database: stage_db
      +schema: seed_stage
    prod:
      +database: prod_db
      +schema: seed_prod
```
Run for dev:
```sh
dbt seed --target dev
```
Run for prod:
```sh
dbt seed --target prod
```
✅ **Benefit:** Each environment gets **its own version** of the seed data.

---

## **📌 4. Automate Seed Updates in CI/CD**
Modify **GitHub Actions** to run `dbt seed` **only when the seed file changes**.  
📂 **.github/workflows/dbt_ci.yml**
```yaml
name: dbt CI/CD Pipeline

on:
  push:
    branches:
      - main
    paths:
      - 'seeds/**'  # Run only if seed files change

jobs:
  dbt_seed:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v2

      - name: Setup Python
        uses: actions/setup-python@v3
        with:
          python-version: 3.9

      - name: Install dbt
        run: pip install dbt-snowflake

      - name: Run dbt Seed
        run: dbt seed
```
✅ **Benefit:** Runs `dbt seed` only if **seed files are modified**, saving **execution time**.

---

## **📌 5. Dynamic Seed Data Generation (Python + Jinja)**
Instead of **manually updating CSVs**, generate them **dynamically** using Python.

📂 **scripts/generate_seeds.py**
```python
import pandas as pd
from faker import Faker

fake = Faker()
data = {
    "id": range(1, 101),
    "name": [fake.name() for _ in range(100)],
    "email": [fake.email() for _ in range(100)],
    "signup_date": pd.date_range("2024-01-01", periods=100).strftime("%Y-%m-%d"),
}

df = pd.DataFrame(data)
df.to_csv("seeds/dynamic_customers.csv", index=False)
print("✅ Seed file generated successfully!")
```
Run before `dbt seed`:
```sh
python scripts/generate_seeds.py
dbt seed
```
✅ **Benefit:** No need to **manually edit CSVs**—they are **auto-generated**.

---

## **📌 6. Secure & Mask Sensitive Data**
If your seed file contains **PII data**, apply **masking techniques**.

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
✅ **Benefit:** Prevents **exposing sensitive information**.

---

## **📌 7. Automate Large-Scale Seed File Loads**
If you have **millions of rows**, optimize with **parallel execution**.

### **🔹 1. Split Large CSVs into Chunks**
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
✅ **Benefit:** Improves **query performance**.

### **🔹 2. Use `dbt seed --threads` for Parallel Execution**
```sh
dbt seed --threads 4
```
✅ **Benefit:** Speeds up **large CSV file processing**.

---

## **🎯 Final Summary**
| Feature | Technique | Command |
|---------|-----------|---------|
| 🔄 **Incremental Seeding** | Merge new records | `dbt seed --full-refresh` (Cautious use) |
| 🏢 **Multi-Environment Seeding** | Store in different databases | `dbt seed --target dev` |
| 🤖 **CI/CD Automation** | Run seed only if files changed | GitHub Actions workflow |
| 📜 **Audit Logs** | Store reference audit data | `SELECT * FROM {{ ref('audit_events') }}` |
| 🐍 **Python-Based Seeding** | Generate dynamic CSVs | `python scripts/generate_seeds.py` |
| 🔒 **Secure Data** | Mask sensitive columns | Use `column_types` in `dbt_project.yml` |
| 🚀 **Performance Optimization** | Split large files & parallel execution | `dbt seed --threads 4` |

---

## **🚀 Final Thoughts**
With these **advanced techniques**, you can:  
✅ Automate `dbt seed` for **real-time updates**  
✅ Load seeds efficiently for **large-scale databases**  
✅ Secure seed data and **mask sensitive information**  
✅ Scale across **multiple environments**  

