### **🚀 Integrating Airflow with AWS MWAA (Managed Workflows for Apache Airflow)**
Now that we have a working **DBT + Airflow pipeline**, let’s deploy it on **AWS Managed Workflows for Apache Airflow (MWAA)**. MWAA eliminates the need to manage Airflow infrastructure, making it **scalable, secure, and AWS-integrated**.  

---

## **📌 Step 1: Set Up AWS MWAA Environment**
1️⃣ **Go to AWS Console** → Search **"MWAA"**  
2️⃣ **Click "Create environment"** and configure:
   - **Name**: `customer-360-airflow`
   - **Airflow Version**: Choose latest stable version (e.g., `2.5.1`)
   - **S3 Bucket**: Create an S3 bucket (e.g., `s3://mwaa-dbt-pipeline`)
   - **DAGs Folder**: `s3://mwaa-dbt-pipeline/dags/`
   - **Execution Role**: Attach IAM policies:
     - `AmazonMWAAFullAccess`
     - `AmazonS3FullAccess`
     - `AmazonRDSFullAccess` (for Aurora access)
     - `AWSGlueConsoleFullAccess` (if using Glue)
   - **Networking**: Select a **VPC, subnets, and security groups**.
   - **MWAA Environment Class**: Choose `mw1.medium` for cost-effective scaling.

---

## **📌 Step 2: Upload DAGs & DBT Artifacts to S3**
AWS MWAA **reads DAGs from an S3 bucket**, so we need to upload our DBT Airflow DAGs.

1️⃣ **Move Airflow DAGs to S3**  
```sh
aws s3 cp dbt_customer_360_dag.py s3://mwaa-dbt-pipeline/dags/
```

2️⃣ **Upload DBT Project** to S3  
```sh
aws s3 sync ~/dbt/customer_360_project s3://mwaa-dbt-pipeline/dbt/
```

---

## **📌 Step 3: Configure MWAA to Use DBT**
1️⃣ **Install DBT in MWAA**
   - In MWAA, go to **"Plugins and Requirements"**  
   - Add `requirements.txt` in S3 (`s3://mwaa-dbt-pipeline/requirements.txt`)
   - Inside `requirements.txt`:
     ```txt
     dbt-core
     dbt-postgres
     apache-airflow-providers-dbt-cloud
     apache-airflow-providers-amazon
     ```
   - Restart MWAA to install.

2️⃣ **Configure Airflow Connections**
   - **Go to MWAA UI** (`MWAA > Open Airflow UI`)
   - **Add Postgres Connection**:  
     - Connection ID: `postgres_customer360`
     - Connection Type: `Postgres`
     - Host: `<aurora-cluster-endpoint>.rds.amazonaws.com`
     - Schema: `customer360_db`
     - Login: `dbadmin`
     - Password: `yourpassword`
     - Port: `5432`

   - **Add S3 Connection**:
     - Connection ID: `s3_dbt_artifacts`
     - Type: `Amazon S3`
     - Extra:  
       ```json
       { "aws_access_key_id": "YOUR_ACCESS_KEY", "aws_secret_access_key": "YOUR_SECRET_KEY" }
       ```

---

## **📌 Step 4: Modify Airflow DAG to Use MWAA**
Modify `dbt_customer_360_dag.py` to use **S3 storage** instead of local DBT paths.

```python
from airflow import DAG
from airflow.providers.amazon.aws.operators.s3 import S3CopyObjectOperator
from airflow.providers.amazon.aws.transfers.s3_to_s3 import S3ToS3Operator
from airflow.providers.postgres.operators.postgres import PostgresOperator
from airflow.operators.bash import BashOperator
from airflow.utils.dates import days_ago
import os

S3_DBT_PATH = "s3://mwaa-dbt-pipeline/dbt/"
DBT_PROFILES_DIR = "/usr/local/airflow/dbt/"

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
    tags=["dbt", "mwaa", "customer_360"]
) as dag:

    # Step 1: Sync DBT files from S3 to local MWAA
    sync_dbt_files = BashOperator(
        task_id="sync_dbt_files",
        bash_command=f"aws s3 sync {S3_DBT_PATH} {DBT_PROFILES_DIR}"
    )

    # Step 2: Run DBT Snapshots
    dbt_snapshot = BashOperator(
        task_id="dbt_snapshot",
        bash_command=f"cd {DBT_PROFILES_DIR} && dbt snapshot"
    )

    # Step 3: Run DBT Incremental Models
    dbt_run_incremental = BashOperator(
        task_id="dbt_run_incremental",
        bash_command=f"cd {DBT_PROFILES_DIR} && dbt run"
    )

    # Step 4: Run DBT Tests
    dbt_test = BashOperator(
        task_id="dbt_test",
        bash_command=f"cd {DBT_PROFILES_DIR} && dbt test"
    )

    # Step 5: Upload DBT Artifacts back to S3
    upload_dbt_artifacts = S3CopyObjectOperator(
        task_id="upload_dbt_artifacts",
        source_bucket_name="mwaa-dbt-pipeline",
        source_bucket_key="dbt/target/",
        dest_bucket_name="mwaa-dbt-pipeline",
        dest_bucket_key="dbt-results/"
    )

    # Task Dependencies
    sync_dbt_files >> dbt_snapshot >> dbt_run_incremental >> dbt_test >> upload_dbt_artifacts
```

---

## **📌 Step 5: Deploy the Updated DAG to MWAA**
1️⃣ Upload DAG to S3  
```sh
aws s3 cp dbt_customer_360_dag.py s3://mwaa-dbt-pipeline/dags/
```
2️⃣ Restart MWAA  
   - Go to **MWAA Console** → **Restart Environment**.

3️⃣ Check DAG in **MWAA UI** → **Trigger DAG** to test.

---

## **📌 Step 6: Automate MWAA & DBT Pipeline**
### **1️⃣ Enable CloudWatch Logs for Monitoring**
   - Go to **MWAA Console** → Select your environment → **Enable CloudWatch Logs**.
   - Set log level: `INFO`

### **2️⃣ Use AWS EventBridge for Scheduling**
   - Create an **EventBridge rule** to trigger DAGs **every 6 hours**.

```json
{
  "source": ["aws.mwaa"],
  "detail-type": ["MWAA Job"],
  "detail": {
    "dag_name": ["dbt_customer_360_pipeline"]
  }
}
```

### **3️⃣ Alerting with AWS SNS for Failures**
   - Set up **AWS SNS** for email/SMS alerts if Airflow DAG fails.

---

## **🚀 Final Outcome**
✅ **Airflow DAGs on AWS MWAA**  
✅ **DBT Artifacts Managed in S3**  
✅ **Data Processing Automated in Aurora**  
✅ **Monitoring with CloudWatch**  
✅ **Alerting with AWS SNS & EventBridge**  

---
