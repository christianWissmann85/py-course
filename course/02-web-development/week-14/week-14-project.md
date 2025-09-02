# Week 14 Project: Blog Platform Backend

## 🎯 Project Goal
To design and build the complete backend for a multi-user blog platform. This project will involve creating a robust database schema, implementing a full RESTful API for interacting with the data, and managing the database lifecycle with professional-grade tools.

## 📚 Background
You've spent this week diving deep into databases, moving from raw SQL to the powerful SQLAlchemy ORM and migration tools. This project is your opportunity to synthesize all of those skills to build a real-world application. A blog is a classic example because it involves several related entities (users, posts, comments) and a clear set of actions, making it perfect for practicing your database design and API development skills.

We will build the API using **FastAPI**, as it integrates seamlessly with SQLAlchemy and Pydantic for data validation.

```bash
# Make sure you are in your project directory
poetry add fastapi uvicorn sqlalchemy passlib[bcrypt] python-jose[cryptography]
# For PostgreSQL (recommended)
poetry add psycopg2-binary
# For Migrations
poetry add alembic
# For Caching (Bonus)
poetry add redis
```

## 📋 Project Requirements

### 1. Database Schema Design
This is the foundation of your project. You need to define the models for the core entities of the blog.

**Required Models:**
-   **`User`**:
    -   `id` (primary key)
    -   `email` (string, unique, indexed)
    -   `hashed_password` (string)
    -   `is_active` (boolean, default: `True`)
-   **`Post`**:
    -   `id` (primary key)
    -   `title` (string, indexed)
    -   `content` (text)
    -   `created_at` (datetime, default: now)
    -   `owner_id` (foreign key to `users.id`)
-   **`Comment`**:
    -   `id` (primary key)
    -   `content` (text)
    -   `created_at` (datetime, default: now)
    -   `post_id` (foreign key to `posts.id`)
    -   `owner_id` (foreign key to `users.id`)

**Relationships:**
-   A `User` can have many `Posts`.
-   A `User` can have many `Comments`.
-   A `Post` can have many `Comments`.
-   A `Post` has one `User` (the author).
-   A `Comment` has one `User` (the author) and one `Post`.

Use SQLAlchemy ORM to define these models and their relationships (`relationship`, `back_populates`, etc.).

### 2. Database Migrations with Alembic
-   You **must** use Alembic to manage your schema.
-   Initialize an Alembic environment.
-   Create an initial migration that generates all the tables based on your SQLAlchemy models.
-   As you develop, if you need to change a model (e.g., add a column), you must create a new migration for it.

### 3. API Endpoints
Your FastAPI application should expose a RESTful API with the following endpoints. Use Pydantic for request and response models to ensure type safety and automatic documentation.

**Authentication:**
-   `POST /users/`: Register a new user. The password in the request body must be hashed before being saved to the database.
-   `POST /token`: Log a user in. Takes a username (email) and password, verifies them, and returns a JWT access token.

**Posts:**
-   `GET /posts/`: Get a list of all posts (paginated).
-   `POST /posts/`: Create a new post. Requires authentication.
-   `GET /posts/{post_id}`: Get a single post by its ID.
-   `PUT /posts/{post_id}`: Update a post. Requires authentication, and the user must be the post's owner.
-   `DELETE /posts/{post_id}`: Delete a post. Requires authentication, and the user must be the post's owner.

**Comments:**
-   `GET /posts/{post_id}/comments`: Get all comments for a specific post.
-   `POST /posts/{post_id}/comments`: Add a new comment to a post. Requires authentication.

### 4. Security
-   **Password Hashing**: Do not store plain-text passwords. Use `passlib` to hash passwords upon registration and verify them at login.
-   **Authentication**: Protect endpoints that modify data using FastAPI's dependency injection system. A dependency should verify the JWT token from the `Authorization` header and return the current user.

## 🏆 Bonus Challenges
1.  **Add a "Likes" System**:
    -   Create a `Like` model that links a `User` and a `Post`. This will be a many-to-many relationship through an association table.
    -   Add endpoints: `POST /posts/{post_id}/like` to like a post and `DELETE /posts/{post_id}/like` to unlike it.
    -   Display the like count on post endpoints.

2.  **Asynchronous Database Operations**:
    -   Refactor your database layer to be fully asynchronous.
    -   Use an async driver for your database (e.g., `asyncpg` for PostgreSQL).
    -   Use SQLAlchemy's `AsyncEngine` and `AsyncSession`.
    -   Make all your API endpoints `async def`.

3.  **Redis Caching**:
    -   Integrate Redis to cache expensive or frequently accessed data.
    -   Cache the response of `GET /posts/`. Invalidate the cache whenever a new post is created or an existing one is updated/deleted.
    -   Cache the response of `GET /posts/{post_id}` with a TTL. Invalidate it when the post or its comments are changed.

## 💡 Tips
-   **Structure Your Project**: Organize your code into logical modules: `models.py`, `schemas.py` (for Pydantic), `crud.py` (for database functions), `main.py` (for FastAPI endpoints), `database.py` (for engine/session setup), and `security.py` (for auth helpers).
-   **Start with Models and Migrations**: Get your database schema and migrations working before you write a single API endpoint.
-   **Implement Auth First**: User registration and login are prerequisites for most other features.
-   **Dependency Injection is Key**: Use FastAPI's `Depends` for everything: getting a database session (`def get_db()`), getting the current user (`def get_current_user()`), etc.
-   **Test as You Go**: While a full test suite isn't a core requirement, using the interactive API docs (`/docs`) provided by FastAPI is a great way to test your endpoints as you build them.
