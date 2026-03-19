# Class 3: Python Basics

| Class | Duration | Project Milestone |
|-------|----------|-------------------|
| 3 of 15 | 1 hour | Write basic Python functions |

## Learning Objectives
- [ ] Declare variables with appropriate types
- [ ] Use operators and control flow
- [ ] Write simple functions

## Prerequisites
- Classes 1-2 completed
- Python 3.8+ installed

---

# 🔄 SETUP RECAP

Before we start, let's verify everyone's environment:

```python
# Quick check - run this in your terminal
python --version        # Should be 3.8+
pip show pyspark        # Should show pyspark installed
```

### ✅ Checkpoint: Environment Ready?
- [ ] Python 3.8+ installed
- [ ] VS Code connected to VM (or local setup)
- [ ] Can run `python` in terminal

**If issues:** Pair with a neighbor who has it working.

---

# 📖 INSTRUCTOR-LED

## 1. Variables and Data Types

```python
# Variables store data
name = "John"           # String
age = 25                # Integer  
gpa = 3.75              # Float
is_student = True       # Boolean

# Check type
print(type(name))       # <class 'str'>
```

### Type Hints

```python
def greet(name: str) -> str:
    return f"Hello, {name}!"
```

Reference: [Python Data Types](https://www.w3schools.com/python/python_datatypes.asp)

---

## 2. Operators

```python
# Arithmetic
10 + 5    # 15
10 - 5    # 5
10 * 5    # 50
10 / 5    # 2.0
10 % 3    # 1 (modulo)

# Comparison
10 == 10  # True
10 != 5   # True
10 > 5    # True

# Logical
True and False  # False
True or False   # True
not True        # False
```

---

## 3. Control Flow

```python
# If-else
score = 85
if score >= 80:
    grade = "A"
elif score >= 70:
    grade = "B"
else:
    grade = "C"
print(grade)  # A

# For loop
for i in range(3):
    print(i)  # 0, 1, 2

# While loop
count = 0
while count < 3:
    print(count)
    count += 1
```

---

## 4. Functions

```python
def add(a: int, b: int) -> int:
    """Add two numbers."""
    return a + b

result = add(5, 3)
print(result)  # 8

# Default parameters
def greet(name: str, greeting: str = "Hello") -> str:
    return f"{greeting}, {name}!"

print(greet("Alice"))           # Hello, Alice!
print(greet("Bob", "Hi"))       # Hi, Bob!
```

### ✅ Checkpoint
Create a function that takes a number and returns "even" or "odd".

---

# ✏️ STUDENT PRACTICE

## Exercise 1: Temperature Converter

```python
def celsius_to_fahrenheit(celsius: float) -> float:
    """Convert Celsius to Fahrenheit. Formula: F = C × 9/5 + 32"""
    # YOUR CODE HERE
    pass

# Test
print(celsius_to_fahrenheit(0))    # Expected: 32.0
print(celsius_to_fahrenheit(100))  # Expected: 212.0
```

<details>
<summary>💡 Solution</summary>

```python
def celsius_to_fahrenheit(celsius: float) -> float:
    return celsius * 9/5 + 32
```
</details>

---

## Exercise 2: Grade Calculator

```python
def score_to_grade(score: int) -> str:
    """
    Convert score to grade:
    80-100: A, 70-79: B, 60-69: C, below 60: F
    """
    # YOUR CODE HERE
    pass

# Test
print(score_to_grade(85))  # Expected: A
print(score_to_grade(72))  # Expected: B
print(score_to_grade(55))  # Expected: F
```

<details>
<summary>💡 Solution</summary>

```python
def score_to_grade(score: int) -> str:
    if score >= 80:
        return "A"
    elif score >= 70:
        return "B"
    elif score >= 60:
        return "C"
    else:
        return "F"
```
</details>

---

## Exercise 3: List Operations

```python
numbers = [1, 2, 3, 4, 5]

# YOUR TASKS:
# 1. Print the sum of all numbers
# 2. Print the average
# 3. Print only even numbers using a loop
```

<details>
<summary>💡 Solution</summary>

```python
print(sum(numbers))                    # 15
print(sum(numbers) / len(numbers))     # 3.0
for n in numbers:
    if n % 2 == 0:
        print(n)                       # 2, 4
```
</details>

---

# 📝 QUICK CHECK

1. What is the output of `10 % 3`?
   - a) 3.33
   - b) 1
   - c) 3

2. What does `->` indicate in a function?
   - a) Assignment
   - b) Return type hint
   - c) Arrow function

3. Which loop is best when you know the number of iterations?
   - a) while
   - b) for
   - c) do-while

<details>
<summary>Answers</summary>
1. b) 1
2. b) Return type hint
3. b) for
</details>

---

# 📋 SUMMARY

| Concept | Example |
|---------|---------|
| Variable | `name = "John"` |
| Type hint | `def func(x: int) -> str:` |
| If-else | `if x > 0: ... elif: ... else:` |
| For loop | `for i in range(n):` |
| Function | `def add(a, b): return a + b` |

---

# ⏭️ NEXT CLASS

**Class 4: Python for Spark (OOP)**
- Classes and objects
- Modules and imports
- Building reusable code

**Preparation:** Review today's function exercises
