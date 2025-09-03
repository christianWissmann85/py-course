# Week 7, Day 2: Packages

## 🎯 Learning Objectives

- [ ] Structure related modules into a Python package.
- [ ] Understand the role of the `__init__.py` file in defining a package.
- [ ] Use relative imports to import modules within the same package.
- [ ] Control what names are imported from a package using `__all__`.

## 📚 Concepts

### 1. What is a Package?

As your application grows, you might end up with many modules. A **package** is a way to structure your application's modules by grouping them into a directory hierarchy.

Any directory that contains a special file named `__init__.py` is considered a Python package. This file can be empty, but its presence is what matters.

A typical package structure might look like this:

```
my_project/
├── my_app/
│   ├── __init__.py
│   ├── main.py
│   ├── models.py
│   └── utils/
│       ├── __init__.py
│       └── string_helpers.py
└── tests/
```

Here, `my_app` and `my_app.utils` are both packages.

### 2. The Role of `__init__.py`

The `__init__.py` file serves two main purposes:

1.  It tells Python that the directory should be treated as a package.
2.  It can contain initialization code for the package. This code is executed the first time the package or one of its modules is imported.

A common use for `__init__.py` is to make functions or classes from modules inside the package available at the top level of the package. This creates a cleaner public API for your package.

```python
# In my_app/utils/string_helpers.py
def is_palindrome(s: str) -> bool: ...

# In my_app/utils/__init__.py
# This makes is_palindrome available directly from the `utils` package
from .string_helpers import is_palindrome

# Now, in another file, you can do this:
from my_app.utils import is_palindrome # Instead of the longer from my_app.utils.string_helpers import ...
```

### 3. Subpackages

As seen in the example structure, you can nest packages inside other packages. `my_app.utils` is a **subpackage** of `my_app`. This allows for a deep and logical organization of very large codebases.

### 4. Relative Imports

When importing modules _within the same package_, you can use **relative imports**.

- A single dot (`.`) refers to the current package.
- Two dots (`..`) refer to the parent package.

```python
# In my_app/main.py

# Import `is_palindrome` from the sibling module `string_utils`
# inside the same package `my_app`.
from . import models
from .utils import is_palindrome # .utils refers to the utils subpackage

# This is generally preferred over absolute imports (from my_app import models)
# because it makes the package more self-contained and easier to rename.
```

**Important**: You can only use relative imports in files that are part of a package. Trying to run a file that uses relative imports directly as a script will result in an `ImportError`. You must run it as a module using the `-m` flag.

### 5. Controlling Imports with `__all__`

If a user writes `from my_package import *`, what names get imported? By default, Python imports all names that don't start with an underscore. You can explicitly define what gets imported by providing a list of strings called `__all__` in your `__init__.py`.

```python
# In my_app/utils/__init__.py
from .string_helpers import is_palindrome, reverse_string
from .math_helpers import is_prime

# Only `is_palindrome` and `is_prime` will be imported
# when a user does `from my_app.utils import *`
__all__ = ["is_palindrome", "is_prime"]
```

This is considered good practice for any package that's meant to be used as a library, as it creates a clear public API and prevents internal helper functions from leaking into the user's namespace.

## 🔹 Quick Exercise

Create a small package structure and configure its `__init__.py`.

1.  **Create the structure**:
    ```
    my_package/
    ├── __init__.py
    ├── core.py
    └── utils.py
    ```
2.  **Add code**:
    - In `core.py`, define `def main_function(): ...`
    - In `utils.py`, define `def helper_function(): ...`
3.  **Configure `__init__.py`**:
    - In `my_package/__init__.py`, import `main_function` from `.core` and `helper_function` from `.utils`.
    - Set `__all__` so that only `main_function` is considered part of the public API.

```python
# my_package/__init__.py

# Import functions from submodules to make them accessible at the package level
from .core import main_function
from .utils import helper_function

# Define the public API of the package
__all__ = ['main_function']
```

Now, another script could do `from my_package import main_function`, which is cleaner than `from my_package.core import main_function`.

## 📝 Daily Assignment

**Goal**: Refactor your utility modules from Day 1 into a well-structured, installable package.

1.  **Restructure Your Project**: Your `my-first-poetry-app` project should have a structure like this. If it doesn't already, refactor it.
    ```
    my-first-poetry-app/
    ├── my_app/
    │   ├── __init__.py
    │   ├── string_utils.py
    │   ├── math_utils.py
    │   └── cli.py
    └── tests/
    ```
2.  **Configure the Top-Level `__init__.py`**:
    - In `my_app/__init__.py`, import the most important functions from `string_utils` and `math_utils`.
    - Define `__all__` to create a clear public API for your `my_app` package. Choose 3-4 functions to expose.
3.  **Use Relative Imports**:
    - In `my_app/cli.py`, change your imports to be relative (e.g., `from .math_utils import is_prime`).
4.  **Create a `main` entry point**:
    - The `cli.py` module should contain the main user-facing logic. It should have a `main` function and an `if __name__ == "__main__":` block.
5.  **Run as a Package**:
    - From the root of `my-first-poetry-app`, run your application as a module: `poetry run python -m my_app.cli`.
    - This is how real Python applications are typically run.
6.  **Verify**: Ensure the script runs, produces the correct output, and passes all linter and type checks.

## ⚠️ Common Mistakes

- **Missing `__init__.py` files**: If a directory doesn't have an `__init__.py`, Python won't see it as a package, and you won't be able to import from it. (Note: This is partially relaxed in Python 3 with "namespace packages", but for regular packages, `__init__.py` is essential).
- **Confusion between absolute and relative imports**:
  - `from my_app import utils` (absolute): Works from anywhere, as long as `my_app` is in Python's path.
  - `from . import utils` (relative): Only works from within the `my_app` package.
- **Running files directly**: Running `python my_app/cli.py` will fail if `cli.py` uses relative imports. You must run it with `python -m my_app.cli` from the parent directory.

## 📖 Further Reading

- [Python Docs: Packages](https://docs.python.org/3/tutorial/modules.html#packages)
- [Real Python: Python Modules and Packages: An Introduction](https://realpython.com/python-modules-packages/)
- [PEP 420 -- Implicit Namespace Packages](https://peps.python.org/pep-0420/) (For advanced understanding)
