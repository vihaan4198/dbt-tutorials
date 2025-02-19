

### **🛠️ Prerequisites:**
1. **AWS account** with EC2, S3, IAM, and other services configured.
2. **dbt project** set up.
3. **Amazon Managed Workflows for Apache Airflow (MWAA)** (optional) or custom EC2-based Airflow setup.
4. **Python and Apache Airflow** installed locally for development (optional).
5. **AWS CLI** set up and configured for your account.

---

### **💡 Goal:**
We will set up **Airflow** in **AWS** and trigger dbt models using **Airflow DAGs**. We will manage this using **Amazon MWAA** (Managed Workflows for Apache Airflow) for simplicity. The main tasks will include:

- **Create an S3 bucket** to store DAGs and logs.
- **Create an Airflow environment** using MWAA.
- **Create and run dbt models** using Airflow DAGs.

---

## **📌 Step 1: Set Up an S3 Bucket for Airflow**

We'll start by creating an **S3 bucket** to store the necessary files for Airflow.

1. **Log in to AWS Console**.
2. **Navigate to S3** and **Create a new bucket**.
   - Name the bucket, for example: `my-airflow-dags-bucket`.
   - Select a region close to your Airflow environment.
   - Leave the other settings as default.
   - **Enable versioning** for keeping track of changes in your DAG files.
   
3. **Upload your dbt project to S3**:
   - You need to upload your **dbt models**, **profiles.yml**, and any necessary files to this bucket.
   - Place them under a directory like `dags/dbt` inside the bucket for organization.

---

## **📌 Step 2: Set Up Airflow in AWS (MWAA)**

**Amazon Managed Workflows for Apache Airflow (MWAA)** is the easiest way to run Airflow in AWS. Here’s how to set it up:

1. **Go to MWAA Service**:
   - Navigate to **Amazon MWAA** in the AWS Console.
   - Click **Create environment** to create a new environment.

2. **Configure Environment**:
   - **Name**: Give your environment a name, for example: `dbt-airflow-env`.
   - **Execution role**: Create an **IAM role** or use an existing one with permissions for Airflow to access S3, CloudWatch, and other necessary services.
   - **Airflow version**: Choose the latest version of Airflow (e.g., 2.x).
   - **Network settings**: Choose the VPC, Subnet, and Security Group for the Airflow environment. If you need to access Snowflake or any other database, make sure the network settings allow this.

3. **Specify S3 Bucket**:
   - Under **DAGs folder**, specify the **S3 bucket** you created earlier.
   - Provide the S3 path to your DAGs folder (e.g., `s3://my-airflow-dags-bucket/dags/dbt/`).

4. **Logs**:
   - Choose **CloudWatch Logs** for storing Airflow logs.
   
5. **Create Environment**:
   - After completing the configuration, click **Create environment**.
   - AWS will take a few minutes to set up your MWAA environment.

---

## **📌 Step 3: Create the Airflow DAG to Trigger dbt Models**

Now that MWAA is set up, we can create an **Airflow DAG** to trigger dbt models. This DAG will run dbt commands such as `dbt run`, `dbt test`, etc.

1. **Create the Airflow DAG Python Script**:
   
   Create a Python file for the DAG, e.g., `dbt_dag.py`, and upload it to your S3 bucket in the `dags/dbt/` folder. This will ensure that MWAA picks it up.

2. **Define the DAG**:
   
   Here's a simple example of a DAG that runs dbt models and tests:

```python
from airflow import DAG
from airflow.providers.apache.airflow.providers.dbt.cloud.operators.dbt import DbtRunOperator, DbtTestOperator
from airflow.utils.dates import days_ago
from airflow.utils.dates import datetime

# Default arguments
default_args = {
    'owner': 'airflow',
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
    'start_date': datetime(2023, 1, 1),
}

# Define the DAG
dag = DAG(
    'dbt_airflow_example',
    default_args=default_args,
    description='A simple DAG to run dbt models',
    schedule_interval='@daily',  # Schedule daily
    catchup=False,
)

# Task 1: Run dbt models
run_dbt = DbtRunOperator(
    task_id='run_dbt_models',
    dag=dag,
    dbt_bin='dbt',  # Path to dbt binary
    profiles_dir='/usr/local/airflow/dags/dbt',  # Directory containing profiles.yml
    models='my_dbt_model',  # Specify the model or tag you want to run
)

# Task 2: Run dbt tests (Optional)
run_dbt_tests = DbtTestOperator(
    task_id='run_dbt_tests',
    dag=dag,
    dbt_bin='dbt',
    profiles_dir='/usr/local/airflow/dags/dbt',
    models='my_dbt_model',  # Specify the model or tag you want to test
)

# Task dependencies
run_dbt >> run_dbt_tests
```

### **Explanation**:
- **DbtRunOperator**: This operator triggers dbt models (runs `dbt run`).
- **DbtTestOperator**: This operator runs dbt tests (runs `dbt test`).
- **profiles_dir**: This is where the dbt profile (i.e., `profiles.yml`) is stored. We assume it is uploaded in the `dags/dbt/` folder on S3.
- **task dependencies**: `run_dbt` must be completed before `run_dbt_tests`.

---

## **📌 Step 4: Test and Debug the DAG**

1. **Airflow UI**:
   - After deploying your DAG to S3, you can monitor it in the **Airflow UI**.
   - Navigate to **Amazon MWAA** and click on your environment.
   - Click **Airflow UI** to open the Airflow web interface.
   
2. **Check Logs**:
   - You can see logs in the **Airflow UI** or CloudWatch for each task run.
   - If there are any issues with dbt, you can debug using the logs provided in the Airflow UI.

---

## **📌 Step 5: Schedule and Automate dbt Runs**

Once your DAG is set up, it will run according to the schedule you've defined (in this case, daily). You can adjust the schedule as needed:

- **Schedule example**: `@daily` will run the DAG every day at midnight.
- **Cron syntax**: You can also specify more complex schedules using cron syntax like `0 0 * * *` for daily runs at midnight.

---

## **📌 Step 6: Monitor the DAG**

1. **Airflow UI**:
   - You can monitor DAG runs directly from the **Airflow UI** and check the status of each task.
   - Airflow will also allow you to retry tasks, check logs, and make adjustments if needed.

2. **CloudWatch Logs**:
   - All task logs will be available in **CloudWatch Logs** if you configured them properly during the MWAA setup.

---

## **📌 Best Practices and Optimization**

1. **Use of Virtual Environments**:
   - It's a good idea to use a virtual environment or Docker for your dbt setup to avoid dependency conflicts.
   - This is especially important when managing dbt and Airflow in production.

2. **Error Handling**:
   - Set up retries and failure notifications in Airflow. If a task fails, you can specify **email notifications** or alerts to notify the team.

3. **Version Control**:
   - Version control your DAGs and dbt models to ensure reproducibility and maintainability of your workflows.

4. **Logging and Monitoring**:
   - Log everything! Keep track of dbt runs, failures, and changes to ensure smooth production workflows.

---

### **🚀 Summary:**

| Step                     | Description                                      |
|--------------------------|--------------------------------------------------|
| **Step 1**: Set up S3 bucket | Create an S3 bucket to store Airflow files (DAGs and logs). |
| **Step 2**: Set up MWAA | Use Amazon Managed Workflows for Apache Airflow (MWAA) for setting up the Airflow environment in AWS. |
| **Step 3**: Create dbt DAG | Define a Python script with Airflow DAG to run dbt models and tests. |
| **Step 4**: Test and Debug | Use Airflow UI and CloudWatch logs to monitor and debug the DAG. |
| **Step 5**: Schedule and Automate | Schedule your DAG runs using cron syntax or Airflow’s built-in schedule options. |
| **Step 6**: Monitoring | Use Airflow UI and CloudWatch logs for continuous monitoring and optimization. |

---

By following this guide, you'll be able to automate the running of **dbt models** in **AWS** using **Airflow DAGs** and **MWAA**.

