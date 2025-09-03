# Week 11, Day 2: AsyncIO Deep Dive - Running Tasks Concurrently

## 🎯 Learning Objectives
- [ ] Move beyond sequential `await` to run multiple async operations concurrently.
- [ ] Understand the role of `asyncio.Task` objects for scheduling coroutines on the event loop.
- [ ] Use `asyncio.gather()` to run a list of awaitables concurrently and wait for all to complete.
- [ ] Handle exceptions and timeouts in concurrent tasks.

## 📚 Concepts

### 1. The Problem with Sequential `await`
Yesterday, we wrote an async `main` function that looked like this:
```python
async def main():
    await cook_rice()
    await cook_vegetables()
    await cook_meat()
```
This code is asynchronous, but it is not **concurrent**. Each `await` blocks the `main` function until that specific operation is complete. The total time is the sum of all individual times. To get a real performance benefit, we need to start all the "cooking" tasks at the same time and let them run concurrently.

### 2. `asyncio.Task`
An `asyncio.Task` is an object that schedules a coroutine to run on the event loop "in the background". When you wrap a coroutine in a task, the event loop can start running it as soon as it gets a chance, without you having to explicitly `await` it right away.

The most common way to create a task is with `asyncio.create_task()`.

```python
import asyncio

async def my_coro():
    print("Starting...")
    await asyncio.sleep(1)
    print("...Finished")

async def main():
    # Schedule my_coro to run on the event loop.
    # The event loop will start it as soon as possible.
    task = asyncio.create_task(my_coro())

    # We can do other things here while the task runs in the background.
    print("Task scheduled.")
    await asyncio.sleep(2) # Wait long enough for the task to finish
    print("Main function done.")
```
This is better, but we still need a way to wait for the task to be officially complete and get its result.

### 3. `asyncio.gather()`
`asyncio.gather()` is the primary tool for running a list of awaitable objects (like coroutines or tasks) concurrently. It takes all the awaitables, schedules them to run, and then waits for all of them to finish. It returns a list of their results in the same order.

This is the solution to our cooking problem:

```python
import asyncio
import time

async def cook(food: str, duration: int):
    print(f"Start cooking {food}...")
    await asyncio.sleep(duration)
    print(f"{food} is ready!")
    return f"{food} done"

async def main():
    start_time = time.perf_counter()

    # gather() runs all these coroutines concurrently
    results = await asyncio.gather(
        cook("Rice", 3),
        cook("Vegetables", 5),
        cook("Meat", 8)
    )

    end_time = time.perf_counter()
    print(f"Finished all cooking in {end_time - start_time:.2f} seconds.")
    print(f"Results: {results}")

# The total time will be roughly the time of the LONGEST task (8 seconds),
# not the sum of all tasks.
asyncio.run(main())
```

### 4. Exception Handling with `gather`
If one of the awaitables passed to `gather()` raises an exception, the exception is immediately propagated to the code that is `await`ing `gather()`. By default, `gather()` will not cancel the other running tasks.

To get all results, including exceptions, you can use `return_exceptions=True`.

```python
async def succeed(): return "Success"
async def fail(): raise ValueError("Failure")

async def main():
    # With return_exceptions=False (default)
    try:
        await asyncio.gather(succeed(), fail(), succeed())
    except ValueError as e:
        print(f"Caught expected error: {e}")

    # With return_exceptions=True
    results = await asyncio.gather(
        succeed(),
        fail(),
        succeed(),
        return_exceptions=True
    )
    for result in results:
        if isinstance(result, Exception):
            print(f"Task failed with: {result}")
        else:
            print(f"Task succeeded with: {result}")
```

### 5. Timeouts
What if a network request takes too long? You can wrap an awaitable in `asyncio.wait_for()` to set a timeout. If the operation doesn't complete within the timeout period, it will raise an `asyncio.TimeoutError`.

```python
async def long_running_task():
    await asyncio.sleep(10)

async def main():
    try:
        # Wait a maximum of 5 seconds for the task.
        await asyncio.wait_for(long_running_task(), timeout=5.0)
    except asyncio.TimeoutError:
        print("The task took too long!")
```

## 🔹 Quick Exercise

Complete the `fetch_multiple` function below. It should take a list of URLs and use `asyncio.gather()` to fetch them all concurrently.

```python
import asyncio

async def fetch_data(url: str, delay: float = 1.0) -> str:
    """A coroutine that simulates fetching data from a URL."""
    await asyncio.sleep(delay)
    return f"Data from {url}"

async def fetch_multiple(urls: list[str]) -> list[str]:
    """
    Fetches multiple URLs concurrently.
    """
    tasks = [fetch_data(url) for url in urls]
    return await asyncio.gather(*tasks)

async def main():
    urls = ["a.com", "b.com", "c.com"]
    # This should take about 1 second, not 3.
    results = await fetch_multiple(urls)
    assert results == ["Data from a.com", "Data from b.com", "Data from c.com"]
    print("Successfully fetched multiple URLs concurrently.")

# --- Run the main coroutine ---
asyncio.run(main())
```

## 📝 Daily Assignment
**Goal**: Build a simple, concurrent web scraper.

1.  **Create Project File and Add Dependency**:
    -   In your project, create `my_first_poetry_app/concurrent_scraper.py`.
    -   You'll need an async HTTP client. `httpx` is a great choice. Add it to your project: `poetry add httpx`.
2.  **Implement a `fetch_status` coroutine**:
    -   `async def fetch_status(client: httpx.AsyncClient, url: str) -> tuple[int, str]`:
    -   This coroutine should take an `httpx.AsyncClient` instance and a URL.
    -   It should make a `GET` request to the URL.
    -   It should return a tuple containing the HTTP status code and the URL.
3.  **Implement Error and Timeout Handling**:
    -   Wrap the request logic in a `try...except` block to catch potential `httpx.RequestError` exceptions. If an error occurs, return `(0, url)` as a signal.
    -   Wrap the `await client.get(...)` call in `asyncio.wait_for(..., timeout=10.0)` to prevent hanging on a slow site. Catch the `TimeoutError` and return `(0, url)`.
4.  **`main()` coroutine**:
    -   Define a list of 10-20 URLs to check. Include some that might be slow or invalid. (e.g., `https://httpbin.org/delay/5`, `https://non-existent-domain.xyz`)
    -   Use an **async context manager** (`async with httpx.AsyncClient() as client:`) to create a client session.
    -   Inside the `async with` block, create a list of `fetch_status` tasks for all your URLs.
    -   Use `asyncio.gather(*tasks)` to run them all concurrently.
    -   Loop through the results and print a summary of each URL's status.
5.  **Verify**: Run your script. It should process all URLs in parallel and finish in roughly the time of the longest single request (or the timeout), not the sum of all requests. It should also handle errors gracefully.

## ⚠️ Common Mistakes
- **Creating tasks but not `await`ing them**: If you `create_task` but your program exits before the task is done, it will be cancelled. `asyncio.gather` is the most common way to ensure you wait for all your tasks to complete.
- **Not using a session object for multiple requests**: When using `httpx`, it's much more efficient to create a single `AsyncClient` and reuse it for all requests, rather than creating a new one for each request. The `async with` block handles this perfectly.
- **Mixing blocking and async code**: Calling a regular blocking function (like `requests.get()` or `time.sleep()`) inside a coroutine will block the entire event loop and negate all benefits of `asyncio`.

## 📖 Further Reading
- [Python Docs: Coroutines and Tasks](https://docs.python.org/3/library/asyncio-task.html)
- [Real Python: Getting Started With `asyncio`](https://realpython.com/async-io-python/#the-asyncio-package-and-the-asyncawait-syntax)
- [`httpx` Documentation](https://www.python-httpx.org/)
