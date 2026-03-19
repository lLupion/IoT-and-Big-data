# Class 10: CI/CD for Data Pipelines

| Class | Duration | Project Milestone |
|-------|----------|-------------------|
| 10 of 15 | 1 hour | Set up GitHub Actions for automated testing |

## Learning Objectives
By the end of this lesson, you will be able to:
- [ ] Explain CI/CD concepts and benefits
- [ ] Set up GitHub Actions for automated testing
- [ ] Structure a Python/PySpark project for CI/CD

## Prerequisites
- Lessons 03-07 completed
- GitHub account
- Git installed locally

## Why This Matters
Manual deployments are error-prone. "It works on my machine" is not a deployment strategy. CI/CD automates testing and deployment, catching bugs early and ensuring consistent releases.

## Recall from Lesson 5
You wrote unit tests for your ETL functions. Now we'll run those tests automatically every time you push code.

---

# 📖 INSTRUCTOR-LED SECTION

## 1. What is CI/CD?

| Term | Meaning | Example |
|------|---------|---------|
| **CI** | Continuous Integration | Auto-run tests on every push |
| **CD** | Continuous Delivery | Auto-deploy to staging/production |

### The CI/CD Pipeline

```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│  Code   │ ──▶ │  Test   │ ──▶ │  Build  │ ──▶ │ Deploy  │
│  Push   │     │  (CI)   │     │  (CI)   │     │  (CD)   │
└─────────┘     └─────────┘     └─────────┘     └─────────┘
                    │
              ❌ Fail = Stop
              ✅ Pass = Continue
```

Reference: [https://en.wikipedia.org/wiki/CI/CD](https://en.wikipedia.org/wiki/CI/CD)

---

## 2. Why CI/CD for Data Engineering?

| Without CI/CD | With CI/CD |
|---------------|------------|
| "I think it works" | Tests prove it works |
| Manual deployment | Automated deployment |
| "Works on my machine" | Works everywhere |
| Fear of changes | Confidence to refactor |
| Bugs in production | Bugs caught early |

### Data Pipelines Fit CI/CD Because:

- ETL scripts are modular (independent functions)
- Functions can be unit tested
- Scripts can be added/removed without affecting others

---

## 3. Project Structure

```
my-etl-project/
├── .github/
│   └── workflows/
│       └── ci.yml          # GitHub Actions config
├── src/
│   └── etl/
│       ├── __init__.py
│       ├── transformations.py
│       └── validations.py
├── tests/
│   ├── __init__.py
│   ├── test_transformations.py
│   └── test_validations.py
├── pyproject.toml          # Package config
├── requirements.txt        # Dependencies
└── .gitignore
```

---

## 4. GitHub Actions Workflow

Create `.github/workflows/ci.yml`:

```yaml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Set up Python
      uses: actions/setup-python@v5
      with:
        python-version: '3.10'
    
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
        pip install pytest
    
    - name: Run tests
      run: pytest tests/ -v
```

### Workflow Triggers

| Trigger | When |
|---------|------|
| `push` | Code pushed to branch |
| `pull_request` | PR opened/updated |
| `schedule` | Cron schedule |
| `workflow_dispatch` | Manual trigger |

### ✅ Checkpoint 1
Create a new GitHub repository and add the workflow file above.

---

## 5. Example: Complete CI Setup

**requirements.txt:**
```
pyspark==3.5.0
chispa==0.9.4
pytest==8.0.0
```

**src/etl/transformations.py:**
```python
from pyspark.sql.functions import col, when

def add_temperature_status(df):
    """Add status column based on temperature."""
    return df.withColumn("status",
        when(col("temperature") > 30, "HOT")
        .when(col("temperature") < 10, "COLD")
        .otherwise("NORMAL")
    )

def filter_valid_readings(df):
    """Remove invalid temperature readings."""
    return df.filter(
        (col("temperature") >= -50) & 
        (col("temperature") <= 100)
    )
```

**tests/test_transformations.py:**
```python
import pytest
from pyspark.sql import SparkSession
from chispa.dataframe_comparer import assert_df_equality
from src.etl.transformations import add_temperature_status, filter_valid_readings

@pytest.fixture(scope="session")
def spark():
    return SparkSession.builder.master("local[*]").getOrCreate()

def test_add_temperature_status(spark):
    input_data = [(35.0,), (25.0,), (5.0,)]
    input_df = spark.createDataFrame(input_data, ["temperature"])
    
    expected_data = [(35.0, "HOT"), (25.0, "NORMAL"), (5.0, "COLD")]
    expected_df = spark.createDataFrame(expected_data, ["temperature", "status"])
    
    result_df = add_temperature_status(input_df)
    assert_df_equality(result_df, expected_df)

def test_filter_valid_readings(spark):
    input_data = [(25.0,), (150.0,), (-100.0,)]
    input_df = spark.createDataFrame(input_data, ["temperature"])
    
    expected_data = [(25.0,)]
    expected_df = spark.createDataFrame(expected_data, ["temperature"])
    
    result_df = filter_valid_readings(input_df)
    assert_df_equality(result_df, expected_df)
```

---

## 6. CI/CD Workflow in Practice

```
Developer                    GitHub                      Production
    │                           │                            │
    │  1. Write code + tests    │                            │
    │ ─────────────────────────▶│                            │
    │                           │  2. CI runs tests          │
    │                           │ ◀────────────────          │
    │  3. See results           │                            │
    │ ◀─────────────────────────│                            │
    │                           │                            │
    │  4. Merge to main         │                            │
    │ ─────────────────────────▶│                            │
    │                           │  5. CD deploys             │
    │                           │ ──────────────────────────▶│
```

---

# ✏️ STUDENT PRACTICE SECTION

## Exercise 1: Create CI Workflow

1. Create a new GitHub repository
2. Add the project structure shown above
3. Create the CI workflow file
4. Push and verify the workflow runs

**Verification:** Go to Actions tab in GitHub and see the workflow run.

---

## Exercise 2: Add a New Test

Add a test for a new function `celsius_to_fahrenheit`:

```python
# src/etl/transformations.py
def celsius_to_fahrenheit(df):
    """Convert temperature from Celsius to Fahrenheit."""
    return df.withColumn("temp_f", col("temperature") * 9/5 + 32)

# tests/test_transformations.py
def test_celsius_to_fahrenheit(spark):
    # YOUR CODE HERE
    # Input: [(0,), (100,)]
    # Expected: [(0, 32.0), (100, 212.0)]
    pass
```

<details>
<summary>💡 Solution</summary>

```python
def test_celsius_to_fahrenheit(spark):
    input_data = [(0.0,), (100.0,)]
    input_df = spark.createDataFrame(input_data, ["temperature"])
    
    expected_data = [(0.0, 32.0), (100.0, 212.0)]
    expected_df = spark.createDataFrame(expected_data, ["temperature", "temp_f"])
    
    result_df = celsius_to_fahrenheit(input_df)
    assert_df_equality(result_df, expected_df)
```
</details>

---

## Exercise 3: Break the Build

1. Intentionally write a failing test
2. Push to GitHub
3. Observe the CI failure
4. Fix the test
5. Push again and see it pass

This demonstrates the "safety net" of CI.

---

## Discussion: Is CI/CD Worth It?

![image-20250206-170815.png](07-ci-cd-images/image-20250206-170815.png)

### Time Investment Analysis

| Task | Time per occurrence | Frequency | 5-year total |
|------|---------------------|-----------|--------------|
| Manual SSH deploy | 5 min | 3x/week | 65 hours |
| Manual testing | 15 min | 5x/week | 325 hours |
| CI/CD setup | 4 hours | Once | 4 hours |

**Conclusion:** CI/CD pays off quickly for any project that lives more than a few weeks.

---

## Quick Check

1. What does CI stand for?
   - a) Code Integration
   - b) Continuous Integration
   - c) Complete Installation

2. When does a GitHub Actions workflow run with `on: push`?
   - a) Only on manual trigger
   - b) Every time code is pushed
   - c) Only on pull requests

3. What happens if a test fails in CI?
   - a) Deployment continues anyway
   - b) Pipeline stops, code not merged
   - c) Test is skipped

<details>
<summary>Answers</summary>

1. b) Continuous Integration
2. b) Every time code is pushed
3. b) Pipeline stops, code not merged
</details>

---

## Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| Workflow not running | Wrong file path | Must be `.github/workflows/*.yml` |
| Python not found | Missing setup step | Add `actions/setup-python` |
| Module not found | Dependencies not installed | Add `pip install -r requirements.txt` |
| Tests pass locally, fail in CI | Environment differences | Use same Python version |

---

## Summary

| Concept | Key Point |
|---------|-----------|
| CI | Automatically test on every push |
| CD | Automatically deploy after tests pass |
| GitHub Actions | Free CI/CD for GitHub repos |
| Workflow | YAML file defining CI/CD steps |
| Benefits | Catch bugs early, consistent deployments |

---

## What's Next?

In **Lesson 9**, we'll learn about Orchestration with Apache Airflow. You'll schedule your ETL jobs to run automatically and manage dependencies between tasks.

**Preparation:** Install Docker (Airflow runs in containers)
