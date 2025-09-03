# Week 11, Day 3: Async Context Managers & Iterators

## 🎯 Learning Objectives
- [ ] Implement asynchronous context managers using `async with`.
- [ ] Create classes that support the async context manager protocol with `__aenter__` and `__aexit__`.
- [ ] Iterate over asynchronous data streams using `async for`.
- [ ] Write asynchronous generators with `async def` and `yield`.

## 📚 Concepts

### 1. Asynchronous Context Managers: `async with`
Just as a regular `with` statement manages resources like files, an `async with` statement manages resources that require `await` for their setup and teardown (e.g., a database connection in an async library).

```python
import httpx

async def main():
    # httpx.AsyncClient() is an async context manager.
    # Its __aenter__ method is awaited to set up the connection pool.
    async with httpx.AsyncClient() as client:
        response = await client.get("https://www.example.com")
        print(response.status_code)
    # Its __aexit__ method is awaited here to cleanly close connections.
```
The `async with` statement ensures that the resource's cleanup logic is always called, even if errors occur within the block.

### 2. The `__aenter__` and `__aexit__` Methods
To create your own async context manager, you implement the async context management protocol in your class:
- **`async def __aenter__(self)`**: Called when entering the `async with` block. Can perform `await` operations. Its return value is bound to the `as` variable.
- **`async def __aexit__(self, exc_type, exc_value, traceback)`**: Called when exiting the block. Can perform `await` operations for cleanup.

```python
import asyncio

class AsyncDatabaseConnection:
    async def __aenter__(self):
        print("Connecting to the database...")
        await asyncio.sleep(1) # Simulate async connection
        print("Connected.")
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("Disconnecting from the database...")
        await asyncio.sleep(1) # Simulate async disconnection
        print("Disconnected.")

    async def query(self, q: str):
        await asyncio.sleep(0.5)
        return f"Result for '{q}'"

async def main():
    async with AsyncDatabaseConnection() as db:
        result = await db.query("SELECT * FROM users")
        print(result)
```

### 3. Asynchronous Iterators: `async for`
Just as `for` iterates over a regular iterable, `async for` iterates over an **asynchronous iterable**.

An asynchronous iterable is an object with an `__aiter__` method. An asynchronous iterator is an object with an `__anext__` method, which must be a coroutine.

The `async for` loop handles the `await`ing of `__anext__` for you.

### 4. Asynchronous Generators
The easiest way to create an asynchronous iterator is with an **asynchronous generator**. You define it with `async def` and use `yield` inside it, just like a regular generator. The key difference is that you can also use `await` inside an async generator.

This is perfect for creating a stream of data where fetching each item is an I/O operation.

```python
import asyncio
from typing import AsyncGenerator

async def number_stream(count: int) -> AsyncGenerator[int, None]:
    """
    An async generator that yields numbers with a delay.
    """
    for i in range(count):
        # Simulate an async operation to get the next number
        await asyncio.sleep(0.5)
        yield i

async def main():
    # `async for` handles calling __anext__ and awaiting it.
    async for number in number_stream(5):
        print(number)
```

### 5. Typing Async Protocols
The `typing` module provides `AsyncContextManager`, `AsyncIterable`, and `AsyncIterator` for type hinting.

```python
from typing import AsyncContextManager, AsyncIterable

# Assuming DBConnection is a defined class
# def get_db_conn() -> AsyncContextManager[DBConnection]:
#     ...

async def process_items(stream: AsyncIterable[int]):
    async for item in stream:
        print(item)
```

## 🔹 Quick Exercise

Complete the `AsyncResource` class below. It should be an async context manager that simulates acquiring and releasing a resource.

```python
import asyncio

class AsyncResource:
    async def __aenter__(self):
        print("Acquiring resource...")
        await asyncio.sleep(1)
        print("Resource acquired.")
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("Releasing resource...")
        await asyncio.sleep(1)
        print("Resource released.")

    async def use(self):
        print("Using the resource...")
        await asyncio.sleep(1)

async def main():
    async with AsyncResource() as res:
        await res.use()

# --- Run the main coroutine ---
asyncio.run(main())
```

## 📝 Daily Assignment
**Goal**: Build an asynchronous log file reader that streams new lines as they are added to a file.

1.  **Create Project Files**:
    -   `my_first_poetry_app/async_tail.py`: Your main application.
    -   `my_first_poetry_app/log_writer.py`: A separate script to simulate writing to a log file.
2.  **Implement `log_writer.py`**:
    -   This should be a simple synchronous script.
    -   In an infinite loop, it should write a new line with the current timestamp to a file (`app.log`) every second.
3.  **Implement `async_tail.py`**:
    -   **`tail_file(filepath: str) -> AsyncGenerator[str, None]`**: This will be your async generator.
        -   It should open the file.
        -   In an infinite `while True` loop:
            -   Read a line from the file.
            -   If there is a line, `yield` it.
            -   If there is no line (you've reached the end of the file), `await asyncio.sleep(0.1)` to pause without blocking, then try reading again.
    -   **`main()` coroutine**:
        -   Run the `log_writer.py` script as a separate process in the background. (You can use `asyncio.create_subprocess_exec`).
        -   Use `async for` to iterate over the `tail_file` generator.
        -   Print each new line as it comes in.
        -   Add logic to stop after receiving 10 lines.
4.  **Run and Verify**:
    -   Run `async_tail.py`. You should see it print log lines in real-time as the `log_writer.py` script adds them.

## ⚠️ Common Mistakes
- **Using `with` instead of `async with`**: If an object is an async context manager (has `__aenter__`/`__aexit__`), you must use `async with`. Using a plain `with` will result in a `TypeError`.
- **Using `for` instead of `async for`**: Similarly, you must use `async for` to iterate over an async iterator/generator.
- **Blocking inside an async generator**: If you perform a blocking operation (like `time.sleep()`) inside an `async def`, you block the entire event loop. The `yield` doesn't release control if the function itself is blocked. Always `await` non-blocking operations like `asyncio.sleep()`.

## 📖 Further Reading
- [Python Docs: Asynchronous Context Managers and `async with`](https://docs.python.org/3/reference/compound_stmts.html#async-with)
- [Python Docs: Asynchronous Iterators and `async for`](https://docs.python.org/3/reference/compound_stmts.html#async-for)
- [Real Python: Asynchronous Iterators and Generators](https://realpython.com/async-io-python/#asynchronous-iteration-and-asynchronous-generators)
