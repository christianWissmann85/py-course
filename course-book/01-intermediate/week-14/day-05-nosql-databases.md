# Week 14, Day 5: NoSQL & Alternative Databases

## 🎯 Learning Objectives

- [ ] Understand the fundamental concepts of NoSQL databases and how they differ from relational (SQL) databases.
- [ ] Learn about the different categories of NoSQL databases (Key-Value, Document, Column-family, Graph).
- [ ] Get hands-on experience with Redis, a popular in-memory key-value store.
- [ ] Get a conceptual overview of MongoDB, a popular document store.
- [ ] Develop a framework for choosing the right database for a given task.

## 📚 Concepts

### 1. What is NoSQL?

**NoSQL** ("Not Only SQL") is a broad term for databases that do not use the traditional relational model of tables, rows, and columns. They arose from the need for massive scalability, flexible data models, and high performance, particularly for web-scale applications.

**Key Differences from SQL Databases:**
| Feature | SQL (Relational) | NoSQL |
|-------------------|------------------------------------------------|-----------------------------------------------------|
| **Schema** | Strict, predefined schema (schema-on-write) | Dynamic or flexible schema (schema-on-read) |
| **Data Model** | Tables with rows and columns | Varies: Key-value, documents (JSON/BSON), graphs, etc. |
| **Scalability** | Typically scales vertically (bigger server) | Typically scales horizontally (more servers) |
| **Consistency** | Strong consistency (ACID) | Often favors availability over consistency (BASE) |
| **Joins** | Rich support for `JOIN` operations | Generally no or limited support for joins |

### 2. Categories of NoSQL Databases

1.  **Key-Value Stores**: The simplest model. Data is stored as a dictionary-like collection of keys and values. Extremely fast for simple lookups.

    - _Examples_: **Redis**, Amazon DynamoDB.
    - _Use Cases_: Caching, session management, real-time leaderboards.

2.  **Document Stores**: Store data in flexible, semi-structured documents, typically in a JSON-like format (e.g., BSON for MongoDB).

    - _Examples_: **MongoDB**, CouchDB.
    - _Use Cases_: Content management, user profiles, IoT data.

3.  **Column-Family Stores**: Store data in tables with rows and columns, but the columns are dynamic and can vary from row to row. Optimized for wide datasets.

    - _Examples_: Apache Cassandra, Google Bigtable.
    - _Use Cases_: Big data analytics, logging, time-series data.

4.  **Graph Databases**: Designed to store and navigate relationships. Data is modeled as nodes (entities) and edges (relationships).
    - _Examples_: Neo4j, Amazon Neptune.
    - _Use Cases_: Social networks, fraud detection, recommendation engines.

### 3. Hands-On with Redis

**Redis** is an incredibly fast, in-memory key-value store. Because it holds data in RAM, reads and writes are near-instantaneous. It can persist data to disk, but its primary use case is as a high-speed cache or message broker.

First, install the Python client: `poetry add redis`
You also need a Redis server running. The easiest way is with Docker: `docker run -d -p 6379:6379 redis`

```python
import redis

# Connect to the Redis server. decode_responses=True makes it return strings instead of bytes.
r = redis.Redis(host='localhost', port=6379, decode_responses=True)

# --- Basic Key-Value Operations ---
# Set a value. The key is 'user:1:name', the value is 'Alice'.
r.set('user:1:name', 'Alice')

# Get a value
name = r.get('user:1:name')
print(f"Retrieved name: {name}")

# --- Using Hashes for Objects ---
# It's common to store object fields in a hash.
user_data = {
    "name": "Bob",
    "email": "bob@example.com",
    "visits": 10
}
r.hset('user:2', mapping=user_data)

# Get all fields of a hash
retrieved_user = r.hgetall('user:2')
print(f"Retrieved user: {retrieved_user}")

# --- Setting a Time-To-Live (TTL) for Caching ---
# Set a key that will automatically expire in 60 seconds.
r.set('my_cache_key', 'some temporary value', ex=60)

# --- Atomic Operations ---
# Redis operations are atomic. This is great for counters.
r.incr('page_views')
r.incr('page_views')
current_views = r.get('page_views')
print(f"Current page views: {current_views}") # "2"
```

### 4. Conceptual Overview of MongoDB

**MongoDB** is the most popular document database. It stores data in a format called BSON (Binary JSON), which is a typed, binary-encoded version of JSON.

- **Collection**: The equivalent of a SQL table.
- **Document**: The equivalent of a SQL row. A document is a JSON-like object.

**Example Document:**

```json
{
  "_id": "653d4f8b2c3d1a4b9f0e1a2b", // Like a primary key
  "username": "charlie",
  "email": "charlie@mongodb.com",
  "posts": [
    { "title": "My first post", "likes": 10 },
    { "title": "My second post", "likes": 25 }
  ],
  "profile": {
    "bio": "Loves databases",
    "location": "Palo Alto"
  }
}
```

Notice how the schema is flexible. The `posts` array and `profile` object are embedded directly in the user document. This avoids the need for `JOIN`s and can be very efficient if you often need to retrieve a user's profile and posts at the same time.

### 5. How to Choose?

There is no single "best" database. The choice depends entirely on your application's needs.

- **Start with a Relational DB (like PostgreSQL) by default.** They are mature, reliable, and their structured nature is a good fit for most applications, especially early on. You already know SQL and SQLAlchemy!
- **Use Redis for Caching or High-Speed Counters.** If you need to offload work from your primary database or handle real-time data, add Redis into your stack.
- **Consider a Document DB (like MongoDB) when:**
  - Your data has a flexible or unpredictable schema.
  - Your data is naturally document-oriented (e.g., articles, product catalogs).
  - You need to scale horizontally very easily.
- **Consider a Graph DB when:**
  - The relationships between your data are the most important part of your application (e.g., "who follows whom," "what's connected to what").

## 🔹 Quick Exercise

Write a Python function `cache_user_profile(user_id: int, user_data: dict)` that uses Redis to cache a user's profile data (as a hash) with a TTL of 5 minutes (300 seconds).

```python
import redis

r = redis.Redis(host='localhost', port=6379)

def cache_user_profile(user_id: int, user_data: dict):
    """Caches a user profile in Redis using a hash with a TTL."""
    key = f"user:{user_id}:profile"
    # Use a pipeline to ensure atomicity
    with r.pipeline() as pipe:
        # Set the hash values
        pipe.hset(key, mapping=user_data)
        # Set the expiration on the key
        pipe.expire(key, 300)
        # Execute both commands
        pipe.execute()
    print(f"Cached profile for user {user_id}")

# Example usage:
# profile = {"name": "Dana", "email": "dana@redis.com"}
# cache_user_profile(3, profile)
```

## 📝 Daily Assignment

**Goal**: Integrate Redis to add a caching layer to your "To-Do List" application.

1.  **Setup**:
    - Install `redis`: `poetry add redis`.
    - Ensure you have a Redis server running (e.g., via Docker).
2.  **Refactor the `Database` class**:
    - Modify your `Database` class from the ORM assignment (or create a new wrapper class). It should now also accept a Redis connection instance in its `__init__`.
3.  **Implement Caching Logic**:
    - Modify the `get_todo_by_id` method.
    - Before querying the SQL database, it should first try to fetch the todo from Redis. The Redis key could be `todo:{todo_id}`.
    - If the data is in Redis (a "cache hit"), deserialize it (e.g., from JSON) and return it directly.
    - If the data is not in Redis (a "cache miss"), query the SQL database as before.
    - After getting the data from the SQL database, serialize it (e.g., to a JSON string) and store it in Redis with a TTL of, say, 60 seconds before returning it.
4.  **Implement Cache Invalidation**:
    - When you update or delete a todo, you need to invalidate the cache to avoid serving stale data.
    - Modify your `update_todo_status` and `delete_todo` methods to also delete the corresponding key from Redis (e.g., `r.delete(f"todo:{todo_id}")`).
5.  **Write a Test Script**:
    - Write a `main.py` script that demonstrates the caching.
    - Call `get_todo_by_id` for the same ID twice. The first call should be slow (cache miss), and the second should be fast (cache hit). You can add print statements to show this.
    - Call `update_todo_status` for that ID, and then call `get_todo_by_id` again to show that the cache was invalidated and it's another cache miss.

## 📖 Further Reading

- [Redis University](https://redis.com/try-free/) (Free courses on Redis)
- [MongoDB University](https://learn.mongodb.com/) (Free courses on MongoDB)
- [Introduction to NoSQL - AWS](https://aws.amazon.com/nosql/)
- [Martin Fowler: NoSQL](https://martinfowler.com/nosql.html)
