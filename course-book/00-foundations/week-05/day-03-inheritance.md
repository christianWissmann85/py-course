# Week 5, Day 3: Inheritance

## 🎯 Learning Objectives

- [ ] Understand the concept of inheritance and create child classes that inherit from a parent class.
- [ ] Use `super()` to call methods from the parent class.
- [ ] Override parent class methods in a child class to provide specialized behavior.
- [ ] Correctly apply type hints in class hierarchies.

## 📚 Concepts

### 1. Inheritance Basics

Inheritance is a core OOP concept that allows a class (the **child** or **subclass**) to inherit attributes and methods from another class (the **parent** or **superclass**). This promotes code reuse and creates a logical hierarchy.

You define a child class by putting the parent class name in parentheses after the child class name.

```python
# Parent class
class Animal:
    def __init__(self, name: str):
        self.name = name

    def speak(self) -> str:
        return "Some generic animal sound"

# Child class inherits from Animal
class Dog(Animal):
    # Dog gets the __init__ and speak methods from Animal for free!
    pass

# Create instances
generic_animal = Animal("Creature")
my_dog = Dog("Fido")

print(generic_animal.speak()) # "Some generic animal sound"
print(my_dog.speak())       # "Some generic animal sound"
print(f"My dog's name is {my_dog.name}") # "My dog's name is Fido"
```

### 2. Method Overriding

A child class can provide its own specific implementation of a method that it inherited from a parent class. This is called **method overriding**.

```python
class Cat(Animal):
    # Override the speak method
    def speak(self) -> str:
        return "Meow"

my_cat = Cat("Whiskers")
print(my_cat.speak()) # "Meow"
```

### 3. Using `super()`

When you override a method, you might still want to execute the logic from the parent's version of that method. The `super()` function lets you call the parent class's method. This is most common in `__init__`.

```python
class Dog(Animal):
    def __init__(self, name: str, breed: str):
        # Call the parent's __init__ method to handle the `name` attribute.
        super().__init__(name)
        # Now, add the child's specific attribute.
        self.breed = breed

    # Override speak, but also include the parent's behavior
    def speak(self) -> str:
        parent_sound = super().speak()
        return f"The dog barks, but inside it's thinking: '{parent_sound}'"

my_dog = Dog("Buddy", "Golden Retriever")
print(f"Name: {my_dog.name}, Breed: {my_dog.breed}")
print(my_dog.speak())
```

### 4. Multiple Inheritance and MRO

A class can inherit from multiple parent classes. This is called **multiple inheritance**.

```python
class Flyer:
    def fly(self): print("Flying")

class Swimmer:
    def swim(self): print("Swimming")

class Duck(Flyer, Swimmer):
    pass

my_duck = Duck()
my_duck.fly()
my_duck.swim()
```

When a method is called, Python looks for it in the current class first, then in its parent classes in the order they are listed. This order is called the **Method Resolution Order (MRO)**. You can view a class's MRO with `ClassName.mro()`. Multiple inheritance can get complicated and should be used with care.

### 5. Typing Inheritance

When working with class hierarchies, you can use the parent class as a type hint. This allows you to write functions that can accept an instance of the parent class _or any of its child classes_.

```python
def make_animal_speak(animal: Animal) -> None:
    # This function accepts an Animal, Dog, or Cat
    print(f"{animal.name} says: {animal.speak()}")

my_dog = Dog("Rex", "German Shepherd")
my_cat = Cat("Fluffy")

make_animal_speak(my_dog) # OK
make_animal_speak(my_cat) # OK
```

This is a key principle of polymorphism in OOP.

## 🔹 Quick Exercise

Complete the `Dog` class below. It should inherit from `Animal` and override the `speak` method.

```python
class Animal:
    def __init__(self, name: str):
        self.name = name

    def speak(self) -> str:
        # This is a placeholder that should be overridden.
        # Raising NotImplementedError is a good practice for this.
        raise NotImplementedError("Subclasses must implement this method")

class Dog(Animal):
    # The __init__ is inherited automatically, no need to redefine it
    # if it's the same as the parent's.

    def speak(self) -> str:
        """Overrides the parent speak method."""
        # Return the sound a dog makes
        pass

# --- Tests ---
my_dog = Dog("Rex")
assert isinstance(my_dog, Animal)
assert my_dog.name == "Rex"
assert my_dog.speak() == "Woof!"

try:
    animal = Animal("Generic")
    animal.speak()
except NotImplementedError:
    print("Correctly raised NotImplementedError for base class.")

```

## 📝 Daily Assignment

**Goal**: Model a small hierarchy of e-commerce products using inheritance.

1.  **Create Project File**: In your `models.py` file, create a base class and several child classes.
2.  **Create a Base `Product` Class**:
    - Attributes: `product_id: str`, `name: str`, `price: float`.
    - Methods:
      - `__init__` to set the attributes.
      - `get_description(self) -> str`: Returns a generic description string.
3.  **Create Child Classes**:
    - `Book(Product)`:
      - Additional attribute: `author: str`.
      - Override `__init__`: Use `super()` to call the parent `__init__` and then set the `author`.
      - Override `get_description()`: Return a string that includes the author.
    - `Electronics(Product)`:
      - Additional attribute: `warranty_period_months: int`.
      - Override `__init__` and `get_description()` similarly.
    - `Clothing(Product)`:
      - Additional attributes: `size: str`, `color: str`.
      - Override `__init__` and `get_description()`.
4.  **Create a "Shopping Cart" Function**:
    - Write a standalone function `print_product_descriptions(products: list[Product]) -> None`.
    - This function should take a list of `Product` objects (which can include `Book`, `Electronics`, etc.).
    - It should loop through the list and print the result of each object's `get_description()` method.
5.  **`main()` function**:
    - Create a list containing instances of `Book`, `Electronics`, and `Clothing`.
    - Pass this list to your `print_product_descriptions` function to demonstrate that it works with all product types (polymorphism).
6.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes

- **Forgetting to call `super().__init__`**: If a child class defines its own `__init__` but doesn't call `super().__init__`, the parent's `__init__` is never run, and the object won't have the parent's attributes.
- **Incorrect `super()` signature**: In Python 3, `super()` with no arguments works correctly inside methods. In older code or complex scenarios, you might see `super(ClassName, self).__init__()`. Stick to the no-argument version.
- **Violating the Liskov Substitution Principle**: This principle states that objects of a superclass should be replaceable with objects of a subclass without affecting the correctness of the program. For example, if your overridden child method has a completely different signature or behavior, it can cause problems.
- **Overusing inheritance**: Inheritance creates a strong coupling between classes. Sometimes, **composition** (having an object as an attribute inside another object) is a more flexible design. Ask yourself "is it an 'is-a' relationship (inheritance) or a 'has-a' relationship (composition)?".

## 📖 Further Reading

- [Real Python: Inheritance and Composition](https://realpython.com/inheritance-composition-python/)
- [Python Docs: `super()`](https://docs.python.org/3/library/functions.html#super)
- [Wikipedia: Liskov Substitution Principle](https://en.wikipedia.org/wiki/Liskov_substitution_principle)
