# Week 12, Day 5: Performance & Best Practices

## 🎯 Learning Objectives

- [ ] Review when to use Threads vs. Processes vs. Asyncio.
- [ ] Understand the overhead associated with threads and processes.
- [ ] Learn best practices for writing clean, safe, and maintainable concurrent code.
- [ ] Learn basic techniques for profiling and debugging concurrent applications.

## 📚 Concepts

### 1. The Grand Summary: Threads vs. Processes vs. Asyncio

We've covered a lot of ground. Let's consolidate the core concepts into a clear decision-making framework.

| Feature              | `threading`                                                 | `multiprocessing`                                  | `asyncio` (Preview)                                        |
| -------------------- | ----------------------------------------------------------- | -------------------------------------------------- | ---------------------------------------------------------- |
| **Best For**         | **I/O-bound** tasks (network, disk)                         | **CPU-bound** tasks (heavy computation)            | **High-volume I/O-bound** tasks (e.g., 10k+ connections)   |
| **Concurrency Type** | Pre-emptive multitasking (OS switches threads)              | Parallelism (multiple CPU cores)                   | Cooperative multitasking (tasks yield control)             |
| **GIL Impact**       | Yes, only one thread runs Python bytecode at a time.        | No, each process has its own GIL.                  | Yes, but it's less of an issue as code is single-threaded. |
| **Memory**           | Shared memory (easy to share data, risk of race conditions) | Separate memory (data sharing requires IPC, safer) | Shared memory (single thread, no race conditions)          |
| **Overhead**         | Low (creating a thread is cheap)                            | High (creating a process is expensive)             | Very Low (creating a task is extremely cheap)              |
| **Complexity**       | Medium (requires locks for safety)                          | High (requires IPC, `if __name__ == "__main__"`)   | High (requires a different programming model)              |

**Rule of Thumb:**

1.  Is your task **CPU-bound**? -> Use **`multiprocessing`**.
2.  Is your task **I/O-bound**? -> Use **`threading`**.
3.  Are you dealing with a massive number of I/O-bound tasks where efficiency is critical? -> Consider **`asyncio`**.

### 2. Understanding Overhead

Concurrency is not free. There is always a performance cost.

- **Thread Overhead**: Creating and managing threads has a cost in both memory and CPU time. If your tasks are extremely short, the overhead of threading might be greater than the time saved by running them concurrently.
- **Process Overhead**: This is much higher than thread overhead. A new process requires the OS to allocate a separate memory space and initialize a new Python interpreter. This can take significant time. Furthermore, passing data between processes (IPC) is much slower than accessing shared memory in threads.
- **Locking Overhead**: Acquiring and releasing locks is not instantaneous. In high-contention scenarios (many threads trying to acquire the same lock), threads can spend more time waiting for locks than doing useful work. This is called **lock contention**.

**Amdahl's Law**: This law states that the speedup from parallelization is limited by the portion of the program that is sequential. If 10% of your program must run sequentially, you can never achieve more than a 10x speedup, no matter how many cores you add.

### 3. Best Practices for Concurrent Code

1.  **Keep Locked Sections Small and Fast**: The "critical section" (the code inside a `with lock:`) should be as short as possible. Do all non-shared work _outside_ the lock to minimize lock contention.
2.  **Avoid Shared State When Possible**: The best way to avoid race conditions is to avoid sharing mutable state altogether. If each thread works on its own data and only communicates results back via a queue, the need for locks diminishes dramatically.
3.  **Use High-Level Abstractions**: Prefer `concurrent.futures` over the `threading` and `multiprocessing` modules directly. The code is cleaner, safer (executors handle joining automatically), and easier to switch between threads and processes.
4.  **Always Use Context Managers (`with` statement)**: For locks, pools, and executors. This guarantees that resources are properly acquired and released, even if errors occur.
5.  **Be Wary of Deadlocks**: A simple way to avoid deadlocks is to ensure that all threads acquire multiple locks in the same, consistent order.
6.  **Use Daemon Threads Responsibly**: `daemon=True` is useful for background tasks that you don't need to wait for (like logging or monitoring). But be careful: if the main program exits, daemon threads are terminated abruptly, which could leave resources in a bad state.
7.  **Guard Your `multiprocessing` Code**: Always, always, always put the code that starts processes inside an `if __name__ == "__main__":` block.

### 4. Profiling and Debugging

Debugging concurrent code is notoriously difficult because the bugs (like race conditions) can be non-deterministic. They might appear on one run and disappear on the next.

- **Profiling**: Python's built-in `cProfile` module can help you understand where your program is spending its time. However, it's not always great at showing time spent waiting on I/O or locks.
  - `python -m cProfile -o my_program.prof my_program.py`
  - Tools like `snakeviz` can then be used to visualize the output: `snakeviz my_program.prof`.
- **Logging is Your Best Friend**: Use the `logging` module. Have each thread log what it's doing with its thread ID (`threading.get_ident()`) or name. This creates a timeline that you can inspect to understand the sequence of events.
- **Simplify and Reproduce**: If you have a bug, try to create the smallest possible version of the program that can still reproduce it. Reduce the number of threads, simplify the shared data, and add extensive logging.
- **Reasoning About State**: For a potential race condition, stop and think: "What happens if the OS pauses this thread right here and runs another one?" Walk through the possible interleavings of execution in your head or on a whiteboard.

## 🔹 Quick Exercise

Consider the following code snippet. Identify potential performance issues or violations of best practices. How would you improve it?

```python
import threading

# Global variables are a form of shared state
my_data = []
my_lock = threading.Lock()

def process_data():
    # This function does a lot of work while holding the lock
    my_lock.acquire()

    # Step 1: Some initial computation
    new_items = [i*i for i in range(1000)]

    # Step 2: Modify the shared data
    my_data.extend(new_items)

    # Step 3: Some final computation on the shared data
    total = sum(my_data)
    print(f"Current total is {total}")

    my_lock.release()
```

**Answer**:

1.  The lock is held for the entire duration of the function. The initial computation (`new_items = ...`) does not depend on shared data and should be done _before_ acquiring the lock.
2.  The `print` statement also holds the lock, which is unnecessary.
3.  It doesn't use a `with` statement, so if an error occurs between `acquire` and `release`, the lock is never released.

**Improved Version**:

```python
def process_data_improved():
    # Step 1: Do work that doesn't need the lock first.
    new_items = [i*i for i in range(1000)]

    # Use a context manager for the lock
    with my_lock:
        # Step 2: The critical section is now much smaller.
        my_data.extend(new_items)
        # Reading the data for the print statement might still need the lock
        # depending on requirements, but let's assume it does for now.
        total = sum(my_data)

    # Step 3: The print can happen outside the lock.
    print(f"Current total is {total}")
```

## 📝 Weekly Project

**Goal**: Build a parallel web crawler.

A web crawler starts at a given URL, downloads the page, parses it for new links, and then follows those links to crawl more pages, up to a certain depth or number of pages.

**Your Task**:

1.  **Project Setup**: In your Poetry project, create `my_first_poetry_app/web_crawler.py`. You will need `requests` for downloading and `BeautifulSoup4` for parsing HTML. (`poetry add beautifulsoup4`).
2.  **Core Logic (Single-threaded first!)**:
    - Write a function `crawl(url: str)` that:
      - Downloads the content of the URL.
      - Parses the HTML to find all `<a>` tags.
      - Extracts the `href` attribute from each tag.
      - Cleans and normalizes these links (e.g., turn relative URLs like `/about` into absolute URLs like `http://example.com/about`).
      - Returns a set of all valid, new URLs found on the page.
3.  **Concurrent Implementation**:
    - Use a `ThreadPoolExecutor` to manage the crawling tasks.
    - You need to keep track of URLs you have already visited to avoid crawling the same page multiple times or getting into infinite loops. A `set` is perfect for this.
    - You also need a way to manage the queue of URLs that are waiting to be crawled. A `collections.deque` is a good choice.
    - **This is a tricky problem because you have shared state**: the set of visited URLs and the queue of URLs to visit. You **must** use a `threading.Lock` to protect access to both of these data structures.
4.  **Putting It Together (The Algorithm)**:
    - Initialize:
      - `urls_to_crawl = collections.deque([start_url])`
      - `visited_urls = {start_url}`
      - `lock = threading.Lock()`
    - The main loop:
      - Create a `ThreadPoolExecutor`.
      - As long as there are URLs in `urls_to_crawl`, submit new tasks to the executor.
      - Be careful not to submit more tasks than you intend to run concurrently. You can use a `Semaphore` or simply manage the number of active `Future` objects.
      - When a `crawl` task finishes, it will return a set of new URLs. In a "done" callback (or after getting the `.result()`), you need to:
        - Acquire the lock.
        - For each new URL, check if it's in `visited_urls`. If not, add it to `visited_urls` and `urls_to_crawl`.
        - Release the lock.
5.  **Set a Limit**: Your crawler should stop after a certain number of pages have been crawled or a certain depth has been reached.

This project combines I/O-bound work, a high-level executor, and manual synchronization, touching on all the key topics of this week.
