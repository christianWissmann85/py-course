# Week 2, Day 1: Variables & Basic Types

## 🎯 Learning Objectives
- [ ] Master the declaration of variables with clear, explicit type annotations.
- [ ] Understand the concept of immutability and which basic types are immutable.
- [ ] Learn how to use type aliases to create more readable type hints.
- [ ] Use `Literal` and `Final` from the `typing` module to create more precise and safe types.

## 📚 Concepts

### 1. Variable Declaration with Types
In Python, you declare a variable by assigning a value to it. With type hints, you can also declare its expected type. This makes your code clearer and allows MyPy to catch errors. The syntax is `variable_name: type = value`.

```python
user_name: str = "Jules"
items_count: int = 5
price: float = 99.95
is_available: bool = True
```

### 2. Primitive Types & Immutability
The basic building blocks of data in Python are the primitive types:
- `int`: Integers (e.g., `10`, `-3`).
- `float`: Floating-point numbers (e.g., `3.14`, `-0.001`).
- `str`: Strings of text (e.g., `"Hello"`, `'Python'`).
- `bool`: Boolean values (`True` or `False`).

All of these primitive types are **immutable**. This means that once they are created, their value cannot be changed. Any operation that seems to "modify" them actually creates a new object in memory.

```python
name: str = "Jules"
# This does not change the original string "Jules".
# It creates a new string "Jules Verne" and reassigns the `name` variable to it.
name = name + " Verne"
```

### 3. Type Inference vs. Explicit Annotation
MyPy is often smart enough to infer the type of a variable from its assigned value.

```python
# MyPy infers `user_id` is an `int`
user_id = 123
```

However, for this course and for professional-grade code, we **always prefer explicit type annotations**. It makes the author's intent clear and reduces ambiguity, both for human readers and for the type checker.

### 4. Constants with `Final`
Sometimes, you want to declare a variable whose value should never be changed. This is called a constant. Python doesn't have a built-in `const` keyword, but the `typing` module provides `Final`.

`Final` tells the type checker that a variable should not be reassigned.

```python
from typing import Final

GRAVITATIONAL_CONSTANT: Final[float] = 9.81

# MyPy will flag this line as an error:
# GRAVITATIONAL_CONSTANT = 10.0
# error: Cannot assign to a "Final" variable
```

### 5. Precise Types with `Literal`
What if a variable can only have a few specific string values? For example, a status that can only be `"active"`, `"inactive"`, or `"pending"`. Using `str` as the type is too broad. `Literal` lets you be more precise.

```python
from typing import Literal

OrderStatus = Literal["new", "processing", "shipped", "delivered"]

# Now, `order_status` can only be one of those four strings.
order_status: OrderStatus = "processing"

# MyPy will flag this as an error:
# order_status = "cancelled"
# error: Invalid value "cancelled" for type Literal["new", "processing", "shipped", "delivered"]
```
We also used a **type alias** here by assigning the `Literal` to the `OrderStatus` variable. This makes the type hint reusable and much cleaner.

## 🔹 Quick Exercise

Complete the following code by filling in the blanks (`___`) with appropriate values. Create a file, add the code, and run MyPy to verify that your annotations and values are correct.

```python
from typing import Final, Literal

# 1. An integer representing a user's age.
age: int = ___

# 2. A string for a user's name.
name: str = ___

# 3. A floating-point constant for the value of PI.
#    This value should not be changed.
PI: Final[float] = ___

# 4. A variable representing a user's login status.
#    It can only be 'online' or 'offline'.
UserStatus = Literal["online", "offline"]
current_status: UserStatus = ___

# 5. A boolean indicating if a task is complete.
is_task_done: bool = ___

print(f"User: {name}, Age: {age}, Status: {current_status}")
print(f"PI is {PI}. Is the task done? {is_task_done}")
```

## 📝 Daily Assignment

**Goal**: Build a simple, type-safe calculator program.

1.  **Create Project File**: In your `my-first-poetry-app` project, create a new file: `my_first_poetry_app/calculator.py`.
2.  **Define Constants**:
    - Use `Final` to define constants for the four basic arithmetic operations: `ADD: Final[str] = "+"` , `SUBTRACT: Final[str] = "-"`, etc.
3.  **Use Type Aliases**:
    - Create a `Operator` type alias using `Literal` that can only be one of the four operation constants you defined.
    - Create a `Number` type alias that can be either an `int` or a `float` (Hint: use `Union` from `typing`).
4.  **Implement the `calculate` function**:
    - Create a function `calculate(a: Number, b: Number, operator: Operator) -> Number | None:`.
    - This function should take two numbers and an operator.
    - It should return the result of the calculation.
    - It should handle division by zero by returning `None`.
5.  **Implement a `main` function**:
    - Create a `main() -> None:` function.
    - Inside `main`, call your `calculate` function with various inputs, including a division by zero case.
    - Print the results in a user-friendly way.
6.  **Run and Verify**:
    - Run your script with `poetry run python my_first_poetry_app/calculator.py`.
    - Run the type checker with `poetry run mypy .` and fix all errors.

## ⚠️ Common Mistakes
- **Confusing `Final` with immutability**: `Final` prevents reassignment of the variable name. It does *not* make the object itself immutable. If you have a `Final[list]`, you can still `append` to the list.
- **Using type aliases incorrectly**: A type alias is just a name. You use it in the type annotation spot, not as a function or class.
  ```python
  # Correct
  status: OrderStatus = "shipped"
  # Incorrect
  status = OrderStatus("shipped")
  ```
- **Overly broad types**: Using `str` when `Literal["a", "b"]` is more accurate. The goal of modern typing is to be as precise as possible.

## 📖 Further Reading
- [Python Docs: `typing` — Support for type hints](https://docs.python.org/3/library/typing.html) (The official source of truth)
- [MyPy Docs: Type aliases](https://mypy.readthedocs.io/en/stable/type_aliases.html)
- [MyPy Docs: Final names](https://mypy.readthedocs.io/en/stable/final_attrs.html)
