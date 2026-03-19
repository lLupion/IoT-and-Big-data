# Class 9: Joining Data

| Class | Duration | Project Milestone |
|-------|----------|-------------------|
| 9 of 15 | 1 hour | Join sensor readings with location metadata |

## Learning Objectives
By the end of this lesson, you will be able to:
- [ ] Explain different types of SQL joins
- [ ] Apply joins in PySpark
- [ ] Choose the appropriate join type for different scenarios

## Prerequisites
- Lesson 06: Data Lifecycle
- Understanding of relational data concepts

## Why This Matters
Real-world data is rarely in one table. Sensor readings need location info. Orders need customer details. Joins combine data from multiple sources to create complete pictures.

## Recall from Lesson 6
You designed data layers for sensor data. Now imagine you have a separate table with sensor locations. How do you combine them?

---

# 📖 INSTRUCTOR-LED SECTION

## 1. Types of Joins

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│   INNER JOIN       LEFT JOIN        RIGHT JOIN       FULL OUTER    │
│                                                                     │
│    ┌───┬───┐       ┌───┬───┐        ┌───┬───┐       ┌───┬───┐     │
│    │ A │ B │       │ A │ B │        │ A │ B │       │ A │ B │     │
│    │░░░│   │       │███│   │        │   │███│       │███│███│     │
│    │░░░│░░░│       │███│░░░│        │░░░│███│       │███│███│     │
│    │   │░░░│       │   │░░░│        │░░░│███│       │███│███│     │
│    └───┴───┘       └───┴───┘        └───┴───┘       └───┴───┘     │
│                                                                     │
│   Only matching    All A +          All B +         All from       │
│   rows             matching B       matching A      both tables    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

Reference: [SQL Join Types Explained Visually](https://www.atlassian.com/data/sql/sql-join-types-explained-visually)

---

## 2. Sample Data

**Sensors Table (df_sensors)**

| module_id | temperature | timestamp |
|-----------|-------------|-----------|
| sensor_01 | 25.3 | 2025-01-20 |
| sensor_02 | 26.1 | 2025-01-20 |
| sensor_03 | 24.8 | 2025-01-20 |

**Locations Table (df_locations)**

| module_id | building | floor |
|-----------|----------|-------|
| sensor_01 | Building A | 1 |
| sensor_02 | Building A | 2 |
| sensor_04 | Building B | 1 |

Notice: `sensor_03` has no location, `sensor_04` has no readings.

---

## 3. Join Syntax in PySpark

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("Joins").getOrCreate()

# Create sample DataFrames
sensors = [("sensor_01", 25.3), ("sensor_02", 26.1), ("sensor_03", 24.8)]
df_sensors = spark.createDataFrame(sensors, ["module_id", "temperature"])

locations = [("sensor_01", "Building A", 1), ("sensor_02", "Building A", 2), ("sensor_04", "Building B", 1)]
df_locations = spark.createDataFrame(locations, ["module_id", "building", "floor"])
```

### Inner Join (Default)

```python
df_inner = df_sensors.join(df_locations, "module_id", "inner")
df_inner.show()
```

**Result:**
| module_id | temperature | building | floor |
|-----------|-------------|----------|-------|
| sensor_01 | 25.3 | Building A | 1 |
| sensor_02 | 26.1 | Building A | 2 |

❌ `sensor_03` dropped (no location)
❌ `sensor_04` dropped (no readings)

### Left Join

```python
df_left = df_sensors.join(df_locations, "module_id", "left")
df_left.show()
```

**Result:**
| module_id | temperature | building | floor |
|-----------|-------------|----------|-------|
| sensor_01 | 25.3 | Building A | 1 |
| sensor_02 | 26.1 | Building A | 2 |
| sensor_03 | 24.8 | NULL | NULL |

✅ All sensors kept
❌ `sensor_04` dropped

### Right Join

```python
df_right = df_sensors.join(df_locations, "module_id", "right")
df_right.show()
```

**Result:**
| module_id | temperature | building | floor |
|-----------|-------------|----------|-------|
| sensor_01 | 25.3 | Building A | 1 |
| sensor_02 | 26.1 | Building A | 2 |
| sensor_04 | NULL | Building B | 1 |

❌ `sensor_03` dropped
✅ All locations kept

### Full Outer Join

```python
df_outer = df_sensors.join(df_locations, "module_id", "outer")
df_outer.show()
```

**Result:**
| module_id | temperature | building | floor |
|-----------|-------------|----------|-------|
| sensor_01 | 25.3 | Building A | 1 |
| sensor_02 | 26.1 | Building A | 2 |
| sensor_03 | 24.8 | NULL | NULL |
| sensor_04 | NULL | Building B | 1 |

✅ All sensors kept
✅ All locations kept

### ✅ Checkpoint 1
What is the default join type in Spark? (Answer: inner)

---

## 4. Choosing the Right Join

| Scenario | Join Type | Why |
|----------|-----------|-----|
| Only want complete records | Inner | Drops incomplete data |
| Keep all from primary table | Left | Primary table is "left" |
| Keep all from lookup table | Right | Lookup table is "right" |
| Can't lose any data | Full Outer | Keeps everything |
| All combinations | Cross | Cartesian product (careful!) |

---

## 5. Advanced: Self-Join

Join a table to itself. Useful for hierarchical data.

```python
# Employees with manager info
employees = [
    (1, "Alice", None),    # CEO, no manager
    (2, "Bob", 1),         # Reports to Alice
    (3, "Carol", 1),       # Reports to Alice
    (4, "Dave", 2),        # Reports to Bob
]
df_emp = spark.createDataFrame(employees, ["id", "name", "manager_id"])

# Self-join to get manager names
df_with_manager = df_emp.alias("e").join(
    df_emp.alias("m"),
    col("e.manager_id") == col("m.id"),
    "left"
).select(
    col("e.name").alias("employee"),
    col("m.name").alias("manager")
)
```

**Result:**
| employee | manager |
|----------|---------|
| Alice | NULL |
| Bob | Alice |
| Carol | Alice |
| Dave | Bob |

---

# ✏️ STUDENT PRACTICE SECTION

## Exercise 1: Basic Joins

Given these tables:

**Students**
| student_id | name | class_of |
|------------|------|----------|
| 1 | John | 2024 |
| 2 | Jane | 2025 |
| 3 | Sarah | 2025 |
| 4 | Oliver | 2026 |

**Advisors**
| advisor | class_of |
|---------|----------|
| Dr. Hill | 2024 |
| Dr. Jung | 2025 |

```python
students = [(1, "John", 2024), (2, "Jane", 2025), (3, "Sarah", 2025), (4, "Oliver", 2026)]
df_students = spark.createDataFrame(students, ["student_id", "name", "class_of"])

advisors = [("Dr. Hill", 2024), ("Dr. Jung", 2025)]
df_advisors = spark.createDataFrame(advisors, ["advisor", "class_of"])

# YOUR TASKS:
# 1. Inner join - how many rows?
# 2. Left join - what happens to Oliver?
# 3. Which join keeps all students?
```

<details>
<summary>💡 Solution</summary>

```python
# 1. Inner join - 3 rows (John, Jane, Sarah)
df_students.join(df_advisors, "class_of", "inner").show()

# 2. Left join - Oliver has NULL advisor
df_students.join(df_advisors, "class_of", "left").show()

# 3. Left join keeps all students
```
</details>

---

## Exercise 2: Enrich Sensor Data

Join sensor readings with location metadata:

```python
# Sensor readings
readings = [
    ("sensor_01", 25.3, "2025-01-20"),
    ("sensor_02", 26.1, "2025-01-20"),
    ("sensor_03", 24.8, "2025-01-20"),
    ("sensor_01", 25.8, "2025-01-21"),
]
df_readings = spark.createDataFrame(readings, ["module_id", "temperature", "date"])

# Location metadata
locations = [
    ("sensor_01", "Building A", "Room 101", 1),
    ("sensor_02", "Building A", "Room 201", 2),
]
df_locations = spark.createDataFrame(locations, ["module_id", "building", "room", "floor"])

# YOUR TASKS:
# 1. Join readings with locations (keep all readings)
# 2. Select: date, building, room, temperature
# 3. Filter to only Building A
# 4. How many readings have no location info?
```

<details>
<summary>💡 Solution</summary>

```python
# 1. Left join to keep all readings
df_enriched = df_readings.join(df_locations, "module_id", "left")

# 2. Select columns
df_result = df_enriched.select("date", "building", "room", "temperature")

# 3. Filter to Building A
df_building_a = df_result.filter(col("building") == "Building A")
df_building_a.show()

# 4. Count readings with no location
no_location = df_enriched.filter(col("building").isNull()).count()
print(f"Readings without location: {no_location}")  # Answer: 2
```
</details>

---

## Exercise 3: Find Missing Data

Use joins to find sensors without readings and readings without sensors:

```python
# Which sensors have no readings? (anti-join)
df_no_readings = df_locations.join(df_readings, "module_id", "left_anti")

# Which readings have no sensor info? (anti-join)
df_no_sensor = df_readings.join(df_locations, "module_id", "left_anti")
```

**Question:** What does `left_anti` join return?

<details>
<summary>💡 Answer</summary>

`left_anti` returns rows from the left table that have NO match in the right table. It's useful for finding missing data.
</details>

---

## Quick Check

1. What is the default join type in PySpark?
   - a) Left
   - b) Inner
   - c) Outer

2. Which join keeps all rows from both tables?
   - a) Inner
   - b) Left
   - c) Full Outer

3. What happens to unmatched rows in an inner join?
   - a) Filled with NULL
   - b) Dropped
   - c) Duplicated

4. When would you use a self-join?
   - a) Joining two different tables
   - b) Hierarchical data (employee-manager)
   - c) Aggregating data

<details>
<summary>Answers</summary>

1. b) Inner
2. c) Full Outer
3. b) Dropped
4. b) Hierarchical data
</details>

---

## Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| Duplicate columns | Same column name in both tables | Use `alias()` or specify columns |
| Cartesian product | Missing join condition | Always specify join key |
| Unexpected row count | Wrong join type | Check with small sample first |
| NULL values | Unmatched rows in outer join | Handle NULLs with `coalesce()` |

---

## Summary

| Join Type | Keeps From Left | Keeps From Right | Use Case |
|-----------|-----------------|------------------|----------|
| Inner | Only matched | Only matched | Complete records only |
| Left | All | Only matched | Keep primary table |
| Right | Only matched | All | Keep lookup table |
| Full Outer | All | All | Can't lose any data |
| Cross | All × All | All × All | All combinations |
| Left Anti | Unmatched only | - | Find missing data |

---

## What's Next?

In **Lesson 8**, we'll learn about CI/CD (Continuous Integration/Continuous Deployment). You'll automate testing and deployment of your ETL code using GitHub Actions.

**Preparation:** Create a GitHub account if you don't have one.
