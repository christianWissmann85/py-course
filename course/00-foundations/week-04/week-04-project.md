# Week 4 Project: Functional Data Processor

## 🎯 Project Objective
To build a data processing pipeline using a functional programming style. This project will challenge you to use the concepts from this week—pure functions, higher-order functions, generators, and decorators—to create a memory-efficient and elegant solution. You will process a large dataset without loading it all into memory at once.

## 📋 Core Requirements
You will build a script that reads a large CSV file of sales data, processes it through a pipeline of generator functions, and calculates a summary report.

### 1. Project Setup
-   In your `my-first-poetry-app` project, create a new file: `my_first_poetry_app/functional_processor.py`.
-   You will also need a sample CSV file. Create a function to generate a large `sales_data.csv` file with at least 100,000 rows. The CSV should have the columns: `product_id`, `category`, `price`, `quantity`, `sale_date`.

### 2. The Data Pipeline (Generators)
Your processing logic must be implemented as a chain of generators. Each step in the pipeline should be a generator function that takes an iterator as input and yields processed data.

-   **`read_sales_data(filepath: str) -> Generator[dict, None, None]`**: Reads the CSV file row by row and yields each row as a dictionary. Use `csv.DictReader`.
-   **`filter_by_category(rows: Generator[dict, None, None], category: str) -> Generator[dict, None, None]`**: A generator that yields only the rows that match a given category.
-   **`add_total_price_field(rows: Generator[dict, None, None]) -> Generator[dict, None, None]`**: A generator that takes rows, calculates `price * quantity`, and yields a new dictionary with an added `total_price` field. Handle potential `ValueError`s from the data gracefully (e.g., by skipping the row and logging a warning).
-   **`extract_field(rows: Generator[dict, None, None], field: str) -> Generator[Any, None, None]`**: A generic generator that yields only the value of a specific field from each row.

### 3. Decorator Usage
-   Create a `@time_it` decorator that measures and prints the execution time of any function it decorates.
-   Apply this decorator to your main processing function to see how long the entire pipeline takes to run.

### 4. Pure Functions for Calculation
-   `calculate_total_revenue(sales_totals: Generator[float, None, None]) -> float`: A pure function that takes a generator of sales totals and returns their sum. You can use `sum()` for this.

### 5. Main Processing Logic
-   Create a `main` function that sets up and runs the pipeline.
-   **The pipeline should**:
    1.  Read the sales data using `read_sales_data`.
    2.  Filter the data for a specific category (e.g., "electronics").
    3.  Add the `total_price` field to the filtered data.
    4.  Extract just the `total_price` field.
    5.  Pass the final generator to `calculate_total_revenue` to get the result.
-   The `main` function should be decorated with `@time_it`.
-   Print the final calculated revenue.

### 6. Type Safety & Code Quality
-   The entire application must be fully type-annotated and pass `mypy --strict`.
-   The code must be well-documented with docstrings.
-   The code must pass all `ruff check .` linter checks.

## ✅ Definition of Done
- [ ] A `functional_processor.py` script exists.
- [ ] A function to generate a large sample CSV file is present.
- [ ] The data processing is implemented as a chain of four distinct generator functions.
- [ ] A `@time_it` decorator is implemented and used.
- [ ] The pipeline correctly calculates the total revenue for a specific category.
- [ ] The solution is memory-efficient (it does not load the entire file into memory at once).
- [ ] The entire codebase is type-safe, documented, and passes the linter.

## 🏆 Bonus Challenges
-   **Error Handling Decorator**: Create an `@error_handler` decorator that can wrap a generator function. If the generator encounters an exception (e.g., a `ValueError` during type conversion), the decorator should log the error and gracefully continue processing the rest of the data instead of crashing.
-   **Pipeline Function**: Create a higher-order `pipeline` function that takes an initial iterator and a list of functions (your generator functions) and chains them all together, returning the final processed iterator. This makes the `main` function even cleaner.
-   **Configuration Driven**: Instead of hardcoding the category to filter, read it from a simple configuration file or pass it as a command-line argument.

## 💡 Tips
-   The power of this pattern is its laziness. No data is read or processed until you start iterating over the final result (e.g., when `calculate_total_revenue` calls `sum()`).
-   Test each generator function independently before chaining them together.
-   Use `print()` statements inside your generators initially to visualize how the data flows through the pipeline one item at a time. Remove them once you understand the flow.
