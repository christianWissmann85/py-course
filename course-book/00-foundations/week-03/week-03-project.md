# Week 3 Project: Data Structure Library

## 🎯 Project Objective

To deepen your understanding of how fundamental data structures work by implementing them from scratch. You will build your own simplified, generic, and type-safe versions of a List, Stack, and Queue. This project will force you to think about the underlying mechanics and performance characteristics of the tools you use every day.

## 📋 Core Requirements

You will create a Python module containing several custom data structure classes.

### 1. Project Setup

- In your `my-first-poetry-app` project, create a new file: `my_first_poetry_app/data_structures.py`.
- You will need `TypeVar` from the `typing` module to make your data structures generic.

### 2. Part 1: Custom Generic List (`MyList`)

Implement a class `MyList[T]` that wraps a standard Python `list` to provide a simplified interface.

- Use a `TypeVar('T')` to make it generic.
- The constructor `__init__(self)` should initialize an empty internal list (e.g., `self._items: list[T] = []`).
- Implement the following methods:
  - `push(self, item: T) -> None`: Adds an item to the end of the list (like `append`).
  - `pop(self) -> T`: Removes and returns the last item. Should raise an `IndexError` if the list is empty.
  - `get(self, index: int) -> T`: Returns the item at a given index. Should raise an `IndexError` for invalid indices.
  - `is_empty(self) -> bool`: Returns `True` if the list is empty.
  - `__len__(self) -> int`: The dunder method that allows `len(my_list_instance)` to work.
  - `__str__(self) -> str`: A string representation for printing.

### 3. Part 2: Custom Stack (`MyStack`)

Implement a `MyStack[T]` class that represents a Stack (Last-In, First-Out).

- You can (and should) use your `MyList` class internally to store the data. This is called **composition**.
- Implement the following methods:
  - `push(self, item: T) -> None`: Adds an item to the top of the stack.
  - `pop(self) -> T`: Removes and returns the top item from the stack.
  - `peek(self) -> T`: Returns the top item without removing it.
  - `is_empty(self) -> bool` and `__len__`.

### 4. Part 3: Custom Queue (`MyQueue`)

Implement a `MyQueue[T]` class that represents a Queue (First-In, First-Out).

- For the core requirement, you can use a Python `list` internally.
- Implement the following methods:
  - `enqueue(self, item: T) -> None`: Adds an item to the back of the queue.
  - `dequeue(self) -> T`: Removes and returns the front item from the queue.
  - `peek(self) -> T`: Returns the front item without removing it.
  - `is_empty(self) -> bool` and `__len__`.
- **Analysis**: Add a comment in your code explaining the time complexity of your `dequeue` operation. (Hint: removing from the front of a list is inefficient).

### 5. Type Safety & Code Quality

- Your entire module must pass `mypy --strict`.
- Your code must be fully documented with docstrings explaining what each class and method does.
- The code must be formatted and pass all `ruff check .` linter checks.

## ✅ Definition of Done

- [ ] A `data_structures.py` file exists with `MyList`, `MyStack`, and `MyQueue` classes.
- [ ] All classes are generic and fully type-annotated.
- [ ] All specified methods are implemented correctly.
- [ ] `MyStack` correctly uses `MyList` for its internal storage.
- [ ] The inefficiency of the list-based `MyQueue.dequeue` is explained in a comment.
- [ ] The code is fully documented with docstrings.
- [ ] `mypy --strict .` and `ruff check .` both pass without errors.

## 🏆 Bonus Challenges

- **Efficient Queue**: Create a new class `MyDequeQueue[T]` that implements the same Queue interface but uses `collections.deque` internally for efficient `O(1)` enqueues and dequeues.
- **Performance Test**: Write a `main` function that compares the performance of your `MyQueue` vs. `MyDequeQueue`. Enqueue 100,000 items and then dequeue all of them, timing how long the dequeue process takes for each implementation.
- **Make them Iterable**: Implement the `__iter__` method on your data structures so you can use them in a `for` loop.

## 💡 Tips

- Think about edge cases: What happens if you `pop` from an empty stack? What if you `get` an index that's out of bounds? Your code should handle this, usually by raising an `IndexError`.
- Composition is a powerful idea. Notice how you can build a `Stack` by _using_ a `List`, without having to rewrite all the list logic.
- When implementing the inefficient queue, `list.pop(0)` is the method you'll need for the dequeue operation.
