# Week 10, Day 4: Function Composition & Pipelines

## 🎯 Learning Objectives

- [ ] Understand function composition as a core functional programming pattern.
- [ ] Implement a `pipe` function to chain multiple functions together into a data processing pipeline.
- [ ] Contrast function pipelines with method chaining.
- [ ] Use the `operator` module to create point-free functions for your pipelines.

## 📚 Concepts

### 1. Function Composition

Function composition is the process of combining two or more functions to produce a new function. The output of one function becomes the input of the next. In mathematics, this is written as `h(x) = f(g(x))`.

```python
def double(x: int) -> int:
    return x * 2

def add_one(x: int) -> int:
    return x + 1

# We can manually compose them
def double_then_add_one(x: int) -> int:
    return add_one(double(x))

# Or create a higher-order function to do it for us
def compose(f, g):
    return lambda x: f(g(x))

composed_function = compose(add_one, double)
result = composed_function(10) # result is 21
```

This pattern allows us to build complex operations from small, simple, and testable building blocks.

### 2. The Pipeline Pattern

While `compose` is elegant, the evaluation order (`f(g(x))`) is inside-out, which can be hard to read. A more readable pattern is the **pipeline**. A pipeline applies functions in the order they are listed.

We can write a `pipe` function that takes a value and a series of functions, and "pipes" the value through each one.

```python
from typing import Any, Callable, Iterable

def pipe(value: Any, *funcs: Callable) -> Any:
    """
    Passes a value through a sequence of functions.
    pipe(x, f, g, h) is equivalent to h(g(f(x))).
    """
    result = value
    for func in funcs:
        result = func(result)
    return result

# Example usage:
def double(x): return x * 2
def add_one(x): return x + 1
def to_string(x): return f"Result: {x}"

output = pipe(
    10,
    double,
    add_one,
    to_string
)
print(output) # "Result: 21"
```

This is much more readable as it flows from top to bottom, just like we read text.

### 3. Method Chaining vs. Pipelines

You may have seen a similar pattern with **method chaining**, especially in libraries like Pandas.

```python
# Method Chaining (Pandas example)
# df.filter(...).groupby(...).sum()

# Functional Pipeline
# pipe(df, filter_func, groupby_func, sum_func)
```

- **Method Chaining**: Requires the object itself (`df`) to have all the methods (`.filter`, `.groupby`). This is an object-oriented approach.
- **Functional Pipeline**: Uses standalone functions. This is a functional approach. It's more flexible because you can use any function in your pipeline; it doesn't have to be a method of the object.

### 4. The `operator` Module

Sometimes you need simple functions like "add two numbers" or "get an item from a dict". The `operator` module provides functions that are equivalent to Python's intrinsic operators. This is useful when a higher-order function needs a function argument.

```python
import operator
from functools import reduce

# Instead of lambda:
reduce(lambda x, y: x + y, [1, 2, 3])

# You can use the operator module:
reduce(operator.add, [1, 2, 3])
```

This can make code more readable and is sometimes slightly faster than an equivalent `lambda`.

### 5. Lazy Evaluation

Pipelines work beautifully with the generators we learned about. Because generators are lazy, a pipeline built with them only does the minimum work necessary.

```python
# A pipeline of generators
numbers = range(1, 1_000_000)
squared = (n * n for n in numbers)
evens = (n for n in squared if n % 2 == 0)

# No computation has happened yet!

# Get the first 5 results
first_five = [next(evens) for _ in range(5)]
print(first_five) # [4, 16, 36, 64, 100]

# The pipeline only computed values until it had found 5 even squares.
# It didn't process all one million numbers. This is incredibly efficient.
```

## 🔹 Quick Exercise

Implement the `pipe` function from the concepts section. It should take an initial value and then any number of functions (`*funcs`), and apply them in sequence.

```python
from typing import Any, Callable

def pipe(value: Any, *funcs: Callable) -> Any:
    """
    Passes a value through a sequence of functions.
    The first function in funcs is called with `value`.
    Each subsequent function is called with the result of the previous one.
    """
    result = value
    for func in funcs:
        result = func(result)
    return result

# --- Tests ---
def double(x): return x * 2
def add(x, y): return x + y
def to_string(x): return str(x)

from functools import partial

assert pipe(5, double, partial(add, 10)) == 20 # (5*2)+10
assert pipe("hello", str.upper, partial(str.replace, 'H', 'J')) == "JELLO"
```

## 📝 Daily Assignment

**Goal**: Build a data processing pipeline library using the concepts learned today.

1.  **Create Project File**: In your project, create `my_first_poetry_app/pipeline_lib.py`.
2.  **Implement `pipe`**: Implement the `pipe` function from the Quick Exercise.
3.  **Implement Curried Helper Functions**: Create several "curried" helper functions that can be used in your pipelines. A curried function takes some configuration and returns another function.
    - `select_keys(keys_to_keep: list[str]) -> Callable[[dict], dict]`: Returns a function that takes a dictionary and returns a new dictionary containing only the specified keys.
    - `rename_key(old_name: str, new_name: str) -> Callable[[dict], dict]`: Returns a function that renames a key in a dictionary.
    - `as_type(key: str, new_type: Callable) -> Callable[[dict], dict]`: Returns a function that converts the value of a specific key to a new type.
4.  **Build a Pipeline**:

    - In a `main` function, define some sample data (e.g., a list of dictionaries).
    - Use your `pipe` function to chain together your helper functions to process the data.
    - Example pipeline:

      ```python
      data = {"user_id": 101, "user_name": "alice", "email": "a@b.com", "age_str": "30"}

      pipeline = pipe(
          data,
          select_keys(["user_name", "age_str"]),
          rename_key("user_name", "name"),
          as_type("age_str", int)
      )
      # Expected output of pipeline: {"name": "alice", "age_str": 30}
      ```

5.  **Lazy Pipeline (Bonus)**:
    - Create a new function `pipe_lazy(*funcs)` that takes only functions and returns a new single function that represents the entire pipeline. This is a more advanced composition.
    - `my_pipeline = pipe_lazy(f, g, h)`
    - `result = my_pipeline(initial_value)`
6.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes

- **Incorrectly ordered `compose`**: The mathematical definition of `compose(f, g)` is `f(g(x))`. It's easy to implement it backwards as `g(f(x))`. This is why `pipe` is often more intuitive for data processing.
- **Mixing up data and functions**: In a pipeline, the first argument is the data, and the rest are functions to apply to it.
- **Not using `partial` for functions with multiple arguments**: If a function in your pipeline needs more than one argument (like `add(x, 10)`), you must use `functools.partial` to "fix" the extra arguments, so it becomes a function that only takes one argument (the value from the pipeline).

## 📖 Further Reading

- [Toolz: A functional utility library for Python](https://toolz.readthedocs.io/en/latest/index.html) (A popular and powerful library that includes `pipe` and `compose`)
- [Real Python: Python's `operator` Module](https://realpython.com/python-operator-module/)
- [Fn.py: Functional programming in Python](https://github.com/kachayev/fn.py) (Another library with functional tools)
