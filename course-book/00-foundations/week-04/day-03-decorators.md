# Week 4, Day 3: Decorators

## 🎯 Learning Objectives
- [ ] Understand that functions are first-class objects in Python and how this enables decorators.
- [ ] Create and apply simple custom decorators to add functionality to existing functions.
- [ ] Use `functools.wraps` to preserve the metadata of the original function.
- [ ] Learn how to correctly type decorators using `Callable` and `TypeVar`.

## 📚 Concepts

### 1. Functions are First-Class Objects
In Python, functions are "first-class citizens." This means you can treat them like any other object:
- Assign them to variables.
- Store them in data structures (lists, dicts).
- Pass them as arguments to other functions.
- Return them from other functions.

This is the fundamental concept that makes decorators possible.

### 2. Decorator Basics
A decorator is a function that takes another function as an argument, adds some functionality (or "decorates" it), and returns another function, all without altering the source code of the original function.

The `@` syntax is "syntactic sugar" for this process.

```python
# This is the decorator function
def my_decorator(func):
    # This is the new function that will replace the original
    def wrapper():
        print("Something is happening before the function is called.")
        func() # Call the original function
        print("Something is happening after the function is called.")
    return wrapper

# Using the decorator with @ syntax
@my_decorator
def say_hello():
    print("Hello!")

# Now, when we call say_hello, we are actually calling the wrapper.
say_hello()

# The @ syntax is equivalent to this:
# def say_hello():
#     print("Hello!")
# say_hello = my_decorator(say_hello)
```

### 3. Using `functools.wraps`
One problem with the simple decorator above is that it loses the metadata of the original function. The name, docstring, and annotations of `say_hello` are replaced by those of `wrapper`.

```python
# print(say_hello.__name__) # would print 'wrapper'
```

`functools.wraps` is a decorator that you apply to your `wrapper` function to fix this. It copies the metadata from the original function (`func`) to the wrapper.

```python
import functools

def better_decorator(func):
    @functools.wraps(func) # Use wraps here
    def wrapper(*args, **kwargs): # Use *args, **kwargs to be generic
        print("Before...")
        result = func(*args, **kwargs) # Pass arguments through
        print("...After")
        return result
    return wrapper

@better_decorator
def add(a: int, b: int) -> int:
    """This is the add function docstring."""
    return a + b

print(add.__name__)     # 'add'
print(add.__doc__)      # 'This is the add function docstring.'
```

### 4. Typing Decorators
Typing decorators can be tricky. The key is to use `TypeVar` and `Callable`. A `TypeVar` can be "bound" to `Callable` to represent any kind of function.

```python
from typing import Any, Callable, TypeVar

# F represents any function type.
# By binding it to Callable, we ensure it's a function.
F = TypeVar('F', bound=Callable[..., Any])

def timer_decorator(func: F) -> F:
    """A typed decorator that times a function's execution."""
    @functools.wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        import time
        start_time = time.perf_counter()
        value = func(*args, **kwargs)
        end_time = time.perf_counter()
        run_time = end_time - start_time
        print(f"Finished {func.__name__!r} in {run_time:.4f} secs")
        return value

    # The cast is sometimes needed to help MyPy understand
    # that the decorated wrapper has the same signature as the original.
    from typing import cast
    return cast(F, wrapper)
```
- `Callable[..., Any]` means a callable that takes any arguments (`...`) and returns any type (`Any`).
- The decorator takes a function of type `F` and returns a function of the same type `F`.

### 5. Decorators with Arguments (Decorator Factories)
What if you want to pass arguments to the decorator itself, like `@retry(attempts=3)`? You need another layer of function definition—a function that *returns* a decorator. This is called a decorator factory.

```python
def repeat(num_times: int):  # 1. The factory, takes decorator arguments
    def decorator_repeat(func): # 2. The actual decorator
        @functools.wraps(func)
        def wrapper(*args, **kwargs): # 3. The wrapper
            for _ in range(num_times):
                func(*args, **kwargs)
        return wrapper
    return decorator_repeat

@repeat(num_times=3)
def greet(name):
    print(f"Hello {name}")

greet("World") # Will print "Hello World" three times.
```

## 🔹 Quick Exercise

Complete the `timer` decorator below. It should measure the time a function takes to execute and print it.

```python
import time
import functools
from typing import Any, Callable, TypeVar

F = TypeVar('F', bound=Callable[..., Any])

def timer(func: F) -> F:
    """A decorator that prints the execution time of the decorated function."""
    @functools.wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        # 1. Record the start time (time.perf_counter())
        # 2. Call the original function with its arguments and store the result
        # 3. Record the end time
        # 4. Calculate and print the duration
        # 5. Return the result of the original function call
        pass # Implement here

    from typing import cast
    return cast(F, wrapper)


@timer
def slow_function(delay: float) -> None:
    """A sample function that sleeps for a given delay."""
    time.sleep(delay)

# This should run, print a time around 1.0 seconds, and then finish.
slow_function(1)
```

## 📝 Daily Assignment
**Goal**: Create a small library of useful, practical decorators.

1.  **Create Project File**: In `my-first-poetry-app`, create a new file: `my_first_poetry_app/decorators.py`.
2.  **Implement a Logging Decorator**:
    - Create a decorator `@log_activity`.
    - When a decorated function is called, it should log a message like `"Calling my_function(a=1, b=2)"`.
    - When the function finishes, it should log `"my_function returned: 3"`.
    - Use Python's built-in `logging` module for this.
3.  **Implement a Caching/Memoization Decorator**:
    - Create a decorator `@cache_result`.
    - This decorator should store the results of function calls in a dictionary.
    - When the decorated function is called with a set of arguments for the first time, it should compute the result and store it in the cache (e.g., `cache[(arg1, arg2)] = result`).
    - On subsequent calls with the same arguments, it should return the cached result directly without re-computing it.
    - Test this with a slow function (like a recursive Fibonacci) to see the performance improvement.
    - Note: Python's `functools` module has `@lru_cache` which does this professionally. Your goal is to implement the basic concept yourself.
4.  **Implement a Decorator Factory**:
    - Create a decorator factory `@require_permission(permission: str)`.
    - This decorator will wrap a function. The wrapper should check a "simulated" global set of user permissions.
    - `USER_PERMISSIONS: set[str] = {"read", "write"}`
    - If the required `permission` is in `USER_PERMISSIONS`, the function runs. Otherwise, it should raise a `PermissionError`.
5.  **`main()` function**:
    - Write a `main()` function that demonstrates all three of your decorators on sample functions.
6.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes
- **Forgetting `@functools.wraps`**: This is the most common mistake. It leads to confusing behavior when debugging or introspecting your code.
- **Not handling arguments correctly**: The `wrapper` function must accept `*args` and `**kwargs` and pass them to the original function to be a general-purpose decorator.
- **Incorrectly nesting for decorator factories**: A decorator with arguments requires three levels of functions. Getting the nesting or the return values wrong is easy to do.
- **Complex typing**: Decorator typing can get very complex with `ParamSpec` for forwarding argument types perfectly. For now, `TypeVar('F', bound=Callable)` is a great, robust starting point.

## 📖 Further Reading
- [Real Python: Python Decorators 101](https://realpython.com/primer-on-python-decorators/)
- [Python Docs: `functools.wraps`](https://docs.python.org/3/library/functools.html#functools.wraps)
- [PEP 318 -- Decorators for Functions and Methods](https://peps.python.org/pep-0318/) (The original proposal)
