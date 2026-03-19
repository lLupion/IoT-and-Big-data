# Class 5: Spark ETL - Reading Data

| Class | Duration | Project Milestone |
|-------|----------|-------------------|
| 5 of 15 | 1 hour | Read sensor CSV into Spark DataFrame |

## Learning Objectives
- [ ] Create a SparkSession
- [ ] Read CSV files into DataFrames
- [ ] Explore DataFrame structure and schema

## Prerequisites
- Classes 3-4: Python fundamentals
- PySpark installed: `pip install pyspark`

## Why This Matters
Every data pipeline starts with reading data. Spark can read from CSV, JSON, Parquet, databases, and more. Understanding schemas is critical for data quality.

---

# 📖 INSTRUCTOR-LED

## 1. Creating SparkSession

```python
from pyspark.sql import SparkSession

# Create SparkSession (entry point to Spark)
spark = SparkSession.builder \
    .appName("SensorETL") \
    .master("local[*]") \
    .getOrCreate()

# Verify
print(f"Spark version: {spark.version}")
```

| Parameter | Meaning |
|-----------|---------|
| `appName` | Name shown in Spark UI |
| `master` | `local[*]` = use all CPU cores |
| `getOrCreate` | Reuse existing or create new |

### ✅ Checkpoint 1
Run the code. Do you see the Spark version?

---

## 2. Reading CSV Files

### Basic Read

```python
# Read CSV with automatic schema detection
df = spark.read.csv("sensors.csv", header=True, inferSchema=True)

# View data
df.show(5)

# View schema
df.printSchema()
```

**Sample Output:**
```
+-------------------+-----------+--------+-----------+
|          timestamp|temperature|humidity|  module_id|
+-------------------+-----------+--------+-----------+
|2025-01-20 10:00:00|       25.3|    60.5|  sensor_01|
|2025-01-20 10:01:00|       25.5|    59.8|  sensor_01|
+-------------------+-----------+--------+-----------+

root
 |-- timestamp: string (nullable = true)
 |-- temperature: double (nullable = true)
 |-- humidity: double (nullable = true)
 |-- module_id: string (nullable = true)
```

### Read Options

```python
df = spark.read.csv(
    "sensors.csv",
    header=True,           # First row is header
    inferSchema=True,      # Auto-detect types
    sep=",",               # Delimiter
    nullValue="NA"         # Treat "NA" as null
)
```

---

## 3. Explicit Schema (Recommended)

```python
from pyspark.sql.types import StructType, StructField, StringType, FloatType, TimestampType

schema = StructType([
    StructField("timestamp", TimestampType(), True),
    StructField("temperature", FloatType(), True),
    StructField("humidity", FloatType(), True),
    StructField("module_id", StringType(), True)
])

df = spark.read.csv("sensors.csv", header=True, schema=schema)
df.printSchema()
```

### Why Explicit Schema?

| inferSchema | Explicit Schema |
|-------------|-----------------|
| Reads data twice | Reads once |
| May guess wrong | You control types |
| Slower | Faster |

---

## 4. Exploring DataFrames

```python
# Row count
print(f"Rows: {df.count()}")

# Column names
print(f"Columns: {df.columns}")

# Summary statistics
df.describe().show()

# First N rows as list
rows = df.take(3)
print(rows)
```

---

# ✏️ STUDENT PRACTICE

## Exercise 1: Create Sample Data

Since we may not have a CSV file, create data directly:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("Practice").getOrCreate()

# Create sample sensor data
data = [
    ("2025-01-20 10:00:00", 25.3, 60.5, "sensor_01"),
    ("2025-01-20 10:01:00", 25.5, 59.8, "sensor_01"),
    ("2025-01-20 10:00:00", 26.1, 58.2, "sensor_02"),
    ("2025-01-20 10:01:00", None, 57.5, "sensor_02"),  # Missing temp!
    ("2025-01-20 10:00:00", 150.0, 55.0, "sensor_03"), # Invalid!
]
columns = ["timestamp", "temperature", "humidity", "module_id"]

df = spark.createDataFrame(data, columns)
df.show()
```

---

## Exercise 2: Explore the Data

Using the DataFrame above:

```python
# YOUR TASKS:
# 1. Print the schema
# 2. Count total rows
# 3. Count rows where temperature is NOT null
# 4. Show only the module_id column
```

<details>
<summary>💡 Solution</summary>

```python
# 1. Schema
df.printSchema()

# 2. Total rows
print(f"Total: {df.count()}")

# 3. Non-null temperature
from pyspark.sql.functions import col
print(f"Non-null temp: {df.filter(col('temperature').isNotNull()).count()}")

# 4. Select column
df.select("module_id").show()
```
</details>

---

## Exercise 3: Read with Schema

Define an explicit schema and create the DataFrame:

```python
from pyspark.sql.types import StructType, StructField, StringType, FloatType

# YOUR TASK: Define schema for:
# - timestamp (String)
# - temperature (Float)
# - humidity (Float)  
# - module_id (String)

schema = StructType([
    # YOUR CODE HERE
])

df = spark.createDataFrame(data, schema)
df.printSchema()
```

<details>
<summary>💡 Solution</summary>

```python
schema = StructType([
    StructField("timestamp", StringType(), True),
    StructField("temperature", FloatType(), True),
    StructField("humidity", FloatType(), True),
    StructField("module_id", StringType(), True)
])
```
</details>

---

# 📝 QUICK CHECK

1. What method reads a CSV file in Spark?
   - a) `spark.load.csv()`
   - b) `spark.read.csv()`
   - c) `spark.open.csv()`

2. What does `inferSchema=True` do?
   - a) Creates a new schema
   - b) Auto-detects column types
   - c) Validates the schema

3. Which is faster for large files?
   - a) inferSchema
   - b) Explicit schema
   - c) Same speed

<details>
<summary>Answers</summary>
1. b) `spark.read.csv()`
2. b) Auto-detects column types
3. b) Explicit schema
</details>

---

# 📋 SUMMARY

| Operation | Code |
|-----------|------|
| Create session | `SparkSession.builder.getOrCreate()` |
| Read CSV | `spark.read.csv("file.csv", header=True)` |
| Show data | `df.show()` |
| Show schema | `df.printSchema()` |
| Count rows | `df.count()` |

---

# ⏭️ NEXT CLASS

**Class 6: Spark ETL - Transform & Write**
- Filter rows
- Add/modify columns
- Write to Parquet

**Preparation:** Keep your SparkSession code ready
