# Week 12, Day 1: Threading Basics

## 🎯 Learning Objectives
- [ ] Understand the difference between a process and a thread.
- [ ] Learn about Python's Global Interpreter Lock (GIL) and its impact on multi-threaded performance.
- [ ] Create and run basic threads using the `threading` module.
- [ ] Understand the concept of thread safety and how to use locks to prevent race conditions.

## 📚 Concepts

### 1. Concurrency vs. Parallelism
- **Concurrency**: The ability to have multiple tasks *in progress* at the same time. The tasks can be interleaved on a single CPU core. This is what `asyncio` does.
- **Parallelism**: The ability to have multiple tasks *executing* at the exact same time, typically on multiple CPU cores.

A **process** is an instance of a program running on your computer. It has its own memory space. A **thread** is the smallest unit of execution within a process. A single process can have multiple threads, and they all share the same memory space.

### 2. The Global Interpreter Lock (GIL)
The **Global Interpreter Lock (GIL)** is a mutex (a type of lock) that protects access to Python objects, preventing multiple native threads from executing Python bytecode at the same time.

**What this means**: Even if you have a multi-core CPU, only one thread can be executing Python code at any given moment.

- **Does this make threading useless?** No!
- For **I/O-bound** tasks (like waiting for network requests or reading from disk), the GIL is released while the thread is waiting. This allows another thread to run, making threading very effective for I/O-bound concurrency.
- For **CPU-bound** tasks (like heavy calculations), the GIL prevents threads from achieving true parallelism. In this case, **multiprocessing** (which we'll cover tomorrow) is the correct tool.

### 3. Creating Threads
The `threading` module is Python's standard library for working with threads.

```python
import threading
import time

def worker(name: str):
    print(f"Thread {name}: starting")
    time.sleep(2)
    print(f"Thread {name}: finishing")

# Create a thread instance
# target is the function the thread will run
# args is a tuple of arguments to pass to the target function
thread = threading.Thread(target=worker, args=("Worker 1",))

# Start the thread's execution
thread.start()

print("Main thread: continuing to run while the worker is busy.")

# Wait for the thread to complete its execution
thread.join()

print("Main thread: worker has finished.")
```

### 4. Thread Safety and Race Conditions
Because threads in the same process share memory, multiple threads can try to read and write to the same variable at the same time. This can lead to unpredictable results and corrupted data, a problem known as a **race condition**.

Code that is safe to be called from multiple threads simultaneously is called **thread-safe**.

The primary tool for ensuring thread safety is a **Lock**. A `threading.Lock` can be in one of two states: locked or unlocked. Only one thread can "acquire" the lock at a time. Any other thread that tries to acquire it will be blocked until the lock is released.

```python
import threading

class SafeCounter:
    def __init__(self):
        self._value = 0
        self._lock = threading.Lock()

    def increment(self):
        # The 'with' statement automatically acquires and releases the lock
        with self._lock:
            # This section is a "critical section" - only one thread can be here at a time.
            current_value = self._value
            # In a real race condition, the thread could be swapped out here!
            self._value = current_value + 1
```

### 5. When to Use Threading
- **Best for I/O-bound tasks**: Use threading to run multiple network requests, file operations, or other I/O-bound tasks concurrently. While one thread is waiting for I/O, the GIL is released and another thread can run.
- **Avoid for CPU-bound tasks**: For tasks that are heavy on calculation, the GIL will prevent you from getting a performance boost from multiple threads. Use `multiprocessing` for these.

## 🔹 Quick Exercise

Complete the `create_thread_safe_counter` function below. It should return a function that can be safely called from multiple threads to increment a shared counter.

```python
import threading
from typing import Callable

def create_thread_safe_counter() -> Callable[[], int]:
    """Creates and returns a thread-safe counter function."""
    counter = 0
    lock = threading.Lock()

    def increment() -> int:
        nonlocal counter
        with lock:
            counter += 1
            return counter

    return increment

# --- A simple test (a real test would use multiple threads) ---
counter_func = create_thread_safe_counter()
# assert counter_func() == 1
# assert counter_func() == 2
```

## 📝 Daily Assignment
**Goal**: Build a simple thread pool to download multiple files concurrently.

1.  **Create Project File**: In your project, create `my_first_poetry_app/threaded_downloader.py`.
2.  **Implement a `download_file` function**:
    -   `download_file(url: str, target_dir: Path) -> None`:
    -   This function should take a URL and a directory path.
    -   It should use the `requests` library (you'll need to `poetry add requests`) to download the content from the URL.
    -   It should then save the content to a file in the `target_dir`. The filename can be derived from the URL.
    -   Add `print` or `logging` statements to show when a download starts and finishes.
3.  **Implement the Thread Pool Logic**:
    -   In a `main` function, define a list of URLs to download (e.g., a list of image URLs).
    -   Create a list of `threading.Thread` objects, one for each URL. Each thread should be configured to run your `download_file` function.
    -   Start all the threads.
    -   After starting them, loop through your list of thread objects and call `.join()` on each one. This ensures the main script waits for all downloads to complete before exiting.
4.  **Analyze**:
    -   Run the script. Observe from your print/log statements how the downloads happen concurrently.
    -   Add a timer to your `main` function to measure the total time. Compare this to how long it would take to download the files sequentially in a simple `for` loop.
5.  **Verify**: Check that all files were downloaded correctly. Run the linter and type checker.

## ⚠️ Common Mistakes
- **Forgetting `.join()`**: If you start a bunch of threads but your main script finishes before they do, the threads might be killed abruptly. Calling `.join()` on a thread makes the main thread wait for it to finish.
- **Race Conditions**: Forgetting to use locks when multiple threads modify a shared piece of data (like a list or dictionary). This leads to unpredictable, hard-to-debug errors.
- **Deadlocks**: Thread A acquires Lock 1 and waits for Lock 2. Thread B acquires Lock 2 and waits for Lock 1. Neither thread can proceed. This is a deadlock. A common way to avoid this is to always acquire locks in the same order.
- **Using threading for CPU-bound work**: Newcomers often expect threading to speed up heavy calculations in Python. Due to the GIL, it usually makes them *slower* because of the overhead of managing the threads.

## 📖 Further Reading
- [Real Python: An Introduction to Threading in Python](https://realpython.com/intro-to-python-threading/)
- [Python Docs: `threading` — Thread-based parallelism](https://docs.python.org/3/library/threading.html)
- [David Beazley: Understanding the Python GIL](https://www.youtube.com/watch?v=Obt-vMVdM8s) (A classic, deep-dive talk)
