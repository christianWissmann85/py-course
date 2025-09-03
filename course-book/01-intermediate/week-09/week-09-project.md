# Week 9 Project: A Simple Plugin & Validation Framework

## 🎯 Project Objective
To build a small, reusable framework that demonstrates a deep understanding of the advanced Object-Oriented Programming concepts covered this week. You will create a system that uses a **metaclass** for automatic plugin registration, an **Abstract Base Class (ABC)** to define the core plugin interface, **descriptors** for data validation, and **protocols** for type-hinting callback functions.

This project will show how these advanced features work together to create powerful, flexible, and safe APIs.

## 📋 Core Requirements
You will build a mini-framework for processing "records" (dictionaries). The framework will allow users to define custom `Validator` plugins that are automatically registered and can be used to validate records.

### 1. Project Setup
-   Create a new project directory for this. `poetry new oop-framework`.
-   Structure your code within a `framework/` package.

### 2. The `Validator` ABC (The Interface)
-   In `framework/interfaces.py`, define an ABC named `Validator`.
-   It must have one abstract method: `validate(self, data: dict) -> bool`. This method should return `True` if the data is valid according to the validator's rules, and `False` otherwise.

### 3. The Plugin System (The Metaclass)
-   In `framework/registry.py`, create a `PLUGIN_REGISTRY` dictionary.
-   Create a `ValidatorMeta(type)` metaclass.
-   The metaclass's `__init__` method should check if the class being created has an attribute `plugin_name: str`.
-   If it does, it should register the class in `PLUGIN_REGISTRY` with `plugin_name` as the key.

### 4. Concrete Validator Plugins
-   In `framework/validators.py`, create a base `BaseValidator` class that inherits from `Validator` and uses `ValidatorMeta` as its metaclass.
-   Create at least two concrete validator plugins that inherit from `BaseValidator`:
    -   `class HasNameAndEmail(BaseValidator)`:
        -   `plugin_name = "has_name_email"`
        -   The `validate` method checks if the input `dict` has both a "name" and an "email" key.
    -   `class HasPositiveId(BaseValidator)`:
        -   `plugin_name = "has_positive_id"`
        -   The `validate` method checks if the dict has an "id" key and if its value is an integer greater than 0.

### 5. The `DataRecord` Class (Descriptors)
-   In `framework/models.py`, create a descriptor class `ValidatedString`.
    -   This descriptor should ensure that its value is always a string and is not empty.
-   Create a `DataRecord` class.
    -   It should use your `ValidatedString` descriptor for its `name` and `record_type` attributes.
    -   Its `__init__` should accept `name`, `record_type`, and a `data: dict`.

### 6. The `Callable` Protocol (Type Hinting Callbacks)
-   In `framework/interfaces.py`, define a `LogCallable` protocol.
-   `class LogCallable(Protocol): def __call__(self, message: str) -> None: ...`
-   This defines a "shape" for any function that can be used for logging.

### 7. The `Processor` (Bringing it all together)
-   In `framework/core.py`, create a `Processor` class.
-   Its `__init__` should accept a `logger: LogCallable`.
-   It should have a method `process_record(self, record: DataRecord, validator_name: str) -> bool`.
-   This method should:
    1.  Look up the validator from the global `PLUGIN_REGISTRY` using `validator_name`.
    2.  If the validator is not found, it should log an error using the logger and return `False`.
    3.  If found, it should create an instance of the validator and call its `validate()` method on the `record.data`.
    4.  It should log the result of the validation and return it.

## ✅ Definition of Done
- [ ] The project is structured as a proper package.
- [ ] An `ABC` is used to define the `Validator` interface.
- [ ] A `metaclass` is used to automatically register validator plugins.
- [ ] At least two concrete validator plugins are implemented.
- [ ] A `descriptor` is used for attribute validation in the `DataRecord` class.
- [ ] A `Protocol` is used to type-hint the logger callback function.
- [ ] The `Processor` class correctly uses the registry, validators, and logger to process data.
- [ ] A `main` script demonstrates the full workflow: creating a processor with a logging function, creating data records, and processing them with different validators.
- [ ] The entire codebase is type-safe and passes `mypy --strict`.

## 🏆 Bonus Challenges
-   **Metaclass with Parameters**: Modify your `PluginMeta` to be a factory that accepts arguments, e.g., `@register_plugin(name="my_plugin")`.
-   **Descriptor for Type Enforcement**: Create a `TypedField` descriptor that takes a type as an argument (e.g., `user_id = TypedField(int)`) and ensures that the attribute is always of that type.
-   **Make it a Real Framework**: Publish your mini-framework to TestPyPI so it could be installed and used by another project.

## 💡 Tips
-   **Build Incrementally**: Start with the `ABC`. Then implement the metaclass and registry. Test that plugins are being registered. Then build the `DataRecord` with its descriptor. Finally, build the `Processor` that ties it all together.
-   **Static vs. Runtime**: Pay attention to what happens when. The metaclass runs when a class is *defined*. The descriptor runs when an attribute is *accessed*. The ABC is enforced when a subclass is *instantiated*.
-   **Think about the "Why"**: As you build, ask yourself why each feature is necessary. Why use a metaclass instead of just manually adding to a dict? (Automation, less boilerplate). Why use an ABC instead of a Protocol? (To provide a common base class and shared code, if any).
