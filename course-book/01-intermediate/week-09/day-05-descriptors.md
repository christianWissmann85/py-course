# Week 9, Day 5: Descriptors

## 🎯 Learning Objectives

- [ ] Understand the descriptor protocol (`__get__`, `__set__`, `__delete__`).
- [ ] Create custom descriptor classes to manage attribute access.
- [ ] Understand how Python's properties are actually implemented using descriptors.
- [ ] Use descriptors to create reusable validation logic for class attributes.

## 📚 Concepts

### 1. What are Descriptors?

A descriptor is an object that controls how another object's attributes are accessed. If an object has a `__get__`, `__set__`, or `__delete__` method, it is said to be a descriptor.

Descriptors are the underlying mechanism for properties, methods, static methods, class methods, and `super()`. They are a powerful, low-level feature that allows you to customize attribute access logic.

### 2. The Descriptor Protocol

A class that implements any of these three methods is a descriptor:

- **`__get__(self, instance, owner)`**: Called when the descriptor's value is retrieved.
  - `self`: The descriptor instance itself.
  - `instance`: The instance of the class where the descriptor is being accessed (e.g., `my_obj`).
  - `owner`: The class itself (e.g., `MyClass`).
- **`__set__(self, instance, value)`**: Called when a new value is assigned to the descriptor.
- **`__delete__(self, instance)`**: Called when `del` is used on the descriptor.

### 3. Creating a Descriptor

Let's create a simple descriptor that logs when an attribute is accessed or set.

```python
class LoggingDescriptor:
    def __get__(self, instance, owner):
        print(f"Getting value from {instance}...")
        # A common pattern is to store the actual value on the instance,
        # often with a "private" name to avoid name clashes.
        return instance.__dict__.get(self.name, None)

    def __set__(self, instance, value):
        print(f"Setting value to {value} on {instance}...")
        instance.__dict__[self.name] = value

    def __set_name__(self, owner, name):
        # This saves the public name of the attribute.
        self.name = name

class MyClass:
    # The descriptor is a class attribute
    my_attribute = LoggingDescriptor()

# ---
instance = MyClass()

# This calls LoggingDescriptor.__set__
instance.my_attribute = 10
# Output: Setting value to 10 on <__main__.MyClass object at ...>...

# This calls LoggingDescriptor.__get__
value = instance.my_attribute
# Output: Getting value from <__main__.MyClass object at ...>...
```

### 4. Data vs. Non-Data Descriptors

- **Data Descriptor**: An object with both `__get__` and `__set__` (or `__delete__`).
- **Non-Data Descriptor**: An object with only `__get__`.

This distinction is important for how attributes are looked up. **Data descriptors always override an instance's dictionary.** If you have a data descriptor named `x` and an instance attribute `__dict__['x']`, the descriptor will always be called.

Non-data descriptors, however, can be overridden by an instance attribute. This is how methods work. Methods are non-data descriptors (they only have `__get__`).

### 5. How Properties are Implemented

The `@property` decorator is just elegant syntactic sugar for creating a descriptor.

```python
# When you write this:
class MyClass:
    @property
    def my_attr(self):
        return 42

# Python is effectively doing this:
class MyAttrDescriptor:
    def __get__(self, instance, owner):
        return 42

class MyClass:
    my_attr = MyAttrDescriptor()
```

When you add a `.setter`, you are creating a _data descriptor_ with both `__get__` and `__set__` methods. This is why you can't have an instance attribute with the same name as a property that has a setter.

### 6. Use Case: Reusable Validation

The most common use case for custom descriptors is to create reusable validation logic for attributes.

```python
class PositiveNumber:
    """A descriptor that ensures an attribute is always a positive number."""
    def __set_name__(self, owner, name):
        # This special method saves the attribute's name (e.g., 'width' or 'height')
        self.private_name = '_' + name

    def __get__(self, instance, owner):
        return getattr(instance, self.private_name)

    def __set__(self, instance, value):
        if not isinstance(value, (int, float)) or value <= 0:
            raise ValueError(f"Value must be a positive number, not {value!r}")
        setattr(instance, self.private_name, value)

class Rectangle:
    # We can reuse the same validation logic for multiple attributes!
    width = PositiveNumber()
    height = PositiveNumber()

    def __init__(self, width, height):
        self.width = width # Calls PositiveNumber.__set__
        self.height = height

rect = Rectangle(10, 20)
print(rect.width) # 10, calls PositiveNumber.__get__

try:
    rect.width = -5 # Calls PositiveNumber.__set__ and raises ValueError
except ValueError as e:
    print(e)
```

## 🔹 Quick Exercise

Complete the `ValidatedAttribute` descriptor. It should ensure that any value assigned is a non-empty string.

```python
class ValidatedAttribute:
    """A descriptor that validates for non-empty strings."""
    def __set_name__(self, owner, name):
        self.private_name = '_' + name

    def __get__(self, instance, owner):
        return getattr(instance, self.private_name)

    def __set__(self, instance, value):
        if not isinstance(value, str) or not value:
            raise ValueError("Value must be a non-empty string")
        setattr(instance, self.private_name, value)

class Person:
    name = ValidatedAttribute()

    def __init__(self, name: str):
        self.name = name # This will call the descriptor's __set__ method

# --- Tests ---
p = Person("Alice")
assert p.name == "Alice"

try:
    p.name = "" # Should fail
    assert False, "Should have raised ValueError for empty string"
except ValueError:
    pass

try:
    p.name = 123 # Should fail
    assert False, "Should have raised ValueError for non-string"
except ValueError:
    pass
```

## 📝 Daily Assignment

**Goal**: Build a small ORM-like (Object-Relational Mapper) system where you define typed "fields" for a model class.

1.  **Create Project File**: In your project, create `my_first_poetry_app/orm.py`.
2.  **Create Descriptor "Field" Classes**:
    - `class Field`: A base class for your descriptors.
    - `class IntField(Field)`: A descriptor that ensures the assigned value is an integer.
    - `class CharField(Field)`: A descriptor that ensures the assigned value is a string and, optionally, has a maximum length. The `__init__` of this descriptor should accept `max_length`.
3.  **Create a `Model` Base Class**:
    - This class will be the parent for all your models. Its main job is to have a nice `__repr__` that automatically prints the names and values of all the descriptor fields.
4.  **Create a Concrete `User` Model**:
    - `class User(Model)`:
      - `id = IntField()`
      - `username = CharField(max_length=20)`
      - `email = CharField()`
    - Your `User` class should have an `__init__` that accepts `id`, `username`, and `email` and assigns them to the class attributes (which are your descriptors).
5.  **`main()` function**:
    - Create an instance of `User`.
    - Print the user instance to see your `__repr__` work.
    - Demonstrate the validation by trying to assign an integer to `username` or a string that is too long. Catch the `ValueError`s.
6.  **Verify**: Run the script, type checker, and linter.

## 📦 Week 9 Project: OOP Framework

This week's project is to create a small, reusable framework that combines ABCs, Protocols, and Descriptors. You might design a simple plugin framework, a data validation library, or a basic event system, demonstrating your understanding of these advanced OOP concepts.

See `week-09-project.md` for a more detailed breakdown.

## ⚠️ Common Mistakes

- **State management**: Where do you store the actual value? In the descriptor or in the instance? Storing it on the descriptor makes it a class-level variable, shared by all instances! The common pattern is to store the actual data in the instance's `__dict__` with a "private" name (e.g., `_name`), and the descriptor just manages access to it.
- **`__set_name__`**: This method was added in Python 3.6. Before that, descriptors had to be instantiated with the attribute name, which was more verbose: `name = ValidatedAttribute("name")`. `__set_name__` makes them much cleaner.
- **Complexity**: Descriptors are powerful but can make code harder to follow than a simple `@property`. Use them when you have logic that you want to reuse across many attributes or many classes.

## 📖 Further Reading

- [Python Docs: Descriptor HowTo Guide](https://docs.python.org/3/howto/descriptor.html) (The essential guide)
- [Real Python: Python Descriptors](https://realpython.com/python-descriptors/)
- [Understanding Python Descriptors](https://blog.micheledes.com/2016/09/10/understanding-python-descriptors/)
