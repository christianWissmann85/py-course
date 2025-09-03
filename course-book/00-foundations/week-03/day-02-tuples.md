# Week 3, Day 2: Tuples & NamedTuples

## 🎯 Learning Objectives

- [ ] Understand the immutable nature of tuples and how it differs from lists.
- [ ] Use tuple unpacking for elegant multiple variable assignments.
- [ ] Create simple, immutable data structures using `typing.NamedTuple`.
- [ ] Learn to write type annotations for tuples and `NamedTuple` instances.

## 📚 Concepts

### 1. Tuple Basics and Immutability

A tuple is an ordered, **immutable** collection of items. You create tuples using parentheses `()`. Their main characteristic is that once a tuple is created, you cannot change, add, or remove items.

```python
# A tuple of a string, an integer, and a float
my_tuple: tuple[str, int, float] = ("Alice", 30, 172.5)

# You can access items by index, just like a list
name: str = my_tuple[0]  # "Alice"

# But you CANNOT modify it
# my_tuple[1] = 31  # This will raise a TypeError!
```

The type hint for a tuple can specify the type of each element in order, like `tuple[str, int, float]`. For a tuple with a variable number of items of the same type, you use `tuple[int, ...]`.

### 2. Tuple Unpacking

Tuple unpacking is a very Pythonic feature that allows you to assign the items of a tuple to multiple variables in a single line.

```python
user_data: tuple[str, int, bool] = ("Bob", 42, True)

# Unpack the tuple into three variables
name, age, is_active = user_data

print(f"Name: {name}, Age: {age}, Active: {is_active}")
# Output: Name: Bob, Age: 42, Active: True
```

This is commonly used to return multiple values from a function.

### 3. `typing.NamedTuple`

A regular tuple is great, but accessing items by index (`user_data[0]`) can be unreadable. A `NamedTuple` solves this by giving names to each position. It's a factory function that creates a new, simple, immutable class.

```python
from typing import NamedTuple

# Define a new NamedTuple class called 'Point'
class Point(NamedTuple):
    x: float
    y: float

# Create an instance of our new class
p1: Point = Point(x=10.5, y=20.0)

# Now you can access items by name, which is much clearer
print(f"Coordinates: ({p1.x}, {p1.y})") # Output: Coordinates: (10.5, 20.0)

# You can still access by index if you want
assert p1[0] == 10.5
```

`NamedTuple` gives you the best of both worlds: the immutability and light weight of a tuple, with the readability of an object with named attributes.

### 4. Use Cases for Tuples

- **Returning multiple values from functions**: `return name, age` implicitly returns a tuple.
- **Data that shouldn't change**: Use a tuple for a collection of items that represents a single, fixed entity, like a point `(x, y)` or an RGB color `(r, g, b)`.
- **Dictionary keys**: Since tuples are immutable, they can be used as keys in a dictionary, whereas lists cannot.
  ```python
  locations: dict[tuple[int, int], str] = {}
  locations[(40, -74)] = "New York City"
  ```

### 5. Performance Benefits

Tuples are generally slightly more memory-efficient and faster to create than lists. This is because Python can make certain optimizations knowing that the tuple's size and contents will not change. While you shouldn't choose a tuple over a list for minor performance gains, it's a nice bonus in performance-critical code where you are creating many small collections.

## 🔹 Quick Exercise

Implement the `distance` function below. It should take two `Point` objects (our `NamedTuple`) and calculate the Euclidean distance between them. The formula is `sqrt((x2 - x1)^2 + (y2 - y1)^2)`.

```python
import math
from typing import NamedTuple

class Point(NamedTuple):
    x: float
    y: float

def distance(p1: Point, p2: Point) -> float:
    """
    Calculates the Euclidean distance between two points.
    """
    # Implement the distance formula here.
    # You'll need math.sqrt() and the ** operator for squaring.
    pass

# --- Tests ---
p1 = Point(1, 2)
p2 = Point(4, 6)
# The distance should be 5.0
assert math.isclose(distance(p1, p2), 5.0)

p3 = Point(0, 0)
p4 = Point(5, 12)
# The distance should be 13.0
assert math.isclose(distance(p3, p4), 13.0)
```

## 📝 Daily Assignment

**Goal**: Build a simple coordinate geometry system using `NamedTuple`.

1.  **Create Project File**: In `my-first-poetry-app`, create a new file: `my_first_poetry_app/geometry.py`.
2.  **Define Data Structures**:
    - Define a `Point` `NamedTuple` with `x: float` and `y: float` fields.
    - Define a `Line` `NamedTuple` with `start: Point` and `end: Point` fields.
3.  **Implement Geometry Functions**:
    - `midpoint(line: Line) -> Point`: A function that takes a `Line` and returns its midpoint. The midpoint's coordinates are the average of the start and end coordinates.
    - `slope(line: Line) -> float | None`: A function that calculates the slope of a line (`(y2-y1) / (x2-x1)`). Return `None` for vertical lines (where `x2-x1` is zero) to avoid a `ZeroDivisionError`.
    - `length(line: Line) -> float`: A function that calculates the length of a line. You can reuse the `distance` function from the Quick Exercise.
4.  **`main()` function**:
    - Create several `Point` and `Line` instances.
    - Call each of your geometry functions with these instances.
    - Print the results in a clear, human-readable format, e.g., `The midpoint of Line(...) is Point(...)`.
5.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes

- **Trying to modify a tuple**: `my_tuple[0] = 5` will always fail. If you need to "change" a tuple, you must create a new one.
- **Forgetting the comma for a single-item tuple**: `(1)` is just the number 1 in parentheses. To create a tuple with one item, you need a trailing comma: `(1,)`.
  ```python
  not_a_tuple = (42) # This is an int
  is_a_tuple = (42,) # This is a tuple
  ```
- **Using `NamedTuple` like a full class**: `NamedTuple` is for creating simple, immutable data containers. It's not designed for complex methods or state changes. If you need that, you should use a `dataclass` (Week 5) or a regular `class`.

## 📖 Further Reading

- [Real Python: Python Tuples](https://realpython.com/python-lists-tuples/#python-tuples)
- [Python Docs: `typing.NamedTuple`](https://docs.python.org/3/library/typing.html#typing.NamedTuple)
- [Python Docs: Unpacking Argument Lists](https://docs.python.org/3/tutorial/controlflow.html#unpacking-argument-lists)
