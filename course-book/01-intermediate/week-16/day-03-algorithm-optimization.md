# Week 16, Day 3: Algorithm & Data Structure Optimization

## 🎯 Learning Objectives
- [ ] Revisit Big O notation to analyze algorithm complexity.
- [ ] Understand how choosing the right data structure is a critical optimization technique.
- [ ] Implement caching and memoization to avoid re-computing expensive function calls.
- [ ] Grasp the concept of the space-time tradeoff in algorithm design.
- [ ] Learn to recognize when a more complex algorithm is necessary for performance.

## 📚 Concepts

### 1. It's All About the Algorithm
After you've profiled your code and found a bottleneck, the most significant performance gains often come not from micro-optimizing Python code, but from changing the underlying **algorithm** or **data structure**.

Switching from an O(n²) algorithm to an O(n log n) algorithm will have a far greater impact than any small-scale code tweak as your input size (`n`) grows.

**Big O Notation Recap:**
-   **O(1) - Constant Time**: The time taken is independent of the input size. (e.g., dictionary lookup by key).
-   **O(log n) - Logarithmic Time**: Time increases logarithmically with input size. (e.g., binary search on a sorted list).
-   **O(n) - Linear Time**: Time is directly proportional to the input size. (e.g., iterating through a list).
-   **O(n log n) - Log-Linear Time**: A common complexity for efficient sorting algorithms (e.g., Timsort, Merge Sort).
-   **O(n²) - Quadratic Time**: Time is proportional to the square of the input size. (e.g., nested loops over the same list).
-   **O(2ⁿ) - Exponential Time**: Time doubles with each addition to the input. (e.g., a naive recursive Fibonacci calculation).

Your goal is often to move down this list to a lower complexity class.

### 2. The Right Data Structure for the Job
The choice of data structure has a massive impact on performance. The key is to understand the time complexity of their common operations.

| Operation         | `list`        | `set`         | `dict`        |
|-------------------|---------------|---------------|---------------|
| **Get Item**      | O(1) (by index) | -             | O(1) (by key) |
| **Set Item**      | O(1) (by index) | -             | O(1) (by key) |
| **Check for Item (`in`)** | **O(n)**      | **O(1)**      | **O(1)**      |
| **Add Item**      | O(1) (append) | O(1)          | O(1)          |
| **Delete Item**   | O(n)          | O(1)          | O(1)          |

**The Golden Rule**: If you need to frequently check for the existence of an item (`if item in my_collection:`), you should almost always use a `set` or a `dict`, not a `list`.

**Inefficient Example (O(n²)):**
```python
# Find common elements between two large lists
list1 = list(range(10000))
list2 = list(range(5000, 15000))

# This is O(n*m) because for each item in list1, we do an O(m) search in list2.
common = []
for item in list1:
    if item in list2: # This check is slow!
        common.append(item)
```

**Efficient Example (O(n + m)):**
```python
# Convert one list to a set for fast lookups
set2 = set(list2) # This takes O(m) time

# The 'in' check against a set is O(1) on average.
# The total time is O(m) to build the set + O(n) for the loop.
common = [item for item in list1 if item in set2]
```

### 3. Caching & Memoization
If you have a pure function (one that always returns the same output for the same input) that is expensive to run, you can get a huge performance boost by **caching** its results.

**Memoization** is a specific type of caching where we cache the return values of a function. Python's standard library makes this incredibly easy with `functools.lru_cache`.

**Inefficient Recursive Fibonacci (O(2ⁿ)):**
```python
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    # This is horribly inefficient because fibonacci(n-2), fibonacci(n-3), etc.
    # are re-calculated many, many times.
    return fibonacci(n - 1) + fibonacci(n - 2)
```

**Memoized Fibonacci with `lru_cache` (O(n)):**
```python
from functools import lru_cache

@lru_cache(maxsize=None) # maxsize=None means the cache can grow indefinitely
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    # If fibonacci(n-1) has been calculated, it's retrieved from the cache instantly.
    return fibonacci(n - 1) + fibonacci(n - 2)

# This now runs almost instantly, even for large n.
# fibonacci(40)
```
`lru_cache` ("Least Recently Used" Cache) is a decorator that wraps your function in a memoizing cache.

### 4. The Space-Time Tradeoff
This is a fundamental concept in computer science. You can often make a program faster by using more memory, or make it use less memory by making it slower.

-   **Caching** is a classic example. You use more memory to store cached results, which saves the time of re-computing them.
-   The **Set vs. List** example also shows this. We used extra memory to create `set2`, which made the lookup algorithm much faster.
-   **Pre-computation**: If you have a complex calculation, you might perform it once at the start of your program and store the results in a lookup table (using memory) to make subsequent lookups very fast (saving time).

There is no "right" answer; the correct tradeoff depends on the constraints of your application (e.g., are you running on a memory-constrained device?).

## 🔹 Quick Exercise
You have a function `get_user_permissions(user_id: int)` that makes a slow database call. You want to cache the results for 10 minutes. How would you apply a cache to this function?

While `lru_cache` doesn't have a built-in time limit, what is the *concept* you would use?

**Answer:**
You would use **caching** (specifically, memoization with a Time-To-Live). You would store the result of the database call in a cache (like a dictionary or Redis) with the `user_id` as the key. You would also store a timestamp. Before making a new database call, you would check the cache. If the key exists and the timestamp is less than 10 minutes old, you return the cached result. Otherwise, you make the database call and update the cache.

The `timed_cache` decorator from Week 13, Day 5 is a perfect example of how to implement this.

## 📝 Daily Assignment
**Goal**: Optimize a slow data processing script by applying algorithmic and data structure improvements.

1.  **The Slow Script**: Create a file `slow_script.py`.
    ```python
    import random

    def find_duplicate_transactions(transactions: list[str]) -> list[str]:
        """A very inefficient way to find duplicate transaction IDs."""
        duplicates = []
        for i in range(len(transactions)):
            for j in range(i + 1, len(transactions)):
                if transactions[i] == transactions[j] and transactions[i] not in duplicates:
                    duplicates.append(transactions[i])
        return duplicates

    def get_user_level(user_id: int) -> str:
        """Simulates a slow lookup for a user's access level."""
        # In a real app, this would be a database call.
        levels = {1: "admin", 2: "user", 3: "user", 4: "guest"}
        # Naive search
        for uid, level in levels.items():
            if uid == user_id:
                return level
        return "unknown"

    def main():
        # Generate a large list of transactions with some duplicates
        all_transactions = [f"TXN{random.randint(1000, 10000)}" for _ in range(20000)]
        all_transactions.extend(all_transactions[:50]) # Add 50 duplicates
        random.shuffle(all_transactions)

        print("Finding duplicate transactions...")
        duplicates = find_duplicate_transactions(all_transactions)
        print(f"Found duplicates: {duplicates}")

        print("\nChecking user levels...")
        for i in range(1, 6):
            # This lookup is done repeatedly
            level = get_user_level(i)
            print(f"Level for user {i}: {level}")
            level = get_user_level(i)
            print(f"Level for user {i} (again): {level}")

    if __name__ == "__main__":
        main()
    ```
2.  **Profile**: Run the script and notice how slow it is. Use `cProfile` to confirm which function is the main bottleneck.

3.  **Optimize `find_duplicate_transactions`**:
    -   This function is O(n²).
    -   Rewrite it to be more efficient. **Hint**: Use a `set` to keep track of the items you've already seen. The new version should be O(n).

4.  **Optimize `get_user_level`**:
    -   This function is repeatedly called with the same arguments.
    -   This is a perfect use case for memoization.
    -   Apply the `@functools.lru_cache` decorator to this function.

5.  **Benchmark**:
    -   Time the execution of the original `slow_script.py`.
    -   Time the execution of your new, optimized `optimized_script.py`.
    -   In a markdown file `benchmark.md`, report the time difference and explain *why* your changes made the script faster, referencing the time complexities of the data structures and algorithms you used.

## 📖 Further Reading
- [Python Docs: `functools.lru_cache`](https://docs.python.org/3/library/functools.html#functools.lru_cache)
- [Real Python: Python's Timsort](https://realpython.com/python-timsort/) (An example of a highly optimized algorithm)
- [Problem Solving with Algorithms and Data Structures using Python](https://runestone.academy/ns/books/published/pythonds/index.html) (An excellent free online textbook)
