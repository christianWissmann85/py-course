# Week 2, Day 3: Control Flow with Types

## 🎯 Learning Objectives
- [ ] Write type-safe conditional statements (`if`, `elif`, `else`).
- [ ] Understand how type checkers use "type narrowing" to infer more specific types within code blocks.
- [ ] Use Python 3.10's `match` statement for elegant structural pattern matching.
- [ ] Apply the "guard clause" pattern to write cleaner, less-nested functions.

## 📚 Concepts

### 1. `if/elif/else` with Types
The fundamental control flow statements work exactly as you'd expect, but type checkers add a layer of safety. They can analyze the conditions to understand what's possible within each block.

```python
def check_value(value: int) -> str:
    if value > 100:
        return "Huge"
    elif value > 10:
        return "Medium"
    else:
        return "Small"
```
MyPy knows that `value` is always an `int`, and the function must always return a `str`. If you forget the `else` block, MyPy will complain that your function might not return a value.

### 2. Type Narrowing
This is a powerful feature of static type checkers. When you perform a check (like `isinstance`), MyPy "narrows" the type of the variable to be more specific within that block.

```python
from typing import Union

def process_data(data: Union[str, int]) -> None:
    print(f"Received: {data}")
    if isinstance(data, str):
        # Inside this block, MyPy knows `data` is a `str`.
        # So this is safe:
        print(data.upper())
    else:
        # Inside this block, MyPy knows `data` must be an `int`.
        # So this is safe:
        print(f"Square of data is {data * data}")
```

### 3. The `match` Statement (Python 3.10+)
The `match` statement provides a clean way to handle complex conditional logic, similar to a `switch` statement in other languages but much more powerful. It's called "structural pattern matching".

```python
from typing import Any

def describe(item: Any) -> None:
    match item:
        case str():
            print("It's a string.")
        case int() | float():
            print("It's a number.")
        case list() if len(item) > 3:
            print("It's a long list.")
        case _:  # The underscore is a wildcard, catching anything else.
            print("It's something else.")

describe("hello")  # It's a string.
describe([1, 2, 3, 4, 5]) # It's a long list.
```
The `match` statement also performs type narrowing.

### 4. Guard Clauses (Early Returns)
Guard clauses are a pattern for writing cleaner functions by checking for edge cases or invalid conditions at the very beginning of the function and exiting early. This avoids deeply nested `if/else` blocks.

```python
# --- Nested Style (Avoid) ---
def process_item(item: dict | None) -> str:
    if item is not None:
        if "name" in item:
            # ... more nested logic
            return f"Processing {item['name']}"
        else:
            return "Error: No name key"
    else:
        return "Error: No item"

# --- Guard Clause Style (Prefer) ---
def process_item_clean(item: dict | None) -> str:
    if item is None:
        return "Error: No item"  # Early exit

    if "name" not in item:
        return "Error: No name key" # Early exit

    # Now, the main logic is not nested
    return f"Processing {item['name']}"
```

### 5. Assertion Types
The `assert` statement checks if a condition is true. If it's not, it raises an `AssertionError`. Type checkers also use `assert` for type narrowing. This is useful when you have information about a type that the checker doesn't.

```python
from typing import Optional

def greet(name: Optional[str]) -> None:
    # We assert that name is not None.
    # If it is, the program will crash here.
    assert name is not None, "Name cannot be None for greeting!"

    # After the assert, MyPy knows `name` must be a `str`.
    # So this is type-safe:
    print(f"Hello, {name.upper()}")
```
Assertions are great for internal checks and making types more specific, but they can be disabled in production, so they shouldn't replace proper error handling for user input.

## 🔹 Quick Exercise

Implement the following two functions. Use the patterns discussed today.

```python
from typing import Literal, Optional

def classify_age(age: int) -> Literal["child", "teen", "adult", "senior"]:
    """
    Classifies an age into a category using a match statement or if/elif.
    - child: 0-12
    - teen: 13-19
    - adult: 20-64
    - senior: 65+
    """
    # Implement here using a match statement or if/elif/else
    pass

def safe_divide(a: float, b: float) -> Optional[float]:
    """
    Divides a by b, but returns None if b is zero.
    Uses a guard clause for the zero check.
    """
    # Use a guard clause to handle the case where b is 0.
    # Implement here
    pass

# --- Tests ---
assert classify_age(10) == "child"
assert classify_age(18) == "teen"
assert classify_age(35) == "adult"
assert classify_age(70) == "senior"
assert safe_divide(10, 2) == 5.0
assert safe_divide(10, 0) is None
```

## 📝 Daily Assignment
**Goal**: Build a small, type-safe decision tree system.

1.  **Create Project File**: In `my-first-poetry-app`, create a new file: `my_first_poetry_app/decision_maker.py`.
2.  **Define Data Structures**:
    - Use `TypedDict` or `dataclasses` to define a structure for a `Character`. A character should have attributes like `health: int`, `mana: int`, `has_weapon: bool`.
3.  **Implement Decision Functions**:
    - Create a function `should_attack(character: Character, enemy_is_close: bool) -> bool`. This function should use `if/elif/else` and boolean logic to decide if the character should attack. (e.g., `if character['has_weapon'] and enemy_is_close:`).
    - Create a function `choose_action(character: Character, situation: Literal["combat", "stealth", "rest"]) -> str`. This function must use a `match` statement on the `situation` parameter to return an appropriate action string (e.g., "Attack!", "Hide.", "Meditate.").
4.  **Implement Type Narrowing**:
    - Create a function `process_input(user_input: str | int) -> str`.
    - Inside, use `isinstance` to check if the input is a `str` or an `int`.
    - If it's a string, return its uppercase version.
    - If it's an integer, return its value multiplied by 2, converted to a string.
    - This demonstrates type narrowing.
5.  **Use Guard Clauses**:
    - Create a function `get_character_weapon_damage(character: Character | None) -> int`.
    - Use guard clauses to check if `character` is `None` or if `character['has_weapon']` is `False`. In these cases, return `0`.
    - If all checks pass, return a default damage value, like `10`.
6.  **`main()` function**:
    - Create a `main` function to demonstrate all your decision functions.
    - Create a sample character dictionary.
    - Call your functions with different inputs and print the results.
7.  **Verify**: Run `poetry run python ...` and `poetry run mypy .` and `poetry run ruff check .` and fix all issues.

## ⚠️ Common Mistakes
- **Complex nested `if`s**: Code that looks like an arrowhead (`>`) with many levels of indentation is a "code smell". It's hard to read and maintain. Refactor using guard clauses or by breaking the logic into smaller functions.
- **`match` statement without a wildcard**: If you use a `match` statement and don't include a wildcard case (`case _: ...`), MyPy will warn you if it's possible for a value to not match any case, which could lead to a runtime error.
- **Confusing `is` and `==`**:
  - Use `is` for identity checks, which is best for singletons like `None`, `True`, and `False`. (e.g., `if x is None:`).
  - Use `==` for equality checks, which compares the values of two objects. (e.g., `if name == "Jules":`).
- **Relying on "truthiness" with ambiguous types**: An empty string `""` is `False` in a boolean context. If a function can return `str | None`, checking `if result:` is risky because both `None` and `""` will evaluate to `False`. Be explicit: `if result is not None:`.

## 📖 Further Reading
- [Python Docs: More Control Flow Tools](https://docs.python.org/3/tutorial/controlflow.html) (Covers `if`, `for`, `match`, etc.)
- [Real Python: Structural Pattern Matching](https://realpython.com/python-match-case-statement/)
- [MyPy Docs: Type Narrowing](https://mypy.readthedocs.io/en/stable/type_narrowing.html)
- [Refactoring Guru: Replace Nested Conditional with Guard Clauses](https://refactoring.guru/replace-nested-conditional-with-guard-clauses)
