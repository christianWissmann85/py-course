# Week 4, Day 5: Lambda & Higher-Order Functions

## 🎯 Learning Objectives
- [ ] Write small, anonymous functions using the `lambda` keyword.
- [ ] Understand the concept of higher-order functions.
- [ ] Use the built-in `map`, `filter`, and `reduce` functions for functional-style data processing.
- [ ] Learn basic function composition and partial application patterns.

## 📚 Concepts

### 1. Lambda Expressions
A `lambda` function is a small, anonymous function. It can take any number of arguments, but can only have one expression. The result of this expression is what the function returns. Lambdas are syntactically restricted and cannot contain statements like `if` or `for`.

```python
# A regular function
def add(x: int, y: int) -> int:
    return x + y

# The equivalent lambda function
add_lambda = lambda x, y: x + y

print(add(2, 3))       # 5
print(add_lambda(2, 3)) # 5
```
Lambdas are most useful when you need a short, throwaway function for another function, like a sorting key.

### 2. Higher-Order Functions
A **higher-order function** is a function that does at least one of the following:
- Takes one or more functions as arguments.
- Returns a function as its result.

Decorators, which we learned about on Day 3, are a prime example of higher-order functions. `map`, `filter`, and `reduce` are also classic examples.

### 3. `map`, `filter`, and `reduce`
These are the three classic higher-order functions for processing iterables.

- **`map(function, iterable)`**: Applies a function to every item of an iterable and returns a `map` object (an iterator) of the results.
  ```python
  numbers = [1, 2, 3, 4]
  squared = map(lambda x: x * x, numbers)
  print(list(squared)) # [1, 4, 9, 16]
  ```
- **`filter(function, iterable)`**: Tests each element of an iterable with a function. It returns a `filter` object (an iterator) with the elements for which the function returned `True`.
  ```python
  numbers = [1, 2, 3, 4, 5, 6]
  evens = filter(lambda x: x % 2 == 0, numbers)
  print(list(evens)) # [2, 4, 6]
  ```
- **`functools.reduce(function, iterable)`**: Applies a rolling computation to sequential pairs of values in an iterable. It takes a function that accepts two arguments.
  ```python
  from functools import reduce
  numbers = [1, 2, 3, 4]
  # reduce(lambda x, y: x + y, numbers) calculates (((1+2)+3)+4)
  total = reduce(lambda x, y: x + y, numbers)
  print(total) # 10
  ```
While powerful, `map` and `filter` can often be replaced by more readable list or generator comprehensions. `reduce` can be less readable for simple cases like `sum()`.

### 4. Partial Application
Partial application allows you to "freeze" some portion of a function's arguments, producing a new function with a simplified signature. `functools.partial` is the tool for this.

```python
from functools import partial

def power(base: float, exponent: float) -> float:
    return base ** exponent

# Create a new function 'square' where the exponent is always 2
square = partial(power, exponent=2)

# Create a new function 'cube' where the exponent is always 3
cube = partial(power, exponent=3)

print(square(5)) # 25.0
print(cube(5))   # 125.0
```
This is useful for creating specialized versions of general functions.

### 5. Function Composition
Function composition is the act of combining simple functions to build more complicated ones. In mathematics, `f(g(x))` is the composition of functions `f` and `g`.

```python
def double(x: int) -> int:
    return x * 2

def add_one(x: int) -> int:
    return x + 1

# To get double_and_add_one, we compose them:
def double_and_add_one(x: int) -> int:
    return add_one(double(x))

print(double_and_add_one(5)) # 11  (5 * 2 = 10, 10 + 1 = 11)
```
Writing small, pure, composable functions is a cornerstone of the functional programming style.

## 🔹 Quick Exercise

1.  **Implement `compose`**: Write a higher-order function `compose` that takes two functions, `f` and `g`, and returns a new function that represents their composition, `f(g(x))`.
2.  **Use `map`, `filter`, `reduce`**: Given a list of numbers, use a combination of `map`, `filter`, and `reduce` to find the sum of the squares of only the even numbers.

```python
from functools import reduce
from typing import Callable, TypeVar

T = TypeVar('T')

# 1. Implement compose
def compose(f: Callable[[T], T], g: Callable[[T], T]) -> Callable[[T], T]:
    """Returns a new function that is the composition of f and g."""
    # Return a lambda function that takes x and applies g, then f.
    pass

# --- Test for compose ---
def double(x): return x * 2
def add_five(x): return x + 5

double_then_add_five = compose(add_five, double)
assert double_then_add_five(10) == 25 # (10 * 2) + 5

# 2. Use map, filter, reduce
numbers = [1, 2, 3, 4, 5, 6]
# Expected result: 4*4 + 2*2 + 6*6 = 16 + 4 + 36 = 56
# Hint: First filter for evens, then map to squares, then reduce with sum.
evens = filter(lambda n: n % 2 == 0, numbers)
squares = map(lambda n: n * n, evens)
total = reduce(lambda a, b: a + b, squares)
assert total == 56
```

## 📝 Daily Assignment
**Goal**: Build a data processing pipeline using only functional programming concepts.

1.  **Create Project File**: In `my-first-poetry-app`, create `my_first_poetry_app/functional_pipeline.py`.
2.  **Define Data**: Start with a list of dictionaries representing user data.
    ```python
    users = [
        {"name": "Alice", "age": 30, "is_active": True, "logins": 150},
        {"name": "Bob", "age": 22, "is_active": False, "logins": 30},
        {"name": "Charlie", "age": 28, "is_active": True, "logins": 200},
        {"name": "Diana", "age": 45, "is_active": True, "logins": 50},
    ]
    ```
3.  **Implement a Pipeline**:
    - **Task**: Find the total number of logins for all active users over the age of 25.
    - **Constraint**: You are **not allowed to use `for` or `while` loops**. You must solve this by chaining `map`, `filter`, and `reduce`.
    - **Steps**:
        1.  Use `filter` to select only active users.
        2.  Chain another `filter` to select users older than 25 from the result of the first filter.
        3.  Use `map` to extract the number of logins from the remaining users.
        4.  Use `reduce` to sum up the logins.
4.  **Implement Functional Utils (Bonus)**:
    - Create a `pipeline` function that takes a starting value and a list of functions, and applies each function to the result of the previous one. This is a generalized composition.
    - `pipeline(value, [func1, func2, func3])` should be equivalent to `func3(func2(func1(value)))`.
5.  **`main()` function**:
    - Execute your pipeline and print the final result.
6.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes
- **Overusing `lambda`**: Lambdas are for small, simple functions. If your lambda becomes complex or long, it's a sign you should define a regular function with `def`. This improves readability and allows for docstrings.
- **Materializing `map` and `filter` objects**: `map` and `filter` return iterators, which are lazy and memory-efficient. Calling `list()` on them immediately can defeat the purpose if you are working with large data and could continue processing in a lazy fashion.
- **Making `reduce` unreadable**: `reduce` is very powerful, but can quickly become hard to understand. For simple sums or products, `sum()` or `math.prod()` are much clearer. Use `reduce` when the operation is truly a complex accumulation.
- **Lambda scope**: Lambdas capture variables from the enclosing scope. This can sometimes lead to surprising behavior, especially inside loops.

## 📖 Further Reading
- [Real Python: Python Lambda Functions](https://realpython.com/python-lambda/)
- [Real Python: `map()`, `filter()`, and `reduce()`](https://realpython.com/python-map-filter-reduce/)
- [Python Docs: `functools.partial`](https://docs.python.org/3/library/functools.html#functools.partial)
- [Functional Programming in Python](https://docs.python.org/3/howto/functional.html) (Official Python HOWTO guide)
