# Week 9, Day 2: Protocols & Structural Subtyping

## 🎯 Learning Objectives
- [ ] Understand the concept of "duck typing" and structural subtyping.
- [ ] Define interfaces implicitly using `typing.Protocol`.
- [ ] Use `@runtime_checkable` to verify protocol conformance at runtime.
- [ ] Compare and contrast Protocols with Abstract Base Classes (ABCs).

## 📚 Concepts

### 1. "Duck Typing" and Structural Subtyping
In Python, there is a common saying: "If it walks like a duck and it quacks like a duck, then it must be a duck." This is known as **duck typing**. It means that the type of an object is determined by its behavior (the methods and attributes it has), not by its explicit type or inheritance.

**Structural Subtyping** is the formal, static type checking version of this idea. A type `A` is a structural subtype of `B` if it has all the methods and attributes that `B` has, with matching type signatures. `A` doesn't need to inherit from `B`.

### 2. `typing.Protocol`
`Protocol` (introduced in PEP 544) allows you to define formal interfaces for structural subtyping. A class does not need to inherit from a protocol to be considered a subtype of it; it just needs to have the right "shape".

```python
from typing import Protocol

# Define an interface using Protocol
class Drawable(Protocol):
    def draw(self) -> None:
        """A method to draw the object."""
        ... # The body of a protocol method can be ... or pass

# This class implicitly conforms to the Drawable protocol
# because it has a `draw()` method with the correct signature.
# It does NOT inherit from Drawable.
class Circle:
    def draw(self) -> None:
        print("Drawing a circle.")

class Square:
    def draw(self) -> None:
        print("Drawing a square.")

class Triangle:
    # This class does NOT conform because the method name is wrong.
    def draw_triangle(self) -> None:
        print("Drawing a triangle.")

def render_scene(items: list[Drawable]) -> None:
    """This function can take a list of ANY object that is 'drawable'."""
    for item in items:
        item.draw()

# MyPy will understand that Circle and Square are valid Drawables.
render_scene([Circle(), Square()])

# MyPy will raise an error for this line because Triangle doesn't match the protocol.
# render_scene([Triangle()])
```
This is extremely flexible. It allows you to define an interface that can apply to classes from third-party libraries that you cannot modify.

### 3. `@runtime_checkable`
By default, you cannot use `isinstance()` with a plain `Protocol`.
`isinstance(Circle(), Drawable)` would raise a `TypeError`.

To enable runtime checks, you can decorate your protocol with `@runtime_checkable`.

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class SupportsClose(Protocol):
    def close(self) -> None: ...

class MyFile:
    def close(self) -> None:
        print("Closing file.")

class MySocket:
    def close(self) -> None:
        print("Closing socket.")

print(isinstance(MyFile(), SupportsClose))   # True
print(isinstance(MySocket(), SupportsClose)) # True
print(isinstance("hello", SupportsClose))  # False
```
This can be useful, but the primary power of Protocols is for static type checking.

### 4. Protocol vs. ABC: When to Use Which?
This is a key architectural decision.

- **Use an ABC (Nominal Subtyping)**:
  - When you want to provide a strong, explicit relationship between classes (`isinstance` works by default).
  - When you want to provide common, shared code or default implementations in the base class.
  - When you are building a framework and want users to explicitly subclass from your base class (e.g., `django.views.View`).
  - This is often called "is-a" a relationship. A `Dog` *is an* `Animal`.

- **Use a Protocol (Structural Subtyping)**:
  - When you want to define an interface that can be implemented by any class, including those you don't control.
  - When you want to decouple your code; the function cares only that the object has the right methods, not where it came from.
  - When you want to avoid a complex inheritance hierarchy.
  - This is often called a "has-a" or "behaves-like-a" relationship. A `File` object and a `Socket` object are not related by inheritance, but they both *behave like* a `Closable` thing.

## 🔹 Quick Exercise

Define a `Drawable` protocol and a few classes. One class should conform to it, and another should not. Test that a function typed with the protocol accepts the conforming class.

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None:
        ...

    @property
    def color(self) -> str:
        ...

class Circle:
    def __init__(self, color: str):
        self._color = color

    def draw(self) -> None:
        print(f"Drawing a {self.color} circle.")

    @property
    def color(self) -> str:
        return self._color

class TextBlock:
    def __init__(self, text: str):
        self.text = text
    # This class is NOT drawable

def draw_item(item: Drawable) -> None:
    item.draw()

# --- Tests ---
my_circle = Circle("blue")
my_text = TextBlock("hello")

# This should pass static type checking
draw_item(my_circle)

# This should be flagged as an error by MyPy
# draw_item(my_text)
```

## 📝 Daily Assignment
**Goal**: Refactor the media player from yesterday's assignment to use Protocols instead of ABCs.

1.  **Create Project File**: In your project, create `my_first_poetry_app/media_protocols.py`.
2.  **Define a `Playable` Protocol**:
    -   Create a `Playable` protocol.
    -   It should define the same interface as yesterday: a `play()` method, a `stop()` method, and a `title` read-only property.
3.  **Create Independent Classes**:
    -   Create a `Song` class. **It should NOT inherit from `Playable`**. It should just be a plain class that happens to have the correct methods and properties (`play`, `stop`, `title`).
    -   Create a `Video` class, also as a plain class that happens to conform to the `Playable` protocol's structure.
    -   Create a `Podcast` class that also conforms.
4.  **Create a `MediaPlayer` class**:
    -   This is the same as yesterday. It should have a method `play_media(self, item: Playable) -> None`.
5.  **`main()` function**:
    -   Create instances of `Song`, `Video`, and `Podcast`.
    -   Create a list of these media items.
    -   Create an instance of `MediaPlayer`.
    -   Pass each item to the media player's `play_media` method.
6.  **Analysis and Verification**:
    -   Run `mypy`. It should pass without errors, demonstrating that MyPy understands that your `Song`, `Video`, and `Podcast` classes structurally satisfy the `Playable` protocol, even without inheritance.
    -   In a comment in your code, write a few sentences explaining why a `Protocol` might be a better choice for this `Playable` interface than an ABC.

## ⚠️ Common Mistakes
- **Inheriting from `Protocol` when you don't need to**: While you can inherit from a protocol, the main point is that you don't have to. A class "implements" a protocol just by having the right structure.
- **Forgetting `@runtime_checkable`**: If you need to use `isinstance()` with a protocol, you must add the `@runtime_checkable` decorator. Otherwise, it will raise a `TypeError`.
- **Mismatching method signatures**: For a class to conform to a protocol, its methods must have type-compatible signatures. If the protocol defines `process(self, data: str)` and your class has `process(self, data: int)`, it does not conform.
- **Thinking Protocols have runtime impact**: Without `@runtime_checkable`, protocols are purely a tool for static type checkers like MyPy. They have no effect on how your code actually runs.

## 📖 Further Reading
- [PEP 544 -- Protocols: Structural subtyping (static duck typing)](https://peps.python.org/pep-0544/)
- [Real Python: Python Type Checking (Protocols section)](https://realpython.com/python-type-checking/#duck-types-and-protocols)
- [MyPy Docs: Protocols and structural subtyping](https://mypy.readthedocs.io/en/stable/protocols.html)
