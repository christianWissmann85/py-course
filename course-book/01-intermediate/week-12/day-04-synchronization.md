# Week 12, Day 4: Synchronization Primitives

## 🎯 Learning Objectives

- [ ] Deepen understanding of `Lock` objects for mutual exclusion.
- [ ] Learn to use `RLock` (Re-entrant Lock) for complex synchronization scenarios.
- [ ] Use `Semaphore` objects to limit access to a resource.
- [ ] Understand how to use `Event` objects to signal between threads.
- [ ] Learn about `Barrier` objects for synchronizing a fixed number of threads at a specific point.

## 📚 Concepts

So far, we've used `threading.Lock` to prevent race conditions. A lock is the most fundamental **synchronization primitive**, but the `threading` module provides several others that are useful for more complex coordination between threads.

### 1. `Lock` vs. `RLock` (Re-entrant Lock)

A standard `threading.Lock` cannot be acquired more than once by the same thread. If a thread owns a lock and tries to acquire it again, it will block forever—a deadlock.

```python
import threading

lock = threading.Lock()
print("Acquiring lock for the first time...")
lock.acquire()
print("Acquired.")

# This next line will cause the program to hang, because the lock is not "re-entrant"
# print("Acquiring lock for the second time...")
# lock.acquire()
# print("This will never be printed.")
```

A **re-entrant lock (`threading.RLock`)** _can_ be acquired multiple times by the same thread. It keeps a counter. `acquire()` increments the counter, and `release()` decrements it. The lock is only fully released for other threads to acquire when the counter is zero.

This is useful in recursive functions or in situations where a function that needs a lock calls another function that _also_ needs that same lock.

```python
import threading

# With RLock, this works fine.
rlock = threading.RLock()

def recursive_function(depth):
    with rlock:
        print(f"RLock acquired at depth {depth}.")
        if depth > 0:
            recursive_function(depth - 1)
    print(f"RLock released at depth {depth}.")

# recursive_function(3)
```

### 2. `Semaphore`

A semaphore is a counter that is used to limit the number of threads that can access a resource simultaneously. A `Lock` is essentially a semaphore with a count of 1.

You initialize a semaphore with a value. `acquire()` decrements the counter, and `release()` increments it. If a thread calls `acquire()` when the counter is zero, it will block until another thread calls `release()`.

This is perfect for scenarios like limiting the number of concurrent connections to a database or the number of downloads happening at once.

```python
import threading
import time
import random

# This semaphore will allow up to 3 threads to run the 'worker' function at a time.
max_concurrent_downloads = 3
semaphore = threading.Semaphore(value=max_concurrent_downloads)

def worker(name: str):
    print(f"Thread {name}: waiting to acquire semaphore.")
    with semaphore:
        # This 'with' block will only be entered by 3 threads at a time.
        print(f"Thread {name}: semaphore acquired. Starting work...")
        time.sleep(random.randint(1, 3))
        print(f"Thread {name}: work finished. Releasing semaphore.")

threads = []
for i in range(10):
    t = threading.Thread(target=worker, args=(f"Worker-{i}",))
    threads.append(t)
    t.start()

for t in threads:
    t.join()
```

### 3. `Event`

An `Event` object is a simple but powerful tool for one thread to signal information to other threads. It's like a flag. One thread can set the event, and other threads can wait for it to be set.

- `event.set()`: Sets the internal flag to true. All threads waiting for it are awakened.
- `event.clear()`: Resets the internal flag to false.
- `event.wait()`: Blocks until the internal flag is true.
- `event.is_set()`: Returns `True` if the flag is set.

A common use case is a "start signal". Multiple worker threads can be set up and told to `wait()` on an event. When the main thread is ready, it calls `set()`, and all workers start their tasks at roughly the same time.

```python
import threading
import time

event = threading.Event()

def worker():
    print("Worker: waiting for the event to be set.")
    event.wait() # This will block until event.set() is called
    print("Worker: event has been set! Starting my task.")

t = threading.Thread(target=worker)
t.start()

print("Main: preparing some resources...")
time.sleep(2)
print("Main: resources are ready. Setting the event.")
event.set()

t.join()
```

### 4. `Barrier`

A `Barrier` is used to make a fixed number of threads wait for each other at a certain point in their execution before they are all allowed to proceed.

You create a barrier with a specific number of parties (threads). Each thread calls `barrier.wait()`. The call will block until all `parties` number of threads have called `wait()`. At that point, they are all released simultaneously.

This is useful for synchronizing stages of a multi-threaded algorithm.

```python
import threading
import time
import random

# This barrier requires 3 threads to reach it before any can proceed.
num_threads = 3
barrier = threading.Barrier(num_threads)

def worker(name: str):
    print(f"[{name}] Starting part 1 of my work.")
    time.sleep(random.randint(1, 4))
    print(f"[{name}] Finished part 1. Waiting at the barrier.")

    try:
        # The last thread to arrive will get a non-negative integer return value.
        # Others get -1.
        wait_result = barrier.wait(timeout=5)
        print(f"[{name}] Passed the barrier (wait_result: {wait_result}). Starting part 2.")
    except threading.BrokenBarrierError:
        print(f"[{name}] Barrier was broken! Exiting.")

threads = []
for i in range(num_threads):
    t = threading.Thread(target=worker, args=(f"Thread-{i}",))
    threads.append(t)
    t.start()

for t in threads:
    t.join()
```

If one of the waiting threads times out or is reset, the barrier is considered "broken", and any other threads waiting on it will raise a `threading.BrokenBarrierError`.

## 🔹 Quick Exercise

You are building a system that processes data in three stages. Three different worker threads are responsible for each stage.

- Thread A fetches data.
- Thread B processes the data.
- Thread C saves the data.

Thread B cannot start until Thread A is finished. Thread C cannot start until Thread B is finished. Use two `Event` objects to coordinate this workflow.

## 📝 Daily Assignment

**Goal**: Build a simulation of a "producer-consumer" problem using a `Queue` and synchronization primitives.

The **producer-consumer problem** is a classic concurrency pattern. One or more "producers" generate data and put it into a shared buffer (a queue). One or more "consumers" take data from the buffer and process it.

**Your Task**:

1.  **Create Project File**: `my_first_poetry_app/producer_consumer.py`.
2.  **The Shared `Queue`**: Create a `queue.Queue` object. To make the problem more interesting, give it a `maxsize`, for example, `queue.Queue(maxsize=5)`. This means the queue will block producers if it's full.
3.  **Implement the Producer**:
    - Create a `producer` function that runs in a thread.
    - It should loop a certain number of times (e.g., 20).
    - In each loop, it should generate a piece of data (e.g., a random number or a string like `"item-X"`).
    - It then puts the item onto the shared queue using `queue.put()`.
    - Add `print` statements to show what the producer is doing.
4.  **Implement the Consumer**:
    - Create a `consumer` function that runs in a thread.
    - It should loop forever (`while True`).
    - In the loop, it gets an item from the queue using `queue.get()`. This will block if the queue is empty.
    - It processes the item (e.g., just printing it is fine).
    - After getting and processing, it must call `queue.task_done()`. This is crucial for the final step.
5.  **Putting It Together**:
    - In your main script, create the shared queue.
    - Create and start one or more producer threads.
    - Create and start one or more consumer threads. **Important**: Consumer threads should be daemon threads (`threading.Thread(target=consumer, daemon=True)`). This means they will be shut down automatically when the main program exits.
    - After starting the producers, the main thread should wait for them to finish their work. Use `thread.join()` for each producer thread.
    - After the producers are done, the main thread needs to wait for the consumers to finish processing all the items that are left in the queue. The elegant way to do this is `queue.join()`. This call will block until every item that was ever `.put()` into the queue has had `task_done()` called for it.
    - Finally, print a "Finished" message.

## 📖 Further Reading

- [Real Python: An Intro to Threading in Python (Covers all these primitives)](https://realpython.com/intro-to-python-threading/)
- [Python Docs: `threading` — Synchronization Objects](https://docs.python.org/3/library/threading.html#lock-objects)
