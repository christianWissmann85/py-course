# Week 10, Day 3: Partial Application & Currying

## 🎯 Learning Objectives

- [ ] Use `functools.partial` to "freeze" a function's arguments, creating a new specialized function.
- [ ] Understand the concept of currying and how it differs from partial application.
- [ ] Implement a `curry` decorator to transform a function that takes multiple arguments into a chain of functions that each take a single argument.
- [ ] Build function factories that generate new, customized functions.

## 📚 Concepts

### 1. Partial Application with `functools.partial`

**Partial application** is the process of fixing a number of a function's arguments to create a new, simpler function. Python's `functools` module provides the `partial` object for this.

```python
from functools import partial

def power(base: float, exponent: float) -> float:
    return base ** exponent

# Create a new function 'square' by "freezing" the exponent argument to 2.
square = partial(power, exponent=2)

# The `square` function is now a new callable that only needs the `base`.
print(square(5)) # 25.0
print(square(10))# 100.0

# You can also freeze positional arguments.
power_of_2 = partial(power, 2)
print(power_of_2(8)) # 2 ** 8 = 256
```

`partial` is incredibly useful for adapting existing functions to be used in contexts where a different function signature is expected (e.g., as a callback).

### 2. The Concept of Currying

**Currying** (named after mathematician Haskell Curry) is the technique of transforming a function that takes multiple arguments into a sequence of functions that each take a single argument.

- A function `f(a, b, c)`
- when curried, becomes `f'(a)(b)(c)`

```python
# A curried add function
def add_curried(a):
    def inner(b):
        return a + b
    return inner

add_5 = add_curried(5)
result = add_5(10) # result is 15
```

While Python doesn't have a built-in `curry` function, it's a powerful concept in functional programming that encourages composing small, single-purpose functions.

### 3. Function Factories

A **function factory** is a higher-order function that returns a new, specialized function based on its inputs. Both `partial` and our `add_curried` example are function factories.

This pattern is useful for creating a family of related functions without code duplication.

```python
def make_multiplier(n: int):
    """A factory that creates a multiplier function."""
    def multiplier(x: int) -> int:
        return x * n
    return multiplier

double = make_multiplier(2)
triple = make_multiplier(3)

print(double(10)) # 20
print(triple(10)) # 30
```

### 4. Practical Use Cases

- **Callbacks**: If a system expects a callback function with no arguments (e.g., `callback()`), but your function needs arguments (`my_func(user, data)`), you can use `partial` to wrap it: `callback = partial(my_func, user=current_user, data=some_data)`.
- **Pipelines**: Curried functions are excellent for building declarative data pipelines, as seen in the assignment.
- **Reducing Boilerplate**: Instead of writing many similar classes or functions, you can write a single factory to generate them.

## 🔹 Quick Exercise

Implement a decorator `@curry` that transforms a function of two arguments into a curried function.

```python
from typing import Callable

def curry(func: Callable) -> Callable:
    """
    A decorator that curries a function of two arguments.
    """
    def curried(a):
        def inner(b):
            return func(a, b)
        return inner
    return curried

@curry
def add(a: int, b: int) -> int:
    return a + b

# --- Tests ---
add_10 = add(10)
assert callable(add_10)
assert add_10(5) == 15

# You can also call it directly
assert add(2)(3) == 5
print("Curry decorator works!")
```

## 📝 Daily Assignment

**Goal**: Build a small library of curried functions for data transformation pipelines.

1.  **Create Project File**: In your project, create `my_first_poetry_app/currying_lab.py`.
2.  **Implement a General `@curry` Decorator (Bonus)**:
    - Make the `@curry` decorator from the exercise more robust so it can handle functions with any number of arguments. This is a challenging but rewarding task!
3.  **Create Curried Utility Functions**:
    - `prop(key: str) -> Callable[[dict], Any]`: A curried function. `prop("name")` should return a _new function_ that takes a dictionary and returns the value of the "name" key.
    - `is_greater_than(threshold: int) -> Callable[[int], bool]`: A curried function. `is_greater_than(10)` should return a _new function_ that takes a number and returns `True` if it's > 10.
    - `transform(key: str, transform_func: Callable) -> Callable[[dict], dict]`: A curried function that returns a function to transform a value in a dictionary.
4.  **Build a Pipeline**:
    - Use the `pipe` function from yesterday's lesson.
    - Define a list of user dictionaries.
    - Build a pipeline that:
      1.  Uses `filter` and a composition of `is_greater_than(25)` and `prop('age')` to find users older than 25.
      2.  Uses `map` and `prop('email')` to get the email addresses of those users.
      3.  The pipeline should be a clean chain of functional calls.
5.  **`main()` function**:
    - Execute your pipeline and print the results.
6.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes

- **Confusing Currying and Partial Application**:
  - `partial` takes a function and some of its arguments, and returns a new function with fewer arguments.
  - `curry` takes a function of N arguments and returns a chain of N functions that each take one argument.
  - `partial` is a built-in, practical tool. `curry` is a more theoretical pattern that you often implement yourself.
- **Incorrectly nesting functions**: When writing factories or curried functions, it's easy to get the `return` statements at the wrong level. Remember that each `def` or `lambda` creates a new scope.
- **Losing type information**: Typing higher-order functions can be complex. Without proper use of `Callable` and `TypeVar`, MyPy can lose track of the specific types flowing through your functions.

## 📖 Further Reading

- [Functools — Higher-order functions and operations on callable objects](https://docs.python.org/3/library/functools.html) (Especially `partial`)
- [Wikipedia: Currying](https://en.wikipedia.org/wiki/Currying)
- [Python-Toolz Currying](https://toolz.readthedocs.io/en/latest/curry.html) (A powerful library implementation of these concepts)
