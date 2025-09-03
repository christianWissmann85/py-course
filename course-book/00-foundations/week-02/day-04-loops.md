# Week 2, Day 4: Loops & Iteration

## 🎯 Learning Objectives

- [ ] Write type-safe `for` and `while` loops.
- [ ] Understand the concept of iterables and how Python's `for` loop uses them.
- [ ] Use `enumerate()` and `zip()` to write cleaner, more Pythonic loops.
- [ ] Understand and use `break`, `continue`, and the `else` clause in loops.

## 📚 Concepts

### 1. `for` Loops with Types

The `for` loop in Python iterates over the items of any sequence or iterable. When using type hints, the type checker knows the type of the item being iterated over.

```python
names: list[str] = ["Alice", "Bob", "Charlie"]

# MyPy knows that `name` is a `str` inside this loop.
for name in names:
    print(f"Hello, {name.upper()}")
```

### 2. `while` Loops Safely

A `while` loop repeats as long as a condition is true. It's powerful but carries the risk of creating an infinite loop if the condition never becomes false.

```python
import time

# This loop will run for 5 seconds
start_time: float = time.monotonic()
while time.monotonic() - start_time < 5:
    print("Still looping...")
    time.sleep(1)

# A more common pattern is using a counter
counter: int = 5
while counter > 0:
    print(f"Countdown: {counter}")
    counter -= 1 # Crucial step to prevent an infinite loop!
```

### 3. Iterables and Sequences

An **iterable** is any Python object capable of returning its members one at a time. Examples include all sequence types (like `list`, `str`, and `tuple`) and some non-sequence types (like `dict` and `set`). The `for` loop can iterate over any iterable.

A **sequence** is a specific type of iterable that supports efficient element access using integer indices via the `__getitem__()` special method. `list`, `tuple`, `range` and `str` are examples.

### 4. `enumerate`, `zip`, and `range`

These are essential built-in functions for writing better loops.

- **`range(stop)` or `range(start, stop, step)`**: Generates a sequence of numbers, perfect for looping a specific number of times.
  ```python
  for i in range(5): # Loops with i = 0, 1, 2, 3, 4
      print(i)
  ```
- **`enumerate(iterable)`**: Use this when you need both the index and the item during iteration.
  ```python
  names: list[str] = ["Alice", "Bob", "Charlie"]
  for index, name in enumerate(names):
      print(f"{index}: {name}")
  # Output:
  # 0: Alice
  # 1: Bob
  # 2: Charlie
  ```
- **`zip(*iterables)`**: Use this to loop over two or more iterables at the same time. The loop stops when the shortest iterable is exhausted.
  ```python
  names: list[str] = ["Alice", "Bob"]
  ages: list[int] = [30, 25]
  for name, age in zip(names, ages):
      print(f"{name} is {age} years old.")
  # Output:
  # Alice is 30 years old.
  # Bob is 25 years old.
  ```

### 5. `break`, `continue`, and `else` in Loops

These statements give you more control over your loops.

- **`break`**: Immediately exits the innermost loop.
- **`continue`**: Skips the rest of the current iteration and proceeds to the next one.
- **`else`**: The `else` block runs only if the loop completes _without_ encountering a `break` statement. This is a unique and useful feature of Python's loops.

```python
# Example combining break and else
numbers: list[int] = [1, 3, 5, 7, 8, 9, 11]
for num in numbers:
    if num % 2 == 0:
        print(f"Found an even number: {num}")
        break  # Exit the loop
else:
    # This block only runs if the loop finishes (no break)
    print("No even numbers were found.")

# Example of continue
for i in range(10):
    if i % 2 != 0:
        continue # Skip the print for odd numbers
    print(i) # 0, 2, 4, 6, 8
```

## 🔹 Quick Exercise

Implement the following two functions using the concepts you've learned.

```python
from typing import List, Tuple

def sum_evens(numbers: list[int]) -> int:
    """
    Sums only the even numbers in a list.
    """
    # Implement here using a for loop and an if statement.
    pass

def parallel_lists(list1: list[str], list2: list[int]) -> list[tuple[str, int]]:
    """
    Combines two lists into a list of tuples.
    Example: parallel_lists(['a', 'b'], [1, 2]) -> [('a', 1), ('b', 2)]
    """
    # Implement here using the zip() function and a list comprehension (or a loop).
    pass


# --- Tests ---
assert sum_evens([1, 2, 3, 4, 5, 6]) == 12
assert sum_evens([10, 1, 1, 1]) == 10
assert parallel_lists(['x', 'y', 'z'], [10, 20, 30]) == [('x', 10), ('y', 20), ('z', 30)]
assert parallel_lists(['a'], [1, 2, 3]) == [('a', 1)] # zip stops at the shortest list
```

## 📝 Daily Assignment

**Goal**: Implement several classic algorithms using loops.

1.  **Create Project File**: In `my-first-poetry-app`, create a new file: `my_first_poetry_app/algorithms.py`.
2.  **Implement 5 Algorithms**: Write 5 functions, each implementing a simple algorithm. Ensure all are fully type-annotated.
    - **Suggestions**:
      - `find_max(numbers: list[int]) -> int | None`: Finds the maximum value in a list. Return `None` if the list is empty.
      - `linear_search(items: list[str], target: str) -> int | None`: Finds the index of a target string in a list. Return `None` if not found. Use `enumerate`.
      - `reverse_list(items: list[int]) -> list[int]`: Returns a new list with the items in reverse order. Do not use the built-in `reverse()` method or `[::-1]` slicing. Use a `while` loop or a `for` loop with `range`.
      - `is_prime(n: int) -> bool`: Checks if a number is prime. Use a `for` loop with `break` and `else`. A number `n` is prime if it's not divisible by any number from 2 to `sqrt(n)`.
      - `factorial(n: int) -> int`: Calculates the factorial of a non-negative integer using a loop.
3.  **Create a `main` function**:
    - Write a `main()` function that demonstrates each of your algorithm functions.
    - Use a variety of inputs to test them.
4.  **Verify**: Run your script (`poetry run python ...`), type checker (`poetry run mypy .`), and linter (`poetry run ruff check .`) and fix all issues.

## ⚠️ Common Mistakes

- **Modifying a list while iterating over it**: This can lead to unpredictable behavior, like skipping items.
  ```python
  # Wrong - can lead to bugs!
  numbers = [1, 2, 3, 4]
  for num in numbers:
      if num % 2 == 0:
          numbers.remove(num) # Modifying the list you're looping over
  ```
  Instead, create a new list or iterate over a _copy_ (`for num in numbers[:]`).
- **Off-by-one errors**: When using `range`, it's easy to be off by one. Remember `range(5)` goes from 0 to 4. `range(len(my_list))` is a common pattern, but `enumerate` is often cleaner.
- **Unnecessary `while` loops**: `while` loops are powerful, but `for` loops are generally safer and more readable for iterating over existing data. Use `while` when the number of iterations isn't known beforehand.
- **Forgetting the `else` on a loop is tied to `break`**: The `else` block on a loop is not like the `else` in an `if` statement. It only runs if the loop terminates _naturally_ (not via `break`). This is a common point of confusion.

## 📖 Further Reading

- [Real Python: Python "for" Loops (Definite Iteration)](https://realpython.com/python-for-loop/)
- [Real Python: Python "while" Loops (Indefinite Iteration)](https://realpython.com/python-while-loop/)
- [Trey Hunner: The `for...else` statement is a great, underused Python feature](https://treyhunner.com/2019/05/python-for-else/)
- [Real Python: Python's `zip()` Function for Parallel Iteration](https://realpython.com/python-zip-function/)
