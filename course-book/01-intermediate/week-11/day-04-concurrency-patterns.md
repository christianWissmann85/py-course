# Week 11, Day 4: Concurrency Patterns

## 🎯 Learning Objectives

- [ ] Understand the need for concurrency control in asynchronous applications.
- [ ] Use `asyncio.Semaphore` to limit the number of concurrent operations.
- [ ] Use `asyncio.Lock` to protect access to a shared resource.
- [ ] Implement a basic producer-consumer pattern using `asyncio.Queue`.

## 📚 Concepts

### 1. The Need for Concurrency Control

When you run many tasks concurrently with `asyncio.gather()`, you might overwhelm the service you are calling (e.g., a web API) or exhaust local resources (e.g., open file handles). Concurrency control patterns allow you to manage the flow of your async tasks to prevent these issues.

### 2. `asyncio.Semaphore`

A **semaphore** is a synchronization primitive that limits the number of coroutines that can access a resource or perform an operation at the same time. You initialize it with a counter. Each time a coroutine wants to enter the controlled block, it must `acquire()` the semaphore, which decrements the counter. When it's done, it `release()`s the semaphore, incrementing the counter. If the counter is zero, any new coroutine trying to `acquire()` will wait until another one releases.

The easiest way to use it is with `async with`.

```python
import asyncio

async def limited_worker(semaphore: asyncio.Semaphore, id: int):
    # This ensures that only a certain number of workers can be
    # inside this 'async with' block at the same time.
    async with semaphore:
        print(f"Worker {id} has acquired the semaphore.")
        await asyncio.sleep(2) # Simulate work
        print(f"Worker {id} is releasing the semaphore.")

async def main():
    # Create a semaphore that allows only 3 workers at a time
    semaphore = asyncio.Semaphore(3)

    # Start 10 workers
    tasks = [limited_worker(semaphore, i) for i in range(10)]
    await asyncio.gather(*tasks)

# You will see 3 workers start, then as each one finishes, a new one starts.
# asyncio.run(main())
```

This is the primary tool for rate-limiting concurrent tasks.

### 3. `asyncio.Lock`

A **lock** is like a semaphore with a count of 1. It allows only one coroutine at a time to access a critical section of code. This is used to protect shared resources from being modified by multiple coroutines at once, which could lead to a corrupt state (a "race condition").

```python
import asyncio

async def modify_shared_resource(lock: asyncio.Lock, shared_list: list):
    async with lock:
        # Only one coroutine can be in this block at a time.
        print("Acquired lock...")
        # Simulate reading and then writing to the resource
        current_value = shared_list[-1] if shared_list else 0
        await asyncio.sleep(0.1) # Pause, allowing another task to try to run
        shared_list.append(current_value + 1)
        print("...Released lock")

async def main():
    lock = asyncio.Lock()
    shared_list = []
    tasks = [modify_shared_resource(lock, shared_list) for _ in range(5)]
    await asyncio.gather(*tasks)
    print(f"Final list: {shared_list}") # Should be [1, 2, 3, 4, 5]

# Without the lock, the list would likely be [1, 1, 1, 1, 1] due to the race condition.
# asyncio.run(main())
```

### 4. `asyncio.Queue` (Producer-Consumer Pattern)

The **producer-consumer** pattern is a common concurrency pattern where one or more "producer" tasks create work items and put them into a queue, and one or more "consumer" tasks take items from the queue and process them.

`asyncio.Queue` is a queue designed for async programming.

- `await queue.put(item)`: Adds an item to the queue.
- `await queue.get()`: Removes and returns an item from the queue. It will wait if the queue is empty.
- `queue.task_done()`: Indicates that a formerly enqueued task is complete.
- `await queue.join()`: Blocks until all items in the queue have been received and processed (i.e., `task_done()` has been called for each item).

```python
import asyncio
import random

async def producer(queue: asyncio.Queue):
    for i in range(5):
        item = f"item-{i}"
        await asyncio.sleep(random.random())
        await queue.put(item)
        print(f"Produced {item}")

async def consumer(queue: asyncio.Queue, id: int):
    while True:
        item = await queue.get()
        print(f"Consumer {id} is processing {item}...")
        await asyncio.sleep(random.random() * 2)
        queue.task_done()
        print(f"Consumer {id} finished {item}")

# This pattern allows you to decouple the production of work from its consumption.
```

## 🔹 Quick Exercise

Complete the `rate_limited_fetch` function below. It should use an `asyncio.Semaphore` to ensure that no more than `max_concurrent` requests are active at any given time.

```python
import asyncio

async def fetch_url(url: str):
    print(f"Fetching {url}...")
    await asyncio.sleep(1)
    print(f"...Done with {url}")
    return f"Data from {url}"

async def rate_limited_fetch(urls: list[str], max_concurrent: int = 3):
    semaphore = asyncio.Semaphore(max_concurrent)

    async def fetch_with_limit(url: str):
        async with semaphore:
            return await fetch_url(url)

    tasks = [fetch_with_limit(url) for url in urls]
    await asyncio.gather(*tasks)

# --- Test ---
# asyncio.run(rate_limited_fetch([f"url_{i}" for i in range(10)], 3))
# You should see it fetch 3 URLs at a time.
```

## 📝 Daily Assignment

**Goal**: Build a web crawler that uses producer-consumer and rate-limiting patterns.

1.  **Create Project File**: In your project, create `my_first_poetry_app/crawler.py`. You will need `httpx` and `beautifulsoup4`.
2.  **The `producer` Coroutine**:
    - `async def producer(queue: asyncio.Queue, client: httpx.AsyncClient, start_url: str)`:
    - It should `put` the `start_url` into the queue.
    - You'll need a way to track visited URLs to avoid getting into loops. A `set` is good for this.
3.  **The `consumer` Coroutine**:
    - `async def consumer(queue: asyncio.Queue, client: httpx.AsyncClient, visited_urls: set[str])`:
    - In a loop, it should `get` a URL from the queue.
    - If the URL has already been visited, skip it.
    - Mark the URL as visited.
    - Fetch the HTML of the URL.
    - Use BeautifulSoup to find all other links (`<a href="...">`) on the page.
    - For each new link found, `put` it back into the queue for other consumers to process.
    - Call `queue.task_done()` after processing a URL.
4.  **The `main` coroutine**:
    - Create a `asyncio.Queue` and a `set` for visited URLs.
    - Create a list of consumer tasks (e.g., 5 of them).
    - Create one producer task.
    - Use `asyncio.gather` to start all the tasks.
    - Use `await queue.join()` to wait until the queue is empty (meaning all found pages have been processed).
    - After `join()` completes, cancel all the consumer tasks (as they are in an infinite loop) so the program can exit.
5.  **Rate Limiting (Crucial!)**:
    - Modify your consumer. Wrap the network request part (`await client.get(...)`) in a block controlled by an `asyncio.Semaphore` to limit your crawler to, for example, 5 concurrent requests, to avoid overwhelming the server.

## ⚠️ Common Mistakes

- **Deadlock**: A situation where two or more coroutines are waiting for each other to release a resource, and none can proceed. This can happen with complex lock interactions.
- **Not calling `task_done()`**: When using `queue.join()`, you must call `queue.task_done()` for every item you `get()` from the queue. Forgetting this will cause `queue.join()` to wait forever.
- **Creating a new semaphore for each task**: A semaphore or lock must be created _once_ and then passed as an argument to all the tasks that need to share it.

## 📖 Further Reading

- [Real Python: `asyncio` Concurrency and Synchronization](https://realpython.com/async-io-python/#asyncio-queue)
- [Python Docs: Synchronization Primitives (`Lock`, `Semaphore`)](https://docs.python.org/3/library/asyncio-sync.html)
- [Python Docs: Queues (`Queue`)](https://docs.python.org/3/library/asyncio-queue.html)
