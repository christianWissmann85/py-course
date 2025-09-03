# Week 14, Day 2: SQLAlchemy Core

## 🎯 Learning Objectives

- [ ] Understand the architecture of SQLAlchemy and the difference between Core and ORM.
- [ ] Learn to connect to a database using an `Engine`.
- [ ] Define database tables as Python objects using `Table` and `MetaData`.
- [ ] Build and execute type-safe SQL expressions using the SQLAlchemy Core Expression Language.
- [ ] Manage database transactions explicitly using `Connection` and `Transaction` objects.

## 📚 Concepts

### 1. SQLAlchemy Architecture: Core vs. ORM

SQLAlchemy is a powerful SQL toolkit and Object-Relational Mapper (ORM). It's designed in two distinct layers:

- **SQLAlchemy Core**: This is the foundation. It provides a **SQL Expression Language** which allows you to represent database structures (like tables and columns) and SQL queries as Python objects and expressions. It gives you fine-grained control and is very close to writing raw SQL, but in a Pythonic, type-safe, and vendor-neutral way.
- **SQLAlchemy ORM**: This is built on top of Core. It allows you to map your own custom Python classes (e.g., `User`, `Post`) directly to database tables. It handles the "object-relational mapping" so you can work with your database primarily through your Python objects. We will cover the ORM tomorrow.

Today, we focus on **Core**, which is excellent for when you want the power of SQL without being tied to a specific database's syntax and while still getting the benefits of Python's structure and type safety.

First, install it: `poetry add sqlalchemy`

### 2. The Engine and Connection

The `Engine` is the starting point for any SQLAlchemy application. It establishes a pool of database connections to a specific database.

```python
from sqlalchemy import create_engine

# The engine doesn't connect immediately, it just prepares for connections.
# The connection string format is: "dialect+driver://user:password@host:port/database"
# For SQLite, it's simpler as it's a file-based database.
engine = create_engine("sqlite:///mydatabase.db", echo=True) # echo=True logs the generated SQL

# To actually interact with the DB, you get a Connection from the engine.
# The 'with' block ensures the connection is properly closed.
with engine.connect() as connection:
    # We can execute raw SQL (though we'll soon see a better way)
    result = connection.execute("SELECT 'hello world'")
    print(result.scalar())
```

### 3. Defining Table Metadata

In SQLAlchemy Core, you define your database tables using `Table` objects, which are collected in a `MetaData` object. This creates a Python representation of your database schema.

```python
from sqlalchemy import create_engine, MetaData, Table, Column, Integer, String, ForeignKey

# The MetaData object is a container for our schema objects.
metadata_obj = MetaData()

# Define the 'users' table
users_table = Table(
    "users",
    metadata_obj,
    Column("id", Integer, primary_key=True),
    Column("name", String(50), nullable=False),
    Column("email", String(100), unique=True)
)

# Define the 'posts' table with a foreign key
posts_table = Table(
    "posts",
    metadata_obj,
    Column("id", Integer, primary_key=True),
    Column("title", String(200), nullable=False),
    Column("user_id", Integer, ForeignKey("users.id"), nullable=False)
)

# To create these tables in the database:
# This checks for the existence of each table before creating it.
engine = create_engine("sqlite:///mydatabase.db")
metadata_obj.create_all(engine)
```

### 4. The SQL Expression Language

This is the heart of SQLAlchemy Core. Instead of writing SQL strings, you build queries using Python expressions.

```python
from sqlalchemy import select, insert, update, delete

# SELECT statement
# Equivalent to: SELECT users.id, users.name FROM users WHERE users.name = 'Alice'
stmt = select(users_table.c.id, users_table.c.name).where(users_table.c.name == 'Alice')

with engine.connect() as conn:
    result = conn.execute(stmt)
    for row in result:
        print(f"ID: {row.id}, Name: {row.name}") # Access columns by name

# INSERT statement
# Equivalent to: INSERT INTO users (name, email) VALUES ('Bob', 'bob@example.com')
stmt = insert(users_table).values(name="Bob", email="bob@example.com")
with engine.connect() as conn:
    result = conn.execute(stmt)
    # For inserts, we often want the new primary key
    print(f"Inserted PK: {result.inserted_primary_key}")
    conn.commit() # We must commit the transaction!

# UPDATE statement
stmt = (
    update(users_table)
    .where(users_table.c.name == "Bob")
    .values(email="bobby@example.com")
)

# DELETE statement
stmt = delete(users_table).where(users_table.c.name == "Bob")
```

### 5. Transactions

SQLAlchemy uses a "commit as you go" approach. You must explicitly start and end transactions. The `Connection` object provides this.

```python
# The recommended way is to use a block for the transaction.
# It automatically commits if the block succeeds, or rolls back if an exception occurs.
with engine.connect() as conn:
    with conn.begin() as transaction:
        try:
            # Perform multiple related operations
            conn.execute(
                insert(users_table),
                [
                    {"name": "Charlie", "email": "charlie@example.com"},
                    {"name": "David", "email": "david@example.com"},
                ]
            )
            # If an error happened here, the transaction would automatically roll back.
        except:
            print("Transaction failed, rolling back.")
            transaction.rollback()
            raise
    # If we get here without an error, the transaction was committed.
```

## 🔹 Quick Exercise

Using the `users_table` defined in the concepts, write a SQLAlchemy Core expression to select the email of the user with an `id` of 1.

```python
from sqlalchemy import select

# users_table is assumed to be defined
# stmt = select(...)

# --- Answer ---
stmt = select(users_table.c.email).where(users_table.c.id == 1)

# To run it:
# with engine.connect() as conn:
#     email = conn.execute(stmt).scalar_one_or_none()
#     print(f"User 1's email is: {email}")
```

## 📝 Daily Assignment

**Goal**: Build a data access layer for the "To-Do List" application from yesterday using SQLAlchemy Core.

1.  **Setup**:
    - Create a new file `todo_db.py`.
    - Add `sqlalchemy` to your project: `poetry add sqlalchemy`. You will also need a database driver. For SQLite, none is needed. For PostgreSQL, you'd add `psycopg2-binary`.
2.  **Define Schema**:
    - Inside `todo_db.py`, define a `todos` table using `Table`, `MetaData`, and `Column`.
    - The table should have columns for `id` (integer, primary key), `title` (string, not nullable), and `completed` (boolean, default to `False`).
3.  **Create a `Database` Class**:
    - Create a class `Database` that takes a database URL in its constructor and creates an `Engine`.
    - Add a method `create_db_tables()` that uses `metadata.create_all()` to create the tables.
4.  **Implement CRUD Functions**:
    - Add methods to your `Database` class for each CRUD operation. These methods should use the SQLAlchemy Expression Language, not raw SQL strings.
      - `add_todo(self, title: str) -> int`: Adds a new todo and returns its new ID.
      - `get_all_todos(self) -> list[dict]`: Returns a list of all todos. Each todo should be a dictionary-like object.
      - `get_todo_by_id(self, todo_id: int) -> dict | None`: Returns a single todo or `None`.
      - `update_todo_status(self, todo_id: int, completed: bool) -> None`: Updates a todo's status.
      - `delete_todo(self, todo_id: int) -> None`: Deletes a todo.
5.  **Use Transactions**: Ensure that your write operations (`add`, `update`, `delete`) are wrapped in transactions.
6.  **Create a Main Script**: Create a `main.py` to test your `Database` class. It should:
    - Instantiate the `Database` class with a SQLite URL (`sqlite:///todos.db`).
    - Call `create_db_tables()`.
    - Add a few todos, print them all, update one, get it again, delete it, and finally print all remaining todos.

## 📖 Further Reading

- [SQLAlchemy 2.0 Unified Tutorial](https://docs.sqlalchemy.org/en/20/tutorial/index.html) (The new official tutorial is excellent)
- [SQLAlchemy Core - Data and Schema](https://docs.sqlalchemy.org/en/20/core/metadata.html)
- [SQLAlchemy Core - Expression Language](https://docs.sqlalchemy.org/en/20/core/expression_api.html)
