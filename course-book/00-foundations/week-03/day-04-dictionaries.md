# Week 3, Day 4: Dictionaries & TypedDict

## 🎯 Learning Objectives

- [ ] Master the creation, access, and modification of dictionaries.
- [ ] Use `TypedDict` to provide a static type definition for dictionary structures.
- [ ] Understand the role of hashing in what can be used as a dictionary key.
- [ ] Use dictionary comprehensions for a concise way to create dictionaries.

## 📚 Concepts

### 1. Dictionary Basics

A dictionary (`dict`) is an unordered (in Python < 3.7) or insertion-ordered (Python 3.7+) collection of key-value pairs. Each key in a dictionary must be unique and hashable (e.g., `str`, `int`, `float`, `tuple`).

```python
# Creating a dictionary
user_profile: dict[str, int | str] = {
    "user_id": 123,
    "name": "Alice",
    "level": 5
}

# Accessing a value by its key
user_name: str | int = user_profile["name"] # "Alice"

# Adding or modifying a key-value pair
user_profile["level"] = 6
user_profile["email"] = "alice@example.com"
```

The type hint `dict[KeyType, ValueType]` defines the expected types for keys and values.

### 2. Dictionary Methods

- `.get(key, default=None)`: Safely get a value by key. Returns `None` (or a specified default) if the key doesn't exist, instead of raising a `KeyError`.
- `.keys()`: Returns a view of the dictionary's keys.
- `.values()`: Returns a view of the dictionary's values.
- `.items()`: Returns a view of the dictionary's key-value tuple pairs. This is very useful for looping.

```python
# Looping through a dictionary
for key, value in user_profile.items():
    print(f"{key}: {value}")

# Safe access with .get()
email: str | int | None = user_profile.get("email") # "alice@example.com"
phone: str | int | None = user_profile.get("phone") # None
```

### 3. `TypedDict` for Structure

A standard `dict` type hint doesn't specify _which_ keys should be present. `TypedDict` solves this. It lets you define a "shape" that a dictionary should have, and MyPy will enforce it.

```python
from typing import TypedDict

# Define the structure of a User dictionary
class User(TypedDict):
    user_id: int
    name: str
    email: str | None # This key is optional, but if present, must be str or None

# Create a dictionary that conforms to the User shape
user1: User = {
    "user_id": 101,
    "name": "Bob",
    "email": "bob@example.com"
}

# MyPy will flag an error if a key is missing or has the wrong type
# user2: User = {"user_id": 102, "name": "Charlie"} # Error: missing "email"
```

This is incredibly useful for working with data from APIs or databases.

### 4. `defaultdict` and `Counter`

The `collections` module provides specialized dictionary subclasses.

- `defaultdict`: When you access a missing key, it creates a default value for it instead of raising a `KeyError`.
- `Counter`: A `dict` subclass for counting hashable objects.

```python
from collections import defaultdict, Counter

# defaultdict example: grouping items
fruits: list[str] = ["apple", "banana", "apple", "orange", "banana", "apple"]
grouped: defaultdict[str, int] = defaultdict(int) # int() creates a default value of 0
for fruit in fruits:
    grouped[fruit] += 1
# grouped is now defaultdict(<class 'int'>, {'apple': 3, 'banana': 2, 'orange': 1})

# Counter does this even more easily!
fruit_counts: Counter[str] = Counter(fruits)
print(fruit_counts) # Counter({'apple': 3, 'banana': 2, 'orange': 1})
print(fruit_counts.most_common(1)) # [('apple', 3)]
```

### 5. Dictionary Comprehensions & Hashing

Like list and set comprehensions, dict comprehensions provide a concise way to create dictionaries.

```python
# Create a dictionary of numbers and their squares
squares_dict: dict[int, int] = {x: x*x for x in range(5)}
# {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```

Dictionaries are fast for lookups (`my_dict[key]`) because they use a technique called **hashing**. Python runs the key through a hash function to compute an index where the value should be stored in memory. This is why keys must be **hashable** (i.e., immutable). All of Python's built-in immutable types (`int`, `str`, `tuple`, `frozenset`) are hashable.

## 🔹 Quick Exercise

Implement the `merge_users` function below. The function should take a list of `User` dictionaries and return a single dictionary where the keys are the users' emails and the values are the `User` objects themselves.

```python
from typing import TypedDict, List, Dict

class User(TypedDict):
    name: str
    age: int
    email: str

def merge_users(users: List[User]) -> Dict[str, User]:
    """
    Merges a list of user dictionaries into a single dictionary
    keyed by the user's email address.
    """
    # Use a dictionary comprehension for a concise solution.
    pass

# --- Tests ---
user_list: List[User] = [
    {'name': 'Alice', 'age': 30, 'email': 'alice@example.com'},
    {'name': 'Bob', 'age': 42, 'email': 'bob@example.com'}
]

merged = merge_users(user_list)
assert 'alice@example.com' in merged
assert merged['bob@example.com']['name'] == 'Bob'
print("Merge successful!")
```

## 📝 Daily Assignment

**Goal**: Build a simple in-memory key-value data store using dictionaries and `TypedDict`.

1.  **Create Project File**: In `my-first-poetry-app`, create a new file: `my_first_poetry_app/data_store.py`.
2.  **Define Data Models with `TypedDict`**:
    - Create a `Movie` `TypedDict` with fields like `title: str`, `director: str`, `release_year: int`.
    - Create a `DataStore` `TypedDict` that holds the entire database. It should have one key, `"movies"`, which holds a `dict[str, Movie]`. The key of this inner dict will be the movie title.
3.  **Implement Store Functions**:
    - `create_empty_store() -> DataStore`: A function that returns a new, empty data store.
    - `add_movie(store: DataStore, movie: Movie) -> None`: Adds a movie to the store. The movie's title should be its key in the `store["movies"]` dictionary.
    - `get_movie(store: DataStore, title: str) -> Movie | None`: Retrieves a movie by its title. Use the `.get()` method for safe access.
    - `find_movies_by_year(store: DataStore, year: int) -> list[Movie]`: Returns a list of all movies released in a given year.
4.  **Implement a Cache (Bonus)**:
    - Create a simple `Cache` class that holds a `dict` internally.
    - Modify your `get_movie` function. Before searching the main `store`, it should first check a `Cache` instance. If the movie is in the cache, return it from there. If not, get it from the store, add it to the cache, and then return it. This simulates a common performance optimization pattern.
5.  **`main()` function**:
    - Demonstrate all your functions. Create a store, add several movies, retrieve them, and search by year. Print the results.
6.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes

- **`KeyError`**: Accessing a key that doesn't exist using square brackets (`my_dict['non_existent_key']`). Use `my_dict.get('non_existent_key')` or check `if key in my_dict:` to prevent this.
- **Modifying a dictionary while iterating over it**: This can raise a `RuntimeError`. If you need to add or remove items while looping, iterate over a copy of the keys: `for key in list(my_dict.keys()):`.
- **Assuming key order**: In modern Python (3.7+), dictionaries preserve insertion order. However, this is a relatively new feature. Code that needs to run on older versions should not rely on this. If order is critical and you need to support older Python, use `collections.OrderedDict`.
- **Using mutable keys**: `my_dict[[1, 2]] = 'value'` will raise a `TypeError` because lists are not hashable. You must use an immutable type like a tuple: `my_dict[(1, 2)] = 'value'`.

## 📖 Further Reading

- [Real Python: Dictionaries in Python](https://realpython.com/python-dicts/)
- [Python Docs: `typing.TypedDict`](https://docs.python.org/3/library/typing.html#typing.TypedDict)
- [Python Docs: `collections` — Container datatypes](https://docs.python.org/3/library/collections.html) (Read about `defaultdict` and `Counter`)
