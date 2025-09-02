# Week 10, Day 2: Monads & Functional Patterns

## 🎯 Learning Objectives
- [ ] Understand the core problem that monad-like patterns solve: handling "wrapper" types in sequential computations.
- [ ] Implement a `Maybe` (or `Option`) type to safely handle computations that might return `None`.
- [ ] Implement a `Result` (or `Either`) type to handle computations that can fail.
- [ ] Use these patterns to write clean, chainable data processing pipelines without complex `if/else` checks.

**Heads-up**: "Monad" is a term from category theory with a reputation for being difficult. We are not going to dive deep into the theory. Instead, we'll focus on the **practical patterns** inspired by monads that are incredibly useful in Python for error handling and data flow. The goal is to understand the "why" and "how" of these patterns in a Pythonic context.

## 📚 Concepts

### 1. The Core Problem: Chaining "Wrapped" Values
Imagine a series of functions where each can fail (by returning `None`).

```python
def find_user(name: str) -> dict | None: ...
def get_address(user: dict) -> str | None: ...
def get_zip_code(address: str) -> str | None: ...

# To get a zip code, you end up with this "staircase" of `if` statements:
user = find_user("Alice")
if user is not None:
    address = get_address(user)
    if address is not None:
        zip_code = get_zip_code(address)
        if zip_code is not None:
            print(f"Zip code: {zip_code}")
```
This is called the "pyramid of doom" and it's hard to read and maintain. Monadic patterns solve this by creating a "wrapper" or "container" object that knows how to handle the `None` or error case itself.

### 2. The `Maybe` (or `Option`) Pattern
A `Maybe` represents a value that might be absent. It's a container that is either `Some(value)` or `Nothing`. This makes the possibility of an absent value explicit in the type system, unlike `Optional[T]`, which is just an alias for `T | None`.

The key is to give this container a method (often called `map` or `bind`) that knows how to apply a function to the *wrapped* value.

- If the container is `Some(value)`, it applies the function to `value` and wraps the result in a new `Some`.
- If the container is `Nothing`, it does nothing and just passes the `Nothing` along.

```python
from dataclasses import dataclass
from typing import Generic, TypeVar, Callable, Optional

T = TypeVar('T')
U = TypeVar('U')

@dataclass(frozen=True)
class Maybe(Generic[T]):
    _value: Optional[T]

    def bind(self, func: Callable[[T], 'Maybe[U]']) -> 'Maybe[U]':
        if self._value is None:
            return Maybe(None) # Pass the 'Nothing' along
        return func(self._value)

# Now we can rewrite our functions to return a Maybe
def find_user_maybe(name: str) -> Maybe[dict]: ...
def get_address_maybe(user: dict) -> Maybe[str]: ...
def get_zip_code_maybe(address: str) -> Maybe[str]: ...

# And the call becomes a clean, readable chain:
# zip_code_maybe = (
#     find_user_maybe("Alice")
#     .bind(get_address_maybe)
#     .bind(get_zip_code_maybe)
# )
```
The ugly `if` checks are now hidden inside the `bind` method of our `Maybe` container.

### 3. The `Result` (or `Either`) Pattern
The `Maybe` pattern is great for handling `None`, but what if you want to know *why* something failed? The `Result` pattern handles this. A `Result` is a container that is either `Ok(value)` or `Err(error_details)`.

This is even more powerful because the error can be a string, an exception, or any object with details about the failure.

```python
E = TypeVar('E')

@dataclass(frozen=True)
class Result(Generic[T, E]):
    _value: T | None = None
    _error: E | None = None

    def bind(self, func: Callable[[T], 'Result[U, E]']) -> 'Result[U, E]':
        if self._error is not None:
            return self # Pass the error along
        if self._value is None:
             # This case should ideally not happen if constructed properly
             return Result(_error=self._error)
        return func(self._value)

def parse_int(s: str) -> Result[int, str]:
    try:
        return Result(_value=int(s))
    except ValueError:
        return Result(_error=f"'{s}' is not a valid integer")

# Chaining operations that can fail
result = (
    parse_int("10")
    .bind(lambda x: Result(_value=x * 2))
) # Result(_value=20, _error=None)

result_fail = (
    parse_int("abc")
    .bind(lambda x: Result(_value=x * 2))
) # Result(_value=None, _error="'abc' is not a valid integer")
```

### 4. Functors and the `map` method
A **Functor** is any type that has a `map` method which applies a function to its wrapped value(s). Our `Maybe` and `Result` types can be made into Functors. The `map` method is like `bind`, but it's for applying a regular function that returns a plain value, not another container.

```python
# On our Maybe class
def map(self, func: Callable[[T], U]) -> 'Maybe[U]':
    if self._value is None:
        return Maybe(None)
    return Maybe(func(self._value))

# Usage
length_maybe = Maybe("hello").map(len) # Maybe(5)
```

## 🔹 Quick Exercise

Complete the `Maybe` class below by implementing the `map` method.

```python
from dataclasses import dataclass
from typing import Generic, TypeVar, Callable, Optional

T = TypeVar('T')
U = TypeVar('U')

@dataclass(frozen=True)
class Maybe(Generic[T]):
    _value: Optional[T]

    def map(self, func: Callable[[T], U]) -> 'Maybe[U]':
        """
        If this Maybe has a value, apply the function to it and return
        a new Maybe containing the result. Otherwise, return an empty Maybe.
        """
        if self._value is None:
            return Maybe(None)
        return Maybe(func(self._value))

# --- Tests ---
assert Maybe("hello").map(len) == Maybe(5)
assert Maybe(None).map(len) == Maybe(None)
assert Maybe(10).map(lambda x: x * 2).map(str) == Maybe("20")

print("Maybe.map works!")
```

## 📝 Daily Assignment
**Goal**: Implement both `Maybe` and `Result` patterns and use them to refactor a data processing function.

1.  **Create Project File**: In your project, create `my_first_poetry_app/monadic_patterns.py`.
2.  **Implement `Maybe`**:
    -   Create a complete, generic `Maybe[T]` dataclass.
    -   It should have a `bind(self, func)` method.
    -   It should have a `map(self, func)` method.
    -   Add a static method `Maybe.from_optional(value: Optional[T]) -> Maybe[T]` to make it easy to create.
3.  **Implement `Result`**:
    -   Create a complete, generic `Result[T, E]` dataclass (where `T` is the success type and `E` is the error type).
    -   It should have a `bind(self, func)` method that only calls `func` on success.
    -   It should have a `map(self, func)` method that only calls `func` on success.
    -   It should have helper properties like `is_ok()` and `is_err()`.
4.  **Refactor with the Patterns**:
    -   Start with this "pyramid of doom" code:
        ```python
        import json
        def get_user_score(user_data_str: str) -> str:
            try:
                data = json.loads(user_data_str)
                if "user" in data:
                    user = data["user"]
                    if "score" in user:
                        return f"User score is {user['score']}"
                    else:
                        return "Error: 'score' key missing"
                else:
                    return "Error: 'user' key missing"
            except json.JSONDecodeError:
                return "Error: Invalid JSON"
        ```
    -   Refactor this into a clean pipeline using your `Result` type. Create small functions that each do one step and return a `Result` (e.g., `parse_json`, `get_user_key`, `get_score_key`). Chain them together with `.bind()`.
5.  **Verify**: Write a `main` function that tests your refactored pipeline with valid and invalid JSON strings. Ensure it passes `mypy` and `ruff`.

## ⚠️ Common Mistakes
- **Making it too complicated**: These patterns are meant to *simplify* code. If your implementation is making things harder to read, you might be over-engineering. The goal is to hide `if` statements, not add more layers of complexity.
- **Breaking the chain**: The power of these patterns comes from chaining. The `bind` and `map` methods must always return a new instance of the container (`Maybe` or `Result`) so that the next call in the chain can work.
- **`map` vs. `bind`**: A common confusion.
  - `map` is for applying a simple function `A -> B`.
  - `bind` (or `flat_map`) is for applying a function that *already returns a container*, `A -> Container[B]`. It "flattens" the result to avoid nested containers like `Maybe[Maybe[int]]`.

## 📖 Further Reading
- [F# for Fun and Profit: Railway Oriented Programming](https://fsharpforfunandprofit.com/rop/) (A fantastic, practical explanation of the `Result` pattern)
- [Monads in Python](https://dev.to/rattis/monads-in-python-2d3e) (A more code-focused Python example)
