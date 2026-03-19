# Class 12: Airflow 3 Hands-on Lab

| Class    | Duration | Project Milestone             |
| -------- | -------- | ----------------------------- |
| 12 of 15 | 1 hour   | Deploy and run sensor ETL DAG |

## Learning Objectives

By the end of this lesson, you will be able to:

- [ ] Deploy DAGs to Airflow 3
- [ ] Monitor and debug task execution
- [ ] Chain multiple PySpark jobs

## Prerequisites

- Lesson 09: Orchestration concepts
- Airflow 3 environment running

## Why This Matters

Theory is great, but you need hands-on experience to debug real issues. This lab simulates real-world scenarios you'll encounter in production.

## Recall from Lesson 9

You learned DAG concepts and wrote your first DAG. Now we'll deploy it and handle real execution.

> ⚠️ **Airflow 3 Migration Note**
>
> This lesson uses **Apache Airflow 3**. Key differences from Airflow 2:
>
> - Standard operators (PythonOperator, BashOperator) moved to `airflow.providers.standard`
> - `schedule_interval` is replaced by `schedule`
> - Install `apache-airflow-providers-standard` for standard operators

---

# 📖 INSTRUCTOR-LED SECTION

## 1. Airflow Architecture Review

```
┌─────────────────────────────────────────────────────────┐
│                    Airflow Components                    │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌───────────┐    ┌───────────┐    ┌───────────┐       │
│  │    Web    │    │ Scheduler │    │  Workers  │       │
│  │   Server  │    │           │    │           │       │
│  └─────┬─────┘    └─────┬─────┘    └─────┬─────┘       │
│        │                │                │              │
│        └────────────────┼────────────────┘              │
│                         │                               │
│                  ┌──────┴──────┐                        │
│                  │  Metadata   │                        │
│                  │   Database  │                        │
│                  └─────────────┘                        │
│                                                         │
│  DAG Folder: /opt/airflow/dags/                        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 2. Reading DAG Code

![image-20250213-173429.png](10-airflow-hands-on-images/image-20250213-173429.png)

### Code Components

```python
from datetime import datetime                    # ← Imports
from airflow import DAG
from airflow.providers.standard.operators.python import PythonOperator

def my_function():                               # ← Task function
    print("Hello!")

with DAG(                                        # ← DAG definition
    dag_id="example",
    start_date=datetime(2025, 1, 1),
    schedule="@daily"
) as dag:

    task = PythonOperator(                       # ← Task definition
        task_id="my_task",
        python_callable=my_function
    )
```

### ✅ Checkpoint 1

In the image above, identify:

- 🔴 DAG context manager (`with DAG...`)
- 🔵 Task definitions (`PythonOperator`)
- 🟢 Task functions (`python_callable`)
- 🟣 Dependencies (`>>`)

---

## 3. Deploying a DAG

### Step 1: Create DAG File

```python
# File: dags/sensor_pipeline.py

from datetime import datetime
from airflow import DAG
from airflow.providers.standard.operators.python import PythonOperator
from time import sleep

def extract_data():
    print("Extracting sensor data from CSV...")
    # Simulate extraction
    return {"rows": 1000}

def transform_data():
    print("Transforming data...")
    # Simulate transformation
    sleep(10)
    return {"cleaned_rows": 950}

def load_data():
    print("Loading to Parquet...")
    return "success"

with DAG(
    dag_id="sensor_pipeline_v1",
    start_date=datetime(2025, 1, 1),
    schedule=None,  # Manual trigger for testing
    catchup=False,
    tags=["sensor", "etl"]
) as dag:

    t_extract = PythonOperator(
        task_id="extract",
        python_callable=extract_data
    )

    t_transform = PythonOperator(
        task_id="transform",
        python_callable=transform_data
    )

    t_load = PythonOperator(
        task_id="load",
        python_callable=load_data
    )

    t_extract >> t_transform >> t_load
```

### Step 2: Copy to DAG Folder

```bash
# Copy to your Airflow DAGs folder
cp sensor_pipeline.py $AIRFLOW_HOME/dags/
```

### Step 3: Verify in UI

1. Go to http://localhost:8080
2. Wait 30 seconds for DAG to appear
3. Find `sensor_pipeline_v1` in list
4. Toggle ON to enable

---

## 4. Running and Monitoring

### Trigger a DAG Run

1. Click on DAG name
2. Click "Trigger DAG" button (▶️)
3. Watch tasks turn green

### Task States

| Color     | State        | Meaning                |
| --------- | ------------ | ---------------------- |
| ⬜ White  | No status    | Not yet scheduled      |
| 🟡 Yellow | Running      | Currently executing    |
| 🟢 Green  | Success      | Completed successfully |
| 🔴 Red    | Failed       | Error occurred         |
| 🟠 Orange | Up for retry | Will retry soon        |

### Viewing Logs

1. Click on task instance (colored box)
2. Click "Log" button
3. Read output and errors

---

# ✏️ STUDENT PRACTICE SECTION

## Lab 1: Deploy Your First DAG

1. Create `hello_airflow.py`:

```python
from datetime import datetime
from airflow import DAG
from airflow.providers.standard.operators.python import PythonOperator

def say_hello():
    print("Hello from Airflow!")

def say_goodbye():
    print("Goodbye from Airflow!")

with DAG(
    dag_id="hello_airflow",
    start_date=datetime(2025, 1, 1),
    schedule=None,
    catchup=False
) as dag:

    t1 = PythonOperator(task_id="hello", python_callable=say_hello)
    t2 = PythonOperator(task_id="goodbye", python_callable=say_goodbye)

    t1 >> t2
```

2. Deploy to Airflow
3. Trigger manually
4. Check logs for both tasks

**✅ Success criteria:** Both tasks show green, logs show print statements.

---

## Lab 2: Add PySpark Task

Extend your DAG to include a PySpark job:

```python
def run_spark_etl():
    from pyspark.sql import SparkSession

    spark = SparkSession.builder.appName("AirflowETL").config("spark.driver.host", "127.0.0.1").config("spark.driver.bindAddress", "127.0.0.1").getOrCreate()

    # Create sample data
    data = [("sensor_01", 25.0), ("sensor_02", 30.0)]
    df = spark.createDataFrame(data, ["id", "temp"])

    # Transform
    df_result = df.filter(df.temp > 20)

    print(f"Processed {df_result.count()} rows")
    spark.stop()

# Add to DAG
from airflow.providers.standard.operators.python import PythonVirtualenvOperator

t_spark = PythonVirtualenvOperator(
    task_id="spark_etl",
    python_callable=run_spark_etl,
    requirements=["pyspark==3.5.0"]
)

t1 >> t_spark >> t2
```

**✅ Success criteria:** Spark task runs and shows row count in logs.

---

## Lab 3: Handle Failures

1. Create a task that fails:

```python
def failing_task():
    raise Exception("Intentional failure!")

t_fail = PythonOperator(
    task_id="will_fail",
    python_callable=failing_task,
    retries=2,
    retry_delay=timedelta(seconds=10)
)
```

2. Observe:
   - Task turns red
   - Retries happen
   - Check logs for error message

3. Fix the task and re-run

---

## Lab 4: Parallel Execution

Create a DAG with parallel tasks:

```
         ┌─── validate_temp ───┐
extract ─┤                     ├─▶ load
         └─── validate_humid ──┘
```

```python
def validate_temp():
    print("Validating temperature...")

def validate_humid():
    print("Validating humidity...")

t_extract >> [t_validate_temp, t_validate_humid] >> t_load
```

**✅ Success criteria:** Both validation tasks run simultaneously.

---

## Lab 5: Complete Sensor Pipeline

Build the full pipeline from our project:

```python
# Complete DAG with:
# 1. raw_to_staged (PySpark)
# 2. staged_to_analytics (PySpark)
# 3. generate_report (Python)
# 4. send_notification (Python)

# Dependencies:
# raw_to_staged >> staged_to_analytics >> [generate_report, send_notification]
```

<details>
<summary>💡 Solution</summary>

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.providers.standard.operators.python import PythonOperator, PythonVirtualenvOperator

def raw_to_staged():
    from pyspark.sql import SparkSession
    from pyspark.sql.functions import col

    spark = SparkSession.builder.appName("RawToStaged").getOrCreate()
    # Simulate reading raw data
    data = [("s1", 25.0, 60.0), ("s2", 150.0, 50.0), ("s3", 30.0, 70.0)]
    df = spark.createDataFrame(data, ["id", "temp", "humidity"])

    # Clean
    df_clean = df.filter((col("temp") >= -50) & (col("temp") <= 100))
    print(f"Cleaned: {df_clean.count()} rows")
    spark.stop()

def staged_to_analytics():
    from pyspark.sql import SparkSession
    from pyspark.sql.functions import avg

    spark = SparkSession.builder.appName("Analytics").getOrCreate()
    data = [("s1", 25.0), ("s3", 30.0)]
    df = spark.createDataFrame(data, ["id", "temp"])

    avg_temp = df.agg(avg("temp")).collect()[0][0]
    print(f"Average temperature: {avg_temp}")
    spark.stop()

def generate_report():
    print("Generating daily report...")
    print("Report: 2 sensors, avg temp 27.5°C")

def send_notification():
    print("Sending notification...")
    print("Email sent to team@example.com")

with DAG(
    dag_id="sensor_pipeline_complete",
    start_date=datetime(2025, 1, 1),
    schedule="@daily",
    catchup=False,
    default_args={
        "retries": 1,
        "retry_delay": timedelta(minutes=5)
    }
) as dag:

    t1 = PythonVirtualenvOperator(
        task_id="raw_to_staged",
        python_callable=raw_to_staged,
        requirements=["pyspark==3.5.0"]
    )

    t2 = PythonVirtualenvOperator(
        task_id="staged_to_analytics",
        python_callable=staged_to_analytics,
        requirements=["pyspark==3.5.0"]
    )

    t3 = PythonOperator(task_id="generate_report", python_callable=generate_report)
    t4 = PythonOperator(task_id="send_notification", python_callable=send_notification)

    t1 >> t2 >> [t3, t4]
```

</details>

---

## Troubleshooting Guide

| Symptom                 | Possible Cause       | Solution                          |
| ----------------------- | -------------------- | --------------------------------- |
| DAG not appearing       | Syntax error         | Run `python dags/file.py` locally |
| Task stuck in "queued"  | No workers available | Check worker logs                 |
| Import error            | Missing package      | Add to `requirements`             |
| Task fails immediately  | Python error         | Check task logs                   |
| "DAG has import errors" | Import at top level  | Move imports inside function      |

---

## Summary

| Skill          | Practiced               |
| -------------- | ----------------------- |
| Deploy DAG     | Copy to DAG folder      |
| Trigger DAG    | UI or CLI               |
| Monitor tasks  | Task states, logs       |
| Debug failures | Read logs, fix code     |
| Parallel tasks | `[task1, task2]` syntax |

---

## What's Next?

In **Lesson 11**, we'll move to the cloud. You'll learn about AWS Lambda and AWS Glue for serverless ETL.

**Preparation:** Create an AWS account (free tier available)
