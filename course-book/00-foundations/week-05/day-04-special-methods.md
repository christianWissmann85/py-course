# Week 5, Day 4: Special Methods

## 🎯 Learning Objectives

- [ ] Implement special methods (dunder methods) to integrate custom objects with Python's built-in syntax.
- [ ] Understand how protocols like comparison and arithmetic are implemented using dunder methods.
- [ ] Make your custom objects "Pythonic" by supporting natural operations like `+`, `==`, and `len()`.
- [ ] Correctly type special methods, including the use of `object` and `NotImplemented`.

## 📚 Concepts

Special methods, also called "dunder" (double underscore) methods, are the hooks that let your custom objects work with Python's built-in operators and syntax. By implementing them, you make your objects feel like native Python types.

### 1. String Representation (`__str__`, `__repr__`)

We covered these on Day 1, but they are the most common special methods. They allow `print()`, `str()`, and `repr()` to work on your objects.

### 2. Comparison Methods

These methods allow you to use comparison operators like `==`, `!=`, `<`, `>` on your objects.

- `__eq__(self, other)`: Implements the `==` (equality) operator.
- `__ne__(self, other)`: Implements the `!=` (not equal) operator.
- `__lt__(self, other)`: Implements the `<` (less than) operator.
- `__le__(self, other)`: Implements the `<=` (less than or equal to) operator.
- `__gt__(self, other)`: Implements the `>` (greater than) operator.
- `__ge__(self, other)`: Implements the `>=` (greater than or equal to) operator.

```python
class Person:
    def __init__(self, name: str, age: int):
        self.name = name
        self.age = age

    def __eq__(self, other: object) -> bool:
        # First, check if the other object is of the same type
        if not isinstance(other, Person):
            return NotImplemented # A special singleton to indicate the op is not implemented for this type
        # Then, compare the relevant attributes
        return self.age == other.age

p1 = Person("Alice", 30)
p2 = Person("Bob", 42)
p3 = Person("Charlie", 30)

print(p1 == p2) # False
print(p1 == p3) # True
```

**Note**: The type hint for `other` is `object` because you could be comparing your object to anything.

### 3. Arithmetic Operators

These methods let you use operators like `+`, `-`, `*` with your objects.

- `__add__(self, other)`: `+`
- `__sub__(self, other)`: `-`
- `__mul__(self, other)`: `*`
- `__truediv__(self, other)`: `/`

```python
class Money:
    def __init__(self, amount: float, currency: str):
        self.amount = amount
        self.currency = currency

    def __add__(self, other: 'Money') -> 'Money':
        if self.currency != other.currency:
            raise ValueError("Cannot add money of different currencies")
        new_amount = self.amount + other.amount
        return Money(new_amount, self.currency)

    def __repr__(self) -> str:
        return f"Money({self.amount}, '{self.currency}')"

wallet1 = Money(20, "USD")
wallet2 = Money(30, "USD")
total = wallet1 + wallet2 # This calls wallet1.__add__(wallet2)
print(total) # Money(50.0, 'USD')
```

### 4. Container Protocols

These methods make your object behave like a container (like a `list` or `dict`).

- `__len__(self)`: Lets `len()` work on your object.
- `__getitem__(self, key)`: Implements access with `[]` (e.g., `my_obj[5]`).
- `__setitem__(self, key, value)`: Implements assignment with `[]` (e.g., `my_obj[5] = 'x'`).
- `__delitem__(self, key)`: Implements deletion with `[]` (e.g., `del my_obj[5]`).
- `__contains__(self, item)`: Implements the `in` operator (e.g., `item in my_obj`).

### 5. Context Managers (`__enter__`, `__exit__`)

These methods allow your object to be used in a `with` statement, which is useful for managing resources like file handles or database connections.

- `__enter__(self)`: Called when entering the `with` block. Its return value is assigned to the `as` variable.
- `__exit__(self, exc_type, exc_value, traceback)`: Called when exiting the `with` block. It receives exception details if one occurred. It can handle the exception and suppress it if it returns `True`.

```python
class ManagedFile:
    def __init__(self, filename: str):
        self._filename = filename

    def __enter__(self):
        print("Entering context, opening file...")
        self._file = open(self._filename, 'w')
        return self._file

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self._file:
            print("Exiting context, closing file...")
            self._file.close()

with ManagedFile("hello.txt") as f:
    f.write("Hello from a context manager!")
```

## 🔹 Quick Exercise

Implement the `__add__` and `__eq__` special methods for the `Vector` class below. A vector is a mathematical object with a magnitude and direction, but here we'll represent it simply by its x and y components.

- Vector addition works by adding the corresponding components: `(x1, y1) + (x2, y2) = (x1+x2, y1+y2)`.
- Two vectors are equal if their x and y components are both equal.

```python
class Vector:
    """A simple 2D vector."""
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y

    def __repr__(self) -> str:
        return f"Vector({self.x}, {self.y})"

    def __add__(self, other: 'Vector') -> 'Vector':
        """Adds two vectors together."""
        # Check if other is a Vector instance
        # Return a new Vector with the added components
        pass

    def __eq__(self, other: object) -> bool:
        """Checks if two vectors are equal."""
        # Check if other is a Vector instance
        # Return True if both x and y components are equal
        pass

# --- Tests ---
v1 = Vector(2, 3)
v2 = Vector(3, 4)
v3 = Vector(5, 7)
v4 = Vector(2, 3)

assert v1 + v2 == v3
assert v1 == v4
assert v1 != v2
```

## 📝 Daily Assignment

**Goal**: Create a "pythonic" custom data type that feels like a built-in type by implementing several special methods.

1.  **Create Project File**: In your `models.py` file, create a `Deck` class that represents a deck of playing cards.
2.  **`Card` Class**: First, create a simple `Card` `NamedTuple` or `dataclass` with `rank: str` and `suit: str`.
3.  **`Deck` Class**:
    - `__init__(self)`: Should create a standard 52-card deck. A list of `Card` objects is a good internal representation. The suits are Spades, Hearts, Diamonds, Clubs. The ranks are 2-10, Jack, Queen, King, Ace.
    - `__len__(self)`: Should return the number of cards left in the deck.
    - `__str__(self)`: Should return a user-friendly string like `"Deck of 52 cards"`.
    - `__getitem__(self, position)`: Should allow accessing a card by its index, e.g., `deck[0]`.
4.  **Add Methods**:
    - `shuffle(self) -> None`: Should randomly shuffle the cards in the deck. Use the `random.shuffle()` function.
    - `deal(self) -> Card`: Should remove and return the top card from the deck.
5.  **`main()` function**:
    - Create a new `Deck`.
    - Print the deck (uses `__str__`).
    - Print its length (uses `__len__`).
    - Get the top card using `deck[0]` (uses `__getitem__`).
    - Shuffle the deck.
    - Get the new top card and show that it's different.
    - Deal 5 cards and print the new length of the deck.
6.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes

- **Incorrect `__eq__` implementation**: Forgetting to check `isinstance` first can lead to `AttributeError` if you try to compare your object to something of a different type. Returning `NotImplemented` is the correct way to handle incompatible types.
- **Modifying `self` in arithmetic methods**: Methods like `__add__` should almost always return a _new_ instance of the class, not modify `self` in place. (Methods for in-place operators like `+=` are different: `__iadd__`).
- **`__str__` vs `__repr__` confusion**: A good rule of thumb: `__repr__` is for developers (unambiguous), `__str__` is for users (readable). If you only define one, define `__repr__`.
- **Returning the wrong type**: `__len__` must return an `int`. `__eq__` must return a `bool`. `__str__` must return a `str`. MyPy will help catch these errors.

## 📖 Further Reading

- [A Guide to Python's Magic Methods](https://rszalski.github.io/magicmethods/) (A comprehensive and well-loved guide)
- [Real Python: Python's `__repr__` vs `__str__`](https://realpython.com/python-repr-str/)
- [Python Docs: The Data Model](https://docs.python.org/3/reference/datamodel.html) (The official, technical reference for all special methods)
