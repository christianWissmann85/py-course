# Week 6, Day 2: Custom Exceptions

## 🎯 Learning Objectives

- [ ] Create custom exception classes by inheriting from Python's built-in `Exception` class.
- [ ] Design logical exception hierarchies for your application.
- [ ] Add attributes to your custom exceptions to provide rich error context.
- [ ] Understand when it is appropriate to create a custom exception.

## 📚 Concepts

### 1. Creating a Custom Exception

While Python's built-in exceptions cover many cases, sometimes you need to create your own to represent errors specific to your application's domain. A custom exception is simply a class that inherits from the base `Exception` class (or another more specific exception).

The simplest custom exception is just a new name for an error:

```python
class MyCustomError(Exception):
    """A base exception for my application."""
    pass

def do_something_risky():
    # Some condition that is an error in our application's logic
    is_error = True
    if is_error:
        raise MyCustomError("Something specific went wrong!")

try:
    do_something_risky()
except MyCustomError as e:
    print(f"Caught a custom error: {e}")
```

Using a custom exception makes your `except` blocks more specific and your code's intent clearer.

### 2. Designing Error Hierarchies

Just like Python's built-in exceptions, you can create a hierarchy for your own exceptions. This allows you to catch errors at different levels of granularity. A common pattern is to have a single base exception for your library or application, and then create more specific exceptions that inherit from it.

```python
class MyAppError(Exception):
    """Base exception for all errors in MyApp."""
    pass

class DatabaseError(MyAppError):
    """An error related to the database."""
    pass

class NetworkError(MyAppError):
    """An error related to network communication."""
    pass

try:
    # Code that might raise either error
    ...
except DatabaseError:
    print("Handling a database-specific issue...")
except NetworkError:
    print("Handling a network-specific issue...")
except MyAppError:
    # This would catch any other error inheriting from MyAppError
    print("Handling a generic app issue...")
```

### 3. Adding Attributes for Context

A simple error message string is good, but sometimes you need to provide more structured information about what went wrong. You can do this by adding an `__init__` method to your custom exception class and storing context as attributes.

```python
from typing import Any

class ValidationError(MyAppError):
    """Raised when an input value is invalid."""
    def __init__(self, message: str, field_name: str, invalid_value: Any):
        self.field_name = field_name
        self.invalid_value = invalid_value
        # Call the parent's init to store the message
        super().__init__(f"Validation failed for '{field_name}': {message}")

def validate_age(age: int):
    if not 0 < age < 120:
        raise ValidationError("Age is out of range", field_name="age", invalid_value=age)

try:
    validate_age(200)
except ValidationError as e:
    print(e) # The error message from super().__init__
    print(f"Field: {e.field_name}")
    print(f"Invalid Value: {e.invalid_value}")
```

### 4. When to Create Custom Exceptions

Don't create a custom exception if a built-in one accurately describes the error.

- Trying to convert `"abc"` to an `int`? `ValueError` is perfect.
- Trying to access a list index that's too large? `IndexError` is correct.
- A user provides a password that's too short? This is an application-specific rule. A custom `InvalidPasswordError` is a great choice.

Rule of thumb: Create a custom exception when you want to handle a specific error from your own code differently from a built-in error, or when the error represents a failure in your application's business logic.

## 🔹 Quick Exercise

Complete the `ValidationError` class below by implementing its `__init__` method. It should store the field name, invalid value, and a descriptive message.

```python
from typing import Any

class ValidationError(Exception):
    """Custom validation error."""

    def __init__(self, field: str, value: Any, message: str) -> None:
        self.field = field
        self.value = value
        self.message = message
        # Use super().__init__ to create a helpful, combined error message.
        # For example: f"Validation Error on field '{field}': {message}. Got value: {value!r}"
        super().__init__(f"Validation Error on field '{field}': {message}. Got value: {value!r}")

# --- Tests ---
try:
    raise ValidationError(field="email", value="not-an-email", message="Invalid format")
except ValidationError as e:
    assert e.field == "email"
    assert e.value == "not-an-email"
    assert "Invalid format" in str(e)
    print("Custom exception works as expected!")
```

## 📝 Daily Assignment

**Goal**: Build a small user validation system that uses a hierarchy of custom exceptions.

1.  **Create Project File**: In your `my-first-poetry-app` project, create a new file: `my_first_poetry_app/validators.py`.
2.  **Create an Exception Hierarchy**:
    - `class UserValidationError(Exception)`: A base class for all user validation errors.
    - `class UsernameError(UserValidationError)`: A subclass for username-related issues.
    - `class PasswordError(UserValidationError)`: A subclass for password-related issues.
3.  **Implement Validator Functions**:
    - `validate_username(username: str) -> None`:
      - Raises `UsernameError` if the username is shorter than 3 characters or longer than 20.
    - `validate_password(password: str) -> None`:
      - Raises `PasswordError` if the password is shorter than 8 characters.
      - Raises `PasswordError` if the password does not contain at least one number.
4.  **Implement a Registration Function**:
    - `register_user(username: str, password: str) -> None`:
      - This function should call your two validator functions inside a `try...except` block.
      - It should be able to catch the specific `UsernameError` and `PasswordError`, as well as the general `UserValidationError`.
      - In the `except` blocks, print user-friendly error messages.
      - If no exceptions are raised, print a "User registered successfully!" message.
5.  **`main()` function**:
    - Call `register_user` with several different inputs to trigger all your exception cases (valid, bad username, bad password, etc.).
6.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes

- **Creating too many exceptions**: Don't create a new exception for every single error condition. Group related errors under a common parent exception. The goal is to make handling errors _easier_.
- **Making exceptions too generic**: An exception named `Error` is not helpful. Be specific, like `ApiConnectionError`.
- **Catching your custom exception before a more specific one**: In a `try...except` block, Python checks the `except` clauses in order. If you put `except UserValidationError:` before `except UsernameError:`, the `UsernameError` block will never be reached, because a `UsernameError` _is a_ `UserValidationError`. Always put more specific exceptions first.
- **Not inheriting from `Exception`**: While you can raise any object in Python, only objects that inherit from `BaseException` (and typically `Exception`) are considered proper exceptions.

## 📖 Further Reading

- [Python Docs: User-defined Exceptions](https://docs.python.org/3/tutorial/errors.html#user-defined-exceptions)
- [Real Python: Creating Custom Exceptions in Python](https://realpython.com/python-custom-exceptions/)
- [Google Python Style Guide on Exceptions](https://google.github.io/styleguide/pyguide.html#24-exceptions)
