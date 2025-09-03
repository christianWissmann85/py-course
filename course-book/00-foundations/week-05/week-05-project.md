# Week 5 Project: OOP Task Manager

## 🎯 Project Objective

To build a simple but robust task management system using all the Object-Oriented Programming (OOP) concepts learned this week. You will design a system with multiple interacting classes, use inheritance to model different types of tasks, use dataclasses for your data models, and apply encapsulation principles to protect your data.

## 📋 Core Requirements

You will build a set of classes that can be used to create, manage, and track tasks.

### 1. Project Setup

- In your `my-first-poetry-app` project, create a new file: `my_first_poetry_app/task_manager.py`.
- You will need to import `dataclass` and other tools from `dataclasses` and `typing`.

### 2. The `Task` Hierarchy (Inheritance and Dataclasses)

- **`Task` (Base Class)**:
  - Create a base `Task` class. This should be a `@dataclass`.
  - Attributes: `task_id: str`, `description: str`, `_is_complete: bool = False`. Note the underscore to mark `_is_complete` for internal use.
  - Implement a public `@property` for `is_complete` to expose the value of `_is_complete`.
  - Implement a method `complete_task(self) -> None` that sets `_is_complete` to `True`.
- **`DeadlineTask(Task)` (Child Class)**:
  - Inherits from `Task`.
  - Should also be a `@dataclass`.
  - Adds a `due_date: date` attribute (use the `datetime` module).
  - Overrides the `__str__` method to include the due date in the task's string representation.
- **`Chore(Task)` (Child Class)**:
  - Inherits from `Task`.
  - Should also be a `@dataclass`.
  - Adds an `assignee: str` attribute.
  - Overrides the `__str__` method to include the assignee.

### 3. The `TaskManager` Class (Composition and Encapsulation)

This class will manage a collection of tasks.

- **`__init__(self)`**:
  - Should initialize a private `_tasks` dictionary: `self._tasks: dict[str, Task] = {}`. This is **composition**—the `TaskManager` _has a_ collection of tasks.
- **Methods**:
  - `add_task(self, task: Task) -> None`: Adds a task to the `_tasks` dictionary. The `task_id` should be the key. Prevent duplicate IDs.
  - `get_task(self, task_id: str) -> Task | None`: Retrieves a task by its ID.
  - `mark_complete(self, task_id: str) -> bool`: Finds a task by ID and calls its `complete_task()` method. Returns `True` on success, `False` if the task is not found.
  - `get_all_tasks(self) -> list[Task]`: Returns a list of all tasks.
  - `get_incomplete_tasks(self) -> list[Task]`: Returns a list of only the tasks that are not complete.

### 4. Type Safety & Code Quality

- The entire application must be fully type-annotated and pass `mypy --strict`.
- All classes and methods must have clear docstrings.
- The code must pass all `ruff check .` linter checks.

## ✅ Definition of Done

- [ ] A `task_manager.py` file exists with `Task`, `DeadlineTask`, `Chore`, and `TaskManager` classes.
- [ ] `Task` is a dataclass, and `DeadlineTask` and `Chore` correctly inherit from it and are also dataclasses.
- [ ] Encapsulation is used correctly (e.g., `_is_complete` property, `_tasks` dictionary).
- [ ] `TaskManager` correctly manages a collection of different task types.
- [ ] A `main` function exists that demonstrates creating a `TaskManager`, adding tasks of different types, marking some as complete, and listing all/incomplete tasks.
- [ ] The entire codebase is type-safe, documented, and passes the linter.

## 🏆 Bonus Challenges

- **`__repr__` for Dataclasses**: Dataclasses provide a default `__repr__`. For the child classes, can you make it even better?
- **Persistent Storage**: Create `save_to_file` and `load_from_file` methods on your `TaskManager`. This will require you to think about how to serialize/deserialize objects of different types. (Hint: you might need to store the task type in the data).
- **Generic Task Manager**: Can you make `TaskManager` generic so it could manage other types of items besides `Task`? What would need to change? (This is an advanced design question).

## 💡 Tips

- Start by designing the classes on paper. What data does each class hold? What actions can it perform?
- Build the `Task` hierarchy first. Make sure `DeadlineTask` and `Chore` work as expected before you start the `TaskManager`.
- The `TaskManager` is the orchestrator. Its methods will call the methods on the `Task` objects it contains.
- Use f-strings in your `__str__` and `__repr__` methods to create clean, informative representations of your objects.
