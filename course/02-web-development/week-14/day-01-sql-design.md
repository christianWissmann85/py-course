# Week 14, Day 1: SQL & Database Design

## 🎯 Learning Objectives
- [ ] Understand and write basic SQL (Structured Query Language) commands: `SELECT`, `INSERT`, `UPDATE`, `DELETE`.
- [ ] Learn the fundamentals of relational database design, including tables, columns, and data types.
- [ ] Grasp the concept of normalization and its importance (1NF, 2NF, 3NF).
- [ ] Understand what database indexes are and how they improve query performance.
- [ ] Learn about transactions and the ACID properties that guarantee data integrity.

## 📚 Concepts

### 1. SQL Basics
**SQL (Structured Query Language)** is the standard language for managing and manipulating relational databases.

-   **`CREATE TABLE`**: Defines a new table.
    ```sql
    CREATE TABLE users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT UNIQUE NOT NULL,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    );
    ```
-   **`INSERT`**: Adds a new row of data.
    ```sql
    INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com');
    ```
-   **`SELECT`**: Queries data from a table.
    ```sql
    -- Select all users
    SELECT * FROM users;

    -- Select a specific user by id
    SELECT id, name, email FROM users WHERE id = 1;
    ```
-   **`UPDATE`**: Modifies existing rows.
    ```sql
    UPDATE users SET email = 'alice_new@example.com' WHERE name = 'Alice';
    ```
-   **`DELETE`**: Removes rows.
    ```sql
    DELETE FROM users WHERE name = 'Alice';
    ```

### 2. Relational Database Design
A relational database organizes data into one or more tables of columns and rows.

-   **Table**: Represents an entity (e.g., `users`, `posts`, `products`).
-   **Column (or Field)**: Represents an attribute of the entity (e.g., `name`, `email`, `price`). Each column has a specific **data type** (`INTEGER`, `TEXT`, `REAL`, `TIMESTAMP`).
-   **Row (or Record)**: Represents a single instance of the entity (e.g., one specific user).
-   **Primary Key**: A column (or set of columns) that uniquely identifies each row in a table. `id` is a common choice.
-   **Foreign Key**: A column that links to the primary key of another table, creating a relationship. For example, a `posts` table might have a `user_id` column that is a foreign key to the `users` table's `id`.

```sql
CREATE TABLE posts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    content TEXT,
    user_id INTEGER,
    FOREIGN KEY (user_id) REFERENCES users (id)
);
```

### 3. Normalization
Normalization is the process of organizing columns and tables in a relational database to minimize data redundancy. It involves dividing larger tables into smaller, well-structured tables and defining relationships between them.

-   **First Normal Form (1NF)**:
    -   Ensures that the table is a flat file with no repeating groups.
    -   Each cell must hold a single, atomic value.
    -   *Bad*: A `tags` column with `"python, sql, web"`. *Good*: A separate `tags` table linked to the posts.

-   **Second Normal Form (2NF)**:
    -   Must be in 1NF.
    -   All non-key attributes must be fully dependent on the entire primary key. (This mainly applies to composite primary keys).

-   **Third Normal Form (3NF)**:
    -   Must be in 2NF.
    -   All attributes must be dependent *only* on the primary key, not on other non-key attributes.
    -   *Bad*: A `products` table with `product_id`, `product_name`, `supplier_name`, and `supplier_address`. The supplier's address depends on the supplier, not the product. *Good*: A separate `suppliers` table.

**Goal**: Reduce redundancy and improve data integrity. Over-normalization can sometimes harm performance, so a balance (often 3NF) is desired.

### 4. Indexes
A **database index** is a data structure that improves the speed of data retrieval operations on a database table at the cost of additional writes and storage space.

-   Without an index, the database must scan the entire table to find a specific row (a "full table scan").
-   With an index on a column (e.g., `email`), the database can look up the email in the index (which is sorted, like a book's index) and find the exact location of the row on disk very quickly.

**When to use an index:**
-   On primary keys (usually created automatically).
-   On foreign key columns.
-   On any column that is frequently used in a `WHERE` clause.

```sql
-- Create an index on the 'email' column of the 'users' table.
CREATE INDEX idx_users_email ON users(email);
```
Indexes slow down write operations (`INSERT`, `UPDATE`, `DELETE`) because the index also needs to be updated. Use them judiciously.

### 5. Transactions (ACID)
A **transaction** is a sequence of operations performed as a single logical unit of work. All of the operations in a transaction must succeed; if any part fails, the entire transaction is rolled back.

Transactions provide the **ACID** guarantees:
-   **Atomicity**: All or nothing. The transaction completes entirely or not at all.
-   **Consistency**: The database remains in a valid state before and after the transaction.
-   **Isolation**: Concurrent transactions do not interfere with each other. It appears as if transactions are run sequentially.
-   **Durability**: Once a transaction is committed, it will remain so, even in the event of a power loss or crash.

```sql
BEGIN TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE name = 'Alice';
UPDATE accounts SET balance = balance + 100 WHERE name = 'Bob';

COMMIT; -- Or ROLLBACK; if something went wrong.
```

## 🔹 Quick Exercise
You are building a simple e-commerce site. You need a table for `products` and a table for `categories`. A product can belong to only one category.

Write the `CREATE TABLE` statements for both tables.
-   `products` should have: `id`, `name`, `price`, `description`, and a link to its category.
-   `categories` should have: `id` and `name`.
-   Ensure appropriate primary and foreign keys are set up.

**Answer:**
```sql
CREATE TABLE categories (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL UNIQUE
);

CREATE TABLE products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    price REAL NOT NULL,
    description TEXT,
    category_id INTEGER,
    FOREIGN KEY (category_id) REFERENCES categories (id)
);
```

## 📝 Daily Assignment
**Goal**: Design and document a database schema for a simple social media application.

1.  **Identify Entities**: What are the core entities? You'll likely need `users`, `posts`, `comments`, and `likes`.
2.  **Define Attributes and Relationships**:
    -   For each entity, define its columns and their data types.
    -   How do they relate? A post has an author (a user). A comment belongs to a post AND a user. A like belongs to a post AND a user.
3.  **Draw a Schema Diagram**: Create a simple ERD (Entity-Relationship Diagram). You can do this with a tool like [dbdiagram.io](https://dbdiagram.io/d), or just use text to describe the relationships.
4.  **Write the `CREATE TABLE` Statements**: In a file named `schema.sql`, write the full SQL `CREATE TABLE` statements for all your tables.
    -   Include primary keys, foreign keys, `NOT NULL` constraints, and `UNIQUE` constraints where appropriate.
    -   Add `CREATE INDEX` statements for any foreign keys or columns you expect to be queried frequently.
5.  **Write Sample Queries**: In a separate file named `queries.sql`, write at least five `SELECT` queries to demonstrate how you would retrieve data from your schema.
    -   Example: "Get all posts by a specific user."
    -   Example: "Get all comments for a specific post, along with the commenter's username."
    -   Example: "Count the number of likes for a specific post."

## 📖 Further Reading
- [SQL Zoo - Interactive SQL Tutorial](https://sqlzoo.net/wiki/SQL_Tutorial)
- [W3Schools SQL Tutorial](https://www.w3schools.com/sql/)
- [Database Normalization Explained - Microsoft Docs](https://learn.microsoft.com/en-us/office/troubleshoot/access/database-normalization-description)
- [How Do Database Indexes Work?](https://www.essentialsql.com/what-is-a-database-index/)
