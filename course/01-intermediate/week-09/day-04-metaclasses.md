# Week 9, Day 4: Metaclasses

## 🎯 Learning Objectives
- [ ] Understand that in Python, classes are themselves objects.
- [ ] Grasp the concept of a metaclass as the "class of a class".
- [ ] Create a custom metaclass by inheriting from `type`.
- [ ] Understand the use cases for metaclasses, such as creating Singletons or registering plugins.

**Heads-up**: Metaclasses are one of the most advanced and mind-bending topics in Python. Tim Peters, a Python core developer, famously said, "Metaclasses are deeper magic than 99% of users should ever worry about. If you wonder whether you need them, you don't." While true, understanding them gives you a profound insight into how Python's object model works.

## 📚 Concepts

### 1. Classes are Objects
In Python, everything is an object, and that includes classes. A class is an object that is an instance of another class.

```python
class MyClass:
    pass

print(type(MyClass)) # <class 'type'>
```
The class `MyClass` is an object, and its class is `type`. `type` is the default **metaclass** for all classes in Python.

### 2. `type` as a Metaclass
The `type` object can be used not only to get the type of an object, but also to create new classes dynamically. The call `type(name, bases, attrs)` creates a new class.
- `name`: The name of the new class.
- `bases`: A tuple of parent classes.
- `attrs`: A dictionary of attributes and methods for the new class.

```python
# This:
class MyClass:
    x = 10
    def greet(self):
        return "Hello"

# Is roughly equivalent to this:
def greet_func(self):
    return "Hello"

MyClassDynamic = type(
    'MyClassDynamic',
    (object,), # a tuple of base classes
    {'x': 10, 'greet': greet_func}
)

instance = MyClassDynamic()
print(instance.x)      # 10
print(instance.greet()) # Hello
```
Understanding that `type` is the default class factory is the key to understanding metaclasses.

### 3. Custom Metaclasses
A custom metaclass is a class that inherits from `type`. It allows you to intercept the creation of *other classes* and modify them.

To use a custom metaclass, you specify it with the `metaclass` keyword argument in a class definition.

```python
# 1. Define the metaclass
class MyMeta(type):
    # __new__ is the method that actually creates the new object (the class).
    def __new__(cls, name, bases, attrs):
        print(f"Creating class '{name}' with MyMeta")
        print(f"  Bases: {bases}")
        print(f"  Attributes: {attrs.keys()}")

        # You can modify the attributes here before the class is created
        attrs['__my_meta_attribute__'] = 100

        # Call the parent's __new__ to actually create the class
        return super().__new__(cls, name, bases, attrs)

# 2. Use the metaclass
class MyClassWithMeta(metaclass=MyMeta):
    x = 10
    def y(self): pass

# The print statements in MyMeta.__new__ run when the class is DEFINED,
# not when an instance is created.

print(MyClassWithMeta.__my_meta_attribute__) # 100
```

### 4. `__new__` vs. `__init__` in Metaclasses
When defining a metaclass, you will override `__new__` or `__init__`.
- **`__new__(cls, name, bases, attrs)`**: This is a class method that is called to **create** the new class object. It receives the metaclass (`cls`), and the name, bases, and attributes of the class being created. It must return the new class object. You modify the `attrs` dictionary here.
- **`__init__(cls, name, bases, attrs)`**: This is called **after** the class object has been created by `__new__`. It's used to **initialize** the newly created class. Here, `cls` is the new class object itself.

In general, use `__new__` if you need to modify the class attributes before the class is created. Use `__init__` if you just need to run some code after the class has been created (like registering the new class in a registry).

### 5. Real-World Use Cases
- **APIs and Frameworks**: Frameworks like Django and SQLAlchemy use metaclasses to create powerful, "magical" APIs. For example, when you define a Django Model, a metaclass inspects your class definition and automatically creates all the database fields and methods.
- **Plugin Registries**: A metaclass can automatically register any new class that uses it into a central registry.
- **Enforcing Coding Standards**: A metaclass can inspect a class's methods and attributes at creation time and raise an error if they don't conform to a certain standard (e.g., all methods must have docstrings).
- **Creating Design Patterns**: Implementing patterns like Singleton can be done cleanly with a metaclass.

## 🔹 Quick Exercise

Implement a `SingletonMeta` metaclass. The Singleton pattern ensures that a class has only one instance and provides a global point of access to it.

```python
class SingletonMeta(type):
    """
    A metaclass that creates a Singleton.
    """
    _instances = {} # A dictionary to store the single instance of each class

    def __call__(cls, *args, **kwargs):
        """
        This method is called when you do MyClass().
        If an instance of the class doesn't exist, create it.
        Otherwise, return the existing instance.
        """
        if cls not in cls._instances:
            # Use super().__call__ to call the original __new__ and __init__
            instance = super().__call__(*args, **kwargs)
            cls._instances[cls] = instance
        return cls._instances[cls]

class MySingletonClass(metaclass=SingletonMeta):
    def __init__(self):
        print("Creating a new instance!")

# --- Tests ---
s1 = MySingletonClass() # "Creating a new instance!" should be printed
s2 = MySingletonClass() # Nothing should be printed

assert id(s1) == id(s2)
assert s1 is s2
print("s1 and s2 are the same instance.")
```

## 📝 Daily Assignment
**Goal**: Create a metaclass that automatically registers classes into a plugin system.

1.  **Create Project File**: In your project, create `my_first_poetry_app/plugin_system.py`.
2.  **Create a Plugin Registry**:
    -   At the top level of your module, create an empty dictionary: `PLUGIN_REGISTRY = {}`.
3.  **Create the Metaclass**:
    -   Define a metaclass `PluginMeta(type)`.
    -   Implement the `__init__` method for this metaclass.
    -   Inside `__init__`, if the class being created has a special attribute `_plugin_name` (e.g., `_plugin_name = "my_plugin"`), the metaclass should add the class to the `PLUGIN_REGISTRY` dictionary, using the `_plugin_name` as the key.
4.  **Create a Base Class and Plugins**:
    -   Create a simple base class `PluginBase(metaclass=PluginMeta)`. All your plugins will inherit from this so they use the metaclass.
    -   Create at least two plugin classes that inherit from `PluginBase`:
        -   `class ImagePlugin(PluginBase): _plugin_name = "image_processor"`
        -   `class AudioPlugin(PluginBase): _plugin_name = "audio_processor"`
5.  **`main()` function**:
    -   After all classes have been defined, print the `PLUGIN_REGISTRY`.
    -   Verify that it contains both of your plugin classes, keyed by their `_plugin_name`. This demonstrates that the metaclass automatically registered them upon their creation.
6.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes
- **Overusing Metaclasses**: The biggest mistake is using them when a simpler solution (like a class decorator or a simple function) would suffice. Always ask if there's an easier way first.
- **Modifying `attrs` in `__init__`**: The class is already created by the time `__init__` is called. Modifying the `attrs` dictionary here will not affect the class. Modifications should happen in `__new__`.
- **Breaking the `super()` chain**: Forgetting to call `super().__new__(...)` in your `__new__` implementation means the class will never actually be created, leading to a `TypeError`.

## 📖 Further Reading
- [Real Python: Python Metaclasses](https://realpython.com/python-metaclasses/)
- [Stack Overflow: What is a metaclass in Python?](https://stackoverflow.com/questions/100003/what-is-a-metaclass-in-python) (A famous and very clear explanation)
- [The Magic of Metaclasses in Python](https://blog.ionelmc.ro/2015/02/09/understanding-python-metaclasses/)
