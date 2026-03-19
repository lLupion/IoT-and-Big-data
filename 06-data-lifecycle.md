# Class 8: Data Lifecycle

| Class | Duration | Project Milestone |
|-------|----------|-------------------|
| 8 of 15 | 1 hour | Design data flow: raw → staged → analytics |

## Learning Objectives
By the end of this lesson, you will be able to:
- [ ] Identify data sources and their characteristics
- [ ] Design data layers (raw, staged, analytics)
- [ ] Choose appropriate storage solutions
- [ ] Consider backup and recovery strategies

## Prerequisites
- Lessons 03-05 completed
- Understanding of ETL concepts

## Why This Matters
Without proper data architecture, pipelines become spaghetti code. Data layers separate concerns: raw data is preserved, staged data is cleaned, analytics data is business-ready. This makes debugging, auditing, and scaling much easier.

## Recall from Lesson 4-5
You wrote ETL code to read CSV, transform, and write Parquet. Now we'll design WHERE that data lives and HOW it flows through the system.

---

# 📖 INSTRUCTOR-LED SECTION

## 1. Thinking Corner: Data in Daily Life

Data lifecycle exists everywhere. Think about a photo on your smartphone:

| Stage | Photo Example | Sensor Data Example |
|-------|---------------|---------------------|
| Created | Photo taken | Sensor records temperature |
| Stored | Saved to phone | Saved to local buffer |
| Transferred | Uploaded to cloud | Sent to central server |
| Processed | Cropped, filtered | Cleaned, validated |
| Archived | Moved to cold storage | Moved to data lake |

**Discussion:** What happens to data in these scenarios?
- A student record in a university
- Medication inventory in a hospital
- Smoke concentration from IoT sensors

---

## 2. Source Systems

### Where Does Data Come From?

| Source Type | Example | Characteristics |
|-------------|---------|-----------------|
| Relational DB | MySQL, PostgreSQL | Structured, ACID compliant |
| APIs | REST, GraphQL | Real-time, rate-limited |
| Files | CSV, Excel, JSON | Batch, manual uploads |
| IoT/Streaming | MQTT, Kafka | High volume, continuous |

![image-20250127-152857.png](05-data-lifecycle-images/image-20250127-152857.png)
*(Fundamentals of Data Engineering, Reis & Housley)*

### Key Considerations

| Factor | Questions to Ask |
|--------|------------------|
| **Persistence** | Is data deleted quickly? Need CDC? |
| **Volume** | How much data per day/hour/second? |
| **Velocity** | Batch (daily) or streaming (real-time)? |
| **Quality** | Are there errors, duplicates, nulls? |
| **Schema** | Does schema change over time? |

---

## 3. Data Layer Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   SOURCE    │ ──▶ │     RAW     │ ──▶ │   STAGED    │ ──▶ │  ANALYTICS  │
│   SYSTEMS   │     │  (Bronze)   │     │  (Silver)   │     │   (Gold)    │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                          │                   │                    │
                    Exact copy          Cleaned &            Aggregated &
                    from source         validated            business-ready
```

### Layer Definitions

| Layer | Purpose | Format | Retention |
|-------|---------|--------|-----------|
| **Raw** | Preserve original data | As-is (CSV, JSON) | Long-term |
| **Staged** | Clean, validate, standardize | Parquet | Medium-term |
| **Analytics** | Business aggregations | Parquet | As needed |

Reference: [AWS Data Layer Definitions](https://docs.aws.amazon.com/prescriptive-guidance/latest/defining-bucket-names-data-lakes/data-layer-definitions.html)

### ✅ Checkpoint 1
Draw the data flow for our sensor project:
- Source: IoT sensors → MQTT → CSV files
- Raw: Store CSV as-is
- Staged: Clean + validate + Parquet
- Analytics: Daily averages per sensor

---

## 4. Storage Solutions

### Choosing the Right Storage

| Use Case | Access Pattern | Recommended Storage |
|----------|----------------|---------------------|
| Hot data (frequent access) | Daily queries | S3 Standard, local SSD |
| Warm data (occasional) | Weekly/monthly | S3 Standard-IA |
| Cold data (archive) | Rarely accessed | S3 Glacier |

### AWS S3 Storage Classes

| Class | Use Case | Cost |
|-------|----------|------|
| S3 Standard | Frequently accessed | $$$ |
| S3 Standard-IA | Infrequent access | $$ |
| S3 Glacier | Archive (minutes to retrieve) | $ |
| S3 Glacier Deep Archive | Long-term archive (hours) | ¢ |

Reference: [AWS S3 Storage Classes](https://aws.amazon.com/s3/storage-classes/)

---

## 5. Backup and Recovery

### Key Metrics

| Metric | Definition | Example |
|--------|------------|---------|
| **RPO** (Recovery Point Objective) | Max acceptable data loss | 1 hour = lose up to 1 hour of data |
| **RTO** (Recovery Time Objective) | Max acceptable downtime | 4 hours = system back in 4 hours |

### Backup Strategy Questions

1. How many copies? (Rule of 3-2-1: 3 copies, 2 media types, 1 offsite)
2. How often? (Depends on RPO)
3. How to recover? (Automated vs manual)
4. How long to recover? (Depends on RTO)

---

# ✏️ STUDENT PRACTICE SECTION

## Exercise 1: Design Data Layers

For our sensor data project, define what happens at each layer:

| Layer | Input | Transformations | Output |
|-------|-------|-----------------|--------|
| Raw | CSV from sensors | ??? | ??? |
| Staged | Raw Parquet | ??? | ??? |
| Analytics | Staged Parquet | ??? | ??? |

<details>
<summary>💡 Solution</summary>

| Layer | Input | Transformations | Output |
|-------|-------|-----------------|--------|
| Raw | CSV from sensors | None (preserve as-is) | Parquet (raw/) |
| Staged | Raw Parquet | Filter nulls, validate ranges, add timestamps | Parquet (staged/) |
| Analytics | Staged Parquet | Aggregate by day/sensor, calculate stats | Parquet (analytics/) |
</details>

---

## Exercise 2: Implement Raw to Staged

Write a PySpark job that moves data from raw to staged:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, current_timestamp

spark = SparkSession.builder.appName("RawToStaged").getOrCreate()

# Read raw data
df_raw = spark.read.parquet("raw/sensors/")

# YOUR TASKS:
# 1. Filter out rows where temperature is NULL
# 2. Filter out rows where temperature > 100 or < -50
# 3. Filter out rows where humidity > 100 or < 0
# 4. Add a column 'processed_at' with current timestamp
# 5. Write to staged/sensors/ as Parquet

df_staged = df_raw  # Transform this!

df_staged.write.mode("overwrite").parquet("staged/sensors/")
```

<details>
<summary>💡 Solution</summary>

```python
df_staged = df_raw \
    .filter(col("temperature").isNotNull()) \
    .filter((col("temperature") >= -50) & (col("temperature") <= 100)) \
    .filter((col("humidity") >= 0) & (col("humidity") <= 100)) \
    .withColumn("processed_at", current_timestamp())

df_staged.write.mode("overwrite").parquet("staged/sensors/")
```
</details>

---

## Exercise 3: Implement Staged to Analytics

Write a PySpark job that creates daily aggregations:

```python
from pyspark.sql.functions import avg, max, min, count, to_date

df_staged = spark.read.parquet("staged/sensors/")

# YOUR TASKS:
# 1. Extract date from timestamp column
# 2. Group by module_id and date
# 3. Calculate: avg_temp, max_temp, min_temp, reading_count
# 4. Write to analytics/daily_summary/

# Expected output columns:
# module_id, date, avg_temp, max_temp, min_temp, reading_count
```

<details>
<summary>💡 Solution</summary>

```python
df_analytics = df_staged \
    .withColumn("date", to_date(col("timestamp"))) \
    .groupBy("module_id", "date") \
    .agg(
        avg("temperature").alias("avg_temp"),
        max("temperature").alias("max_temp"),
        min("temperature").alias("min_temp"),
        count("*").alias("reading_count")
    )

df_analytics.write.mode("overwrite").parquet("analytics/daily_summary/")
```
</details>

---

## Exercise 4: Storage Decision

Choose the appropriate S3 storage class for each scenario:

| Scenario | Storage Class | Why? |
|----------|---------------|------|
| Real-time sensor data (queried hourly) | ??? | ??? |
| Monthly compliance reports | ??? | ??? |
| 10-year-old audit logs (legal requirement) | ??? | ??? |

<details>
<summary>💡 Solution</summary>

| Scenario | Storage Class | Why? |
|----------|---------------|------|
| Real-time sensor data | S3 Standard | Frequent access, low latency needed |
| Monthly compliance reports | S3 Standard-IA | Infrequent but needs quick access |
| 10-year-old audit logs | S3 Glacier Deep Archive | Rarely accessed, cost optimization |
</details>

---

## Quick Check

1. What is the purpose of the "Raw" data layer?
   - a) Store cleaned data
   - b) Preserve original data as-is
   - c) Store aggregated metrics

2. What does RPO stand for?
   - a) Recovery Point Objective
   - b) Recovery Process Order
   - c) Raw Processing Output

3. Which layer should contain business-ready aggregations?
   - a) Raw
   - b) Staged
   - c) Analytics

<details>
<summary>Answers</summary>

1. b) Preserve original data as-is
2. a) Recovery Point Objective
3. c) Analytics
</details>

---

## Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| Data loss in raw layer | Transforming before saving raw | Always save raw first |
| Schema mismatch | Source schema changed | Use schema evolution or versioning |
| Storage costs exploding | Wrong storage class | Review access patterns, use lifecycle policies |

---

## Summary

| Concept | Key Point |
|---------|-----------|
| Data Layers | Raw → Staged → Analytics |
| Raw | Preserve original, no transformations |
| Staged | Cleaned, validated, standardized |
| Analytics | Aggregated, business-ready |
| Storage | Match storage class to access pattern |
| Backup | Define RPO and RTO |

---

## What's Next?

In **Lesson 7**, we'll learn about Joining Data. You'll combine sensor readings with location metadata to enrich your analytics.

**Preparation:** Think about what additional data would make sensor readings more useful (location, sensor type, calibration date, etc.)
