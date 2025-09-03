# Week 1, Day 4: Linting with Ruff

## 🎯 Learning Objectives
- [ ] Understand what linting is and why it's crucial for code quality.
- [ ] Learn how to configure Ruff, a high-performance linter, in your project.
- [ ] Practice identifying and fixing common linting issues reported by Ruff.
- [ ] Integrate Ruff into VS Code for real-time feedback.

## 📚 Concepts

### 1. What is Linting?
Linting is the process of using a tool (a linter) to analyze source code to flag programming errors, bugs, stylistic errors, and suspicious constructs. A linter is like a strict, automated code reviewer that never gets tired. It helps enforce a consistent coding style across a team and catches common mistakes before they become bugs.

### 2. Ruff: The New Standard
For years, the Python ecosystem had a collection of separate tools for linting (Pylint, Flake8) and formatting (Black, isort). Ruff has revolutionized this by combining the functionality of dozens of tools into a single, blazing-fast binary written in Rust.

**Why Ruff?**
- **Speed**: It's 10-100x faster than older linters. This means you get feedback almost instantly.
- **All-in-One**: It replaces Flake8, isort (for sorting imports), pyupgrade (for modernizing syntax), and many other tools.
- **Auto-fixing**: Ruff can automatically fix a huge number of the issues it finds.
- **Great Defaults**: It comes with sensible default settings but is highly configurable.

### 3. Ruff Configuration
Like MyPy, Ruff is configured in your `pyproject.toml` file. This allows you to define all your project's tooling in one place.

A good starting configuration looks like this:

```toml
[tool.ruff]
# Set the maximum line length.
line-length = 88

# Select the rule sets you want to enable.
# "E", "F" are standard Flake8 rules (errors, pyflakes)
# "I" is for isort (import sorting)
# "B" is for flake8-bugbear (finds likely bugs)
# "C4" is for flake8-comprehensions (helps write better comprehensions)
# "UP" is for pyupgrade (upgrades syntax to modern versions)
select = ["E", "F", "I", "B", "C4", "UP"]

# You can also ignore specific rules if needed.
# For example, B008 is often disabled as it can be noisy.
ignore = ["B008"]

[tool.ruff.format]
# Opt-in to using Ruff's formatter (replaces Black)
# It's still new, but very promising.
quote-style = "double"

[tool.ruff.isort]
# Configure how imports are sorted.
known-first-party = ["my_first_poetry_app"]
```

### 4. Common Violations
Ruff will catch hundreds of different issues. Some common ones you'll see are:
- `F841: Local variable 'x' is assigned to but never used.` (Unused variable)
- `E501: Line too long (90 > 88 characters).` (Line length)
- `I001: Import block is un-sorted or un-formatted.` (Imports not sorted)
- `F401: 'os' imported but unused.` (Unused import)

### 5. Auto-fixing Issues
This is one of Ruff's most powerful features. To let Ruff try to fix issues automatically, run:

```bash
poetry run ruff check . --fix
```

This will modify your files in place to correct problems like unsorted imports, formatting issues, and more. It's a huge time-saver.

## 💻 Code Examples

### Code with Linting Issues
Save this to a file named `bad_code.py`:

```python
import os
import sys

def my_function( a,b ):
    unused_var = "I am not used"
    if a==b:
        return True
    else:
        return False
```

### Running Ruff
```bash
# First, just check the code
poetry run ruff check bad_code.py

# Expected output will show multiple errors:
# F401: 'os' imported but unused
# F401: 'sys' imported but unused
# E712: Comparison to `True` should be `if cond is True:` or `if cond:`
# E711: Comparison to `None` should be `if cond is None:` (This might be a false positive depending on the exact rule, but it's an example)
# ... and formatting issues.

# Now, let's try to fix it automatically
poetry run ruff check bad_code.py --fix
```

### The Code After Auto-fixing
Ruff will change the file to something like this (depending on your config):

```python
def my_function(a, b):
    unused_var = "I am not used"  # Ruff can't fix unused variables
    return a == b                 # But it can simplify the boolean return
```
*Note: Ruff can't automatically remove unused imports if it's not sure they are safe to remove, nor can it fix the unused variable. You'd have to fix those manually.*

## 🔹 Quick Exercise

1.  **Create `lint_me.py`**: In your project, create a file with the following content:
    ```python
    import math, sys

    def bad_function ( ):
        my_list=[1,2,3,4,5]
        total =0
        for i in my_list: total+=i
        return total
    ```
2.  **Run Ruff**: Run `poetry run ruff check lint_me.py`. Note the errors.
3.  **Run with `--fix`**: Run `poetry run ruff check lint_me.py --fix`. See what changes.
4.  **Configure Line Length**: In your `pyproject.toml`, set `line-length = 30`. Run `ruff check` again. See the new `E501` error.
5.  **Disable a Rule**: The original code has unsorted imports. This is rule `I001`. In `pyproject.toml`, add `"I001"` to the `ignore` list. Run `ruff check` again and see that the import error is gone.

## 📝 Daily Assignment

1.  **Configure Ruff Comprehensively**: In your `my-first-poetry-app` project, create a `[tool.ruff]` section in your `pyproject.toml`. Use the `select` key to enable at least these rule sets: `E`, `F`, `W`, `I`, `B`, `C4`, `UP`, `N`, `SIM`. Look up what each of these does in the [Ruff documentation](https://docs.astral.sh/ruff/rules/).
2.  **Create a "Bad" File**: Create a new file `my_first_poetry_app/to_refactor.py`.
3.  **Write Messy Code**: In this file, write at least 50 lines of "messy" Python code. Intentionally violate as many linting rules as you can think of. Ideas:
    - Long lines.
    - Unused imports and variables.
    - Badly formatted code.
    - Complex boolean checks (`if x == True:`).
    - Unnecessary `else` blocks after a `return`.
    - Use old-style type annotations if you can (`from typing import List` when `list` would work).
4.  **Fix Everything**:
    - First, run `poetry run ruff check . --fix` to let Ruff do the heavy lifting.
    - Then, run `poetry run ruff check .` again. Manually fix every single remaining issue until Ruff reports `All checks passed!`.

## ⚠️ Common Mistakes

-   **Having conflicting formatters**: If you use Ruff's formatter, uninstall Black or disable it in your editor. If you use Black, make sure Ruff's `line-length` matches Black's. If they fight, you'll have a frustrating experience.
-   **Not integrating with your editor**: The biggest benefit of a fast linter is real-time feedback. Make sure you install the Ruff VS Code extension and configure it to see errors as you type.
-   **Ignoring rules blindly**: Don't just add rules to the `ignore` list to make errors go away. Take the time to understand *why* the linter is flagging something. It's almost always for a good reason.
-   **Forgetting to check all files**: Running `ruff check my_file.py` only checks that one file. Use `ruff check .` to check the entire project.

## 📖 Further Reading

-   [Ruff Official Documentation](https://docs.astral.sh/ruff/)
-   [Ruff's list of implemented rules](https://docs.astral.sh/ruff/rules/) (An excellent resource to learn about common Python pitfalls)
-   [Using Ruff with pre-commit](https://docs.astral.sh/ruff/integrations/#pre-commit)
