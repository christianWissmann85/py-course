# Week 5, Day 1: Classes & Objects

## 🎯 Learning Objectives
- [ ] Understand the relationship between classes (blueprints) and objects (instances).
- [ ] Define custom classes using the `class` keyword.
- [ ] Use the `__init__` method to initialize the state of an object.
- [ ] Differentiate between instance attributes and class attributes and type them correctly.

## 📚 Concepts

### 1. Class Definition
A **class** is a blueprint for creating objects. An **object** (or **instance**) is a specific realization of a class. For example, `str` is a class, and `"hello"` is an object (an instance of the `str` class).

Object-Oriented Programming (OOP) is a paradigm based on this concept. It allows us to bundle data (attributes) and the functions that operate on that data (methods) together into a single unit.

You define a class using the `class` keyword. By convention, class names use `PascalCase` (or `CapWords`).

```python
class MyFirstClass:
    pass

# Create an instance of the class
instance1 = MyFirstClass()
instance2 = MyFirstClass()

print(type(instance1)) # <class '__main__.MyFirstClass'>
```

### 2. The `__init__` Method and `self`
The `__init__` method is a special "dunder" (double underscore) method that acts as the **initializer** for the class. It's called automatically when you create a new instance of the class. Its job is to set up the initial state (the attributes) of the object.

The first parameter of any instance method, including `__init__`, is always a reference to the instance itself. By convention, this parameter is always named `self`.

```python
class Car:
    # The initializer method
    def __init__(self, make: str, model: str, year: int) -> None:
        print(f"Creating a {year} {make} {model}...")
        # These are instance attributes
        self.make = make
        self.model = model
        self.year = year
        self.is_running = False

# When we call Car(...), Python calls __init__ behind the scenes.
# my_car is passed as the `self` argument.
my_car = Car("Toyota", "Corolla", 2021)
```

### 3. Instance vs. Class Attributes
- **Instance Attributes**: These belong to a specific instance of a class. They are defined inside `__init__` using `self.attribute_name = value`. Each instance has its own copy of these attributes. In the `Car` example, `make`, `model`, `year`, and `is_running` are all instance attributes.

- **Class Attributes**: These are shared by *all* instances of a class. They are defined directly inside the class, outside of any method.

```python
class Dog:
    # This is a class attribute, shared by all dogs.
    species: str = "Canis lupus familiaris"

    def __init__(self, name: str, age: int):
        # These are instance attributes.
        self.name = name
        self.age = age

dog1 = Dog("Fido", 5)
dog2 = Dog("Buddy", 3)

# Both instances share the same class attribute
print(f"{dog1.name}'s species: {dog1.species}") # Canis lupus familiaris
print(f"{dog2.name}'s species: {dog2.species}") # Canis lupus familiaris

# But they have different instance attributes
print(f"{dog1.name} is {dog1.age} years old.") # Fido is 5 years old.
print(f"{dog2.name} is {dog2.age} years old.") # Buddy is 3 years old.
```

### 4. Methods with Types
A **method** is a function that is defined inside a class. It operates on the data of an instance. The first argument is always `self`.

```python
class Car:
    def __init__(self, make: str, model: str, year: int) -> None:
        self.make = make
        self.model = model
        self.year = year
        self.is_running = False

    # This is an instance method
    def start_engine(self) -> None:
        if not self.is_running:
            self.is_running = True
            print("Engine started.")
        else:
            print("Engine is already running.")

    def stop_engine(self) -> None:
        if self.is_running:
            self.is_running = False
            print("Engine stopped.")
        else:
            print("Engine is already off.")
```

### 5. String Representation: `__str__` and `__repr__`
What happens when you `print()` an object? By default, you get a useless representation like `<__main__.Car object at 0x...>`. You can provide a better string representation by defining dunder methods:
- `__str__(self) -> str`: Called by `print()` and `str()`. Should return a user-friendly string.
- `__repr__(self) -> str`: Called when the object is inspected in the console. Should return an unambiguous, official string representation of the object, ideally one that can be used to recreate the object (`eval(repr(obj)) == obj`).

If `__str__` is not defined, `__repr__` is used as a fallback. It's good practice to define at least `__repr__`.

```python
class Car:
    # ... (init from before) ...

    def __repr__(self) -> str:
        return f"Car(make='{self.make}', model='{self.model}', year={self.year})"

    def __str__(self) -> str:
        status = "running" if self.is_running else "off"
        return f"A {self.year} {self.make} {self.model}. The engine is {status}."

my_car = Car("Toyota", "Corolla", 2021)
print(my_car)         # Calls __str__
print(repr(my_car))   # Calls __repr__
```

## 🔹 Quick Exercise

Complete the `BankAccount` class below by implementing the `__init__`, `deposit`, and `withdraw` methods.

```python
class BankAccount:
    """A simple, type-safe bank account class."""

    def __init__(self, owner: str, balance: float = 0.0) -> None:
        """Initializes a bank account with an owner and an optional starting balance."""
        # Assign owner and balance to instance attributes
        pass

    def deposit(self, amount: float) -> None:
        """
        Deposits a positive amount into the account.
        Raises a ValueError if the amount is not positive.
        """
        # Check if amount is positive, then add to balance
        pass

    def withdraw(self, amount: float) -> None:
        """
        Withdraws a positive amount from the account.
        Raises a ValueError if the amount is not positive or if funds are insufficient.
        """
        # Check for positive amount and sufficient funds, then subtract from balance
        pass

    def __str__(self) -> str:
        return f"Account Owner: {self.owner}, Balance: ${self.balance:.2f}"

# --- Tests ---
account = BankAccount("John Doe", 100.0)
assert account.balance == 100.0
account.deposit(50.0)
assert account.balance == 150.0
account.withdraw(75.0)
assert account.balance == 75.0
print(account) # Account Owner: John Doe, Balance: $75.00
```

## 📝 Daily Assignment
**Goal**: Practice modeling real-world concepts by creating several distinct classes.

1.  **Create Project File**: In `my-first-poetry-app`, create a new file: `my_first_poetry_app/models.py`.
2.  **Create 5 Classes**: Define at least 5 different classes that could be used in an application. Each class must have:
    - An `__init__` method with fully typed arguments.
    - At least 3 instance attributes.
    - At least 1 class attribute.
    - At least 2 instance methods (besides `__init__`).
    - A useful `__str__` and `__repr__` method.
3.  **Class Ideas**:
    - `Book` (attributes: `title`, `author`, `isbn`, `page_count`)
    - `Playlist` (attributes: `name`, `creator`, `songs` (a list))
    - `InventoryItem` (attributes: `name`, `price`, `quantity_on_hand`)
    - `Recipe` (attributes: `name`, `ingredients` (a dict), `cook_time_minutes`)
    - `Vector2D` (attributes: `x`, `y`)
4.  **`main()` function**:
    - In a `main` function, create at least two instances of each of your classes.
    - Call the methods on your instances and print the results to demonstrate they work.
5.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes
- **Forgetting `self`**: Forgetting to include `self` as the first parameter of an instance method is a very common error for beginners. It results in a `TypeError` about a missing positional argument.
- **Confusing class and instance attributes**: Using `self.species = "..."` inside `__init__` creates an instance attribute that shadows the class attribute. Use `ClassName.species` to refer to the class attribute if you need to.
- **Calling methods from `__init__`**: It's generally safe, but be careful when calling methods that might be overridden in subclasses (a topic for Day 3).
- **Not defining `__repr__`**: A class without a `__repr__` is harder to debug. It's a small effort that pays off significantly.

## 📖 Further Reading
- [Python Docs: Classes](https://docs.python.org/3/tutorial/classes.html) (The official tutorial)
- [Real Python: Object-Oriented Programming (OOP) in Python 3](https://realpython.com/python3-object-oriented-programming/)
