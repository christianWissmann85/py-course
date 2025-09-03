# Week 5, Day 2: Properties & Encapsulation

## 🎯 Learning Objectives
- [ ] Understand the principle of encapsulation and its importance in OOP.
- [ ] Use Python's conventions for "private" attributes (`_` and `__`).
- [ ] Create read-only attributes using the `@property` decorator.
- [ ] Implement custom getter, setter, and deleter logic for class attributes.

## 📚 Concepts

### 1. Encapsulation Principles
Encapsulation is one of the fundamental concepts of Object-Oriented Programming (OOP). It refers to the bundling of data (attributes) with the methods that operate on that data. More importantly, it's about restricting direct access to an object's internal state.

**Why?**
- **Data Integrity**: Prevents external code from putting an object into an invalid or inconsistent state.
- **Flexibility**: The internal implementation of a class can change without affecting the code that uses it.
- **Simplicity**: Hides complexity. The user of a class only needs to know about its public interface, not its internal details.

### 2. "Private" Attributes in Python
Unlike languages like Java or C++, Python does not have a `private` keyword to enforce privacy. Instead, it relies on conventions.

- **Single Underscore (`_`)**: A name prefixed with a single underscore (e.g., `self._balance`) is a convention that tells other programmers, "This is for internal use only. Don't touch it directly from outside the class." It's a "gentleman's agreement" with no technical enforcement.

- **Double Underscore (`__`)**: A name prefixed with two underscores (e.g., `self.__id`) triggers **name mangling**. Python changes the name to `_ClassName__attribute_name`. This makes it much harder to access accidentally from outside the class and is primarily used to avoid naming conflicts in inheritance.

```python
class MyClass:
    def __init__(self):
        self.public = "I am public"
        self._internal = "I am for internal use"
        self.__mangled = "I am name-mangled"

instance = MyClass()
print(instance.public)      # OK
print(instance._internal)   # Works, but you shouldn't do it.
# print(instance.__mangled) # AttributeError!
print(instance._MyClass__mangled) # This is how you access it.
```

### 3. The `@property` Decorator: Getters
What if you want to expose an attribute as read-only, or run some code when an attribute is accessed? The `@property` decorator lets you define a method that acts like an attribute. This is the "getter".

```python
class Circle:
    def __init__(self, radius: float):
        if radius < 0:
            raise ValueError("Radius cannot be negative.")
        self._radius = radius # The "private" internal value

    @property
    def radius(self) -> float:
        """The radius of the circle. This is a read-only property."""
        print("Getting radius...")
        return self._radius

    @property
    def area(self) -> float:
        """A calculated, read-only property."""
        return 3.14159 * (self._radius ** 2)

c = Circle(10)
print(c.radius) # Looks like attribute access, but calls the radius() method.
print(c.area)   # Calls the area() method.
# c.radius = 12 # AttributeError: can't set attribute
```

### 4. Setters and Deleters
To allow changing a property, you define a **setter** method. You decorate a method with `@property_name.setter`.

```python
class Circle:
    # ... __init__ and radius property from before ...

    @radius.setter
    def radius(self, value: float) -> None:
        """The setter for the radius property. Allows validation."""
        print(f"Setting radius to {value}...")
        if value < 0:
            raise ValueError("Radius cannot be negative.")
        self._radius = value

c = Circle(10)
c.radius = 12 # This now calls the setter method!
print(c.radius) # 12
```
This pattern allows you to maintain a simple public interface (`c.radius`) while having complex validation and logic hidden inside the class. You can also define a `@radius.deleter` method.

### 5. Property vs. Method
When should you use a property versus a regular method?
- **Use a Property** if the access feels like getting a characteristic or attribute of the object. It should be fast and not have complex side effects. `circle.area` makes sense.
- **Use a Method** if you are performing an action. `circle.draw()` makes sense. `circle.get_area()` is less Pythonic than `circle.area`.

## 🔹 Quick Exercise

Complete the `Temperature` class below. It should store the temperature internally in Celsius but allow getting and setting it in either Celsius or Fahrenheit.

**Formulas**:
- Fahrenheit to Celsius: `(F - 32) * 5/9`
- Celsius to Fahrenheit: `(C * 9/5) + 32`

```python
class Temperature:
    def __init__(self, celsius: float):
        # We'll store the temperature in Celsius internally.
        self._celsius = celsius

    @property
    def celsius(self) -> float:
        """Returns the temperature in Celsius."""
        return self._celsius

    @celsius.setter
    def celsius(self, value: float) -> None:
        """Sets the temperature in Celsius."""
        self._celsius = value

    @property
    def fahrenheit(self) -> float:
        """Calculates and returns the temperature in Fahrenheit."""
        # Implement the C to F conversion formula here.
        pass

    @fahrenheit.setter
    def fahrenheit(self, value: float) -> None:
        """
        Takes a Fahrenheit value, converts it to Celsius,
        and stores it.
        """
        # Implement the F to C conversion and set self._celsius.
        pass

# --- Tests ---
t = Temperature(25)
assert t.celsius == 25
assert round(t.fahrenheit) == 77

t.fahrenheit = 32
assert t.celsius == 0
assert t.fahrenheit == 32
```

## 📝 Daily Assignment
**Goal**: Build a class that heavily uses properties and encapsulation to ensure its data is always valid.

1.  **Create Project File**: In your `models.py` file from yesterday (or a new file), define a class `Product`.
2.  **Define Attributes with Backing Fields**:
    - The `Product` should have `name` (a `str`) and `price` (a `float`).
    - In `__init__`, store these as `self._name` and `self._price`.
3.  **Implement Properties**:
    - Create a public `name` property.
        - The getter simply returns `self._name`.
        - The setter should validate that the new name is not an empty string. If it is, raise a `ValueError`.
    - Create a public `price` property.
        - The getter simply returns `self._price`.
        - The setter should validate that the price is a positive number. If not, raise a `ValueError`.
4.  **Implement a Read-Only Property**:
    - Add a `tax` property that is read-only (no setter).
    - It should calculate and return the tax on the product (e.g., 20% of the price).
5.  **`main()` function**:
    - Create an instance of your `Product`.
    - Demonstrate that the properties work:
        - Print the name, price, and tax.
        - Change the price and see the tax update automatically.
        - Try to set an invalid price (e.g., `-10`) or an empty name inside a `try...except` block to show that your validation works.
6.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes
- **Accessing the property inside its own getter/setter**: This will cause an infinite recursion.
  ```python
  @property
  def name(self) -> str:
      return self.name # Wrong! This calls the getter again.

  # Correct:
  @property
  def name(self) -> str:
      return self._name # Access the internal backing field.
  ```
- **Confusing properties with methods**: Remember to access a property without parentheses (`my_obj.area`), but call a method with them (`my_obj.get_area()`).
- **Thinking `__` makes an attribute truly private**: It doesn't. It's a mechanism to avoid name clashes, not a security feature. A determined programmer can still access a name-mangled attribute.

## 📖 Further Reading
- [Python Docs: `@property`](https://docs.python.org/3/library/functions.html#property)
- [Real Python: Python Properties](https://realpython.com/python-property/)
- [Python's "Private" Variables](https://www.python-engineer.com/posts/private-variables-in-python/)
