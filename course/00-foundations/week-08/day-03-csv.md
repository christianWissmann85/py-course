# Week 8, Day 3: CSV & Tabular Data

## 🎯 Learning Objectives
- [ ] Read and write CSV (Comma-Separated Values) files using Python's built-in `csv` module.
- [ ] Use `csv.DictReader` and `csv.DictWriter` to work with CSV data as dictionaries.
- [ ] Understand and handle CSVs with and without headers.
- [ ] Apply type annotations and data validation when processing tabular data.

## 📚 Concepts

### 1. The `csv` Module
CSV is a very common plain-text format for storing tabular data. Each line is a row, and commas separate the values in each row. While you could `split(',')` each line, this is brittle and fails if, for example, a data field contains a comma.

Python's `csv` module handles all the complexities of parsing CSV files, including quoted fields and different dialects.

- **Reading**: `csv.reader` creates an iterator that reads a CSV file row by row, where each row is a list of strings.
- **Writing**: `csv.writer` provides a `writerow` method to write a list of values to a file as a comma-separated line.

```python
import csv

# --- Writing a CSV ---
header = ['name', 'age']
data = [['Alice', 30], ['Bob', 42]]

with open('data.csv', 'w', newline='', encoding='utf-8') as f:
    writer = csv.writer(f)
    writer.writerow(header) # Write the header
    writer.writerows(data)  # Write all data rows

# --- Reading a CSV ---
with open('data.csv', 'r', encoding='utf-8') as f:
    reader = csv.reader(f)
    header_read = next(reader) # Read the header row
    print(f"Header: {header_read}")
    for row in reader:
        # Each row is a list of strings
        print(f"Name: {row[0]}, Age: {row[1]}")
```
**Note**: The `newline=''` argument in `open()` is crucial when working with the `csv` module to prevent blank rows from being written.

### 2. `DictReader` and `DictWriter`
Working with indices (`row[0]`) is not very readable. It's often better to work with dictionaries, where keys are the column headers.
- `csv.DictReader`: Reads rows as dictionaries, using the first row as headers.
- `csv.DictWriter`: Writes rows from dictionaries. You must specify the `fieldnames` (headers).

```python
import csv

# --- Writing with DictWriter ---
data_dicts = [
    {'name': 'Charlie', 'area': 'Engineering'},
    {'name': 'Diana', 'area': 'Marketing'},
]
fieldnames = ['name', 'area']

with open('data_dict.csv', 'w', newline='', encoding='utf-8') as f:
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows(data_dicts)

# --- Reading with DictReader ---
with open('data_dict.csv', 'r', encoding='utf-8') as f:
    reader = csv.DictReader(f)
    for row_dict in reader:
        # Each row is an OrderedDict or dict
        print(f"{row_dict['name']} works in {row_dict['area']}")
```

### 3. Dialects and Formatting Parameters
CSVs can have different delimiters (e.g., tabs instead of commas, creating a TSV file), quoting rules, etc. These variations are called **dialects**. You can specify them when creating a reader/writer.

```python
# Example of reading a tab-separated file with no quotes
with open('data.tsv', 'r') as f:
    reader = csv.reader(f, delimiter='\t', quoting=csv.QUOTE_NONE)
    # ...
```

### 4. Large CSV Files
`csv.reader` and `csv.DictReader` are iterators. They read the file line by line, which makes them very memory-efficient. You can process CSV files of any size (even many gigabytes) without running out of memory, because the entire file is never loaded at once. This is the same principle we saw with generators.

### 5. Data Validation and Typing
Data from a CSV file always comes in as **strings**. You are responsible for converting and validating the data. This is a common source of bugs.

```python
with open('data.csv', 'r') as f:
    reader = csv.DictReader(f)
    for row in reader:
        try:
            name: str = row['name']
            # Always validate and convert types!
            age: int = int(row['age'])
            if age < 0:
                raise ValueError("Age cannot be negative")
            print(f"Processing {name}, age {age}")
        except (KeyError, ValueError) as e:
            print(f"Skipping malformed row {row}: {e}")
```

## 🔹 Quick Exercise

Complete the `read_csv_typed` function below. It should read a CSV file and convert each row into a `TypedDict`.

```python
import csv
from typing import Iterator, TypedDict, List

class User(TypedDict):
    name: str
    id: int

def read_csv_typed(filename: str) -> Iterator[User]:
    """
    Reads a CSV with 'name' and 'id' columns and yields User TypedDicts.
    Skips rows with conversion errors.
    """
    with open(filename, 'r', encoding='utf-8') as f:
        reader = csv.DictReader(f)
        for row in reader:
            try:
                user: User = {'name': row['name'], 'id': int(row['id'])}
                yield user
            except (ValueError, KeyError):
                continue

# --- Test Setup ---
with open("users.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerow(["name", "id"])
    writer.writerow(["Alice", "101"])
    writer.writerow(["Bob", "not_an_id"])
    writer.writerow(["Charlie", "103"])

# --- Tests ---
users = list(read_csv_typed("users.csv"))
assert len(users) == 2
assert users[0] == {"name": "Alice", "id": 101}
assert users[1] == {"name": "Charlie", "id": 103}
print("Typed CSV reading works!")
```

## 📝 Daily Assignment
**Goal**: Build a CSV processing pipeline that transforms and validates data.

1.  **Create Project File**: In your project, create `my_first_poetry_app/csv_processor.py`.
2.  **Generate Sample Data**:
    -   Write a function to generate a `products.csv` file. It should have columns `product_id`, `name`, `category`, and `price`.
    -   Include some "dirty" data in your file: some rows with a non-numeric price, some with missing columns.
3.  **Implement a `process_products` function**:
    -   `process_products(input_path: str, output_path: str, category_filter: str) -> None`
    -   This function should:
        1.  Read from `input_path` using `csv.DictReader`.
        2.  Process each row:
            -   **Validate**: Skip any row where `price` is not a valid positive float or any required field is missing. Log a warning for skipped rows.
            -   **Filter**: Skip any row where the `category` does not match `category_filter`.
            -   **Transform**: Add a new column, `price_with_tax`, calculated as `price * 1.20`.
        3.  Write the processed, valid rows to `output_path` using `csv.DictWriter`. The output file should contain the original columns plus the new `price_with_tax` column.
4.  **`main()` function**:
    -   Generate your sample `products.csv`.
    -   Call `process_products` to create a new `electronics_with_tax.csv` file, filtering for the "electronics" category.
    -   Read the output file to verify its contents.
5.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes
- **Forgetting `newline=''`**: When opening a file for the `csv` module, always use `newline=''`. If you don't, you may get extra blank rows in your output file on some operating systems.
- **Assuming data types**: All data read from a CSV is a string until you convert it. `row['age'] + 1` will raise a `TypeError`. You must do `int(row['age']) + 1`.
- **Trusting headers**: `DictReader` assumes the first row is a valid header. If the file is malformed, this can cause issues. In robust systems, you might validate the header row explicitly.
- **Not handling `KeyError`**: When using `DictReader`, if a row is shorter than the header, accessing a key that doesn't exist for that row will raise a `KeyError`. Using `row.get('key')` is safer.

## 📖 Further Reading
- [Python Docs: `csv` — CSV File Reading and Writing](https://docs.python.org/3/library/csv.html)
- [Real Python: Reading and Writing CSV Files in Python](https://realpython.com/python-csv/)
