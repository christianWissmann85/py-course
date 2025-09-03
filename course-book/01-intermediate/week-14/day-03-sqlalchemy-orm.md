# Week 14, Day 3: SQLAlchemy ORM

## 🎯 Learning Objectives

- [ ] Understand the concept of an Object-Relational Mapper (ORM) and its benefits.
- [ ] Learn to define database tables as Python classes (models) using the Declarative API.
- [ ] Establish relationships between models (`one-to-many`, `many-to-many`).
- [ ] Understand the role of the `Session` as a unit of work for database interactions.
- [ ] Learn to create, retrieve, update, and delete data using the ORM's query API.

## 📚 Concepts

### 1. What is an ORM?

An **Object-Relational Mapper (ORM)** is a technique that lets you query and manipulate data from a database using an object-oriented paradigm. Instead of writing SQL, you interact with your database through Python objects.

**Benefits:**

- **Abstraction**: You work with Python classes and objects, not tables and rows.
- **Productivity**: Reduces the amount of boilerplate code needed for CRUD (Create, Read, Update, Delete) operations.
- **Type Safety**: Your models are classes, so you get benefits from type checkers like MyPy.
- **Maintainability**: Business logic is encapsulated in your Python models.

SQLAlchemy's ORM is built on top of its Core, so you can always drop down to the Core Expression Language if you need more control.

### 2. Declarative Mapping: Defining Models

The most common way to use the ORM is with the **Declarative API**. You define your tables as regular Python classes that inherit from a `Base` class.

```python
from sqlalchemy import create_engine, Column, Integer, String, ForeignKey
from sqlalchemy.orm import declarative_base, sessionmaker

# 1. Create a Base class
Base = declarative_base()

# 2. Define your model class, inheriting from Base
class User(Base):
    __tablename__ = 'users'  # The name of the table in the database

    # Define columns as class attributes using Column
    id: int = Column(Integer, primary_key=True)
    name: str = Column(String(50), nullable=False)
    email: str = Column(String(100), unique=True)

    def __repr__(self) -> str:
        return f"User(id={self.id!r}, name={self.name!r})"

# To create the table:
engine = create_engine("sqlite:///mydatabase.db")
Base.metadata.create_all(engine)

# To interact with the database, we need a Session
Session = sessionmaker(bind=engine)
session = Session()
```

### 3. The Session: A Unit of Work

The `Session` object is the primary interface for the ORM. Think of it as a "workspace" for your objects. You load objects into the session, modify them, and then "commit" the session to save all your changes to the database at once. It manages the transaction for you.

- `session.add(obj)`: Stages a new object to be inserted.
- `session.add_all([obj1, obj2])`: Stages multiple new objects.
- `session.query(Model)`: Starts a query for a specific model.
- `session.commit()`: Saves all staged changes (inserts, updates, deletes) to the database.
- `session.rollback()`: Discards all staged changes.
- `session.close()`: Closes the session.

It's best practice to use a `with` block to manage the session's lifecycle.

```python
# Create a new user
new_user = User(name="Alice", email="alice@orm.com")
session.add(new_user)
session.commit() # The user is now saved in the database.

# The user object now has an 'id' assigned by the database
print(f"New user created with ID: {new_user.id}")
```

### 4. Querying with the ORM

The ORM provides a rich query API that lets you filter, order, and retrieve objects.

```python
from sqlalchemy import select

# Get all users
all_users = session.query(User).all()
print(all_users)

# Get a user by their primary key
user_by_pk = session.get(User, 1) # Efficiently gets user with id=1
print(user_by_pk)

# A more modern, 2.0-style query using select()
stmt = select(User).where(User.name == "Alice")
alice = session.scalars(stmt).first() # .scalars() gets the User object, not the row
print(alice)

# Updating an object
if alice:
    alice.email = "alice_updated@orm.com"
    session.commit() # The change is detected and an UPDATE statement is issued.

# Deleting an object
if alice:
    session.delete(alice)
    session.commit()
```

### 5. Defining Relationships

This is where the ORM truly shines. You can define relationships between your models, and SQLAlchemy will manage the joins for you.

```python
from sqlalchemy.orm import relationship

class Post(Base):
    __tablename__ = 'posts'
    id: int = Column(Integer, primary_key=True)
    title: str = Column(String(200), nullable=False)
    user_id: int = Column(Integer, ForeignKey("users.id"), nullable=False)

    # This creates a 'posts' attribute on the User model
    # and an 'author' attribute on the Post model.
    author = relationship("User", back_populates="posts")

# We need to add the corresponding relationship to the User model
User.posts = relationship("Post", back_populates="author", cascade="all, delete-orphan")

# Now, you can navigate the relationship
user = session.get(User, 1)
print(f"Posts by {user.name}:")
for post in user.posts:
    print(f"- {post.title}")

# Create a new post for a user
new_post = Post(title="My First Post", author=user)
session.add(new_post)
session.commit()
```

- `relationship()`: Defines the link between two models.
- `back_populates`: Ensures that when you assign one side of the relationship (e.g., `post.author = user`), the other side (`user.posts`) is updated automatically.
- `cascade`: Defines what happens to related objects when an action is taken on the parent. `all, delete-orphan` is a common setting for one-to-many, meaning posts are deleted if they are removed from a user's `posts` collection.

## 🔹 Quick Exercise

Define a `Comment` model that has a many-to-one relationship with the `Post` model. A post can have many comments, but a comment belongs to only one post.

- The `Comment` model should have `id`, `content` (string), and a `post_id` (foreign key).
- Add the necessary `relationship()` calls to both the `Post` and `Comment` models.

```python
# --- Answer ---
class Comment(Base):
    __tablename__ = 'comments'
    id: int = Column(Integer, primary_key=True)
    content: str = Column(String, nullable=False)
    post_id: int = Column(Integer, ForeignKey("posts.id"), nullable=False)

    # Relationship to Post
    post = relationship("Post", back_populates="comments")

# Add the corresponding relationship to the Post model
Post.comments = relationship("Comment", back_populates="post", cascade="all, delete-orphan")
```

## 📝 Daily Assignment

**Goal**: Refactor the "To-Do List" database layer from yesterday to use the SQLAlchemy ORM.

1.  **Create a `Todo` Model**:

    - In a new file `todo_orm.py`, define a `Todo` class that maps to your `todos` table.
    - It should inherit from a declarative `Base`.
    - It should have `id`, `title`, and `completed` attributes defined as `Column`s.

2.  **Create a `Database` Class**:

    - Create a `Database` class that sets up the engine and session.
    - It should have a method to create the tables (`Base.metadata.create_all(engine)`).
    - It should manage a `Session` object.

3.  **Implement ORM-based CRUD Functions**:

    - Rewrite the CRUD functions from yesterday to use the ORM's session and query API.
      - `add_todo(self, title: str) -> Todo`: Creates a `Todo` object, adds it to the session, commits, and returns the new object.
      - `get_all_todos(self) -> list[Todo]`: Returns a list of all `Todo` objects.
      - `get_todo_by_id(self, todo_id: int) -> Todo | None`: Returns a single `Todo` object or `None`.
      - `update_todo_status(self, todo_id: int, completed: bool) -> Todo | None`: Gets a `Todo` object, updates its `completed` status, commits, and returns the updated object.
      - `delete_todo(self, todo_id: int) -> bool`: Gets a `Todo` object, deletes it, commits, and returns `True` if successful.

4.  **Add a `User` Model and Relationship**:

    - Create a `User` model (`id`, `name`).
    - Modify the `Todo` model to include a `user_id` foreign key.
    - Establish a one-to-many relationship between `User` and `Todo`. A user can have many todos.
    - Update your CRUD functions to handle this relationship. For example, `add_todo` should now take a `user_id`, and you should have a new function `get_todos_by_user(user_id: int)`.

5.  **Write a Test Script**: Write a `main.py` that tests your new ORM-based database layer, including the user-todo relationship.

## 📖 Further Reading

- [SQLAlchemy ORM Tutorial (2.0 style)](https://docs.sqlalchemy.org/en/20/orm/tutorial.html)
- [Defining Relationships with the ORM](https://docs.sqlalchemy.org/en/20/orm/basic_relationships.html)
- [The Session Explained](https://docs.sqlalchemy.org/en/20/orm/session_basics.html)
