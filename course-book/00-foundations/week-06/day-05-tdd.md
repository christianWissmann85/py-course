# Week 6, Day 5: Test-Driven Development (TDD)

## 🎯 Learning Objectives

- [ ] Understand the principles and workflow of Test-Driven Development (TDD).
- [ ] Practice the "Red-Green-Refactor" cycle.
- [ ] Use test coverage tools to measure how much of your code is exercised by tests.
- [ ] Understand the basic concept of mocking for isolating units in tests.

## 📚 Concepts

### 1. TDD Principles

Test-Driven Development is a software development process that reverses the traditional "write code, then test it" sequence. In TDD, you follow a short, repetitive cycle:

1.  **Write a test** for a small piece of functionality you want to add.
2.  **Run the test** and watch it fail (this is the "Red" phase). This is important to ensure your test is actually testing something and doesn't pass by default.
3.  **Write the minimum amount of code** necessary to make the test pass (the "Green" phase).
4.  **Refactor** the code you just wrote to improve its design, while ensuring the test still passes.
5.  Repeat the cycle for the next piece of functionality.

This process leads to a comprehensive test suite, better code design, and a rapid, iterative workflow.

### 2. The Red-Green-Refactor Cycle

This is the core mantra of TDD.

- **Red**: Write a failing test. This test defines the _what_—what you want the code to do. For example, a test `test_add_two_numbers` that asserts `add(2, 3) == 5`. Before the `add` function even exists, this test will fail.
- **Green**: Write the simplest possible code to make the test pass. Don't add extra features. Don't try to be clever. Just make the bar green.
  ```python
  # The simplest code to make `test_add_two_numbers` pass.
  def add(a, b):
      return 5 # This is a valid, if incomplete, first step!
  ```
  Or more realistically:
  ```python
  def add(a, b):
      return a + b
  ```
- **Refactor**: Now that you have a passing test acting as a safety net, you can clean up your code. Is the variable naming clear? Can you make it more efficient? Can you remove duplication? You can make these changes with confidence, because if you break something, your test will fail and you'll know immediately.

### 3. Test Coverage

Test coverage is a metric that measures the percentage of your codebase that is executed by your test suite. 100% coverage means every single line of your production code was run during your tests.

`pytest-cov` is a plugin for `pytest` that measures this.

```bash
# Run pytest and generate a coverage report
poetry run pytest --cov=my_app
```

High coverage is a good goal, but it's not a guarantee of quality. It's possible to have 100% coverage but still have poor tests that don't check for the right things. However, low coverage is a clear sign that you have untested code.

### 4. Mocking (A Brief Introduction)

What if the function you're testing depends on something slow or unpredictable, like a network API or a database? You can't have your fast unit tests depending on these things.

**Mocking** is the practice of replacing these real dependencies with "fake" objects that you can control in your test. Python's `unittest.mock` library (which works with `pytest`) is the standard tool for this.

```python
from unittest.mock import Mock

# Create a mock object that pretends to be a database API
mock_db = Mock()

# Configure its return value for a specific call
mock_db.get_user.return_value = {"name": "Alice", "id": 1}

# Now you can pass this mock object to your function instead of a real database.
user = get_user_from_db(db_api=mock_db, user_id=1)

# You can even assert that the mock was called correctly
mock_db.get_user.assert_called_once_with(user_id=1)
```

We will cover mocking in more detail in later weeks, but it's a key enabler for TDD.

### 5. Unit vs. Integration Tests

- **Unit Test**: Tests a single "unit" of code (usually a function or method) in isolation. Dependencies are often mocked. These tests are fast and form the foundation of your test suite. TDD primarily produces unit tests.
- **Integration Test**: Tests how multiple units work together. For example, testing that your code can correctly call a real database. These are slower and more complex but are crucial for testing the connections between parts of your system.

## 🔹 Quick Exercise

Let's practice the TDD cycle for a simple function, `is_valid_email`.

**Step 1: Red**
Write the first test case in `tests/test_validator.py`.

```python
# tests/test_validator.py
from my_app.validator import is_valid_email

def test_is_valid_email_simple_case():
    assert is_valid_email("test@example.com") is True
```

Run `pytest`. It will fail because `is_valid_email` doesn't exist yet.

**Step 2: Green**
Write the simplest possible code in `my_app/validator.py` to make it pass.

```python
# my_app/validator.py
def is_valid_email(email: str) -> bool:
    return True # Simplest code to make the test pass!
```

Run `pytest`. It passes.

**Step 3: Red (Again)**
Add a new test for a failing case.

```python
# tests/test_validator.py
def test_is_valid_email_no_at_symbol():
    assert is_valid_email("testexample.com") is False
```

Run `pytest`. The new test fails.

**Step 4: Green (Again)**
Improve your implementation to handle the new case.

```python
# my_app/validator.py
def is_valid_email(email: str) -> bool:
    return "@" in email
```

Run `pytest`. Both tests now pass.

**Step 5: Refactor**
The code is simple, but we could add a regex for a more robust check. The tests give us the safety to do so.

```python
# my_app/validator.py
import re
def is_valid_email(email: str) -> bool:
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))
```

Run `pytest`. Both tests still pass. The refactor was successful. You would continue this cycle, adding more test cases for edge cases.

## 📝 Daily Assignment

**Goal**: Build a new feature for your `BankAccount` class using a strict TDD workflow.

1.  **Get Ready**: Open your `models.py` file (or wherever your `BankAccount` class is) and create a corresponding `tests/test_models.py`.
2.  **Feature: Transfer Money**: You want to add a `transfer` method to your `BankAccount` class. It should move money from one account to another.
3.  **TDD Cycle 1: Successful Transfer**
    - **Red**: In `test_models.py`, write a test `test_transfer_successful`. Create two bank accounts. Call `account1.transfer(account2, 50)`. Assert that `account1.balance` has decreased by 50 and `account2.balance` has increased by 50. Run the test; it will fail because `.transfer` doesn't exist.
    - **Green**: Implement the `transfer` method on the `BankAccount` class with the minimum code to make the test pass.
    - **Refactor**: Is the code clean? Is it readable?
4.  **TDD Cycle 2: Insufficient Funds**
    - **Red**: Write a test `test_transfer_insufficient_funds`. Try to transfer more money than is in the account. Assert that a `ValueError` (or a custom exception) is raised. Use `pytest.raises` for this.
    - **Green**: Modify your `transfer` method to check the balance and raise the exception.
5.  **TDD Cycle 3: Invalid Amount**
    - **Red**: Write a test `test_transfer_negative_amount`. Try to transfer a negative amount. Assert that a `ValueError` is raised.
    - **Green**: Add a check for a positive transfer amount.
6.  **Measure Coverage**: Run `poetry run pytest --cov` and ensure your `BankAccount` class has 100% test coverage. If not, add tests for any lines that were missed.

## 📦 Week 6 Project: Robust CLI Application

This week's project is to build a command-line application that is robust, well-tested, and uses logging and custom exceptions. You could build a simple To-Do list manager or a contact book.

**Key Requirements**:

- **Comprehensive Error Handling**: Use `try...except` and your own custom exceptions to handle all foreseeable errors (e.g., file not found, invalid user input, item not found).
- **Custom Exceptions**: Design a small hierarchy of custom exceptions for your application.
- **Full Test Suite**: Use TDD to develop the core logic. Aim for high test coverage.
- **Logging System**: Integrate logging to record application events, user actions, and errors to both the console and a file.

See `week-06-project.md` for a more detailed breakdown.

## ⚠️ Common Mistakes

- **Skipping the "Red" step**: It's tempting to write the code first. Don't. Seeing the test fail first proves that your test is working correctly and is not passing for the wrong reasons.
- **Writing too much code in the "Green" step**: The goal is to write only enough code to pass the current failing test. This keeps the cycle short and focused.
- **Forgetting to refactor**: The "Refactor" step is where the design of your code improves. Skipping it leads to messy code that just happens to pass the tests.
- **Treating 100% coverage as the goal**: High coverage is a _by-product_ of good TDD, not the goal itself. The goal is well-tested, well-designed, working software.

## 📖 Further Reading

- [TestDriven.io: Introduction to Test-Driven Development](https://testdriven.io/blog/test-driven-development-in-python/)
- [FreeCodeCamp: The TDD "Red-Green-Refactor" Mantra](https://www.freecodecamp.org/news/the-tdd-red-green-refactor-mantra-4356633a93c9/)
- [Python Docs: `unittest.mock`](https://docs.python.org/3/library/unittest.mock.html) (The standard library for mocking)
