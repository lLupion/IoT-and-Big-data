# Class 11: Orchestration with Apache Airflow

| Class | Duration | Project Milestone |
|-------|----------|-------------------|
| 11 of 15 | 1 hour | Understand DAGs and Airflow concepts |

## Learning Objectives
By the end of this lesson, you will be able to:
- [ ] Explain workflow orchestration concepts
- [ ] Create DAGs in Apache Airflow
- [ ] Configure task dependencies and scheduling

## Prerequisites
- Lessons 03-08 completed
- Docker installed

## Why This Matters
ETL jobs need to run on schedule. Job B depends on Job A. Failures need alerts. Orchestration tools like Airflow manage all this complexity so you don't have to write cron jobs and bash scripts.

## Recall from Lesson 6
You designed a pipeline: Raw → Staged → Analytics. Now we'll automate it to run daily with proper dependencies.

---

# 📖 INSTRUCTOR-LED SECTION

## 1. What is Orchestration?

Orchestration answers:
- **When** should jobs run? (Schedule)
- **What order** should they run? (Dependencies)
- **What if** something fails? (Retry, alerts)

### Without Orchestration

```bash
# Fragile cron-based approach
0 1 * * * python raw_to_staged.py
0 2 * * * python staged_to_analytics.py  # Hope job 1 finished!
```

### With Orchestration

```python
raw_to_staged >> staged_to_analytics  # Explicit dependency
```

---

## 2. Apache Airflow Overview

| Feature | Description |
|---------|-------------|
| Open source | Free, large community |
| Python-based | DAGs defined in Python |
| Web UI | Monitor and manage workflows |
| Extensible | 1000+ integrations |

### Key Concepts

| Term | Definition |
|------|------------|
| **DAG** | Directed Acyclic Graph - the workflow |
| **Task** | A single unit of work |
| **Operator** | Template for a task (Python, Bash, etc.) |
| **Dependency** | Task B runs after Task A |

---

## 3. DAGs as Graphs

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Extract   │ ──▶ │  Transform  │ ──▶ │    Load     │
└─────────────┘     └─────────────┘     └─────────────┘
      │                                        │
      │            ┌─────────────┐             │
      └──────────▶ │   Notify    │ ◀───────────┘
                   └─────────────┘
```

- **Directed**: Arrows show flow direction
- **Acyclic**: No loops (can't go back)

![image-20250210-173704.png](09-orchestration-images/image-20250210-173704.png)

---

## 4. Your First DAG

```python
from datetime import datetime
from airflow import DAG
from airflow.operators.python import PythonOperator

# Define functions
def extract():
    print("Extracting data...")
    return "extracted_data"

def transform():
    print("Transforming data...")
    return "transformed_data"

def load():
    print("Loading data...")
    return "done"

# Define DAG
with DAG(
    dag_id="my_first_dag",
    start_date=datetime(2025, 1, 1),
    schedule_interval="@daily",  # Run daily
    catchup=False                # Don't backfill
) as dag:
    
    task_extract = PythonOperator(
        task_id="extract",
        python_callable=extract
    )
    
    task_transform = PythonOperator(
        task_id="transform",
        python_callable=transform
    )
    
    task_load = PythonOperator(
        task_id="load",
        python_callable=load
    )
    
    # Define dependencies
    task_extract >> task_transform >> task_load
```

### Schedule Intervals

| Preset | Cron Equivalent | Meaning |
|--------|-----------------|---------|
| `@daily` | `0 0 * * *` | Midnight daily |
| `@hourly` | `0 * * * *` | Every hour |
| `@weekly` | `0 0 * * 0` | Sunday midnight |
| `None` | - | Manual trigger only |

### ✅ Checkpoint 1
Identify in the code above:
- DAG definition
- Task definitions
- Dependencies

---

## 5. Running PySpark in Airflow

Use `PythonVirtualenvOperator` to isolate PySpark:

```python
from airflow.operators.python import PythonVirtualenvOperator

def run_spark_job():
    from pyspark.sql import SparkSession
    
    spark = SparkSession.builder.appName("AirflowJob").getOrCreate()
    
    df = spark.read.parquet("/data/raw/")
    df_clean = df.filter(df.temperature.isNotNull())
    df_clean.write.mode("overwrite").parquet("/data/staged/")
    
    spark.stop()

task_spark = PythonVirtualenvOperator(
    task_id="run_spark",
    python_callable=run_spark_job,
    requirements=["pyspark==3.5.0"],
    system_site_packages=False
)
```

**Why virtualenv?** Isolates PySpark dependencies from Airflow's Python environment.

---

## 6. Airflow Web UI

### DAGs List
![image-20250210-182059.png](09-orchestration-images/image-20250210-182059.png)

### DAG Graph View
![image-20250210-182029.png](09-orchestration-images/image-20250210-182029.png)

### Key UI Features

| Feature | Location | Purpose |
|---------|----------|---------|
| Toggle DAG | DAGs list | Enable/disable |
| Trigger | DAG page | Manual run |
| Graph | DAG page | Visualize dependencies |
| Logs | Task instance | Debug failures |

![image-20250210-185059.png](09-orchestration-images/image-20250210-185059.png)

---

## 7. When to Use Airflow

### ✅ Use Airflow When:
- Complex dependencies between jobs
- Need scheduling and monitoring
- Multiple data sources/destinations
- Team needs visibility into pipelines

### ❌ Don't Use Airflow When:
- Simple cron job is enough
- Real-time streaming (use Kafka)
- Dynamic pipelines that change every run
- Team doesn't know Python

---

# ✏️ STUDENT PRACTICE SECTION

## Exercise 1: Create a Simple DAG

Create a DAG with 3 tasks that print messages:

```python
# File: dags/hello_dag.py

from datetime import datetime
from airflow import DAG
from airflow.operators.python import PythonOperator

def say_hello():
    print("Hello!")

def say_world():
    print("World!")

def say_done():
    print("Done!")

# YOUR CODE: Create DAG with these tasks
# Dependencies: hello >> world >> done
```

<details>
<summary>💡 Solution</summary>

```python
with DAG(
    dag_id="hello_world",
    start_date=datetime(2025, 1, 1),
    schedule_interval=None,
    catchup=False
) as dag:
    
    t1 = PythonOperator(task_id="hello", python_callable=say_hello)
    t2 = PythonOperator(task_id="world", python_callable=say_world)
    t3 = PythonOperator(task_id="done", python_callable=say_done)
    
    t1 >> t2 >> t3
```
</details>

---

## Exercise 2: Sensor ETL DAG

Create a DAG for our sensor data pipeline:

```python
# Tasks:
# 1. raw_to_staged: Read CSV, clean, write Parquet
# 2. staged_to_analytics: Aggregate daily stats
# 3. notify: Print completion message

# Dependencies: raw_to_staged >> staged_to_analytics >> notify
```

<details>
<summary>💡 Solution</summary>

```python
from datetime import datetime
from airflow import DAG
from airflow.operators.python import PythonVirtualenvOperator, PythonOperator

def raw_to_staged():
    from pyspark.sql import SparkSession
    from pyspark.sql.functions import col
    
    spark = SparkSession.builder.appName("RawToStaged").getOrCreate()
    df = spark.read.csv("/data/raw/sensors.csv", header=True, inferSchema=True)
    df_clean = df.filter(col("temperature").isNotNull())
    df_clean.write.mode("overwrite").parquet("/data/staged/sensors")
    spark.stop()

def staged_to_analytics():
    from pyspark.sql import SparkSession
    from pyspark.sql.functions import avg, to_date, col
    
    spark = SparkSession.builder.appName("StagedToAnalytics").getOrCreate()
    df = spark.read.parquet("/data/staged/sensors")
    df_agg = df.withColumn("date", to_date(col("timestamp"))) \
        .groupBy("module_id", "date") \
        .agg(avg("temperature").alias("avg_temp"))
    df_agg.write.mode("overwrite").parquet("/data/analytics/daily")
    spark.stop()

def notify():
    print("Sensor ETL pipeline complete!")

with DAG(
    dag_id="sensor_etl",
    start_date=datetime(2025, 1, 1),
    schedule_interval="@daily",
    catchup=False
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
    
    t3 = PythonOperator(
        task_id="notify",
        python_callable=notify
    )
    
    t1 >> t2 >> t3
```
</details>

---

## Quick Check

1. What does DAG stand for?
   - a) Data Analysis Graph
   - b) Directed Acyclic Graph
   - c) Dynamic Airflow Graph

2. What operator runs a Python function?
   - a) BashOperator
   - b) PythonOperator
   - c) SparkOperator

3. What does `>>` mean in Airflow?
   - a) Greater than
   - b) Dependency (run after)
   - c) Parallel execution

<details>
<summary>Answers</summary>

1. b) Directed Acyclic Graph
2. b) PythonOperator
3. b) Dependency (run after)
</details>

---

## Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| DAG not appearing | Syntax error in file | Check Airflow logs |
| Import error | Module not installed | Add to `requirements` |
| Task stuck | Dependency not met | Check upstream tasks |

---

## Summary

| Concept | Key Point |
|---------|-----------|
| DAG | Workflow as a directed graph |
| Task | Single unit of work |
| Operator | Template for tasks |
| `>>` | Defines dependency |
| Schedule | When DAG runs |

---

## What's Next?

In **Lesson 10**, we'll do hands-on Airflow practice. You'll deploy a complete ETL pipeline and learn to debug common issues.

**Preparation:** Make sure Docker is running.
