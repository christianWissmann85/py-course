# Week 12, Day 2: Multiprocessing

## 🎯 Learning Objectives

- [ ] Understand the use case for multiprocessing for CPU-bound tasks.
- [ ] Learn how to create and manage processes using the `multiprocessing` module.
- [ ] Understand how to use `Pool` objects to manage a group of worker processes.
- [ ] Learn about the different methods for sharing data between processes (e.g., `Queue`, `Pipe`).

## 📚 Concepts

### 1. Why Multiprocessing?

Yesterday, we learned that Python's Global Interpreter Lock (GIL) prevents multiple threads from executing Python code at the same time. This makes threading unsuitable for speeding up CPU-bound tasks.

**Multiprocessing** is the solution. The `multiprocessing` module bypasses the GIL by creating new **processes** instead of threads. Each process gets its own Python interpreter and its own memory space. This allows your Python code to take full advantage of multiple cores on a CPU, achieving true parallelism.

- **Use `threading` for**: I/O-bound concurrency.
- **Use `multiprocessing` for**: CPU-bound parallelism.

### 2. Spawning a Process

Creating a `Process` is very similar to creating a `Thread`.

```python
import multiprocessing
import os
import time

def cpu_intensive_task(name: str):
    pid = os.getpid()
    print(f"Process {name} (PID: {pid}): starting")
    # Simulate a CPU-bound task
    _ = [i*i for i in range(10**7)]
    print(f"Process {name} (PID: {pid}): finishing")

if __name__ == "__main__":
    # The __name__ == "__main__" guard is crucial for multiprocessing
    # It prevents child processes from re-importing and re-executing the script's code

    start_time = time.time()

    # Create and start a process
    process = multiprocessing.Process(target=cpu_intensive_task, args=("Worker 1",))
    process.start()

    print("Main process (PID: %s): continuing to run.", os.getpid())

    # Wait for the process to finish
    process.join()

    print(f"Main process: worker has finished in {time.time() - start_time:.2f} seconds.")
```

### 3. Managing a Pool of Workers

Often, you have a large number of tasks to run. Managing individual processes can be tedious. The `multiprocessing.Pool` object provides a convenient way to manage a pool of worker processes.

A `Pool` distributes tasks to the available worker processes. A common and powerful pattern is to use `pool.map`. It works just like the built-in `map` function but runs the operations in parallel.

```python
import multiprocessing
import time

def square(x: int) -> int:
    # This is our CPU-bound function
    return x * x

if __name__ == "__main__":
    numbers = range(10)

    # Use a 'with' statement for automatic cleanup of the pool
    # By default, Pool() creates a process for each CPU core on your machine
    with multiprocessing.Pool() as pool:
        start_time = time.time()

        # 'map' blocks until all results are ready
        results = pool.map(square, numbers)

        duration = time.time() - start_time
        print(f"Pool.map took: {duration:.4f} seconds")
        print(f"Results: {results}")

    # For comparison, let's do it sequentially
    start_time = time.time()
    sequential_results = list(map(square, numbers))
    duration = time.time() - start_time
    print(f"\nSequential map took: {duration:.4f} seconds")
    print(f"Results: {sequential_results}")
```

**Note**: For very small tasks, the overhead of creating processes and sending data between them can make multiprocessing _slower_ than a sequential approach. It shines when the individual tasks are computationally expensive.

### 4. Sharing Data Between Processes

Since processes have separate memory spaces, they cannot share data directly like threads. You must use special mechanisms for Inter-Process Communication (IPC).

- **`multiprocessing.Queue`**: A thread-safe and process-safe queue. It's the most common way to pass messages (like data or tasks) between processes. One process can `.put()` an item, and another can `.get()` it.
- **`multiprocessing.Pipe`**: Creates a pair of connection objects connected by a pipe. It's generally faster than a `Queue` but can only be used between two specific processes.
- **`multiprocessing.Value` / `multiprocessing.Array`**: Allow sharing data through shared memory, but they are more complex to use correctly as you need to handle your own synchronization with Locks.

**Example with a `Queue`**:

```python
import multiprocessing
import time

def worker(queue: multiprocessing.Queue):
    """A worker process that gets items from a queue and processes them."""
    while True:
        item = queue.get()
        if item is None:  # A "sentinel" value to signal the end
            break
        print(f"Processing {item}")
        time.sleep(1)

if __name__ == "__main__":
    q = multiprocessing.Queue()
    p = multiprocessing.Process(target=worker, args=(q,))
    p.start()

    # Add tasks to the queue
    for i in range(5):
        print(f"Adding task {i} to the queue")
        q.put(i)

    # Add the sentinel value to tell the worker to exit
    q.put(None)

    # Wait for the worker to finish
    p.join()
    print("Worker process finished.")
```

## 🔹 Quick Exercise

Write a function `parallel_sum(numbers: list[int]) -> int` that uses a `multiprocessing.Pool` to calculate the sum of squares of a list of numbers in parallel.

```python
import multiprocessing
from typing import List

def square(x: int) -> int:
    return x * x

def parallel_sum(numbers: List[int]) -> int:
    """Calculates the sum of squares in parallel."""
    # Hint: Use pool.map to get the squares, then use the built-in sum()
    # on the results.
    with multiprocessing.Pool() as pool:
        squared_numbers = pool.map(square, numbers)
        return sum(squared_numbers)

# --- Test ---
# if __name__ == "__main__":
#     nums = list(range(1000))
#     result = parallel_sum(nums)
#     expected = sum(x*x for x in nums)
#     assert result == expected
#     print("Test passed!")
```

## 📝 Daily Assignment

**Goal**: Use `multiprocessing` to speed up a CPU-bound data processing task.

1.  **Create Project File**: In your project, create `my_first_poetry_app/parallel_processor.py`.
2.  **Generate Sample Data**:
    - Create a function `generate_large_file(filename: str, num_lines: int)` that writes a large text file.
    - Each line should contain a random integer between 1 and 1,000,000.
3.  **Implement a Processing Function**:
    - Create a function `is_prime(n: int) -> bool` that checks if a number is prime. This will be our simulated "heavy" CPU task.
4.  **Implement the Parallel Logic**:
    - In your `main` function (guarded by `if __name__ == "__main__":`), first call `generate_large_file` to create a dataset.
    - Read all the numbers from the file into a list.
    - Use `multiprocessing.Pool` and `pool.map` to apply your `is_prime` function to every number in the list. This will return a list of booleans (`[True, False, True, ...]`).
    - Count the number of `True` values in the result to find the total number of primes.
5.  **Analyze**:
    - Time the entire process.
    - Now, modify your code to do the same task sequentially (using a simple `for` loop or the built-in `map`) and time it.
    - Compare the parallel vs. sequential execution times. You should see a significant speedup on a multi-core machine.
6.  **Verify**: Run the linter and type checker on your code.

## ⚠️ Common Mistakes

- **Forgetting `if __name__ == "__main__":`**: This is the most common error. Without it, child processes will re-import the script, leading to an infinite loop of process creation that will crash your system.
- **Passing Un-picklable Objects**: Data sent between processes must be "picklable". Most basic Python objects are, but things like file handles, database connections, or lambda functions are not. This will raise a `PicklingError`.
- **Ignoring Communication Overhead**: Sending large amounts of data between processes can be slow. If the task itself is very fast, this overhead can negate the benefits of parallelism.
- **Shared State Issues**: Trying to modify global variables or shared objects without proper IPC mechanisms (`Queue`, `Pipe`, etc.). This won't work as expected because each process has its own memory.

## 📖 Further Reading

- [Python Docs: `multiprocessing` — Process-based parallelism](https://docs.python.org/3/library/multiprocessing.html)
- [Real Python: An Introduction to Parallel Programming with `multiprocessing`](https://realpython.com/python-multiprocessing/)
