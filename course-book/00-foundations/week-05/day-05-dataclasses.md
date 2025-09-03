# Week 5, Day 5: Dataclasses

## 🎯 Learning Objectives

- [ ] Use the `@dataclass` decorator to reduce boilerplate code in classes that primarily store data.
- [ ] Understand the benefits of dataclasses, such as auto-generated special methods.
- [ ] Configure dataclass behavior using `field()` and decorator arguments.
- [ ] Create immutable data structures using frozen dataclasses.

## 📚 Concepts

### 1. The `@dataclass` Decorator

Often, you write classes that are primarily just containers for data. You have to write `__init__`, `__repr__`, `__eq__`, and other dunder methods over and over. The `@dataclass` decorator, introduced in Python 3.7, automates all of this for you.

You apply it to a class, and it automatically generates methods based on the type-annotated attributes you declare.

```python
from dataclasses import dataclass

# --- The old way ---
class PointOld:
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y

    def __repr__(self) -> str:
        return f"PointOld(x={self.x}, y={self.y})"

    def __eq__(self, other) -> bool:
        if not isinstance(other, PointOld):
            return NotImplemented
        return self.x == other.x and self.y == other.y

# --- The dataclass way ---
@dataclass
class Point:
    x: float
    y: float

# That's it! __init__, __repr__, and __eq__ are generated for you!
p1 = Point(10.5, 20.0)
p2 = Point(10.5, 20.0)

print(p1)       # Point(x=10.5, y=20.0)  <-- nice __repr__
print(p1 == p2) # True                   <-- useful __eq__
```

Dataclasses are still regular Python classes; the decorator just adds the boilerplate methods for you at compile time.

### 2. Field Configuration with `field()`

The `dataclasses.field()` function gives you more control over each attribute. A common use case is providing a mutable default value, like a list. You must use a `default_factory` to do this, which prevents the common mistake of sharing a single mutable default across all instances.

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Playlist:
    name: str
    owner: str
    # Use field() and default_factory for mutable defaults
    songs: List[str] = field(default_factory=list)

p1 = Playlist("Rock Hits", "Alice")
p1.songs.append("Bohemian Rhapsody")

p2 = Playlist("Indie Jams", "Bob")
# p2.songs is a new, empty list, not the same one as p1's.
print(p1) # Playlist(name='Rock Hits', owner='Alice', songs=['Bohemian Rhapsody'])
print(p2) # Playlist(name='Indie Jams', owner='Bob', songs=[])
```

You can also use `field()` to exclude an attribute from the `__init__` or `__repr__` methods.

### 3. Post-init Processing with `__post_init__`

Sometimes you need to run some logic after the `__init__` method has been called (e.g., to compute a value based on other attributes). Dataclasses provide a special method, `__post_init__`, for this purpose.

```python
@dataclass
class Circle:
    radius: float
    # Exclude 'area' from the initializer
    area: float = field(init=False)

    def __post_init__(self):
        # This runs after the main __init__
        if self.radius < 0:
            raise ValueError("Radius cannot be negative")
        self.area = 3.14159 * (self.radius ** 2)

c = Circle(10)
print(c) # Circle(radius=10, area=314.159)
```

### 4. Frozen Dataclasses (Immutability)

If you want to create an immutable data structure, you can set `frozen=True`. This makes instances of the dataclass read-only. Attempting to change an attribute after creation will raise a `FrozenInstanceError`.

Frozen dataclasses are also hashable, meaning you can use their instances as keys in a dictionary or items in a set.

```python
@dataclass(frozen=True)
class ImmutablePoint:
    x: float
    y: float

p = ImmutablePoint(1, 2)
# p.x = 5 # Raises dataclasses.FrozenInstanceError

# Because it's hashable, you can do this:
point_map = {p: "Origin"}
```

### 5. Comparison and Ordering

By default, `@dataclass` only generates `__eq__`. If you want ordering methods (`<`, `<=`, etc.), you can set `order=True`. The decorator will generate them for you, comparing the attributes in the order they are defined in the class.

```python
@dataclass(order=True)
class Product:
    # The sort_index will be used for comparison first
    sort_index: int = field(init=False, repr=False)
    name: str
    price: float

    def __post_init__(self):
        # We can set the sort_index here.
        # Let's say we want to sort primarily by price.
        self.sort_index = self.price

p1 = Product("Apple", 1.00)
p2 = Product("Banana", 0.50)

print(p1 > p2) # True, because p1's price (and thus sort_index) is higher
```

## 🔹 Quick Exercise

Complete the `Product` dataclass below. It should have a `name`, `price`, and a list of `tags` which defaults to an empty list.

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Product:
    name: str
    price: float
    # Use the field() function and a default_factory to handle the mutable list.
    tags: List[str] = field(default_factory=list)

# --- Tests ---
p1 = Product("Laptop", 1200.0)
p1.tags.append("electronics")
assert p1.tags == ["electronics"]

# Create another product to ensure tags list is not shared
p2 = Product("Book", 25.0)
assert p2.tags == []
```

## 📝 Daily Assignment

**Goal**: Refactor some of the classes you built earlier in the week to use dataclasses.

1.  **Create Project File**: In your `models.py` file, you'll be refactoring existing classes or creating new ones.
2.  **Refactor `Product` Hierarchy**:
    - Take the `Product`, `Book`, `Electronics`, and `Clothing` classes from Day 3's assignment.
    - Convert the base `Product` class into a dataclass.
    - Can you make the child classes dataclasses as well? How does inheritance work with dataclasses? Research and implement this.
3.  **Create a Data Model with All Features**:
    - Create a new dataclass `InventoryItem`.
    - It should have `name: str`, `unit_price: float`, and `quantity: int`.
    - It must be **immutable** (frozen).
    - It must be **orderable** (sortable). The primary sort key should be the `name`.
    - It should have a calculated `total_value` attribute that is computed in `__post_init__` but is not included in the `__repr__`.
4.  **`main()` function**:
    - Create instances of your refactored `Book` and `Electronics` classes.
    - Create a list of `InventoryItem` objects.
    - Print the list.
    - Use `.sort()` on the list and print it again to show that the ordering works.
5.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes

- **Using a mutable default without `default_factory`**: `tags: list = []` in a dataclass is a bug waiting to happen, as all instances will share the same list. Always use `field(default_factory=list)`.
- **Forgetting `init=False` for calculated fields**: If an attribute is calculated in `__post_init__`, it shouldn't be part of the `__init__` signature. You must mark it with `field(init=False)`.
- **Trying to modify a frozen instance**: Remember that `@dataclass(frozen=True)` means you cannot change attributes after creation.
- **Complex logic in `__post_init__`**: `__post_init__` is for simple initialization logic. If it becomes very complex, a custom `__init__` in a regular class might be a better choice. Dataclasses are best for data-heavy, logic-light classes.

## 📖 Further Reading

- [Python Docs: `dataclasses`](https://docs.python.org/3/library/dataclasses.html) (The official documentation is excellent)
- [Real Python: The Ultimate Guide to Python Dataclasses](https://realpython.com/python-dataclasses/)
- [PEP 557 -- Data Classes](https://peps.python.org/pep-0557/)
