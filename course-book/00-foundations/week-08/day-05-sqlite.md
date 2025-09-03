# Week 8, Day 5: Database Basics (SQLite)

## 🎯 Learning Objectives

- [ ] Understand what a database is and why SQLite is a useful starting point.
- [ ] Use Python's built-in `sqlite3` module to connect to a database.
- [ ] Execute SQL queries to create tables, insert data, and retrieve data.
- [ ] Understand the importance of transactions for maintaining data integrity.

## 📚 Concepts

### 1. SQLite Basics

A database is an organized collection of data. A relational database (which uses SQL - Structured Query Language) organizes data into tables with rows and columns.

**SQLite** is a C library that provides a lightweight, disk-based database that doesn’t require a separate server process. It's built into Python and is perfect for learning, prototyping, and small to medium-sized applications. The entire database is stored in a single `.db` file.

### 2. Connection and Cursor

To interact with an SQLite database, you need two objects:

- **Connection**: Represents your connection to the database file.
- **Cursor**: An object used to execute SQL queries and fetch results.

```python
import sqlite3

# This creates a connection to the database file (it will be created if it doesn't exist).
con = sqlite3.connect("my_database.db")

# A cursor is needed to execute queries.
cur = con.cursor()

# Do stuff...

# Always close the connection when you're done.
con.close()
```

A `with` statement is the best way to handle connections, as it automatically handles committing or rolling back transactions and closing the connection.

### 3. Executing SQL Queries

You use the cursor's `.execute()` method to run SQL commands.

```python
import sqlite3

with sqlite3.connect("my_database.db") as con:
    cur = con.cursor()

    # CREATE TABLE: Defines the schema for a table
    cur.execute("""
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY,
            name TEXT NOT NULL,
            email TEXT UNIQUE NOT NULL
        )
    """)

    # INSERT: Adds a new row of data
    # Use '?' as a placeholder to prevent SQL injection attacks!
    cur.execute("INSERT INTO users (name, email) VALUES (?, ?)", ("Alice", "alice@example.com"))

    # SELECT: Retrieves data
    res = cur.execute("SELECT name, email FROM users")

    # Fetch the results
    user = res.fetchone() # Fetches one row
    print(user) # ('Alice', 'alice@example.com')
```

**Security Note**: Never use f-strings to put values into an SQL query. This is vulnerable to **SQL injection**. Always use the `?` placeholder style.

### 4. Transactions

A **transaction** is a sequence of operations performed as a single logical unit of work. All operations in a transaction must complete successfully, or none of them are applied. This is known as **atomicity**.

In Python's `sqlite3` module, a transaction is implicitly opened when you make your first data-modifying statement (like `INSERT` or `UPDATE`). To save the changes to the database, you must call `connection.commit()`. To undo them, you call `connection.rollback()`.

The `with con:` statement simplifies this:

- If the `with` block completes successfully, it automatically calls `commit()`.
- If an exception occurs inside the `with` block, it automatically calls `rollback()`.

```python
# The 'with' statement handles the transaction for us.
try:
    with sqlite3.connect("my_database.db") as con:
        cur = con.cursor()
        cur.execute("INSERT INTO users (name, email) VALUES (?, ?)", ("Bob", "bob@example.com"))
        cur.execute("INSERT INTO users (name, email) VALUES (?, ?)", ("Charlie", "charlie@example.com"))
        # This next line will fail because the email must be unique
        cur.execute("INSERT INTO users (name, email) VALUES (?, ?)", ("Bad", "bob@example.com"))
except sqlite3.IntegrityError as e:
    print(f"Transaction failed: {e}")
    # The 'with' block automatically rolls back.
    # Neither Bob nor Charlie will be added to the database.
```

### 5. Best Practices

- **Use `with` statements**: Always manage your connections with a `with` block.
- **Use placeholders (`?`)**: Never format queries with f-strings.
- **Fetch what you need**: `fetchone()` gets one row, `fetchall()` gets all rows as a list (which can use a lot of memory), or you can simply iterate over the cursor (`for row in cur:`).
- **Separate SQL from code**: In larger apps, it's common to store SQL queries in separate files or constants rather than embedding them directly in your functions.

## 🔹 Quick Exercise

Complete the `get_db` context manager below. A context manager is a function decorated with `@contextmanager` that can be used in a `with` statement. It should connect to the database, `yield` the cursor, and then ensure the connection is closed.

```python
import sqlite3
from contextlib import contextmanager
from typing import Generator

@contextmanager
def get_db(db_path: str) -> Generator[sqlite3.Cursor, None, None]:
    """A context manager for a database connection."""
    con = sqlite3.connect(db_path)
    try:
        # Yield the cursor to the 'with' block
        yield con.cursor()
        # Commit changes if the block was successful
        con.commit()
    except Exception:
        # Rollback on any error
        con.rollback()
        raise
    finally:
        # This cleanup code runs no matter what
        print("Closing database connection.")
        con.close()

# --- Test ---
DB_FILE = "test.db"
with get_db(DB_FILE) as cur:
    cur.execute("CREATE TABLE IF NOT EXISTS test (id INTEGER)")
    cur.execute("INSERT INTO test (id) VALUES (1)")

# Check if the data was written
con = sqlite3.connect(DB_FILE)
assert con.execute("SELECT COUNT(*) FROM test").fetchone()[0] == 1
con.close()
```

## 📝 Daily Assignment

**Goal**: Build a simple database application to manage a "to-do" list.

1.  **Create Project File**: In your project, create `my_first_poetry_app/todo_db.py`.
2.  **Setup Function**:
    - Write a `setup_database(db_path: str) -> None` function that connects to the database and creates a `todos` table if it doesn't exist. The table should have `id` (integer primary key), `task` (text, not null), and `is_complete` (integer, 0 or 1).
3.  **CRUD Functions**: Implement the four "CRUD" (Create, Read, Update, Delete) operations:
    - `add_task(db_path: str, task_text: str) -> None`
    - `get_all_tasks(db_path: str) -> list[tuple]`: Should return a list of all task rows.
    - `mark_task_complete(db_path: str, task_id: int) -> None`
    - `delete_task(db_path: str, task_id: int) -> None`
    - All these functions should connect to the database, perform their operation, and close the connection. Use `with sqlite3.connect(...)` to manage transactions.
4.  **`main()` function**:
    - Call `setup_database`.
    - Add a few tasks.
    - List all tasks.
    - Mark one task as complete.
    - Delete a task.
    - List the remaining tasks.
5.  **Verify**: Run the script, type checker, and linter.

## 📦 Week 8 Capstone: Multi-Format Data Processor

This is the final project for the Foundations phase! You will combine everything you've learned to build a tool that processes data from multiple formats (JSON, CSV, XML) and stores the results in an SQLite database.

**Key Requirements**:

- Read data from multiple source formats.
- Perform validation and transformation.
- Store the clean, standardized data in an SQLite database.
- Be fully type-safe and have a good test suite.

See `week-08-capstone-project.md` for a more detailed breakdown.

## ⚠️ Common Mistakes

- **SQL Injection**: The most critical error. Never use string formatting to create queries with user-supplied data.
- **Forgetting to `commit()`**: If you don't use a `with` statement, you must manually call `connection.commit()` or your changes will be lost when the connection is closed.
- **Fetching large result sets**: Calling `.fetchall()` on a query that returns millions of rows can exhaust your system's memory. Iterating over the cursor (`for row in cur:`) is usually safer.
- **Mixing up `fetchone()` and `fetchall()`**: `fetchone()` returns a single row or `None`. `fetchall()` returns a list of all rows (which might be empty). Using one when you expect the other is a common bug.

## 📖 Further Reading

- [Python Docs: `sqlite3` — DB-API 2.0 interface for SQLite databases](https://docs.python.org/3/library/sqlite3.html)
- [Real Python: Using `sqlite3` in Python](https://realpython.com/python-sqlite-sqlalchemy/)
- [SQLite Tutorial](https://www.sqlitetutorial.net/) (A great resource for learning SQL syntax specific to SQLite)
