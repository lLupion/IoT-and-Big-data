# Class 7: Test-Driven Development

| Class | Duration | Project Milestone |
|-------|----------|-------------------|
| 7 of 15 | 1 hour | Add tests for sensor data transformations |

## Learning Objectives
By the end of this lesson, you will be able to:
- [ ] Explain the TDD workflow: Red → Green → Refactor
- [ ] Write unit tests using Python's `unittest`
- [ ] Test PySpark DataFrames using `chispa`

## Prerequisites
- Lesson 04: Spark ETL Basics
- Install: `pip install pytest chispa`

## Why This Matters
Data pipelines fail silently. A bug might produce wrong numbers that go unnoticed for weeks. Tests catch these bugs early and give you confidence to refactor code without breaking things.

## Recall from Lesson 4
You wrote transformations like filtering invalid temperatures and adding status columns. How do you know they work correctly? Tests!

---

# 📖 INSTRUCTOR-LED SECTION

## 1. What is Test-Driven Development?

> **TDD** is writing tests BEFORE writing code. The cycle is:
> 1. 🔴 **Red**: Write a failing test
> 2. 🟢 **Green**: Write minimal code to pass
> 3. 🔄 **Refactor**: Clean up while keeping tests green

[https://en.wikipedia.org/wiki/Test-driven_development](https://en.wikipedia.org/wiki/Test-driven_development)

### Why TDD for Data Engineering?

| Without TDD | With TDD |
|-------------|----------|
| "I think it works" | "Tests prove it works" |
| Bugs found in production | Bugs found immediately |
| Fear of changing code | Confidence to refactor |

---

## 2. Defining a Unit Test

A unit test needs:
1. **Input**: What data goes in?
2. **Expected Output**: What should come out?
3. **Assertion**: Does actual match expected?

### Example: Grade Assignment

**Input:**

| student_id | name | score |
|------------|------|-------|
| 1 | John | 90 |
| 2 | Jane | 72 |

**Expected Output:**

| student_id | name | score | grade |
|------------|------|-------|-------|
| 1 | John | 90 | A |
| 2 | Jane | 72 | B |

---

## 3. Python's unittest Module

```python
import unittest

def score_to_grade(score: int) -> str:
    if score >= 80:
        return "A"
    elif score >= 70:
        return "B"
    else:
        return "F"

class TestScoreToGrade(unittest.TestCase):
    
    def test_grade_a(self):
        self.assertEqual(score_to_grade(90), "A")
        self.assertEqual(score_to_grade(80), "A")
    
    def test_grade_b(self):
        self.assertEqual(score_to_grade(79), "B")
        self.assertEqual(score_to_grade(70), "B")
    
    def test_grade_f(self):
        self.assertEqual(score_to_grade(50), "F")

if __name__ == "__main__":
    unittest.main()
```

**Run it:**
```bash
python test_grades.py
```

**Expected Output:**
```
...
----------------------------------------------------------------------
Ran 3 tests in 0.001s

OK
```

### ✅ Checkpoint 1
Run the test above. Do all 3 tests pass?

---

## 4. Testing PySpark with Chispa

DataFrame comparison is complex. `chispa` makes it easy:

```bash
pip install chispa
```

```python
import unittest
from pyspark.sql import SparkSession
from chispa.dataframe_comparer import assert_df_equality

class TestSensorETL(unittest.TestCase):
    
    @classmethod
    def setUpClass(cls):
        """Create SparkSession once for all tests"""
        cls.spark = SparkSession.builder \
            .master("local[*]") \
            .appName("Testing") \
            .getOrCreate()
    
    @classmethod
    def tearDownClass(cls):
        """Stop SparkSession after all tests"""
        cls.spark.stop()
    
    def test_filter_invalid_temperature(self):
        # ARRANGE: Create input data
        input_data = [
            ("sensor_01", 25.0),
            ("sensor_02", 3000.0),  # Invalid!
            ("sensor_03", 30.0),
        ]
        input_df = self.spark.createDataFrame(input_data, ["module_id", "temperature"])
        
        # ARRANGE: Define expected output
        expected_data = [
            ("sensor_01", 25.0),
            ("sensor_03", 30.0),
        ]
        expected_df = self.spark.createDataFrame(expected_data, ["module_id", "temperature"])
        
        # ACT: Run the transformation
        from pyspark.sql.functions import col
        result_df = input_df.filter(col("temperature") < 100)
        
        # ASSERT: Compare DataFrames
        assert_df_equality(result_df, expected_df, ignore_row_order=True)

if __name__ == "__main__":
    unittest.main()
```

---

## 5. TDD Workflow Demo

Let's build a function using TDD:

### Step 1: 🔴 Write Failing Test

```python
def test_add_status_column(self):
    input_data = [("s1", 35.0), ("s2", 25.0), ("s3", 5.0)]
    input_df = self.spark.createDataFrame(input_data, ["id", "temp"])
    
    expected_data = [("s1", 35.0, "HOT"), ("s2", 25.0, "NORMAL"), ("s3", 5.0, "COLD")]
    expected_df = self.spark.createDataFrame(expected_data, ["id", "temp", "status"])
    
    result_df = add_status(input_df)  # Function doesn't exist yet!
    
    assert_df_equality(result_df, expected_df)
```

**Run:** Test fails with `NameError: name 'add_status' is not defined`

### Step 2: 🟢 Write Minimal Code to Pass

```python
from pyspark.sql.functions import col, when

def add_status(df):
    return df.withColumn("status",
        when(col("temp") > 30, "HOT")
        .when(col("temp") < 10, "COLD")
        .otherwise("NORMAL")
    )
```

**Run:** Test passes! ✅

### Step 3: 🔄 Refactor (if needed)

Code is clean, no refactoring needed.

---

# ✏️ STUDENT PRACTICE SECTION

## Exercise 1: Test the Grade Function

Add tests for edge cases:

```python
class TestScoreToGrade(unittest.TestCase):
    
    def test_boundary_80(self):
        """Score of exactly 80 should be A"""
        # YOUR CODE HERE
        pass
    
    def test_boundary_70(self):
        """Score of exactly 70 should be B"""
        # YOUR CODE HERE
        pass
    
    def test_zero(self):
        """Score of 0 should be F"""
        # YOUR CODE HERE
        pass
```

<details>
<summary>💡 Solution</summary>

```python
def test_boundary_80(self):
    self.assertEqual(score_to_grade(80), "A")

def test_boundary_70(self):
    self.assertEqual(score_to_grade(70), "B")

def test_zero(self):
    self.assertEqual(score_to_grade(0), "F")
```
</details>

---

## Exercise 2: Test Sensor Validation

Write a test for a function that validates sensor readings:
- Temperature must be between -50 and 100
- Humidity must be between 0 and 100
- Invalid rows should be removed

```python
def test_validate_sensor_readings(self):
    input_data = [
        ("s1", 25.0, 60.0),   # Valid
        ("s2", 150.0, 50.0),  # Invalid temp
        ("s3", 20.0, 120.0),  # Invalid humidity
        ("s4", 30.0, 70.0),   # Valid
    ]
    input_df = self.spark.createDataFrame(input_data, ["id", "temp", "humidity"])
    
    expected_data = [
        ("s1", 25.0, 60.0),
        ("s4", 30.0, 70.0),
    ]
    expected_df = self.spark.createDataFrame(expected_data, ["id", "temp", "humidity"])
    
    # YOUR CODE: Implement validate_readings() function
    result_df = validate_readings(input_df)
    
    assert_df_equality(result_df, expected_df, ignore_row_order=True)
```

<details>
<summary>💡 Solution</summary>

```python
from pyspark.sql.functions import col

def validate_readings(df):
    return df.filter(
        (col("temp") >= -50) & (col("temp") <= 100) &
        (col("humidity") >= 0) & (col("humidity") <= 100)
    )
```
</details>

---

## Exercise 3: TDD Challenge

Using TDD, create a function `categorize_air_quality` that:
- Takes a DataFrame with `gas_concentration` column
- Adds `air_quality` column:
  - "GOOD" if gas_concentration < 0.03
  - "MODERATE" if 0.03 <= gas_concentration < 0.06
  - "POOR" if gas_concentration >= 0.06

**Step 1:** Write the test first (it will fail)
**Step 2:** Implement the function
**Step 3:** Run test until it passes

<details>
<summary>💡 Solution</summary>

```python
# Test
def test_categorize_air_quality(self):
    input_data = [(0.01,), (0.04,), (0.08,)]
    input_df = self.spark.createDataFrame(input_data, ["gas_concentration"])
    
    expected_data = [(0.01, "GOOD"), (0.04, "MODERATE"), (0.08, "POOR")]
    expected_df = self.spark.createDataFrame(expected_data, ["gas_concentration", "air_quality"])
    
    result_df = categorize_air_quality(input_df)
    assert_df_equality(result_df, expected_df)

# Implementation
def categorize_air_quality(df):
    return df.withColumn("air_quality",
        when(col("gas_concentration") < 0.03, "GOOD")
        .when(col("gas_concentration") < 0.06, "MODERATE")
        .otherwise("POOR")
    )
```
</details>

---

## Quick Check

1. In TDD, what comes first?
   - a) Write code, then test
   - b) Write test, then code
   - c) Write documentation

2. What does `assert_df_equality` do?
   - a) Checks if DataFrames have same schema
   - b) Checks if DataFrames have same data
   - c) Both schema and data

3. What color represents a failing test in TDD?
   - a) Green
   - b) Red
   - c) Yellow

<details>
<summary>Answers</summary>

1. b) Write test, then code
2. c) Both schema and data
3. b) Red
</details>

---

## Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `DataFramesNotEqualError` | Row order differs | Add `ignore_row_order=True` |
| `SchemasNotEqualError` | Column types differ | Check schema with `printSchema()` |
| `AssertionError` | Expected != Actual | Print both to debug |

---

## Summary

| Concept | Key Point |
|---------|-----------|
| TDD Cycle | Red → Green → Refactor |
| Unit Test | Input + Expected Output + Assertion |
| chispa | `assert_df_equality(actual, expected)` |
| Best Practice | Test edge cases and boundaries |

---

## What's Next?

In **Lesson 6**, we'll learn about Data Lifecycle - how data flows from source systems through raw, staged, and analytics layers. You'll design the architecture for our sensor data pipeline.

**Preparation:** Think about where sensor data comes from and where it needs to go.
