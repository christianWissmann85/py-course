# Week 10, Day 5: Recursion & Tail Call Optimization

## 🎯 Learning Objectives
- [ ] Understand and implement recursive functions to solve problems.
- [ ] Identify the base case and recursive step in a recursive algorithm.
- [ ] Understand the concept of the call stack and the risk of stack overflow errors.
- [ ] Learn about tail call optimization and why Python does not implement it.

## 📚 Concepts

### 1. Recursion Basics
A **recursive function** is a function that calls itself. It's a powerful way to solve problems that can be broken down into smaller, self-similar sub-problems.

Every recursive function must have two parts:
1.  **Base Case**: A condition that stops the recursion. Without a base case, the function would call itself forever.
2.  **Recursive Step**: The part of the function where it calls itself, but with an input that moves it closer to the base case.

```python
def factorial(n: int) -> int:
    # Base Case: factorial of 0 or 1 is 1
    if n <= 1:
        return 1
    # Recursive Step: n * factorial of (n-1)
    else:
        return n * factorial(n - 1)

print(factorial(5)) # 120 (which is 5 * 4 * 3 * 2 * 1)
```
Each call to `factorial` adds a new frame to the **call stack**. `factorial(5)` calls `factorial(4)`, which calls `factorial(3)`, and so on. When `factorial(1)` returns, the frames are unwound one by one.

### 2. Tail Recursion
A recursive function is **tail-recursive** if the recursive call is the very last thing the function does. There is no computation performed on the result of the recursive call.

```python
# NOT tail-recursive: it does a multiplication AFTER the recursive call
def factorial(n: int) -> int:
    if n <= 1: return 1
    return n * factorial(n - 1) # <--- n * ... happens after

# IS tail-recursive: the recursive call is the last action
def factorial_tail(n: int, accumulator: int = 1) -> int:
    if n <= 1: return accumulator
    return factorial_tail(n - 1, n * accumulator) # <--- last thing is the call
```

### 3. Python's Limitation: No Tail Call Optimization (TCO)
Some programming languages perform **Tail Call Optimization (TCO)**. If a function is tail-recursive, the compiler can optimize it to not create a new stack frame for the recursive call. It essentially turns the recursion into an efficient loop, preventing stack overflows.

**Python does not have TCO.** Guido van Rossum, the creator of Python, has provided several reasons, primarily that it would make debugging and stack traces more confusing. This means that even if you write a perfectly tail-recursive function in Python, it can still cause a `RecursionError: maximum recursion depth exceeded` if it runs too many times.

```python
# This will fail in Python, even though it's tail-recursive
# factorial_tail(3000) # RecursionError
```

### 4. The Trampoline Pattern
To get around Python's lack of TCO, you can use a design pattern called a **trampoline**. A trampoline is a loop that iteratively runs a function. Instead of making a deep recursive call, the function returns another function to be called next. This keeps the call stack flat.

```python
from typing import Callable, Any

def trampoline(func: Callable) -> Callable:
    """A decorator to simulate tail-call optimization."""
    def wrapper(*args, **kwargs):
        f = func(*args, **kwargs)
        # Keep calling the returned function as long as it's a callable
        while callable(f):
            f = f()
        return f
    return wrapper

# Now we write our function to return another function (a "thunk")
def factorial_tramp(n: int, acc: int = 1):
    if n <= 1:
        return acc
    # Instead of calling itself, it returns a function that will make the next call
    return lambda: factorial_tramp(n - 1, acc * n)

@trampoline
def factorial_safe(n: int, acc: int = 1):
    return factorial_tramp(n, acc)

# This will now work without a RecursionError, albeit more slowly.
# print(factorial_safe(3000))
```
This is an advanced pattern and rarely needed in practice, but it's a fascinating solution to the problem.

### 5. When to Use Recursion in Python
Given the lack of TCO, when should you use recursion?
- **For naturally recursive data structures**: Processing trees or nested data structures is often much cleaner with recursion.
- **For "divide and conquer" algorithms**: Algorithms like Merge Sort or Quick Sort are elegantly expressed recursively.
- **When the recursion depth is guaranteed to be small**: If you know you'll only ever recurse a few dozen or hundred times, the risk of stack overflow is negligible and the cleaner code may be worth it.

For simple linear iteration, a `for` or `while` loop is almost always more efficient and "Pythonic" than recursion.

## 🔹 Quick Exercise

The `trampoline` decorator from the concepts section is provided. Your task is to write the `factorial_thunk` function that works with it. A "thunk" is a function that wraps an expression to delay its evaluation. In our case, it's the function returned by the recursive step.

```python
from typing import Callable, Any

def trampoline(func: Callable) -> Callable:
    def wrapper(*args, **kwargs):
        f = func(*args, **kwargs)
        while callable(f):
            f = f()
        return f
    return wrapper

def factorial_thunk(n: int, accumulator: int = 1) -> Any:
    """
    Returns the final value if at the base case, otherwise
    returns a 'thunk' (a zero-argument lambda) that continues the recursion.
    """
    if n <= 1:
        return accumulator
    else:
        return lambda: factorial_thunk(n - 1, accumulator * n)

@trampoline
def factorial(n: int) -> int:
    return factorial_thunk(n, 1)

# --- Tests ---
assert factorial(5) == 120
# This would fail with regular recursion, but should work with the trampoline.
# Note: This might be slow to run.
# assert isinstance(factorial(1000), int)
```

## 📝 Daily Assignment
**Goal**: Implement several classic recursive algorithms and analyze their behavior.

1.  **Create Project File**: In your project, create `my_first_poetry_app/recursion_lab.py`.
2.  **Implement Recursive Algorithms**:
    -   `recursive_sum(numbers: list[int]) -> int`: Calculates the sum of a list of numbers recursively. (Hint: The sum of a list is the first element plus the sum of the rest of the list).
    -   `is_palindrome_recursive(text: str) -> bool`: Checks if a string is a palindrome using recursion. (Hint: A string is a palindrome if its first and last characters match, and the substring between them is also a palindrome).
    -   `find_files(path: Path, extension: str) -> Generator[Path, None, None]`: A recursive function that traverses a directory tree and `yield`s the path of any file that matches the given extension. Use `os.listdir` or `pathlib.Path.iterdir`.
3.  **Analyze Stack Depth**:
    -   Inside your `is_palindrome_recursive` function, add a `depth` parameter with a default value of 0.
    -   In the recursive call, pass `depth + 1`.
    -   Print the depth at the start of each call to visualize how deep the recursion goes.
4.  **`main()` function**:
    -   Demonstrate each of your recursive functions.
    -   For `find_files`, create a dummy directory structure to test it on.
5.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes
- **Missing or incorrect base case**: This is the most common cause of infinite recursion and `RecursionError`. Every recursive function must have a condition that stops it.
- **Recursive step doesn't progress toward the base case**: If the recursive call uses the same arguments, it will never reach the base case. `my_func(n)` must call `my_func(n-1)` or something similar that changes the state.
- **Using recursion for simple loops**: `for i in range(n): ...` is much more efficient and readable in Python than a recursive function for simple iteration. Don't use recursion just to be clever.
- **Assuming TCO**: Writing a tail-recursive function in Python is a good academic exercise, but don't assume it will protect you from stack overflows in production code.

## 📖 Further Reading
- [Real Python: Thinking Recursively in Python](https://realpython.com/python-thinking-recursively/)
- [What is Tail Call Optimization?](https://stackoverflow.com/questions/310974/what-is-tail-call-optimization)
- [Python's Guido van Rossum on Tail Recursion](http://neopythonic.blogspot.com/2009/04/tail-recursion-elimination.html)
