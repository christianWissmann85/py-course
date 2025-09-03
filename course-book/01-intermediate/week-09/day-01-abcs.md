# Week 9, Day 1: Abstract Base Classes (ABCs)

## 🎯 Learning Objectives

- [ ] Understand the concept of an "interface" in programming.
- [ ] Use Python's `abc` module to create Abstract Base Classes (ABCs).
- [ ] Define abstract methods and properties that child classes must implement.
- [ ] Use ABCs to enforce a common API across multiple classes.

## 📚 Concepts

### 1. What are Interfaces and ABCs?

In object-oriented programming, an **interface** is a description of all the methods and attributes that an object must have to be considered of a certain type. It's like a contract. If a class promises to follow the "Drawable" interface, it must have a `draw()` method.

Python provides a mechanism to formalize this contract: **Abstract Base Classes (ABCs)**. An ABC is a class that cannot be instantiated on its own. Its purpose is to be inherited from and to define a common interface for its subclasses.

### 2. The `abc` Module and `@abstractmethod`

Python's built-in `abc` module provides the tools to create ABCs.

- **`ABC`**: A helper class to create an ABC by inheriting from it.
- **`@abstractmethod`**: A decorator to indicate that a method is abstract and must be implemented by any concrete (non-abstract) child class.

```python
from abc import ABC, abstractmethod

# This is an Abstract Base Class. You cannot create an instance of it.
class DataProcessor(ABC):

    @abstractmethod
    def read_data(self) -> str:
        """Read data from a source."""
        # An abstract method can have a body, but it's often just `pass`.
        pass

    @abstractmethod
    def process_data(self, data: str) -> str:
        """Process the data."""
        pass

# This is a concrete implementation of the ABC.
class TextFileProcessor(DataProcessor):
    def __init__(self, filepath: str):
        self.filepath = filepath

    # It MUST implement all abstract methods from the parent.
    def read_data(self) -> str:
        with open(self.filepath, 'r') as f:
            return f.read()

    def process_data(self, data: str) -> str:
        return data.upper()

# You can now instantiate the concrete class.
# text_processor = TextFileProcessor("my_file.txt")

# Trying to instantiate the ABC directly will raise a TypeError.
# invalid_processor = DataProcessor() # TypeError: Can't instantiate abstract class
```

If you create a subclass that fails to implement one of the abstract methods, you will get a `TypeError` when you try to instantiate it.

### 3. Abstract Properties

You can also define abstract properties. This forces subclasses to provide a property with a specific name.

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @property
    @abstractmethod
    def area(self) -> float:
        """The area of the shape."""
        pass

class Square(Shape):
    def __init__(self, side: float):
        self.side = side

    @property
    def area(self) -> float:
        # We provide a concrete implementation for the abstract property.
        return self.side ** 2
```

### 4. Interface Design with ABCs

ABCs are a powerful tool for designing robust systems. By defining an interface with an ABC, you can write functions that operate on any object that conforms to that interface, without needing to know the object's specific class.

```python
def report_data(processor: DataProcessor) -> None:
    """
    This function can take ANY object that is a subclass of DataProcessor.
    """
    print("--- Starting Data Report ---")
    data = processor.read_data()
    processed = processor.process_data(data)
    print("Processed Data:")
    print(processed)
    print("--- End of Report ---")

# We can pass any concrete implementation to it.
# text_proc = TextFileProcessor("my_file.txt")
# report_data(text_proc)

# If we had another implementation, it would also work:
# class WebProcessor(DataProcessor): ...
# web_proc = WebProcessor("http://example.com")
# report_data(web_proc)
```

### 5. When to Use ABCs

- When you want to enforce that a group of related classes all share a common set of methods (a common API).
- When you are designing a framework or library and want to provide a clear "plugin" architecture for users.
- When you want to ensure at development time (via type checkers) and at runtime that a class fulfills its contract.

Tomorrow, we will look at `Protocols`, which offer a more flexible, "duck-typed" way of defining interfaces.

## 🔹 Quick Exercise

Define an abstract base class `DataProcessor` with one abstract method, `process`. Then create a concrete implementation `UpperCaseProcessor` that inherits from it and implements the method.

```python
from abc import ABC, abstractmethod

class DataProcessor(ABC):
    @abstractmethod
    def process(self, data: str) -> str:
        """Process the data in some way."""
        pass

class UpperCaseProcessor(DataProcessor):
    def process(self, data: str) -> str:
        """Converts the data to uppercase."""
        return data.upper()

# --- Tests ---
processor = UpperCaseProcessor()
assert isinstance(processor, DataProcessor)
assert processor.process("hello") == "HELLO"

try:
    DataProcessor()
    assert False, "Should not be able to instantiate an ABC"
except TypeError:
    print("Correctly caught TypeError for ABC instantiation.")
```

## 📝 Daily Assignment

**Goal**: Design a set of interfaces for a media player application using ABCs.

1.  **Create Project File**: In your project, create a new file: `my_first_poetry_app/media.py`.
2.  **Define a `Playable` ABC**:
    - Create an ABC called `Playable`.
    - It should have two abstract methods:
      - `play(self) -> None`
      - `stop(self) -> None`
    - It should also have one abstract read-only property:
      - `title(self) -> str`
3.  **Create Concrete Classes**:
    - Create a `Song` class that inherits from `Playable`.
      - It should have `__init__` with attributes like `_title`, `artist`, `duration`.
      - Implement all the abstract methods and the property from `Playable`. The `play` method can just print a message like `"Playing song: {self.title} by {self.artist}"`.
    - Create a `Video` class that inherits from `Playable`.
      - It should have `__init__` with attributes like `_title`, `director`, `resolution`.
      - Implement all the abstract methods and the property.
4.  **Create a `MediaPlayer` class**:
    - This class does _not_ inherit from `Playable`.
    - It should have a method `play_media(self, item: Playable) -> None`.
    - This method takes any object that conforms to the `Playable` interface and calls its `play()` method.
5.  **`main()` function**:
    - Create instances of `Song` and `Video`.
    - Create an instance of `MediaPlayer`.
    - Use the media player to play both the song and the video to demonstrate polymorphism.
6.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes

- **Forgetting to inherit from `ABC`**: A class is only an ABC if it inherits from `ABC`.
- **Forgetting the `@abstractmethod` decorator**: If you define a method in an ABC but forget the decorator, subclasses will not be required to implement it, defeating the purpose.
- **Instantiating an ABC**: You cannot create a direct instance of a class that has abstract methods. You must create an instance of a concrete subclass that has implemented all of them.
- **Mismatching signatures**: When a subclass implements an abstract method, its signature should match the signature defined in the parent class (or be compatible with it). MyPy will help you catch these mismatches.

## 📖 Further Reading

- [Python Docs: `abc` — Abstract Base Classes](https://docs.python.org/3/library/abc.html)
- [Real Python: Abstract Base Classes in Python](https://realpython.com/python-interface/)
- [PEP 3119 -- Introducing Abstract Base Classes](https://peps.python.org/pep-3119/)
