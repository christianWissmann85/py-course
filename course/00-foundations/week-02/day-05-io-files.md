# Week 2, Day 5: Input/Output & Files

## 🎯 Learning Objectives
- [ ] Handle user input from the command line safely, including validation and type conversion.
- [ ] Perform basic file operations (reading and writing) using modern, type-safe practices.
- [ ] Use the `pathlib` module to handle filesystem paths in an object-oriented way.
- [ ] Get a basic understanding of how to work with common data formats like JSON and CSV.

## 📚 Concepts

### 1. Input with Validation
The built-in `input()` function is used to get input from the user in the terminal. **Crucially, `input()` always returns a string.** You must manually convert it to any other type you need, and you should always validate it.

```python
# Get user's age and convert to int
raw_age: str = input("Enter your age: ")

try:
    age: int = int(raw_age)
    print(f"In 10 years, you will be {age + 10}.")
except ValueError:
    print("Invalid input. Please enter a number.")
```
Using a `try...except` block is essential for handling cases where the user enters text that cannot be converted to the desired type.

### 2. Print Formatting
As we saw on Day 2, f-strings are the best way to format output for the `print()` function. They are clean, fast, and allow you to embed expressions directly.

```python
user: str = "Jules"
items: int = 3
total_cost: float = 74.95

print(f"User '{user}' bought {items} items for a total of ${total_cost:.2f}.")
# The :.2f formats the float to have exactly two decimal places.
```

### 3. File Reading/Writing
The standard way to work with files is using the `open()` function within a `with` statement. The `with` statement ensures the file is automatically closed even if errors occur.

```python
# --- Writing to a file ---
# 'w' mode overwrites the file if it exists.
with open("greeting.txt", "w", encoding="utf-8") as f:
    f.write("Hello, world!\n")
    f.write("This is a second line.\n")

# --- Reading from a file ---
lines_read: list[str] = []
with open("greeting.txt", "r", encoding="utf-8") as f:
    for line in f:
        # line includes the trailing newline, so we strip it
        lines_read.append(line.strip())

print(lines_read)
# Output: ['Hello, world!', 'This is a second line.']
```
Always specify `encoding="utf-8"`. It's the modern standard and avoids many common issues.

### 4. `pathlib` for Path Handling
For decades, Python programmers built paths by concatenating strings. This was error-prone. The modern solution is the `pathlib` module. It provides a `Path` object that represents a filesystem path with methods for common operations.

```python
from pathlib import Path

# Get the path to the current directory
current_dir: Path = Path.cwd()
print(f"Current directory: {current_dir}")

# Create a path to a new file. The / operator is overloaded to join paths.
# This works correctly on Windows, macOS, and Linux.
file_path: Path = current_dir / "data" / "my_file.txt"
print(f"File path: {file_path}")

# You can check if a path exists
print(f"Does the file exist? {file_path.exists()}")

# Create parent directories if they don't exist
file_path.parent.mkdir(parents=True, exist_ok=True)
```

### 5. Data Formats: JSON and CSV
- **JSON (JavaScript Object Notation)**: A human-readable format for representing data as nested key-value pairs. It's the de-facto standard for web APIs. Python's `json` module makes it easy to work with.
- **CSV (Comma-Separated Values)**: A simple text format for representing tabular data (like a spreadsheet). Each line is a row, with columns separated by commas. Python's `csv` module helps parse and create these files.

```python
import json
from typing import Any

# --- JSON Example ---
user_data: dict[str, Any] = {"name": "Alice", "age": 30, "is_active": True}

# Save dict to a JSON file
with open("user.json", "w") as f:
    json.dump(user_data, f, indent=4) # indent makes it readable

# Load dict from a JSON file
with open("user.json", "r") as f:
    loaded_data: dict[str, Any] = json.load(f)

print(f"Loaded from JSON: {loaded_data['name']}")
```

## 🔹 Quick Exercise

Implement the following two functions.

```python
import json
from typing import Any, List

def read_integers(filename: str) -> list[int]:
    """
    Reads a file where each line contains an integer.
    Returns a list of those integers.
    """
    # Use a with statement to open the file
    # Loop through the lines, convert each to int, and add to a list.
    pass

def save_json(data: dict[str, Any], filename: str) -> None:
    """
    Saves a dictionary to a file in JSON format.
    """
    # Use the json.dump() function.
    pass


# --- Test Setup ---
with open("test_ints.txt", "w") as f:
    f.write("1\n5\n-3\n")

# --- Tests ---
assert read_integers("test_ints.txt") == [1, 5, -3]
test_dict = {"a": 1, "b": [2, 3]}
save_json(test_dict, "test.json")
with open("test.json", "r") as f:
    assert json.load(f) == test_dict
```

## 📝 Daily Assignment
**Goal**: Build a simple file processor that can read and summarize data from different file formats.

1.  **Create Project File**: In `my-first-poetry-app`, create `my_first_poetry_app/file_processor.py`.
2.  **Generate Sample Files**:
    - Write a function to create a sample CSV file named `data.csv` with columns `name,age,city`.
    - Write a function to create a sample JSON file named `data.json` with a list of user objects (e.g., `[{"name": "...", "age": ..., "city": ...}]`).
3.  **Implement Reader Functions**:
    - `read_csv(filepath: Path) -> list[dict]`: Takes a path to a CSV and returns a list of dictionaries. Use the `csv.DictReader` for this.
    - `read_json(filepath: Path) -> list[dict]`: Takes a path to a JSON file and returns the data.
4.  **Implement Processor Function**:
    - `summarize_data(data: list[dict]) -> None`: Takes a list of user dictionaries and prints a summary, such as:
        - Total number of users.
        - Average age.
        - List of unique cities.
5.  **Add Error Handling**:
    - Wrap your file-reading operations in `try...except` blocks to gracefully handle `FileNotFoundError`.
6.  **`main()` function**:
    - Use `pathlib.Path` to define file paths.
    - Call your functions to generate the sample files.
    - Read the data from both the CSV and JSON files.
    - Pass the data to your `summarize_data` function.
    - Test the error handling by trying to read a file that doesn't exist.
7.  **Verify**: Run the script, type checker, and linter.

## 📦 Week 2 Project: Type-Safe CLI Calculator
This week's project combines everything you've learned about types, control flow, loops, and I/O.

**Goal**: Create a command-line calculator that is fully type-safe, handles user input, and can save/load history from a file.

**Features**:
- **Arithmetic Operations**: Should support add, subtract, multiply, and divide.
- **User Input**: Prompts the user for two numbers and an operator. Handles invalid input gracefully.
- **File History**:
    - After each calculation, append the operation and result to a `history.txt` file (e.g., `10 + 5 = 15`).
    - On startup, the program should offer to show the previous history from the file.
- **Error Handling**: Handles non-numeric input and division by zero.
- **Type Safety**: Fully annotated with MyPy running in strict mode.

See `week-02-project.md` for a more detailed breakdown.

## ⚠️ Common Mistakes
- **Forgetting to `close()` files**: Not using a `with` statement can leave file handles open, which can cause issues in larger applications. The `with` statement handles this for you automatically.
- **Assuming `input()` returns a number**: Always remember `input()` returns a `str`. Forgetting to convert it is a very common bug.
- **Ignoring `FileNotFoundError`**: Trying to read a file that doesn't exist will crash your program. Always wrap file reading in a `try...except FileNotFoundError` block if the file might be missing.
- **Hardcoding path separators**: Writing paths like `"data\\my_file.txt"` (Windows) or `"data/my_file.txt"` (Linux/macOS) makes your code non-portable. Let `pathlib` handle this for you with the `/` operator.

## 📖 Further Reading
- [Real Python: Reading and Writing Files in Python](https://realpython.com/read-write-files-python/)
- [Real Python: `pathlib` — Taming the File System](https://realpython.com/python-pathlib/)
- [Python Docs: `json` — JSON encoder and decoder](https://docs.python.org/3/library/json.html)
- [Python Docs: `csv` — CSV File Reading and Writing](https://docs.python.org/3/library/csv.html)
