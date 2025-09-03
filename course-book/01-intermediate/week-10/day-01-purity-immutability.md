# Week 10, Day 1: Pure Functions & Immutability

## 🎯 Learning Objectives

- [ ] Understand the definition and benefits of pure functions.
- [ ] Differentiate between pure functions and functions with side effects.
- [ ] Use immutable data structures like tuples and frozen dataclasses to prevent accidental modification.
- [ ] Apply functional patterns to create more predictable and testable code.

## 📚 Concepts

### 1. Pure Function Principles

A **pure function** is a function that adheres to two strict rules:

1.  **Given the same input, it will always return the same output.** The function's output depends _only_ on its input arguments. It does not rely on any hidden state, global variables, or I/O.
2.  **It has no observable side effects.** A side effect is any interaction the function has with the outside world beyond returning a value. This includes modifying a mutable argument, changing a global variable, printing to the console, reading from a file, or making a network request.

```python
# PURE function
def add(a: int, b: int) -> int:
    return a + b

# IMPURE function (relies on external state)
c = 10
def add_with_global(a: int) -> int:
    return a + c # Result depends on the global variable 'c'

# IMPURE function (has a side effect)
def add_and_print(a: int, b: int) -> int:
    print(f"Adding {a} and {b}") # Side effect: printing to console
    return a + b
```

### 2. Immutability in Python

An object is **immutable** if its state cannot be modified after it is created. We've already seen several immutable types: `int`, `float`, `str`, `tuple`, and `frozenset`.

Working with immutable data is a cornerstone of functional programming. Instead of changing an object, you create a _new_ object with the updated value. This prevents a whole class of bugs where data is unexpectedly changed by another part of the program.

```python
# Working with an immutable tuple
my_tuple = (1, 2, 3)
# new_tuple = my_tuple + (4,) # Creates a new tuple

# Working with a mutable list
my_list = [1, 2, 3]
# another_var = my_list
# another_var.append(4) # This modifies the original list!
# print(my_list) # [1, 2, 3, 4] <- Surprise!
```

### 3. Frozen Dataclasses

As we saw in Week 5, dataclasses can be made immutable by setting `frozen=True`. This is the primary way to create custom immutable data structures in modern Python. It prevents you or other developers from accidentally changing the state of a data-holding object after it's been created.

```python
from dataclasses import dataclass, replace

@dataclass(frozen=True)
class Point:
    x: float
    y: float

p1 = Point(10, 20)
# p1.x = 15 # Raises FrozenInstanceError

# To "change" a frozen dataclass, you create a new one using `replace`.
p2 = replace(p1, x=15)
print(p1) # Point(x=10, y=20)
print(p2) # Point(x=15, y=20)
```

### 4. Benefits of Purity and Immutability

- **Predictability**: Pure functions are completely predictable. `add(2, 3)` will _always_ be `5`, no matter what else is happening in the program. This makes code much easier to reason about.
- **Testability**: Testing pure functions is trivial. You provide inputs and assert that the output is correct. There's no complex setup or state to manage.
- **Concurrency**: Pure functions are inherently thread-safe. Since they don't modify shared state, you can run them on multiple threads without worrying about race conditions or locks.
- **Fewer Bugs**: A huge number of bugs come from unexpected side effects and shared mutable state. Functional programming patterns eliminate these bugs by design.

## 🔹 Quick Exercise

Complete the `move_point` function below. It must be a **pure function**. It should take a `Point` object and return a _new_ `Point` object with the updated coordinates, leaving the original unchanged.

```python
from dataclasses import dataclass, replace

@dataclass(frozen=True)
class Point:
    x: float
    y: float

def move_point(point: Point, dx: float, dy: float) -> Point:
    """
    Moves a point by a given delta, returning a new Point object.
    """
    return replace(point, x=point.x + dx, y=point.y + dy)

# --- Tests ---
p1 = Point(10, 20)
p2 = move_point(p1, 5, -5)

# Check that a new object was created
assert p1 is not p2
# Check that the original object is unchanged
assert p1 == Point(10, 20)
# Check that the new object has the correct values
assert p2 == Point(15, 15)

print("Pure function `move_point` works correctly!")
```

## 📝 Daily Assignment

**Goal**: Create a small library of pure functions for processing a list of "user" dictionaries, using immutable patterns.

1.  **Create Project File**: In your project, create `my_first_poetry_app/immutable_processing.py`.
2.  **Define an Immutable Data Structure**:
    - Define a `User` `@dataclass(frozen=True)` with fields `user_id: int`, `name: str`, `email: str`.
3.  **Implement Pure Functions**:
    - `update_email(user: User, new_email: str) -> User`: A pure function that returns a _new_ `User` instance with the email updated.
    - `find_user_by_id(users: list[User], user_id: int) -> User | None`: A pure function that searches a list of users.
    - `get_active_users(users: list[User], active_ids: frozenset[int]) -> list[User]`: A pure function that returns a new list containing only the users whose IDs are in the `active_ids` set.
4.  **`main()` function**:
    - Create an initial list of `User` objects.
    - Create a `frozenset` of active user IDs.
    - Demonstrate each of your pure functions.
      - Start with an initial user list.
      - Call `get_active_users` to get a new list.
      - Call `update_email` on one of the users, creating a new user object.
      - Show that the original lists and user objects are never modified.
5.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes

- **Mutating arguments**: The most common way to create an impure function is to modify a mutable argument, like a list or dictionary, in place.

  ```python
  # Impure! Modifies its argument.
  def add_to_list(items: list, item_to_add):
      items.append(item_to_add) # Side effect!

  # Pure version
  def add_to_list_pure(items: list, item_to_add) -> list:
      return items + [item_to_add] # Returns a new list
  ```

- **Relying on hidden inputs**: A function that reads the current time, a global variable, or a file is not pure, because its output can change even if the direct inputs are the same.
- **Confusing immutability**: A `tuple` is immutable, but if it contains a mutable object like a `list`, the _contents_ of that list can still be changed. True immutability requires all nested data to also be immutable. `frozen=True` on a dataclass is a "shallow" freeze.

## 📖 Further Reading

- [Functional Programming in Python](https://docs.python.org/3/howto/functional.html) (Official Python HOWTO)
- [Real Python: Immutability in Python](https://realpython.com/python-immutable-data-types/)
- [Google's "Functional Programming in Python" course](https://developers.google.com/edu/python/functional-programming)
