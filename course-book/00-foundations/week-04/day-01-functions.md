# Week 4, Day 1: Function Fundamentals

## 🎯 Learning Objectives

- [ ] Master the creation of functions with full type-annotated signatures.
- [ ] Understand Python's scope rules (LEGB rule) for variable lookup.
- [ ] Correctly use default arguments and keyword-only arguments.
- [ ] Understand the concept of side effects and aim to write pure functions.

## 📚 Concepts

### 1. Function Definition with Types

A function is a reusable block of code that performs a specific task. A complete, type-annotated function signature includes types for all arguments and the return value.

```python
def add_numbers(a: int, b: int) -> int:
    """Takes two integers and returns their sum."""
    return a + b
```

This signature clearly states: `add_numbers` takes two arguments, `a` (an `int`) and `b` (an `int`), and it returns an `int`.

### 2. Parameters vs. Arguments

- **Parameter**: The variable listed inside the parentheses in the function definition (e.g., `a` and `b` above).
- **Argument**: The actual value that is sent to the function when it is called (e.g., `5` and `10` in `add_numbers(5, 10)`).

Python supports several types of arguments:

- **Positional arguments**: Passed in order (`add_numbers(5, 10)`).
- **Keyword arguments**: Passed by name, so order doesn't matter (`add_numbers(b=10, a=5)`).
- **Default arguments**: Parameters can have a default value, making the corresponding argument optional when calling the function.
- **Keyword-only arguments**: Arguments that _must_ be passed using their keyword name. You define them by placing them after a `*`.

```python
def create_user(name: str, *, is_admin: bool = False, permissions: list[str] | None = None) -> dict:
    """Creates a user, with optional admin status."""
    if permissions is None:
        permissions = []

    user = {"name": name, "is_admin": is_admin, "permissions": permissions}
    return user

# `is_admin` and `permissions` are keyword-only arguments.
user1 = create_user("Alice", is_admin=True)
# This would be an error: create_user("Bob", True)
```

### 3. Return Types and `None`

A function can return a value using the `return` statement. The type of this value should match the return type hint (`-> ...`). If a function does not have a `return` statement, it implicitly returns `None`. In this case, its return type should be annotated as `-> None`.

```python
def print_greeting(name: str) -> None:
    """Prints a greeting and returns nothing."""
    print(f"Hello, {name}!")

result = print_greeting("World")
print(result) # Output: None
```

### 4. Scope and the LEGB Rule

Scope determines the visibility of a variable. Python uses the **LEGB** rule to look up variable names:

1.  **L**ocal: The innermost scope, inside the current function.
2.  **E**nclosing: The scope of any enclosing functions (e.g., in nested functions).
3.  **G**lobal: The top-level scope of the module.
4.  **B**uilt-in: The special scope for Python's built-in names like `list`, `print`, `int`.

Python searches for a variable in this order.

### 5. Side Effects

A function has a **side effect** if it does something other than just compute and return a value. Common side effects include:

- Modifying a mutable object that was passed as an argument.
- Modifying a global variable.
- Printing to the console.
- Reading from or writing to a file.

A function with no side effects is called a **pure function**. Pure functions are easier to test and reason about because for the same input, they _always_ produce the same output.

```python
# Impure function (has a side effect of printing)
def impure_add(a: int, b: int) -> int:
    result = a + b
    print(f"Adding {a} + {b}") # Side effect
    return result

# Pure function (no side effects)
def pure_add(a: int, b: int) -> int:
    return a + b
```

While side effects are often necessary (e.g., to save data), it's a good practice to isolate them from your core logic.

## 🔹 Quick Exercise

Implement the following two functions with full type annotations.

```python
import re

def greet(*, name: str, prefix: str = "Hello") -> str:
    """
    Greets a person with an optional prefix.
    This function should use keyword-only arguments.
    """
    # Implement here
    pass

def is_valid_email(email: str) -> bool:
    """
    Validates an email format using a simple regex.
    Returns True if valid, False otherwise.
    """
    # A simple regex for email validation
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    # Use re.match to check if the pattern matches the start of the string
    # A successful match object is "truthy", no match is "falsy".
    pass

# --- Tests ---
assert greet(name="Alice") == "Hello, Alice"
assert greet(name="Bob", prefix="Hi") == "Hi, Bob"
assert is_valid_email("test@example.com") is True
assert is_valid_email("invalid-email") is False
```

## 📝 Daily Assignment

**Goal**: Create a personal utility library with a variety of small, useful, and pure functions.

1.  **Create Project File**: In `my-first-poetry-app`, create a new file: `my_first_poetry_app/fn_library.py`.
2.  **Implement 10-20 Utility Functions**: Write at least 10 (aim for 20!) small, well-defined, and pure functions. Each must be fully type-annotated.
    - **Math Functions**:
      - `is_even(n: int) -> bool`
      - `area_of_circle(radius: float) -> float`
      - `fibonacci(n: int) -> int` (a recursive version is fine for now)
    - **String Functions**:
      - `capitalize_words(text: str) -> str`
      - `reverse_string(s: str) -> str`
    - **List Functions**:
      - `get_first(items: list) -> any` (make this generic with `TypeVar`)
      - `get_last(items: list) -> any` (also generic)
      - `average(numbers: list[float]) -> float` (handle empty list case)
    - **Data Functions**:
      - `format_user_profile(data: dict) -> str` (use `TypedDict` for the data)
      - `is_in_stock(item: dict) -> bool` (use `TypedDict`)
3.  **`main()` function**:
    - Write a `main()` function that demonstrates at least 5 of your library functions.
4.  **Verify**: Run the script, type checker, and linter, and fix all issues.

## ⚠️ Common Mistakes

- **Modifying default mutable arguments**: A very common and dangerous pitfall. Default arguments are created _once_, when the function is defined. If the default is a mutable type like a `list` or `dict`, any modification to it will persist across function calls.

  ```python
  # Wrong way!
  def add_item(item, my_list: list = []):
      my_list.append(item)
      return my_list

  print(add_item(1)) # [1]
  print(add_item(2)) # [1, 2] <- The list from the previous call was reused!

  # Correct way
  def add_item_safe(item, my_list: list | None = None):
      if my_list is None:
          my_list = []
      my_list.append(item)
      return my_list
  ```

- **Confusing positional and keyword arguments**: A function `def f(a, b):` can be called as `f(1, 2)` or `f(a=1, b=2)` or `f(1, b=2)`, but not `f(a=1, 2)` (positional argument after keyword argument).
- **Returning `None` implicitly**: Forgetting a `return` statement in some branches of an `if/else` can lead to the function sometimes returning `None` unexpectedly. MyPy strict mode helps catch this.

## 📖 Further Reading

- [Real Python: Defining Your Own Python Functions](https://realpython.com/defining-your-own-python-function/)
- [Python Docs: The `global` statement](https://docs.python.org/3/reference/simple_stmts.html#the-global-statement) (For when you need to modify global variables)
- [Real Python: Python Scope & the LEGB Rule](https://realpython.com/python-scope-legb-rule/)
