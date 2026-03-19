# Class 6: Spark ETL - Transform & Write

| Class | Duration | Project Milestone |
|-------|----------|-------------------|
| 6 of 15 | 1 hour | Clean sensor data and write to Parquet |

## Learning Objectives
- [ ] Filter rows based on conditions
- [ ] Add and modify columns
- [ ] Write DataFrames to Parquet format

## Prerequisites
- Class 5: Reading data into Spark

## Recall from Class 5
You read sensor data into a DataFrame. Now we'll clean it (remove invalid readings) and save it.

---

# 📖 INSTRUCTOR-LED

## 1. Setup: Create Sample Data

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, when, lit

spark = SparkSession.builder.appName("Transform").getOrCreate()

data = [
    ("2025-01-20", 25.3, 60.5, "sensor_01"),
    ("2025-01-20", 26.1, 58.2, "sensor_02"),
    ("2025-01-20", None, 55.0, "sensor_03"),   # Missing!
    ("2025-01-20", 150.0, 50.0, "sensor_04"),  # Invalid!
    ("2025-01-21", 24.8, 62.1, "sensor_01"),
]
df = spark.createDataFrame(data, ["date", "temperature", "humidity", "module_id"])
df.show()
```

---

## 2. Filtering Rows

### Single Condition

```python
# Keep only valid temperatures
df_valid = df.filter(col("temperature") <= 100)
df_valid.show()
```

### Multiple Conditions

```python
# Valid temperature AND not null
df_clean = df.filter(
    (col("temperature").isNotNull()) &
    (col("temperature") <= 100)
)
df_clean.show()
```

### SQL-Style Filter

```python
df_clean = df.filter("temperature IS NOT NULL AND temperature <= 100")
```

### ✅ Checkpoint 1
How many rows remain after filtering? (Answer: 3)

---

## 3. Adding/Modifying Columns

### Add Constant Column

```python
df = df.withColumn("unit", lit("celsius"))
```

### Calculate New Column

```python
df = df.withColumn("temp_fahrenheit", col("temperature") * 9/5 + 32)
```

### Conditional Column

```python
df = df.withColumn("status",
    when(col("temperature") > 30, "HOT")
    .when(col("temperature") < 15, "COLD")
    .otherwise("NORMAL")
)
df.show()
```

**Output:**
```
+----------+-----------+--------+-----------+-------+---------------+------+
|      date|temperature|humidity|  module_id|   unit|temp_fahrenheit|status|
+----------+-----------+--------+-----------+-------+---------------+------+
|2025-01-20|       25.3|    60.5|  sensor_01|celsius|          77.54|NORMAL|
|2025-01-20|       26.1|    58.2|  sensor_02|celsius|          78.98|NORMAL|
...
```

---

## 4. Writing to Parquet

```python
# Basic write
df_clean.write.parquet("output/sensors")

# Overwrite existing
df_clean.write.mode("overwrite").parquet("output/sensors")

# Partition by column
df_clean.write.mode("overwrite") \
    .partitionBy("module_id") \
    .parquet("output/sensors_partitioned")
```

### Why Parquet?

| Feature | CSV | Parquet |
|---------|-----|---------|
| Compression | ❌ | ✅ High |
| Schema | ❌ | ✅ Embedded |
| Read speed | Slow | Fast |
| Column pruning | ❌ | ✅ |

---

# ✏️ STUDENT PRACTICE

## Exercise 1: Clean Sensor Data

Starting with the sample data:

```python
data = [
    ("2025-01-20", 25.3, 60.5, "sensor_01"),
    ("2025-01-20", 26.1, 58.2, "sensor_02"),
    ("2025-01-20", None, 55.0, "sensor_03"),
    ("2025-01-20", 150.0, 50.0, "sensor_04"),
    ("2025-01-20", -100.0, 45.0, "sensor_05"),
    ("2025-01-21", 24.8, 62.1, "sensor_01"),
]
df = spark.createDataFrame(data, ["date", "temperature", "humidity", "module_id"])

# YOUR TASKS:
# 1. Filter: temperature is NOT NULL
# 2. Filter: temperature between -50 and 100
# 3. Filter: humidity between 0 and 100
# 4. Show result and count rows
```

<details>
<summary>💡 Solution</summary>

```python
df_clean = df.filter(
    (col("temperature").isNotNull()) &
    (col("temperature") >= -50) &
    (col("temperature") <= 100) &
    (col("humidity") >= 0) &
    (col("humidity") <= 100)
)
df_clean.show()
print(f"Clean rows: {df_clean.count()}")  # 3
```
</details>

---

## Exercise 2: Add Status Column

Add a `temp_status` column:
- "COLD" if temperature < 20
- "WARM" if 20 <= temperature <= 30
- "HOT" if temperature > 30

```python
# YOUR CODE HERE
df_with_status = df_clean  # Add the column

df_with_status.select("module_id", "temperature", "temp_status").show()
```

**Expected:**
```
+-----------+-----------+-----------+
|  module_id|temperature|temp_status|
+-----------+-----------+-----------+
|  sensor_01|       25.3|       WARM|
|  sensor_02|       26.1|       WARM|
|  sensor_01|       24.8|       WARM|
+-----------+-----------+-----------+
```

<details>
<summary>💡 Solution</summary>

```python
df_with_status = df_clean.withColumn("temp_status",
    when(col("temperature") < 20, "COLD")
    .when(col("temperature") <= 30, "WARM")
    .otherwise("HOT")
)
```
</details>

---

## Exercise 3: Complete ETL Pipeline

Combine everything into one pipeline:

```python
# Complete ETL: Read → Clean → Transform → Write
df_final = df \
    .filter(col("temperature").isNotNull()) \
    .filter((col("temperature") >= -50) & (col("temperature") <= 100)) \
    .withColumn("temp_status",
        when(col("temperature") < 20, "COLD")
        .when(col("temperature") <= 30, "WARM")
        .otherwise("HOT")
    )

# Write to parquet
df_final.write.mode("overwrite").parquet("output/sensors_clean")
print("ETL Complete!")
```

---

# 📝 QUICK CHECK

1. Which method adds a new column?
   - a) `addColumn()`
   - b) `withColumn()`
   - c) `newColumn()`

2. What does `mode("overwrite")` do?
   - a) Appends data
   - b) Replaces existing data
   - c) Fails if exists

3. Which format is best for Spark output?
   - a) CSV
   - b) JSON
   - c) Parquet

<details>
<summary>Answers</summary>
1. b) `withColumn()`
2. b) Replaces existing data
3. c) Parquet
</details>

---

# 📋 SUMMARY

| Operation | Code |
|-----------|------|
| Filter | `df.filter(col("x") > 10)` |
| Add column | `df.withColumn("new", lit("value"))` |
| Conditional | `when(cond, val).otherwise(val)` |
| Write | `df.write.parquet("path")` |
| Overwrite | `df.write.mode("overwrite")` |

---

# ⏭️ NEXT CLASS

**Class 7: Test-Driven Development**
- Write tests BEFORE code
- Test DataFrames with chispa
- TDD workflow: Red → Green → Refactor

**Preparation:** Install test libraries: `pip install pytest chispa`
