# **🚀 Extended Tutorial: Trigger dbt Tests and Manage Production Workflows with Airflow**

## **📌 Objective:**
- **Extend** the previous tutorial to include **dbt tests** and ensure **production workflows** are managed efficiently using Apache Airflow.
- Implement **dbt test execution** as part of the DAG.
- Automate **alerts** and **notifications** for test failures.
- Manage and monitor the dbt pipeline as part of a **production-ready workflow**.

---

### **🔹 Prerequisites**
- **Apache Airflow** setup
- **dbt** and the **dbt-airflow provider** installed
- Snowflake/Aurora (or any other supported database) set up in dbt
- Basic understanding of dbt model and tests

---

## **📌 Step 1: Create dbt Tests for Models**

Before proceeding to trigger dbt tests in Airflow, we need to define some **dbt tests** for the models you are working with. dbt provides built-in tests, but you can also create custom tests.

### **📂 Example: Add Tests to Your dbt Models**

Let's assume you have a model like `stg_customers.sql`. You can add tests to this model by creating a corresponding test file.

#### **Example: Add Tests for `stg_customers`**

1. **Create a `tests` directory** in your dbt project.

2. **Create a test file** to validate if the `email` field in `stg_customers` is unique and not null.

#### 📂 **models/tests/test_stg_customers.sql**
```sql
-- Test if the email column is unique
SELECT
    email
FROM {{ ref('stg_customers') }}
GROUP BY email
HAVING COUNT(email) > 1
```

3. **Define the test in dbt**:

   In your dbt model folder, you can create a **schema.yml** file to declare the tests.

#### 📂 **models/schema.yml**
```yaml
version: 2

models:
  - name: stg_customers
    description: "Staging table for customers"
    tests:
      - unique:
          column_name: email
      - not_null:
          column_name: email
```

Here, the `unique` and `not_null` tests are added to the `email` column of `stg_customers`.

---

## **📌 Step 2: Extend the Airflow DAG to Trigger dbt Tests**

Now that we have created the dbt tests, we can extend our Airflow DAG to trigger **dbt test** after the dbt models have run.

### **📂 Modify `dbt_trigger_dag.py` to Add dbt Tests**

1. **Import the dbt test operator** and **other necessary Airflow components**:

```python
from airflow import DAG
from airflow.providers.dbt.operators.dbt import DbtRunOperator, DbtTestOperator
from airflow.utils.dates import days_ago
from datetime import timedelta
```

2. **Add the `DbtTestOperator` to Trigger dbt Tests**:
   Modify the DAG file to include a task for running dbt tests after the models have been executed successfully.

```python
default_args = {
    'owner': 'airflow',
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
    'start_date': days_ago(1),
}

with DAG(
    'dbt_trigger_with_tests_dag',
    default_args=default_args,
    description='An Airflow DAG to trigger dbt models and dbt tests',
    schedule_interval=None,  # Schedule as needed
    catchup=False,
) as dag:

    # Task 1: Run dbt models (like before)
    run_dbt_models = DbtRunOperator(
        task_id='run_dbt_models',
        dbt_conn_id='dbt_connection',  # dbt connection to Snowflake/Aurora
        models='stg_customers, stg_orders',
        profile_name='my_snowflake_dbt',  # dbt profile
        retries=2,
    )

    # Task 2: Run dbt tests (after dbt models run)
    run_dbt_tests = DbtTestOperator(
        task_id='run_dbt_tests',
        dbt_conn_id='dbt_connection',
        models='stg_customers, stg_orders',
        profile_name='my_snowflake_dbt',
    )

    # Define task dependencies
    run_dbt_models >> run_dbt_tests
```

#### **Explanation:**
- **DbtRunOperator**: This triggers the execution of the dbt models (`stg_customers`, `stg_orders`).
- **DbtTestOperator**: This triggers dbt tests for the specified models after the models have been run.
- **Task Dependencies**: The `run_dbt_tests` task will only run after the `run_dbt_models` task has completed.

---

## **📌 Step 3: Add Notification/Alerting for Test Failures**

To manage **production workflows**, we want to be notified in case any dbt tests fail. You can use **Airflow’s alerting system** to send emails or trigger other notification channels (e.g., Slack, PagerDuty, etc.) when a dbt test fails.

### **Configure Email Notifications in Airflow**

1. **Set up email alerts for failures** in Airflow's default arguments:

```python
default_args = {
    'owner': 'airflow',
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
    'start_date': days_ago(1),
    'email_on_failure': True,
    'email_on_retry': False,
    'email': ['your_email@example.com'],  # Email to receive failure notifications
}
```

2. **Configure Email Alert in DbtTestOperator**:
   
   You can set custom email alerts on the `DbtTestOperator` to trigger alerts when dbt tests fail.

```python
run_dbt_tests = DbtTestOperator(
    task_id='run_dbt_tests',
    dbt_conn_id='dbt_connection',
    models='stg_customers, stg_orders',
    profile_name='my_snowflake_dbt',
    on_failure_callback=your_failure_callback_function  # Optional custom callback
)
```

3. **Add Slack Notifications (Optional)**:
   
   To send Slack messages on failure, you can configure the **SlackAPIPostOperator** for custom notification.

```python
from airflow.providers.slack.operators.slack_api import SlackAPIPostOperator

slack_alert = SlackAPIPostOperator(
    task_id='slack_alert',
    token="your_slack_api_token",
    channel="#dbt-alerts",
    text=":x: dbt test failed for {{ task_instance.task_id }}",
    trigger_rule="all_failed",
)

run_dbt_tests >> slack_alert  # Adding Slack alert after dbt test
```

---

## **📌 Step 4: Managing Production Workflows and Scheduling**

1. **Schedule the DAG**: 

   You can schedule the DAG to run at specific intervals (e.g., daily, hourly).

```python
with DAG(
    'dbt_trigger_with_tests_dag',
    default_args=default_args,
    description='An Airflow DAG to trigger dbt models and dbt tests',
    schedule_interval='@daily',  # Schedule to run daily
    catchup=False,
) as dag:
```

2. **Trigger the DAG manually** (if needed):

   You can trigger the DAG manually using the **Airflow UI** or the **Airflow CLI**:

   ```bash
   airflow dags trigger dbt_trigger_with_tests_dag
   ```

3. **Monitor the DAG Runs**: 

   In the **Airflow UI**, you can monitor the progress of each task. Tasks like `run_dbt_models` and `run_dbt_tests` will show the status (success, failed, etc.) and provide logs to help you debug issues.

---

## **📌 Step 5: Best Practices for Managing Production Workflows with Airflow**

1. **Testing and Validation**:
   - Always test your DAG locally using `airflow test` for specific tasks before running the full DAG.
   - Use **unit tests** for dbt models to ensure correctness before deploying them to production.

2. **Error Handling**:
   - Configure retries and alerting (e.g., emails, Slack) for failed tasks.
   - Use `on_failure_callback` to trigger custom actions, such as sending alerts to Slack or another system.

3. **Scalability**:
   - Use **parallel execution** by using task concurrency and the `TaskGroup` to manage parallel dbt models or tests in a single DAG.
   - Divide your workflow into multiple DAGs for large, complex systems.

4. **Logging and Monitoring**:
   - Ensure that **logs are stored** in a centralized location (e.g., S3, local directories) for audit purposes.
   - Leverage **Airflow's UI** for easy task tracking and quick debugging.

---

## **🔥 Summary**

| Step                         | Description                                             |
|------------------------------|---------------------------------------------------------|
| **Step 1:** Define dbt Tests  | Create and add dbt tests (e.g., unique, not_null) for your models. |
| **Step 2:** Extend DAG to Trigger dbt Tests | Use `DbtTestOperator` to trigger dbt tests after models have run. |
| **Step 3:** Set Up Notifications | Set up email and Slack notifications for test failures. |
| **Step 4:** Manage Production Workflows | Schedule DAGs and add dependencies to manage workflows efficiently. |
| **Step 5:** Best Practices | Implement error handling, retry strategies, and optimize for scalability. |

---

