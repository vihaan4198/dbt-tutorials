**DBT Customer 360 Part 3 -integrate Apache Airflow** 


---

# **📌 Step 1: Set Up Airflow for DBT**
### **1️⃣ Install Airflow & Required Dependencies**
If you haven't installed Airflow yet, run:  
```sh
pip install apache-airflow
pip install apache-airflow-providers-dbt-cloud
pip install apache-airflow-providers-postgres
```
For **DBT CLI integration**, install:  
```sh
pip install dbt-core dbt-postgres
```

### **2️⃣ Configure Airflow Environment**
Initialize Airflow if not done yet:  
```sh
airflow db init
airflow scheduler &
airflow webserver -p 8080
```
Login at `http://localhost:8080` (default user: `admin` / `airflow`).

---

# **📌 Step 2: Create DBT DAGs in Airflow**
### **1️⃣ Create Airflow DAGs Directory**
Navigate to your Airflow `dags/` folder:  
```sh
cd ~/airflow/dags
```
Create a **new DAG file** for DBT:
```sh
touch dbt_customer_360_dag.py
```

---

## **🔹 Step 3: Create Airflow DAG for DBT Runs**
Now, let's define an **Airflow DAG** to:
✅ Run **DBT snapshots** (track history)  
✅ Run **DBT incremental models** (fast updates)  
✅ Run **DBT tests** (data validation)  
✅ Send **Slack notifications** (optional)

### **✍️ Create `dbt_customer_360_dag.py`**
```python
from airflow import DAG
from airflow.providers.dbt.cloud.operators.dbt import DbtCloudRunJobOperator
from airflow.operators.bash import BashOperator
from airflow.operators.email import EmailOperator
from airflow.utils.dates import days_ago
from airflow.providers.slack.operators.slack_webhook import SlackWebhookOperator
import os

# Load Slack Webhook from environment
SLACK_WEBHOOK_URL = os.getenv("SLACK_WEBHOOK_URL")

default_args = {
    "owner": "airflow",
    "depends_on_past": False,
    "start_date": days_ago(1),
    "retries": 1
}

with DAG(
    "dbt_customer_360_pipeline",
    default_args=default_args,
    schedule_interval="0 */6 * * *",  # Runs every 6 hours
    catchup=False,
    tags=["dbt", "customer_360"]
) as dag:

    # Step 1: Run DBT Snapshots (Historical Tracking)
    dbt_snapshot = BashOperator(
        task_id="dbt_snapshot",
        bash_command="cd ~/dbt/customer_360_project && dbt snapshot"
    )

    # Step 2: Run DBT Incremental Models
    dbt_run_incremental = BashOperator(
        task_id="dbt_run_incremental",
        bash_command="cd ~/dbt/customer_360_project && dbt run"
    )

    # Step 3: Run DBT Tests
    dbt_test = BashOperator(
        task_id="dbt_test",
        bash_command="cd ~/dbt/customer_360_project && dbt test"
    )

    # Step 4: Send Slack Notification (Optional)
    slack_success_notification = SlackWebhookOperator(
        task_id="slack_success_notification",
        http_conn_id="slack_connection",
        webhook_token=SLACK_WEBHOOK_URL,
        message="✅ DBT Customer 360 pipeline completed successfully!",
        channel="#data-alerts",
        username="airflow"
    )

    # Step 5: Send Email on Failure
    email_alert = EmailOperator(
        task_id="email_alert",
        to="data_team@example.com",
        subject="🚨 DBT Pipeline Failed",
        html_content="DBT Customer 360 pipeline failed. Check Airflow logs.",
        trigger_rule="one_failed"
    )

    # Task Dependencies
    dbt_snapshot >> dbt_run_incremental >> dbt_test
    dbt_test >> [slack_success_notification, email_alert]
```

---

## **📌 Step 4: Deploy and Test the Airflow DAG**
### **1️⃣ Move the DAG to Airflow Folder**
```sh
mv dbt_customer_360_dag.py ~/airflow/dags/
```

### **2️⃣ Restart Airflow Services**
```sh
airflow scheduler restart
airflow webserver restart
```

### **3️⃣ Verify the DAG in Airflow UI**
- Open **Airflow UI** (`http://localhost:8080`).
- Find **"dbt_customer_360_pipeline"** DAG.
- Click **Trigger DAG** ▶️ to test.

---

## **📌 Step 5: Automate Execution & Monitoring**
### **1️⃣ Schedule DAG to Run Every 6 Hours**
We've already set:  
```python
schedule_interval="0 */6 * * *"
```
This means: **Runs every 6 hours**.

### **2️⃣ Setup Slack Alerts for Failures**
- Create a **Slack Incoming Webhook**.
- Add it to `.env`:
  ```sh
  export SLACK_WEBHOOK_URL="https://hooks.slack.com/services/T000/B000/XXXX"
  ```

### **3️⃣ View Logs & Debug Failures**
Check logs:
```sh
airflow dags list
airflow dags trigger dbt_customer_360_pipeline
airflow tasks logs dbt_customer_360_pipeline dbt_run_incremental
```

---

## **🚀 Final Outcome**
✅ **Automated DBT Snapshots** (Track historical changes)  
✅ **Incremental DBT Runs** (Faster updates)  
✅ **Data Testing** (DBT `test`)  
✅ **Slack & Email Alerts** (Proactive Monitoring)  
✅ **Scheduled Every 6 Hours** (Optimized processing)  
