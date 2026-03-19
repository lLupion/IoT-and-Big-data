# Class 4: Python for Spark (OOP)

| Class | Duration | Project Milestone |
|-------|----------|-------------------|
| 4 of 15 | 1 hour | Create SensorReading class for project |

## Learning Objectives
- [ ] Create classes with attributes and methods
- [ ] Import and use modules
- [ ] Organize code across files

## Prerequisites
- Class 3: Python Basics

## Recall from Class 3
You wrote functions like `score_to_grade()`. Now we'll organize related functions into classes.

---

# 📖 INSTRUCTOR-LED

## 1. Classes and Objects

```python
class Student:
    def __init__(self, name: str, student_id: int):
        """Constructor - called when creating instance"""
        self.name = name
        self.student_id = student_id
        self.scores = {}
    
    def add_score(self, subject: str, score: int):
        """Add a score for a subject"""
        self.scores[subject] = score
    
    def get_average(self) -> float:
        """Calculate average score"""
        if not self.scores:
            return 0.0
        return sum(self.scores.values()) / len(self.scores)

# Create instances
alice = Student("Alice", 1001)
alice.add_score("Math", 90)
alice.add_score("English", 85)
print(alice.get_average())  # 87.5

bob = Student("Bob", 1002)
bob.add_score("Math", 75)
print(bob.get_average())  # 75.0
```

### Key Concepts

| Term | Meaning |
|------|---------|
| `class` | Blueprint for objects |
| `self` | Reference to current instance |
| `__init__` | Constructor method |
| Instance | Object created from class |

---

## 2. Project Class: SensorReading

```python
class SensorReading:
    def __init__(self, module_id: str, temperature: float, humidity: float):
        self.module_id = module_id
        self.temperature = temperature
        self.humidity = humidity
    
    def is_valid(self) -> bool:
        """Check if reading is within valid ranges"""
        temp_valid = -50 <= self.temperature <= 100
        humid_valid = 0 <= self.humidity <= 100
        return temp_valid and humid_valid
    
    def to_dict(self) -> dict:
        """Convert to dictionary (useful for Spark)"""
        return {
            "module_id": self.module_id,
            "temperature": self.temperature,
            "humidity": self.humidity
        }

# Test
reading = SensorReading("sensor_01", 25.5, 60.0)
print(reading.is_valid())   # True
print(reading.to_dict())    # {'module_id': 'sensor_01', ...}

bad_reading = SensorReading("sensor_02", 150.0, 50.0)
print(bad_reading.is_valid())  # False
```

### ✅ Checkpoint
What would `SensorReading("s1", -60, 50).is_valid()` return?

---

## 3. Modules and Imports

```python
# File: utils/grading.py
def score_to_grade(score: int) -> str:
    if score >= 80: return "A"
    elif score >= 70: return "B"
    else: return "F"

# File: utils/sensor.py
class SensorReading:
    ...

# File: main.py
from utils.grading import score_to_grade
from utils.sensor import SensorReading

grade = score_to_grade(85)
reading = SensorReading("s1", 25.0, 60.0)
```

### Project Structure

```
my_project/
├── utils/
│   ├── __init__.py    # Makes it a package
│   ├── grading.py
│   └── sensor.py
└── main.py
```

---

# ✏️ STUDENT PRACTICE

## Exercise 1: Complete the Student Class

```python
class Student:
    def __init__(self, name: str, student_id: int):
        self.name = name
        self.student_id = student_id
        self.scores = {}
    
    def add_score(self, subject: str, score: int):
        self.scores[subject] = score
    
    def get_average(self) -> float:
        # YOUR CODE HERE
        pass
    
    def get_grade(self) -> str:
        """Return grade based on average: A(80+), B(70+), C(60+), F"""
        # YOUR CODE HERE
        pass

# Test
s = Student("Test", 1)
s.add_score("Math", 85)
s.add_score("English", 75)
print(s.get_average())  # Expected: 80.0
print(s.get_grade())    # Expected: A
```

<details>
<summary>💡 Solution</summary>

```python
def get_average(self) -> float:
    if not self.scores:
        return 0.0
    return sum(self.scores.values()) / len(self.scores)

def get_grade(self) -> str:
    avg = self.get_average()
    if avg >= 80: return "A"
    elif avg >= 70: return "B"
    elif avg >= 60: return "C"
    else: return "F"
```
</details>

---

## Exercise 2: Extend SensorReading

Add a method to categorize temperature:

```python
class SensorReading:
    def __init__(self, module_id: str, temperature: float, humidity: float):
        self.module_id = module_id
        self.temperature = temperature
        self.humidity = humidity
    
    def get_temp_status(self) -> str:
        """Return: 'COLD' (<15), 'NORMAL' (15-30), 'HOT' (>30)"""
        # YOUR CODE HERE
        pass

# Test
print(SensorReading("s1", 10, 50).get_temp_status())   # COLD
print(SensorReading("s2", 25, 50).get_temp_status())   # NORMAL
print(SensorReading("s3", 35, 50).get_temp_status())   # HOT
```

<details>
<summary>💡 Solution</summary>

```python
def get_temp_status(self) -> str:
    if self.temperature < 15:
        return "COLD"
    elif self.temperature <= 30:
        return "NORMAL"
    else:
        return "HOT"
```
</details>

---

## Exercise 3: Create a Module

1. Create file `sensor_utils.py` with:
   - `SensorReading` class
   - Function `validate_reading(reading) -> bool`

2. Create `main.py` that imports and uses them

<details>
<summary>💡 Solution</summary>

```python
# sensor_utils.py
class SensorReading:
    def __init__(self, module_id: str, temperature: float, humidity: float):
        self.module_id = module_id
        self.temperature = temperature
        self.humidity = humidity

def validate_reading(reading: SensorReading) -> bool:
    return -50 <= reading.temperature <= 100

# main.py
from sensor_utils import SensorReading, validate_reading

r = SensorReading("s1", 25.0, 60.0)
print(validate_reading(r))  # True
```
</details>

---

# 📝 QUICK CHECK

1. What is `self` in a class method?
   - a) The class name
   - b) Reference to current instance
   - c) A reserved variable

2. What file makes a folder a Python package?
   - a) `main.py`
   - b) `__init__()`
   - c) `__init__.py`

3. How do you import a class from a module?
   - a) `import MyClass from module`
   - b) `from module import MyClass`
   - c) `include module.MyClass`

<details>
<summary>Answers</summary>
1. b) Reference to current instance
2. c) `__init__.py`
3. b) `from module import MyClass`
</details>

---

# 📋 SUMMARY

| Concept | Example |
|---------|---------|
| Class | `class Student:` |
| Constructor | `def __init__(self, name):` |
| Method | `def get_average(self):` |
| Instance | `alice = Student("Alice")` |
| Import | `from module import Class` |

---

# ⏭️ NEXT CLASS

**Class 5: Spark ETL - Reading Data**
- Create SparkSession
- Read CSV files
- Understand DataFrames

**Preparation:** Ensure PySpark is installed: `pip install pyspark`
