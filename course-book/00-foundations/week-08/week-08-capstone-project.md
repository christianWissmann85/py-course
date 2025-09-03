# Week 8 Capstone Project: Multi-Format Data Processor

## 🎯 Project Objective
This is the capstone project for the Foundations phase of the course. You will build a complete, robust, and well-tested application that combines everything you have learned over the past eight weeks. The goal is to create a tool that can ingest data from multiple file formats (JSON, CSV, XML), validate it, standardize it, and load it into an SQLite database.

This project simulates a real-world data engineering task and will be an excellent piece for your portfolio.

## 📋 Core Requirements
You will build a command-line application that processes a directory of data files.

### 1. Project Setup
-   This should be a new, standalone Poetry project. Initialize it with `poetry new data-processor`.
-   Structure your project as a proper package (e.g., all source code inside a `data_processor/` directory).
-   Add any necessary dependencies, such as `pytest`.

### 2. Data Models
-   Define a standardized data model for a "Transaction". A `@dataclass` is perfect for this. It should have fields like `transaction_id: str`, `source_format: Literal["json", "csv", "xml"]`, `amount: float`, `timestamp: datetime`.

### 3. The Readers (Data Ingestion)
-   Create a `readers` module (`data_processor/readers.py`).
-   Implement a separate generator function for each file format:
    -   `read_json_data(filepath: Path) -> Generator[dict, None, None]`
    -   `read_csv_data(filepath: Path) -> Generator[dict, None, None]`
    -   `read_xml_data(filepath: Path) -> Generator[dict, None, None]`
-   Each function should take a file path and `yield` dictionaries representing the raw records from the file.

### 4. The Validator and Standardizer
-   Create a `processing` module (`data_processor/processing.py`).
-   Write a function `validate_and_standardize(raw_records: Iterator[dict], source_format: str) -> Generator[Transaction, None, None]`.
-   This generator should:
    1.  Take an iterator of raw dictionaries from one of the readers.
    2.  For each raw dictionary, validate its contents (e.g., does it have the required keys? Is the amount a valid number?).
    3.  If valid, convert it into your standard `Transaction` dataclass instance.
    4.  If invalid, log a warning and skip it.
    5.  `yield` the valid `Transaction` objects.

### 5. The Writer (Database Storage)
-   Create a `database` module (`data_processor/database.py`).
-   Implement a `DatabaseWriter` class.
    -   `__init__(self, db_path: Path)`: Connects to the SQLite database.
    -   `setup_table(self)`: Creates a `transactions` table that matches your `Transaction` dataclass.
    -   `write_transactions(self, transactions: Iterator[Transaction]) -> None`: Takes an iterator of `Transaction` objects and writes them to the database in a single transaction.

### 6. The Main Application
-   Create a `main.py` or `cli.py` module to orchestrate the process.
-   The application should be able to scan an input directory for `.json`, `.csv`, and `.xml` files.
-   For each file, it should:
    1.  Select the correct reader based on the file extension.
    2.  Pass the reader's output to the `validate_and_standardize` function.
    3.  Pass the standardized transactions to the `DatabaseWriter`.
-   Use logging throughout the application to provide insight into the process.

### 7. Testing
-   This project must be developed using **Test-Driven Development (TDD)** for the core logic (processing and database writing).
-   Write unit tests for your reader functions (using sample files), your processing function, and your database writer class.
-   For the database tests, you can use an in-memory SQLite database (`:memory:`) to keep tests fast.
-   Aim for high test coverage (`>90%`).

## ✅ Definition of Done
- [ ] The application can process a directory containing a mix of JSON, CSV, and XML files.
- [ ] Data is correctly parsed, validated, standardized into a `Transaction` object, and loaded into an SQLite database.
- [ ] The application is robust and handles file-not-found errors, corrupted data, and invalid records gracefully.
- [ ] The core logic is well-tested with a `pytest` suite, achieving high coverage.
- [ ] The entire codebase is fully type-annotated and passes `mypy --strict`.
- [ ] The project is structured as a proper Python package.
- [ ] (Bonus) The package is documented and published to TestPyPI.

## 🏆 Bonus Challenges
-   **Concurrency**: Use `concurrent.futures` to process multiple files in parallel. How does this change your design?
-   **Plugin Architecture**: Refactor your readers into a plugin system. The main application should be able to discover and use any reader class that conforms to a specific interface (e.g., an Abstract Base Class).
-   **Async Processing**: Rewrite the I/O-bound parts (reading files, writing to DB) using `asyncio` for a different approach to concurrency.

## 💡 Tips
-   **Start with the Data Model**: A clear, standardized `Transaction` dataclass is the most important part of the project.
-   **TDD is Your Friend**: For a function like `validate_and_standardize`, start with a test for the simplest valid case. Then add tests for all the error cases (missing keys, bad values) one by one.
-   **Separate Concerns**: Your reader's job is just to read. Your processor's job is just to validate and standardize. Your writer's job is just to write. Keeping these separate makes the system much easier to test and maintain.
-   **Think about the "Seams"**: The boundaries between your components (e.g., between the processor and the writer) are important. The `Iterator[Transaction]` that flows between them is a "seam". If you keep these seams clean, your components will be nicely decoupled.
