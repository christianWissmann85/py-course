# Week 12, Day 3: The `concurrent.futures` Module

## 🎯 Learning Objectives
- [ ] Understand the high-level API provided by `concurrent.futures`.
- [ ] Learn how to use the `ThreadPoolExecutor` for I/O-bound tasks.
- [ ] Learn how to use the `ProcessPoolExecutor` for CPU-bound tasks.
- [ ] Understand the `Future` object and how to use it to manage asynchronous results.

## 📚 Concepts

### 1. A High-Level Abstraction
The `threading` and `multiprocessing` modules are powerful but can be verbose. The `concurrent.futures` module, introduced in Python 3.2, provides a much higher-level, simpler API for managing pools of threads or processes.

The key idea is that you submit a "future" (a task you want to run) to an "executor". The executor manages a pool of workers (threads or processes) and runs your task. It returns a `Future` object, which is a placeholder for the result that will eventually be available.

This module provides two main executor classes:
- **`ThreadPoolExecutor`**: Uses a pool of threads. Ideal for I/O-bound concurrency.
- **`ProcessPoolExecutor`**: Uses a pool of processes. Ideal for CPU-bound parallelism.

The beauty is that the API is **identical** for both. You can switch from threads to processes by changing just one line of code, making it easy to experiment and choose the right tool for your task.

### 2. Using an Executor
The most common pattern is to use an executor as a context manager (`with` statement), which handles the creation and proper shutdown of the worker pool.

You submit tasks using the `.submit()` method.

```python
from concurrent.futures import ThreadPoolExecutor
import time
import requests

def download_url(url: str) -> str:
    """A simple I/O-bound task."""
    print(f"Starting download for {url}")
    response = requests.get(url)
    print(f"Finished download for {url}")
    return f"Downloaded {len(response.content)} bytes from {url}"

URLS = [
    "https://www.python.org/",
    "https://www.djangoproject.com/",
    "https://www.realpython.com/",
]

# Using the ThreadPoolExecutor for our I/O-bound task
with ThreadPoolExecutor(max_workers=3) as executor:
    # .submit() immediately returns a Future object
    # It does not wait for the function to complete.
    future1 = executor.submit(download_url, URLS[0])
    future2 = executor.submit(download_url, URLS[1])
    future3 = executor.submit(download_url, URLS[2])

    print("Tasks have been submitted.")

    # To get the result, we call .result() on the future.
    # This call will BLOCK until the future is complete.
    result1 = future1.result()
    result2 = future2.result()
    result3 = future3.result()

    print("\nResults:")
    print(result1)
    print(result2)
    print(result3)
```

### 3. The `Future` Object
A `Future` object represents a computation that may not have completed yet. It's a promise. You can query its state and get its result (or the exception it raised).

- `future.done()`: Returns `True` if the future has completed or been cancelled.
- `future.result()`: Returns the result of the callable. If the callable raised an exception, this method raises the same exception. This method is blocking.
- `future.exception()`: Returns the exception raised by the callable. If no exception was raised, returns `None`. This method is non-blocking.
- `future.add_done_callback(fn)`: Attaches a callable `fn` that will be called with the future as its only argument when the future is done. This is a non-blocking way to handle results.

### 4. `executor.map`
Just like with `multiprocessing.Pool`, executors have a `.map()` method. It's a convenient way to apply a function to every item in an iterable. It's simpler than submitting each task individually and collecting the results.

`executor.map` returns an iterator that yields results *in the order the tasks were submitted*.

```python
from concurrent.futures import ProcessPoolExecutor
import time

def square(x: int) -> int:
    """A simple CPU-bound task."""
    # print(f"Squaring {x}")
    return x * x

if __name__ == "__main__":
    numbers = range(10)

    # Using ProcessPoolExecutor for our CPU-bound task
    with ProcessPoolExecutor() as executor:
        start_time = time.time()

        # 'map' applies the function to each item in 'numbers'
        # It returns an iterator that yields results as they become available
        results = executor.map(square, numbers)

        # Iterating over 'results' will block until each result is ready
        results_list = list(results)

        duration = time.time() - start_time
        print(f"Executor.map took: {duration:.4f} seconds")
        print(f"Results: {results_list}")
```

## 🔹 Quick Exercise

Rewrite the `parallel_sum` function from yesterday's lesson to use a `ProcessPoolExecutor` and its `.map()` method.

```python
from concurrent.futures import ProcessPoolExecutor
from typing import List

def square(x: int) -> int:
    return x * x

def parallel_sum_with_executor(numbers: List[int]) -> int:
    """Calculates the sum of squares in parallel using ProcessPoolExecutor."""
    with ProcessPoolExecutor() as executor:
        # Use executor.map to get the squares of the numbers
        squared_numbers = executor.map(square, numbers)
        # The result of map is an iterator, so you can pass it directly to sum()
        return sum(squared_numbers)

# --- Test ---
# if __name__ == "__main__":
#     nums = list(range(1000))
#     result = parallel_sum_with_executor(nums)
#     expected = sum(x*x for x in nums)
#     assert result == expected
#     print("Test passed!")
```

## 📝 Daily Assignment
**Goal**: Refactor the threaded downloader and the parallel processor from the previous two days to use `concurrent.futures`.

This assignment has two parts.

### Part 1: Refactor the I/O-bound Downloader
1.  **Copy and Rename**: Copy your `threaded_downloader.py` to a new file, `executor_downloader.py`.
2.  **Refactor**: Modify the code to use a `ThreadPoolExecutor` instead of manually creating and managing `threading.Thread` objects.
    -   Use `executor.map` for a concise and elegant solution.
3.  **Analyze**: Run the script. The behavior should be the same, but the code should be cleaner and easier to read.

### Part 2: Refactor the CPU-bound Processor
1.  **Copy and Rename**: Copy your `parallel_processor.py` to a new file, `executor_processor.py`.
2.  **Refactor**: Modify the code to use a `ProcessPoolExecutor` instead of a `multiprocessing.Pool`.
    -   Again, `executor.map` is the ideal tool here.
3.  **Analyze**: Compare the code's complexity. The `concurrent.futures` version is often simpler to write and reason about. The performance should be very similar to the `multiprocessing.Pool` version.

## ⚠️ Common Mistakes
- **Using the Wrong Executor**: Using a `ThreadPoolExecutor` for a CPU-bound task will not give you a speedup due to the GIL. Using a `ProcessPoolExecutor` for a simple I/O-bound task can be slower than a `ThreadPoolExecutor` due to the higher overhead of creating processes.
- **Not Handling Exceptions**: If a function submitted to an executor raises an exception, that exception will be "stored" in the `Future` object. If you never call `.result()` or `.exception()` on that future, the exception will be silently ignored. This can make debugging very difficult.
- **Passing Un-picklable Objects to `ProcessPoolExecutor`**: Just like with `multiprocessing`, any data passed to a `ProcessPoolExecutor` (as arguments or returned from the function) must be picklable.

## 📖 Further Reading
- [Python Docs: `concurrent.futures` — Launching parallel tasks](https://docs.python.org/3/library/concurrent.futures.html)
- [Real Python: Threading and `concurrent.futures` in Python](https://realpython.com/python-concurrency/)
- [A deep dive into `concurrent.futures`](https://www.roguelynn.com/words/asyncio-we-did-it-wrong/) (This article also discusses `asyncio`, which we'll see next week, but the `concurrent.futures` parts are excellent).
