# Week 1, Day 3: Type Hints & MyPy

## 🎯 Learning Objectives
- [ ] Understand the difference between static and dynamic typing.
- [ ] Learn to write basic type annotations for variables and functions.
- [ ] Configure MyPy for strict type checking in a Poetry project.
- [ ] Learn to identify and debug common type errors reported by MyPy.

## 📚 Concepts

### 1. Static vs. Dynamic Typing
Python is a dynamically typed language. This means you don't have to declare the type of a variable when you create it. The interpreter figures it out at runtime.

```python
# Dynamic typing
my_variable = 10         # It's an integer
my_variable = "hello"  # Now it's a string, Python doesn't complain
```

Static typing, found in languages like Java or C++, requires you to declare variable types upfront. This catches many errors before the program even runs.

Python introduced **type hints** (or type annotations) to bring the benefits of static typing to a dynamic language. They are optional "hints" that don't affect how the code runs, but can be checked by external tools like **MyPy**.

### 2. Basic Type Annotations
You can add type hints to variables, function arguments, and function return values.

- **Variables**: `variable_name: type = value`
- **Function arguments**: `def my_func(param_name: type):`
- **Function return value**: `def my_func() -> type:`

```python
# Variable annotation
user_name: str = "Alice"
user_age: int = 30

# Function annotation
def greet(name: str, is_excited: bool = False) -> str:
    message = f"Hello, {name}!"
    if is_excited:
        message += "!!!"
    return message
```

### 3. MyPy Configuration
MyPy is the de-facto standard type checker for Python. It reads your type hints and reports any inconsistencies. For this course, we will use it in its strictest mode to catch the maximum number of potential bugs.

You can configure MyPy in your `pyproject.toml` file:

```toml
[tool.mypy]
python_version = "3.12"
warn_return_any = true
warn_unused_configs = true
# For this course, we start with the strictest settings!
strict = true
```

To run MyPy, you use `poetry run mypy .` from the root of your project.

### 4. Common Type Errors
MyPy will help you find errors like:
- `error: Incompatible types in assignment (expression has type "int", variable has type "str")`
- `error: Argument 1 to "add" has incompatible type "str"; expected "int"`
- `error: Missing return statement`
- `error: "add" does not return a value`

Learning to read and understand these errors is a fundamental skill.

### 5. Benefits of Type Safety
- **Fewer Bugs**: Catches a whole class of errors before you even run your code.
- **Improved Readability**: Type hints make code easier to understand and serve as documentation.
- **Better Tooling**: IDEs like VS Code use type hints to provide better autocompletion and error highlighting.
- **Easier Refactoring**: Makes it safer to make large changes to your codebase.
- **Effective AI Collaboration**: Clear type definitions are essential for getting accurate and useful code from AI assistants.

## 💻 Code Examples

### A function before and after typing
```python
# Before (dynamic)
def calculate_total(items):
    total = 0
    for item in items:
        total += item['price']
    return total

# After (with type hints)
from typing import List, Dict, Any

def calculate_total_typed(items: List[Dict[str, Any]]) -> float:
    total: float = 0.0
    for item in items:
        total += item['price']
    return total

# MyPy can now check that you always pass a list of dictionaries
# and that each dictionary has a 'price' key.
# It also checks that you always return a float.
```

## 🔹 Quick Exercise

Add the correct type hints to the following functions. Test your annotations by saving the code to a file in a Poetry project (with MyPy configured) and running `poetry run mypy <your_file.py>`.

```python
# 1. Add types to this function
def add(a, b):
    return a + b

# 2. Add types to this function, including the optional argument
def greet(name, times=1):
    return f"Hello {name}! " * times

# 3. Add types to this function. What type should a list of strings be?
def process_names(names):
    for name in names:
        print(name.upper())

# 4. What if a function might not return anything?
def find_user(user_id, user_list):
    for user in user_list:
        if user['id'] == user_id:
            return user
    # What happens if the user is not found?
```
*Hint for #4: Look up `Optional` from the `typing` module.*

## 📝 Daily Assignment

1.  **Create a `utils.py` file**: In your `my-first-poetry-app` project from yesterday, create a new file: `my_first_poetry_app/utils.py`.
2.  **Configure MyPy**: Add the `[tool.mypy]` configuration from the "Concepts" section to your `pyproject.toml` and set `strict = true`.
3.  **Annotate 20 Functions**: Write and correctly annotate at least 20 different functions in `utils.py`. Make them simple, but cover a range of scenarios:
    - Functions that take `int`, `str`, `float`, `bool`.
    - Functions with optional arguments (`Optional[str] = None`).
    - Functions that return simple types.
    - Functions that return `None`.
    - Functions that work with lists (`List[int]`).
    - Functions that work with dictionaries (`Dict[str, float]`).
    - Functions that can return one of two types (`Union[int, str]`).
4.  **Run MyPy**: Continuously run `poetry run mypy .` and fix all errors until MyPy reports `Success: no issues found`. This process of fighting and satisfying the type checker is a core skill.

## ⚠️ Common Mistakes

-   **Using `list` instead of `List`**: For older Python versions (pre 3.9), you must import `List`, `Dict`, `Tuple` etc., from the `typing` module. In modern Python (3.9+), you can use the lowercase built-in types `list[int]` and `dict[str, int]`, which is preferred.
-   **Ignoring MyPy errors**: It's tempting to ignore MyPy's complaints, especially when the code "seems to work". This defeats the purpose. Treat MyPy errors as seriously as you would treat a syntax error.
-   **Forgetting to type function return values**: It's easy to type all the arguments but forget the `-> ReturnType:` part. `mypy --strict` will catch this.
-   **Using `Any` as a crutch**: `Any` tells the type checker to "allow anything". While sometimes necessary, overusing it is a sign of poorly typed code. Try to be as specific as possible.

## 📖 Further Reading

-   [MyPy Documentation](https://mypy.readthedocs.io/en/stable/)
-   [Typing module documentation (Python 3.12)](https://docs.python.org/3/library/typing.html)
-   [Real Python: Python Type Checking Guide](https://realpython.com/python-type-checking/)
