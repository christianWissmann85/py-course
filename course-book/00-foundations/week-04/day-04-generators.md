# Week 4, Day 4: Generators & Iterators

## 🎯 Learning Objectives

- [ ] Understand the difference between an iterable and an iterator.
- [ ] Create your own generator functions using the `yield` keyword.
- [ ] Use generator expressions for a concise, memory-efficient way to create iterators.
- [ ] Type-annotate generators and iterators correctly.

## 📚 Concepts

### 1. The Iterator Protocol

Before we can understand generators, we must understand iterators.

- **Iterable**: An object that you can loop over (like a `list`, `str`, `dict`). It has an `__iter__` method that returns an iterator.
- **Iterator**: An object that represents a stream of data. It has a `__next__` method that returns the next item in the stream. When there are no more items, it raises a `StopIteration` exception.

A `for` loop implicitly calls `__iter__` on an iterable to get an iterator, and then calls `__next__` on that iterator until `StopIteration` is raised.

### 2. Generator Functions and `yield`

A **generator function** is a simpler way to create an iterator. Instead of writing a full class with `__iter__` and `__next__`, you just write a function that uses the `yield` keyword.

When a generator function is called, it returns a **generator object** (a type of iterator). The function's code does not run at this point. Each time `__next__` is called on the generator object (e.g., by a `for` loop), the function's code executes until it hits a `yield` statement. The `yield`ed value is returned, and the function's state is paused. The next time `__next__` is called, it resumes right where it left off.

```python
from typing import Generator

# This is a generator function. Its return type is Generator[YieldType, SendType, ReturnType]
# For simple generators, we can use Generator[YieldType, None, None].
def countdown(n: int) -> Generator[int, None, None]:
    print("Starting countdown!")
    while n > 0:
        yield n
        n -= 1
    print("Countdown finished!")

# Calling it creates a generator object, but doesn't run the code yet.
my_generator = countdown(3)
print(type(my_generator)) # <class 'generator'>

# The for loop will call __next__() for us.
for number in my_generator:
    print(f"Yielded: {number}")
```

Output:

```
<class 'generator'>
Starting countdown!
Yielded: 3
Yielded: 2
Yielded: 1
Countdown finished!
```

The key benefit is **lazy evaluation**. The values are generated one at a time, on demand, which is incredibly memory-efficient for large datasets.

### 3. Generator Expressions

A generator expression is a high-performance, memory-efficient generalization of list comprehensions. The syntax is just like a list comprehension, but with parentheses `()` instead of square brackets `[]`.

It creates a generator object without creating the full list in memory.

```python
# A list comprehension - builds the full list in memory.
list_comp = [x*x for x in range(10)]

# A generator expression - creates a generator object.
# No values are computed yet.
gen_expr = (x*x for x in range(10))

print(list_comp) # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
print(gen_expr)  # <generator object <genexpr> at 0x...>

# You can iterate over the generator to get the values.
for value in gen_expr:
    print(value, end=' ') # 0 1 4 9 ... 81
```

Use a generator expression when you are iterating over the result immediately and don't need to store the entire list.

### 4. `yield from`

The `yield from` statement allows one generator to delegate part of its operations to another generator. It's a clean way to chain iterables together.

```python
def combined_generator() -> Generator[str, None, None]:
    # yield from can be used with any iterable
    yield from ['a', 'b', 'c']
    yield from ('d', 'e', 'f')
    yield from "ghi"

for item in combined_generator():
    print(item, end='') # abcdefghi
```

### 5. Infinite Generators

Because generators are lazy, you can even create generators that produce an infinite sequence of values. You just need to be careful to use `break` or some other condition to stop iterating over them.

```python
def infinite_counter(start: int = 0) -> Generator[int, None, None]:
    num = start
    while True:
        yield num
        num += 1

counter = infinite_counter()
for i in range(5):
    print(next(counter)) # 0, 1, 2, 3, 4
```

## 🔹 Quick Exercise

Implement the following two generator functions.

```python
from typing import Generator, TypeVar, List

T = TypeVar('T')

def fibonacci() -> Generator[int, None, None]:
    """
    Generates the Fibonacci sequence indefinitely.
    Sequence: 0, 1, 1, 2, 3, 5, 8, ...
    """
    a, b = 0, 1
    while True:
        # Yield the current value, then update the state
        pass # Implement here

def chunk_list(items: List[T], size: int) -> Generator[List[T], None, None]:
    """
    Yields successive n-sized chunks from a list.
    Example: chunk_list([1,2,3,4,5,6], 2) -> yields [1,2], then [3,4], then [5,6]
    """
    # Hint: Use a for loop with a step, and slicing.
    # range(0, len(items), size)
    pass


# --- Tests ---
fib_gen = fibonacci()
assert [next(fib_gen) for _ in range(6)] == [0, 1, 1, 2, 3, 5]

chunk_gen = chunk_list([1, 2, 3, 4, 5, 6, 7], 3)
assert next(chunk_gen) == [1, 2, 3]
assert next(chunk_gen) == [4, 5, 6]
assert next(chunk_gen) == [7]
```

## 📝 Daily Assignment

**Goal**: Build a small data processing pipeline using a toolkit of generator functions.

1.  **Create Project File**: In `my-first-poetry-app`, create `my_first_poetry_app/pipeline.py`.
2.  **Implement Generator Toolkit**:
    - `read_large_file(filepath: str) -> Generator[str, None, None]`: A generator that reads a large file line by line using `yield`. This avoids loading the whole file into memory.
    - `filter_lines(lines: Generator[str, None, None], keyword: str) -> Generator[str, None, None]`: Takes a generator of lines and yields only those containing a specific keyword.
    - `uppercase_lines(lines: Generator[str, None, None]) -> Generator[str, None, None]`: Takes a generator of lines and yields uppercase versions of them.
3.  **Create a Data Pipeline**:
    - In a `main` function:
      - First, create a large dummy log file (e.g., 100,000 lines) to test with. Some lines should contain the word "ERROR".
      - Create a processing pipeline by chaining your generators together:
        ```python
        # Pseudocode
        lines = read_large_file("dummy.log")
        error_lines = filter_lines(lines, "ERROR")
        processed_lines = uppercase_lines(error_lines)
        ```
      - Iterate over the final `processed_lines` generator and print each line to the console.
4.  **Analyze**: Add comments to your code explaining why this generator-based approach is more memory-efficient than reading the whole file into a list, then filtering it into a new list, etc.
5.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes

- **Confusing a generator function with a normal function**: Calling a generator function does not run its code; it returns a generator object. The code only runs when you iterate over the generator (e.g., with a `for` loop or `next()`).
- **Storing the result of a generator expression**:
  ```python
  my_gen = (i for i in range(10))
  list1 = list(my_gen) # [0, 1, ..., 9]
  list2 = list(my_gen) # []
  ```
  A generator can only be consumed once. After `list1` is created, the generator is exhausted.
- **`return` in a generator**: Using `return` in a generator function will cause it to raise `StopIteration`, effectively ending the generation. It doesn't return a value in the traditional sense.
- **Overusing list comprehensions**: When working with very large datasets that you only need to iterate over once, a generator expression is almost always better than a list comprehension to save memory.

## 📖 Further Reading

- [Real Python: Python Generators 101](https://realpython.com/introduction-to-python-generators/)
- [Python Docs: `yield` expressions](https://docs.python.org/3/reference/expressions.html#yield-expressions)
- [PEP 255 -- Simple Generators](https://peps.python.org/pep-0255/)
