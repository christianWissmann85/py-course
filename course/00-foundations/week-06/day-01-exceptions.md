# Week 6, Day 1: Exception Handling

## 🎯 Learning Objectives
- [ ] Understand the concept of exceptions for error handling.
- [ ] Use `try...except` blocks to gracefully handle runtime errors.
- [ ] Use the `finally` and `else` clauses to execute code under specific conditions.
- [ ] Understand the basics of Python's exception hierarchy.

## 📚 Concepts

### 1. What is an Exception?
An exception is an event, which occurs during the execution of a program, that disrupts the normal flow of the program's instructions. When a Python script raises an exception, it must either be handled immediately or it will terminate the program.

Instead of writing code that checks for every possible error condition with `if` statements, we can write the "happy path" code and handle the exceptions when they occur. This is often described as "It's easier to ask for forgiveness than permission" (EAFP), as opposed to "Look before you leap" (LBYL).

### 2. The `try...except` Block
The core of exception handling is the `try...except` block.
- **`try`**: The code that might raise an exception is placed inside the `try` block.
- **`except`**: If an exception occurs in the `try` block, Python looks for a matching `except` block to handle it.

```python
try:
    # This might cause a ZeroDivisionError
    result = 10 / 0
except ZeroDivisionError as e:
    # This block runs only if a ZeroDivisionError occurs
    print(f"Error: Cannot divide by zero. ({e})")

print("Program continues after the error.")
```
You can catch multiple exceptions by listing them in a tuple: `except (ValueError, TypeError):`. You can also have multiple `except` blocks to handle different exceptions differently.

### 3. The `else` and `finally` Clauses
The `try` statement can have two optional final clauses:
- **`else`**: The code in the `else` block is executed only if the `try` block completes **without raising an exception**. This is useful for code that should run only when the `try` part was successful.
- **`finally`**: The code in the `finally` block is **always executed**, whether an exception occurred or not. This is perfect for cleanup actions, like closing a file or a network connection.

```python
def safe_get_from_dict(d: dict, key: str):
    try:
        value = d[key]
    except KeyError:
        print(f"Key '{key}' not found.")
    else:
        print(f"Successfully retrieved value: {value}")
    finally:
        print("--- Attempt finished ---")

my_dict = {"a": 1}
safe_get_from_dict(my_dict, "a") # Runs `else` and `finally`
safe_get_from_dict(my_dict, "b") # Runs `except` and `finally`
```

### 4. Python's Exception Hierarchy
Exceptions are objects that inherit from a base class, `BaseException`. Most exceptions you'll deal with inherit from the `Exception` class. Some common ones:
- `Exception`
  - `AttributeError`: Raised when an attribute reference or assignment fails.
  - `KeyError`: Raised when a dictionary key is not found.
  - `IndexError`: Raised when a sequence subscript is out of range.
  - `TypeError`: Raised when an operation or function is applied to an object of inappropriate type.
  - `ValueError`: Raised when an operation receives an argument of the right type but an inappropriate value.
  - `ZeroDivisionError`: Raised when the second argument of a division or modulo operation is zero.

You can catch a parent exception to handle all of its child exceptions. For example, `except LookupError:` will catch both `KeyError` and `IndexError`.

### 5. Exception Chaining
When you catch an exception and raise a new one, Python 3 automatically chains them. The original exception is stored in the `__cause__` attribute of the new exception. This provides valuable context when debugging.

```python
def process_data():
    try:
        # Some operation that fails
        int("abc")
    except ValueError as e:
        # Raise a new, more specific exception, but chain the original
        raise TypeError("Invalid data type found during processing") from e

try:
    process_data()
except TypeError as e:
    print(f"Caught error: {e}")
    print(f"Original cause: {e.__cause__}")
```

## 🔹 Quick Exercise

Complete the `safe_divide` function below. It should handle both `ZeroDivisionError` and `TypeError` (if non-numeric types are passed).

```python
from typing import Union

def safe_divide(a: Union[int, float], b: Union[int, float]) -> float | None:
    """
    Divides a by b, handling potential errors.
    Returns the result as a float, or None if an error occurs.
    """
    try:
        # Attempt the division
        result = a / b
    except ZeroDivisionError:
        # Handle the case where b is zero
        print("Error: Cannot divide by zero.")
        return None
    except TypeError:
        # Handle the case where a or b are not numbers
        print("Error: Both inputs must be numbers.")
        return None
    else:
        # This will run if no exception occurred
        print("Division successful.")
        return result
    finally:
        # This will always run
        print(f"Finished attempt to divide {a} by {b}.")

# --- Tests ---
assert safe_divide(10, 2) == 5.0
assert safe_divide(10, 0) is None
# The type checker should ideally catch this, but we're testing runtime handling
assert safe_divide(10, "a") is None
```

## 📝 Daily Assignment
**Goal**: Refactor a simple script to make it robust by adding comprehensive error handling.

1.  **Create Project File**: In your `my-first-poetry-app` project, create a new file: `my_first_poetry_app/robust_script.py`.
2.  **Start with "Fragile" Code**:
    - Write a function `process_user_data(data: list[dict]) -> None`.
    - This function will take a list of user dictionaries, where each dict is supposed to have a `"name"` (str) and `"age"` (int).
    - The function should loop through the list and print a summary for each user, like `f"{user['name']} will be {user['age'] + 5} in five years."`
3.  **Add Error Handling**:
    - The input data might be messy. Use `try...except` blocks inside your loop to handle potential errors for each user dictionary *without* crashing the entire loop.
    - Handle `KeyError` if the `"name"` or `"age"` key is missing.
    - Handle `TypeError` or `ValueError` if the `age` is not a number.
    - In each `except` block, print a helpful error message indicating which user record failed and why.
4.  **Create a `main` function**:
    - Create a list of "messy" user data. Include correct records, records with missing keys, and records with bad data types.
      ```python
      sample_data = [
          {"name": "Alice", "age": 30},
          {"name": "Bob"}, # Missing age
          {"name": "Charlie", "age": "twenty-five"}, # Invalid age type
          {"username": "Diana", "age": 40}, # Missing name
      ]
      ```
    - Call `process_user_data` with your sample data.
5.  **Verify**: The script should run to completion without crashing, printing success messages for valid records and error messages for invalid ones. Ensure it also passes `mypy` and `ruff`.

## ⚠️ Common Mistakes
- **Catching `Exception` too broadly**: `except Exception:` will catch almost every possible error. This can hide bugs and make it hard to debug. It's almost always better to catch specific exceptions (`except ValueError:`) so you know exactly what went wrong.
- **Ignoring the exception**: `except: pass` is one of the most dangerous patterns in Python. It silently swallows the error, leaving you with no idea that something went wrong. At the very least, log the error.
- **Confusing `TypeError` and `ValueError`**:
  - `TypeError`: The *type* of the object is wrong for the operation (e.g., `len(123)`).
  - `ValueError`: The *type* is right, but the *value* is wrong (e.g., `int("abc")`).
- **Putting too much code in the `try` block**: Only include the specific lines of code that might raise the exception you want to handle. Code that can't raise that exception should be outside the `try` block (often in the `else` block).

## 📖 Further Reading
- [Python Docs: Errors and Exceptions](https://docs.python.org/3/tutorial/errors.html)
- [Real Python: Python `try...except`](https://realpython.com/python-try-except/)
- [Python Docs: Built-in Exceptions](https://docs.python.org/3/library/exceptions.html#exception-hierarchy) (The full hierarchy)
