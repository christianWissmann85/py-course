# Week 11, Day 1: Async/Await Basics

## 🎯 Learning Objectives

- [ ] Understand the difference between synchronous and asynchronous code.
- [ ] Define asynchronous functions (coroutines) using `async def`.
- [ ] Use the `await` keyword to pause execution and wait for an asynchronous operation to complete.
- [ ] Use `asyncio.run()` to execute the top-level asynchronous function.

## 📚 Concepts

### 1. Synchronous vs. Asynchronous

- **Synchronous (sync) code**: The code we've written so far. Each line executes in order, one after the other. If a line of code takes a long time to run (like a network request or reading a large file), the entire program blocks and waits.

- **Asynchronous (async) code**: Allows the program to perform other tasks while waiting for a long operation to complete. Instead of blocking, the program can switch to another task. This is called **cooperative multitasking**. It's particularly useful for I/O-bound operations (like network requests, database queries, file I/O) where the program spends most of its time waiting for external resources.

### 2. `async def`: Creating Coroutines

To create an asynchronous function, you use the `async def` syntax. Functions defined this way are called **coroutines**.

```python
import asyncio

# This is a coroutine function.
async def my_coroutine():
    print("Running in a coroutine.")
    # A coroutine can use `await` to call other coroutines.
    await asyncio.sleep(1) # a non-blocking sleep
    print("Finished coroutine.")

# Calling a coroutine function does NOT run it.
# It returns a coroutine object.
coro_obj = my_coroutine()
print(coro_obj) # <coroutine object my_coroutine at 0x...>
```

A coroutine function can be thought of as a "pausable" function.

### 3. `await`: Pausing Coroutines

The `await` keyword is used inside a coroutine to call another coroutine. It tells the event loop, "Pause this function's execution here, and give control back to me. When the operation I'm waiting for is complete, wake me up and continue from this point."

You can **only** use `await` inside an `async def` function.

```python
async def fetch_data(url: str) -> str:
    print(f"Fetching data from {url}...")
    # In a real app, this would be a network request.
    # asyncio.sleep() is a coroutine that simulates waiting for I/O.
    await asyncio.sleep(2)
    print("...data fetched.")
    return f"Data from {url}"

async def main_logic():
    # We 'await' the result of the fetch_data coroutine.
    # This pauses main_logic for 2 seconds.
    result = await fetch_data("https://example.com")
    print(result)
```

### 4. The Event Loop and `asyncio.run()`

So if calling a coroutine doesn't run it, what does? An **event loop**. The event loop is the orchestrator for all your async tasks. It keeps track of which tasks are ready to run and which are paused (waiting for an `await`).

The simplest way to start the event loop and run your top-level async function is with `asyncio.run()`.

```python
import asyncio

async def main():
    print("Hello")
    await asyncio.sleep(1)
    print("World")

# This starts the event loop, runs the `main` coroutine until it completes,
# and then closes the loop.
if __name__ == "__main__":
    asyncio.run(main())
```

You typically have only one `asyncio.run()` call in your entire application, in your main entry point.

### 5. When to Use `asyncio`

`asyncio` is not a magic bullet for making all code faster. It excels at **I/O-bound** problems.

- **Good for `asyncio`**:
  - Web servers and clients (making thousands of network requests concurrently).
  - Database connections.
  - Chat applications with many simultaneous connections.
  - Reading/writing to multiple files.
- **Not good for `asyncio`**:
  - **CPU-bound** tasks (e.g., complex mathematical calculations, image processing). Since `asyncio` runs on a single thread, a long-running CPU-bound task will block the entire event loop, defeating the purpose of async. For CPU-bound work, you should use multiprocessing (Week 12).

## 🔹 Quick Exercise

Complete the `fetch_data` and `main` coroutines below.

```python
import asyncio

async def fetch_data(url: str) -> str:
    """A coroutine that simulates fetching data from a URL."""
    print(f"Start fetching from {url}")
    # Use await to pause for 1 second to simulate a network request.
    await asyncio.sleep(1)
    print(f"Finished fetching from {url}")
    return f"Data from {url}"

async def main() -> None:
    """The main coroutine to run our program."""
    # Use await to call fetch_data and get the result.
    result = await fetch_data("https://example.com")
    print(result)

# Use asyncio.run() to execute your main coroutine.
if __name__ == "__main__":
    asyncio.run(main())
```

## 📝 Daily Assignment

**Goal**: Create a set of simple asynchronous functions and run them.

1.  **Create Project File**: In your project, create `my_first_poetry_app/async_lab.py`.
2.  **Create Several Coroutines**:
    - `async def cook_rice()`: Should print "Cooking rice...", wait for 3 seconds, and then print "Rice is ready!".
    - `async def cook_vegetables()`: Should print "Chopping vegetables...", wait for 2 seconds, then print "Cooking vegetables...", wait for another 2 seconds, and finally print "Vegetables are ready!".
    - `async def cook_meat()`: Should print "Marinating meat...", wait for 4 seconds, and then print "Meat is ready!".
3.  **Create a `main` coroutine**:
    - Define `async def main()`.
    - Inside `main`, call each of your "cook" coroutines in sequence using `await`.
    - Use the `time` module to measure how long the entire `main` function takes to run.
4.  **Run the Program**:
    - Use `if __name__ == "__main__":` and `asyncio.run(main())` to execute your code.
    - Observe the total execution time. It should be the _sum_ of all the individual `sleep` times (around 3 + 2 + 2 + 4 = 11 seconds), because `await` runs them one after another.
5.  **Analysis**: Add a comment in your code explaining why this sequential `await`ing doesn't provide any concurrency yet. We will solve this tomorrow!
6.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes

- **Forgetting `await`**: Calling a coroutine without `await`ing it. This creates the coroutine object but doesn't run it, which is a very common bug. `mypy` and `ruff` will often warn you about this ("coroutine was never awaited").
- **Using `time.sleep()` in async code**: `time.sleep()` is a blocking call. It will freeze your entire application, including the event loop. Always use the non-blocking `asyncio.sleep()` in async code.
- **Putting `async` on a non-async function**: If a function doesn't use `await`, it probably doesn't need to be an `async def` function.
- **Trying to `await` a regular function**: You can only `await` other "awaitables" (like coroutines or `Task` objects). `await my_regular_function()` will raise a `TypeError`.

## 📖 Further Reading

- [Python Docs: `asyncio` — Asynchronous I/O](https://docs.python.org/3/library/asyncio.html)
- [Real Python: Async IO in Python: A Complete Walkthrough](https://realpython.com/async-io-python/)
- [Async Python for the Coder Who Has Outgrown `requests`](https://lucumr.pocoo.org/2016/10/30/i-dont-understand-asyncio/) (A great conceptual article)
