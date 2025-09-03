# Week 8, Day 2: JSON & Data Serialization

## 🎯 Learning Objectives

- [ ] Master the use of Python's `json` module to serialize Python objects into JSON strings and deserialize them back.
- [ ] Handle complex, custom Python objects by writing a custom JSON encoder.
- [ ] Understand the concept of JSON Schema for validating the structure of JSON data.
- [ ] Be aware of alternative serialization formats like Pickle and YAML.

## 📚 Concepts

### 1. The `json` Module

**Serialization** (or encoding) is the process of converting a Python object into a format that can be stored or transmitted (e.g., a string). **Deserialization** (or decoding) is the reverse process.

JSON (JavaScript Object Notation) is the most common format for this on the web. Python's built-in `json` module provides the tools.

- `json.dumps(obj)`: Dumps an object to a JSON formatted **s**tring.
- `json.loads(s)`: Loads an object from a JSON formatted **s**tring.
- `json.dump(obj, fp)`: Dumps an object to a **f**ile **p**ointer (a file opened for writing).
- `json.load(fp)`: Loads an object from a **f**ile **p**ointer.

```python
import json
from typing import Any

# A Python dictionary (serializable by default)
data: dict[str, Any] = {
    "name": "Alice",
    "age": 30,
    "isStudent": False,
    "courses": ["History", "Math"],
    "address": None
}

# Serialize to a string
json_string: str = json.dumps(data, indent=4) # indent makes it human-readable
print(json_string)

# Deserialize from a string back to a Python object
reloaded_data: dict[str, Any] = json.loads(json_string)
assert data == reloaded_data
```

The `json` module can handle `dict`, `list`, `tuple`, `str`, `int`, `float`, `bool`, and `None`. It cannot handle custom objects by default.

### 2. Custom Serialization with `JSONEncoder`

What happens if you try to serialize a custom object or a `datetime`?

```python
import datetime
# json.dumps({"time": datetime.datetime.now()}) # This will raise a TypeError!
```

The `json` module doesn't know how to convert this object to a string. To fix this, you can create a custom `JSONEncoder` and override its `default()` method.

```python
import json
import datetime
from typing import Any

class CustomJSONEncoder(json.JSONEncoder):
    def default(self, obj: Any) -> Any:
        # If the object is a datetime, format it as an ISO string
        if isinstance(obj, datetime.datetime):
            return obj.isoformat()
        # For any other type, let the base class handle it
        return super().default(obj)

data = {"user": "Bob", "timestamp": datetime.datetime.now()}

# Use the `cls` argument to pass your custom encoder
json_string = json.dumps(data, cls=CustomJSONEncoder, indent=4)
print(json_string)
```

### 3. Streaming JSON

For very large JSON files, loading the whole thing into memory can be inefficient. Libraries like `ijson` allow you to parse a JSON file as a stream, processing it piece by piece without holding the entire structure in memory. This is an advanced technique useful for big data.

### 4. JSON Schema

How do you ensure that the JSON data you receive from an API has the correct structure (e.g., the `age` field is an integer, the `name` field is present)? **JSON Schema** is a vocabulary that allows you to annotate and validate JSON documents.

You define a schema (a dictionary describing the rules), and then use a library like `jsonschema` to validate your data against it.

```python
# An example schema
schema = {
    "type": "object",
    "properties": {
        "name": {"type": "string"},
        "age": {"type": "integer", "minimum": 0},
    },
    "required": ["name", "age"]
}

# Data to validate
valid_data = {"name": "Alice", "age": 30}
invalid_data = {"name": "Bob", "age": -5}

# You would use a library like this:
# from jsonschema import validate
# validate(instance=valid_data, schema=schema)   # This passes
# validate(instance=invalid_data, schema=schema) # This raises a ValidationError
```

### 5. Alternative Formats

- **Pickle**: Python's native serialization format. It can serialize almost any Python object, including custom classes, but it is **not secure**. Never unpickle data from an untrusted source, as it can execute arbitrary code.
- **YAML**: A human-readable format that is often used for configuration files. It's a superset of JSON and can handle more complex data types and comments. Requires a third-party library like `PyYAML`.

## 🔹 Quick Exercise

Complete the `CustomEncoder` class below. It should handle both `datetime` objects and `set` objects (which are not serializable by default). A `set` should be serialized into a `list`.

```python
import json
import datetime
from typing import Any

class CustomEncoder(json.JSONEncoder):
    def default(self, obj: Any) -> Any:
        if isinstance(obj, datetime.datetime):
            return obj.isoformat()
        if isinstance(obj, set):
            return list(obj)
        return super().default(obj)

# --- Tests ---
data = {
    "now": datetime.datetime(2023, 1, 1, 12, 0, 0),
    "tags": {"python", "json", "course"}
}
result = json.dumps(data, cls=CustomEncoder)
# Note: set ordering is not guaranteed, so we check contents
reloaded = json.loads(result)
assert reloaded["now"] == "2023-01-01T12:00:00"
assert sorted(reloaded["tags"]) == ["course", "json", "python"]
print("Custom encoder works!")
```

## 📝 Daily Assignment

**Goal**: Build a system that can serialize and deserialize custom class instances to and from JSON.

1.  **Create Project File**: In your project, create a new file: `my_first_poetry_app/serializer.py`.
2.  **Define a `Book` Dataclass**:
    - Create a `@dataclass` for a `Book` with attributes: `title: str`, `author: str`, `published_date: date`. (Use `datetime.date`).
3.  **Implement a Custom `BookEncoder`**:
    - Create a `BookEncoder(json.JSONEncoder)` class.
    - Override the `default` method to handle `Book` objects and `date` objects.
    - When it sees a `Book` object, it should convert it into a dictionary. A good way is to use `dataclasses.asdict`.
    - When it sees a `date` object, it should convert it into an ISO format string (`date.isoformat()`).
4.  **Implement a `book_decoder` function**:
    - Deserialization is often done with a custom function passed to `json.load(s)`'s `object_hook` parameter.
    - `def book_decoder(dct: dict) -> Book | dict:`
    - This function receives a dictionary. It needs to check if the dictionary looks like a serialized `Book` (e.g., does it have the keys `"title"`, `"author"`, etc.?).
    - If it does, it should convert the `published_date` string back into a `date` object (`date.fromisoformat()`) and return a `Book` instance.
    - If it doesn't look like a `Book`, it should return the dictionary unmodified.
5.  **`main()` function**:
    - Create an instance of your `Book` class.
    - Use `json.dumps` with your `BookEncoder` to serialize it to a JSON string. Print the string.
    - Use `json.loads` with your `book_decoder` as the `object_hook` to deserialize the string back into a `Book` object.
    - Assert that the original object is equal to the deserialized object.
6.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes

- **Security risk with `pickle`**: Never unpickle data from an untrusted source. It is a major security vulnerability. Stick to formats like JSON or YAML for external data.
- **Forgetting `default` or `object_hook`**: When serializing/deserializing custom types, it's easy to forget to pass your custom encoder/decoder function to `json.dump(s)`/`json.load(s)`, which will result in a `TypeError`.
- **Assuming dictionary key order in JSON**: While Python `dict`s preserve insertion order (3.7+), the JSON standard does not guarantee key order. Don't write code that relies on the order of keys in a JSON object.
- **Not handling `None` values**: The JSON `null` is converted to Python's `None`. Ensure your type hints (`str | None`) and logic account for this possibility when dealing with optional data.

## 📖 Further Reading

- [Real Python: Working With JSON Data in Python](https://realpython.com/python-json/)
- [JSON Schema Documentation](https://json-schema.org/)
- [Python Docs: `pickle` — Python object serialization](https://docs.python.org/3/library/pickle.html) (Read the security warnings!)
