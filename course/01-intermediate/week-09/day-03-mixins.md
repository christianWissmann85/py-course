# Week 9, Day 3: Mixins & Multiple Inheritance

## 🎯 Learning Objectives
- [ ] Understand and use multiple inheritance to create classes that combine behaviors.
- [ ] Grasp the concept of Method Resolution Order (MRO) and how Python resolves method calls.
- [ ] Use the "mixin" pattern to add reusable, plug-in functionality to classes.
- [ ] Understand the "diamond problem" and how `super()` helps solve it.

## 📚 Concepts

### 1. Multiple Inheritance
A Python class can inherit from multiple parent classes. This allows a child class to inherit methods and attributes from all of them.

```python
class Logger:
    def log(self, message: str) -> None:
        print(f"LOG: {message}")

class Timestamped:
    def get_timestamp(self) -> str:
        from datetime import datetime
        return datetime.now().isoformat()

# This class inherits from BOTH Logger and Timestamped
class Event(Logger, Timestamped):
    def __init__(self, event_name: str):
        self.event_name = event_name

    def record(self) -> None:
        timestamp = self.get_timestamp() # Inherited from Timestamped
        self.log(f"Event '{self.event_name}' occurred at {timestamp}") # Inherited from Logger

my_event = Event("User Login")
my_event.record()
```
While powerful, multiple inheritance can make code complex and hard to reason about if overused.

### 2. Method Resolution Order (MRO)
When you call a method on an object, how does Python know where to find it among all the parent classes? It follows a specific order called the **Method Resolution Order (MRO)**.

The MRO for a class is a linearized list of all its base classes, from the class itself up to `object`. Python searches this list in order and uses the first implementation of the method it finds. You can inspect the MRO of any class using the `mro()` method or the `__mro__` attribute.

```python
print(Event.mro())
# Output might be:
# [<class '__main__.Event'>, <class '__main__.Logger'>, <class '__main__.Timestamped'>, <class 'object'>]
```
Python uses a sophisticated algorithm called C3 linearization to compute the MRO, which ensures a consistent and predictable order, even in complex hierarchies.

### 3. The Mixin Pattern
A **mixin** is a class that provides a specific, self-contained piece of functionality, but is not meant to be instantiated on its own. It's designed to be "mixed in" to other classes via multiple inheritance to add its behavior.

Mixins are a great way to practice **composition over inheritance**. Instead of a deep inheritance tree, you compose a class's functionality from smaller, reusable pieces.

```python
# A mixin for converting an object to a dictionary
class ToDictMixin:
    def to_dict(self) -> dict:
        # A simple implementation (might not work for complex objects)
        return self._asdict() if hasattr(self, '_asdict') else self.__dict__

# A regular data class
from dataclasses import dataclass

@dataclass
class Person(ToDictMixin): # Mix in the functionality
    name: str
    age: int

p = Person("Alice", 30)
# The to_dict method comes from the mixin!
print(p.to_dict()) # {'name': 'Alice', 'age': 30}
```
Mixin class names often end with `Mixin`.

### 4. `super()` and the Diamond Problem
The "diamond problem" is a classic issue in multiple inheritance. It occurs when a class inherits from two classes that both inherit from the same grandparent class.

```
      A
     / \
    B   C
     \ /
      D
```
If `D` calls a method that exists in `B` and `C`, which one should be called? And if `B` and `C` both call the method on `A` (the grandparent), does `A`'s method get called twice?

The combination of Python's MRO and `super()` solves this. `super()` does not necessarily call the "parent" class. Instead, it calls the **next class in the MRO**. This ensures that each class in the inheritance hierarchy is called exactly once, in the correct order.

```python
class A:
    def do_something(self): print("A")

class B(A):
    def do_something(self):
        print("B")
        super().do_something()

class C(A):
    def do_something(self):
        print("C")
        super().do_something()

class D(B, C):
    def do_something(self):
        print("D")
        super().do_something()

d = D()
d.do_something()
# MRO of D is [D, B, C, A, object]
# Output:
# D
# B
# C
# A
```

### 5. Best Practices
- **Favor Composition and Mixins**: Before creating a deep inheritance tree, ask if you can achieve the same result by "mixing in" functionality or by having one object contain another (composition).
- **Keep Mixins Small and Focused**: A mixin should do one thing well (e.g., logging, serialization).
- **Avoid Name Collisions**: Be careful that methods in different mixins don't have the same name, as one will override the other based on the MRO.
- **Use `super()` Consistently**: When using multiple inheritance, ensure all classes in the hierarchy use `super()` so that method calls are correctly forwarded along the MRO.

## 🔹 Quick Exercise

Create a `TimestampMixin` that adds a `created_at` timestamp to any class it's mixed into.

```python
import datetime

class TimestampMixin:
    def __init__(self) -> None:
        # Note: This __init__ will be called as part of the MRO.
        # It's important that it calls super().__init__() to continue the chain.
        self.created_at: datetime.datetime = datetime.datetime.now()
        super().__init__() # VERY IMPORTANT for cooperative multiple inheritance

class Document:
    def __init__(self, content: str) -> None:
        self.content = content
        super().__init__()

# Mix them together! The order matters for the MRO.
class SignedDocument(Document, TimestampMixin):
    def __init__(self, content: str, signature: str):
        self.signature = signature
        # This will call the __init__ methods of Document and TimestampMixin
        # in the correct MRO order.
        super().__init__(content=content)

# --- Tests ---
doc = SignedDocument("Hello World", "Jules")
assert hasattr(doc, "content")
assert hasattr(doc, "signature")
assert hasattr(doc, "created_at")
assert isinstance(doc.created_at, datetime.datetime)

print(f"Content: {doc.content}")
print(f"Signature: {doc.signature}")
print(f"Created At: {doc.created_at}")
```

## 📝 Daily Assignment
**Goal**: Build a small system of configurable components using mixins.

1.  **Create Project File**: In your project, create `my_first_poetry_app/mixins.py`.
2.  **Define Mixin Classes**:
    -   `JSONMixin`: Provides a `to_json(self) -> str` method that serializes the object's `__dict__` to a JSON string.
    -   `XMLMixin`: Provides a `to_xml(self) -> str` method that creates a simple XML representation of the object.
    -   `LoggerMixin`: Provides a `log(self, message)` method that prints a formatted log message, including the class name.
3.  **Define a Base Class**:
    -   Create a simple `Data` class with an `__init__` that accepts `**kwargs` and sets them as attributes.
4.  **Compose Concrete Classes**:
    -   Create a class `JSONLoggerData(JSONMixin, LoggerMixin, Data)` that can be logged and serialized to JSON.
    -   Create a class `XMLLoggerData(XMLMixin, LoggerMixin, Data)` that can be logged and serialized to XML.
5.  **`main()` function**:
    -   Create an instance of `JSONLoggerData` with some data. Call its `log()` and `to_json()` methods.
    -   Create an instance of `XMLLoggerData` with some data. Call its `log()` and `to_xml()` methods.
    -   Print the MRO for one of your composed classes to see the resolution order.
6.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes
- **Forgetting `super().__init__()`**: In a multiple inheritance scenario, if any class in the chain omits the `super().__init__()` call, it breaks the chain and the `__init__` methods of classes further up the MRO will not be called.
- **Creating a "god object"**: Multiple inheritance can be tempting to create a single class that does everything. This is an anti-pattern. Mixins should add focused, orthogonal (independent) functionality.
- **Not understanding the MRO**: The order in which you list parent classes matters. `class D(B, C)` has a different MRO from `class D(C, B)`, which can change which parent's method gets called first.
- **Using multiple inheritance when an ABC or Protocol is better**: If you just need to define a common interface, an ABC or Protocol is often a much cleaner and safer choice than using multiple inheritance as a substitute.

## 📖 Further Reading
- [Real Python: Multiple Inheritance and Mixins](https://realpython.com/python-multiple-inheritance/)
- [Raymond Hettinger: Super considered super!](https://rhettinger.wordpress.com/2011/05/26/super-considered-super/) (A classic talk explaining `super` and the MRO)
- [The Python `super()` Guide](https://www.python-course.eu/python3_super.php)
