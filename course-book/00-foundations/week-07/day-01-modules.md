# Week 7, Day 1: The Module System

## 🎯 Learning Objectives

- [ ] Understand that every Python file is a module.
- [ ] Use various forms of the `import` statement to use code from other modules.
- [ ] Understand the purpose of the `if __name__ == "__main__":` block.
- [ ] Learn about common module attributes like `__name__` and `__doc__`.

## 📚 Concepts

### 1. What are Modules?

In Python, a **module** is simply a file containing Python definitions and statements. The file name is the module name with the suffix `.py` appended. Modules allow you to logically organize your Python code into separate files, making it more manageable and reusable.

When you write `import my_file`, Python looks for a file named `my_file.py` and makes its contents available to you.

### 2. The `import` Statement

There are several ways to import code from a module.

```python
# Assume we have a file named 'math_utils.py' with this content:
#
# PI = 3.14159
# def area_of_circle(radius: float) -> float:
#     return PI * (radius ** 2)

# --- Style 1: Import the entire module ---
import math_utils

# You must prefix names with the module name
circle_area = math_utils.area_of_circle(10)
print(f"The value of PI is {math_utils.PI}")

# --- Style 2: Import specific names from a module ---
from math_utils import area_of_circle, PI

# You can now use the names directly
circle_area = area_of_circle(10)
print(f"The value of PI is {PI}")

# --- Style 3: Import with an alias ---
import math_utils as mu
from math_utils import area_of_circle as area

circle_area = mu.area_of_circle(10) # or area(10)
print(f"The value of PI is {mu.PI}")
```

**Best Practice**: Prefer `import module` or `from module import name`. Avoid `from module import *`, as it pollutes the current namespace and makes it unclear where names are coming from.

### 3. `__name__` and the `__main__` block

Every module has a special attribute called `__name__`.

- If you run the Python file directly (e.g., `python my_module.py`), its `__name__` is set to the string `"__main__"`.
- If you `import` the file into another module, its `__name__` is set to the module's file name (e.g., `"my_module"`).

This allows you to write code that only runs when the file is executed as a script, but not when it's imported. This is done with the `if __name__ == "__main__":` block. It's the standard entry point for a Python script.

```python
# In file 'my_script.py'

def some_function():
    print("Function was called.")

print(f"This line always runs. __name__ is {__name__}")

# This block only runs when you execute `python my_script.py`
if __name__ == "__main__":
    print("This script is being run directly.")
    some_function()
```

If you import this file (`import my_script`), it will print "This line always runs. `__name__` is my_script", but the code inside the `if` block will not execute.

### 4. Other Module Attributes

Modules have several other useful built-in attributes:

- `__doc__`: The module's docstring (the first string literal at the top of the file).
- `__file__`: The path to the file from which the module was loaded.
- `__cached__`: The path to the compiled `.pyc` file.

### 5. Python's Import Cache

When you import a module for the first time, Python executes the code in the file and caches the resulting module object in a dictionary called `sys.modules`. Subsequent imports of the same module are very fast because they just retrieve the object from the cache instead of re-running the file. This means that code at the top level of a module is only ever executed once per program run.

## 🔹 Quick Exercise

1.  Create a file named `math_utils.py`.
2.  In it, define a constant `PI = 3.14159` and a function `calculate_area(radius: float) -> float` that calculates the area of a circle.
3.  Create a second file named `main.py`.
4.  In `main.py`, import the `math_utils` module and use its `calculate_area` function to find the area of a circle with a radius of 5.
5.  Add a `if __name__ == "__main__":` block to `math_utils.py` that prints a message like "This module provides math utilities." This message should _not_ appear when you run `main.py`.

```python
# math_utils.py
PI: float = 3.14159

def calculate_area(radius: float) -> float:
    """Calculates the area of a circle."""
    return PI * (radius ** 2)

if __name__ == "__main__":
    print("This module provides math utilities and is being run directly.")
    print(f"Area of circle with radius 1 is {calculate_area(1)}")


# main.py
# Import the function from the other module
from math_utils import calculate_area

def main():
    radius = 5.0
    area = calculate_area(radius)
    print(f"The area of a circle with radius {radius} is {area}")

if __name__ == "__main__":
    main()
```

## 📝 Daily Assignment

**Goal**: Organize a small project into multiple modules.

1.  **Create Project Files**: In your `my-first-poetry-app` project, create a directory `my_app` if it doesn't exist. Inside it, create three files:
    - `my_app/main.py` (The main entry point)
    - `my_app/string_utils.py`
    - `my_app/math_utils.py`
2.  **Populate Utility Modules**:
    - Move some of the string-related functions you wrote in previous assignments (like `is_palindrome`, `reverse_string`) into `string_utils.py`.
    - Move some of the math-related functions (like `is_prime`, `factorial`) into `math_utils.py`.
3.  **Create the Main Script**:
    - In `main.py`, write a `main()` function that demonstrates functions from _both_ utility modules.
    - Import the functions you need using `from .string_utils import ...` and `from .math_utils import ...`. (The leading dot `.` indicates a relative import from the same package, which we'll cover tomorrow).
    - Use an `if __name__ == "__main__":` block to call your `main()` function.
4.  **Run the Application**:
    - From your project's root directory, run your main script as a module using the `-m` flag: `poetry run python -m my_app.main`. The `-m` flag is important for making relative imports work correctly.
5.  **Verify**: Ensure the script runs, produces the correct output, and passes all linter and type checks.

## ⚠️ Common Mistakes

- **Circular Imports**: Module A imports Module B, and Module B imports Module A. This will raise an `ImportError`. It's a sign that you need to restructure your code, perhaps by moving shared functionality to a third module.
- **Name Clashes**: If you import `f` from `module1` and `f` from `module2`, the second import will overwrite the first. Use aliases to resolve this: `from module1 import f as f1` and `from module2 import f as f2`.
- **Running a file with relative imports directly**: If `main.py` uses `from . import utils`, running `python main.py` will fail with an `ImportError`. You must run it as part of a package with `python -m my_package.main`.
- **Side effects at the top level**: Code at the top level of a module (not inside a function) runs the first time the module is imported. Avoid expensive operations like network requests or file I/O at the top level. Put them inside functions.

## 📖 Further Reading

- [Real Python: Python Modules and Packages](https://realpython.com/python-modules-packages/)
- [Python Docs: The Module System](https://docs.python.org/3/tutorial/modules.html)
- [PEP 338 -- Executing modules as scripts](https://peps.python.org/pep-0338/) (Explains the `-m` flag)
