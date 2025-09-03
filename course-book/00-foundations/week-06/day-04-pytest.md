# Week 6, Day 4: Testing with pytest

## 🎯 Learning Objectives
- [ ] Understand the importance of automated testing.
- [ ] Write basic unit tests for functions using `pytest`.
- [ ] Use `pytest` fixtures to set up and tear down test state.
- [ ] Use `pytest` parametrization to run the same test with multiple different inputs.

## 📚 Concepts

### 1. Why Test?
Automated tests are programs that check if your main program's code works as expected. They are a critical part of modern software development.
- **Confidence**: Tests give you confidence that your code works and that new changes haven't broken existing functionality (this is called preventing "regressions").
- **Documentation**: Well-written tests serve as executable documentation, showing how your code is intended to be used.
- **Better Design**: Writing testable code often forces you to write smaller, more focused, and better-designed functions and classes.

### 2. `pytest` Basics
`pytest` is the most popular testing framework for Python. It makes testing simple and scalable with its minimal boilerplate, powerful features, and rich plugin ecosystem.

To write a test for `pytest`:
1.  Create a file named `test_*.py` or `*_test.py`.
2.  Write functions with names that start with `test_`.
3.  Inside these functions, use the `assert` keyword to check for expected outcomes.

```python
# In a file named 'my_app/math_utils.py'
def add(a: int, b: int) -> int:
    return a + b

# In a file named 'tests/test_math_utils.py'
from my_app.math_utils import add

def test_add_positive_numbers():
    # The assert statement is the test itself.
    # If the condition is True, the test passes. If False, it fails.
    assert add(2, 3) == 5

def test_add_negative_numbers():
    assert add(-1, -1) == -2
```
To run your tests, you simply navigate to your project's root directory in the terminal and run `poetry run pytest`. `pytest` will automatically discover and run your test functions.

### 3. `pytest` Assertions
`pytest` uses the standard `assert` statement but enhances it with "introspection". When an assertion fails, `pytest` provides detailed output showing the values of the variables involved, which makes debugging much easier.

```
# Example of a failing test output
E   assert add(2, 3) == 6
E    + where 5 = add(2, 3)
```
This output clearly shows that the `add(2, 3)` call resulted in `5`, which does not equal the expected `6`.

### 4. Fixtures: Managing Test State
A **fixture** is a function that provides a fixed baseline state or data for your tests. They are perfect for setting up resources that many tests need, like a database connection, a temporary file, or a complex object.

You define a fixture by decorating a function with `@pytest.fixture`. To use it, you add the fixture function's name as an argument to your test function.

```python
import pytest

@pytest.fixture
def sample_user_data() -> dict:
    """A fixture that provides a sample user dictionary."""
    return {"name": "Alice", "email": "alice@example.com", "is_active": True}

# Pytest sees the argument and injects the fixture's return value.
def test_user_is_active(sample_user_data: dict):
    assert sample_user_data["is_active"] is True

def test_user_has_email(sample_user_data: dict):
    assert "@" in sample_user_data["email"]
```

### 5. Parametrization: Running a Test Multiple Times
Sometimes you want to run the same test logic with many different inputs and expected outputs. `@pytest.mark.parametrize` is a decorator that lets you do this without writing duplicate code.

```python
import pytest
from my_app.math_utils import add

# The first argument is a string of comma-separated argument names.
# The second argument is a list of tuples, where each tuple contains the values for one test run.
@pytest.mark.parametrize("a, b, expected", [
    (1, 2, 3),          # Test case 1
    (-1, 1, 0),         # Test case 2
    (0, 0, 0),          # Test case 3
    (100, 200, 300),    # Test case 4
])
def test_add_parametrized(a: int, b: int, expected: int):
    assert add(a, b) == expected
```
This single test function will run four times, once for each tuple in the list.

## 🔹 Quick Exercise

Using the `add` function from the concepts section, complete the two test functions below. The first is a simple test, and the second uses parametrization.

```python
import pytest
# Assume `add` is in a file that can be imported
# from my_app.math_utils import add

def add(a: int, b: int) -> int: return a + b

def test_addition():
    """Tests that add(2, 3) equals 5."""
    # Write the assertion here
    assert add(2, 3) == 5

@pytest.mark.parametrize("a, b, expected", [
    (10, 20, 30),
    (-5, 5, 0),
    (0, -5, -5),
])
def test_add_with_various_inputs(a: int, b: int, expected: int):
    """Tests the add function with various inputs."""
    # Write the assertion here
    assert add(a, b) == expected
```

## 📝 Daily Assignment
**Goal**: Write a comprehensive test suite for a small utility module.

1.  **Create Utility and Test Files**:
    -   In your `my-first-poetry-app` project, create `my_first_poetry_app/data_validator.py`.
    -   In your `tests` directory, create `tests/test_data_validator.py`.
2.  **Implement Utility Functions**: In `data_validator.py`, create and implement these functions:
    -   `is_valid_email(email: str) -> bool`: Checks if a string is a valid email.
    -   `is_valid_age(age: int) -> bool`: Checks if an age is between 18 and 120.
    -   `get_username_from_email(email: str) -> str | None`: Extracts the part before the `@` symbol. Returns `None` if the email is invalid.
3.  **Write Tests in `test_data_validator.py`**:
    -   **Write at least 50 tests in total.** This sounds like a lot, but it's easy with parametrization!
    -   For `is_valid_email`, use `@pytest.mark.parametrize` to test at least 10 valid emails and 10 invalid emails.
    -   For `is_valid_age`, use parametrization to test edge cases: `17`, `18`, `120`, `121`, `0`, `-1`.
    -   For `get_username_from_email`, test both valid and invalid emails.
4.  **Use a Fixture**:
    -   Create a fixture `valid_user_payload()` that returns a dictionary like `{"email": "test@example.com", "age": 30}`.
    -   Write a test that uses this fixture to check that both the email and age from the payload are valid according to your functions.
5.  **Achieve 100% Test Coverage**:
    -   Run your tests with the coverage flag: `poetry run pytest --cov=my_first_poetry_app`.
    -   Analyze the report. If any lines in `data_validator.py` are not covered, write more tests until you reach 100% coverage.

## ⚠️ Common Mistakes
- **Tests depending on each other**: Each test function should be completely independent. Never write a test that relies on another test having run first. Fixtures are the correct way to share setup.
- **Not testing edge cases**: It's easy to test the "happy path" (e.g., `add(2, 3)`). The real value of tests is in checking the edges: what happens with `0`, `-1`, an empty list, `None`?
- **Slow tests**: Tests should be fast. If a test needs to do something slow (like a network request), you should "mock" that slow part out. We'll cover mocking later, but for now, keep tests focused on fast, in-memory operations.
- **Flaky tests**: A test that sometimes passes and sometimes fails without code changes is a "flaky" test. This is often due to relying on things like the current time or random numbers. Tests should be deterministic.

## 📖 Further Reading
- [pytest Official Documentation](https://docs.pytest.org/en/stable/)
- [Real Python: Getting Started With `pytest`](https://realpython.com/pytest-python-testing/)
- [Brian Okken: Python Testing with pytest](https://pragprog.com/titles/bopytest2/python-testing-with-pytest-second-edition/) (An excellent book)
