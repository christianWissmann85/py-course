# Week 16, Day 2: Memory Management in Python

## 🎯 Learning Objectives

- [ ] Understand Python's memory management model, including the private heap.
- [ ] Learn how reference counting works and why it's the primary mechanism for memory management.
- [ ] Understand the role of the cyclic garbage collector in breaking reference cycles.
- [ ] Identify common causes of memory leaks in Python applications.
- [ ] Learn to use `weakref` to create references that don't prevent garbage collection.

## 📚 Concepts

### 1. Python's Memory Model

Unlike languages like C or C++, Python manages memory automatically. You don't need to manually `malloc` or `free` memory. Python's memory manager takes care of this.

- **Private Heap**: Python manages its own memory space, called the private heap. All Python objects and data structures are located in this heap. The programmer does not have access to this heap; it's the Python interpreter that manages it.
- **Object Allocation**: When you create an object (e.g., `x = 10` or `my_list = []`), Python's memory manager allocates a chunk of memory from the heap to store that object.

### 2. Reference Counting

The primary way Python knows when to deallocate an object is through **reference counting**.

- Every object in memory has a count of how many variables (references) are pointing to it.
- When a new reference points to an object, its reference count is incremented.
- When a reference is destroyed (e.g., it goes out of scope or is reassigned), the object's reference count is decremented.
- **When the reference count reaches zero, the object's memory is immediately deallocated.**

```python
import sys

# Create a list object.
# The variable 'a' now points to it. Ref count is 1.
a = []
print(f"Ref count for []: {sys.getrefcount(a) - 1}") # Output: 1

# Create another reference to the same list.
# Now 'a' and 'b' both point to it. Ref count is 2.
b = a
print(f"Ref count for []: {sys.getrefcount(a) - 1}") # Output: 2

# Destroy one reference.
# 'b' now points to None. Ref count for [] is back to 1.
b = None
print(f"Ref count for []: {sys.getrefcount(a) - 1}") # Output: 1

# Destroy the last reference.
# The ref count for [] becomes 0, and it is deallocated.
a = None
# At this point, the list object is gone from memory.
```

_Note: `sys.getrefcount()` itself creates a temporary reference, so its result is always one higher than the actual count._

### 3. The Cyclic Garbage Collector

Reference counting has one major weakness: **reference cycles**. This happens when objects refer to each other, creating a loop where their reference counts will never drop to zero, even if they are no longer accessible from anywhere else in the program.

```python
# A simple reference cycle
a = {}
b = {}
a['b_ref'] = b # a points to b
b['a_ref'] = a # b points to a

# Even if we delete our main references to a and b...
del a
del b

# ...the objects still exist in memory!
# a's ref count is 1 (from b's reference)
# b's ref count is 1 (from a's reference)
```

To solve this, Python has a supplemental **cyclic garbage collector (GC)**.

- The GC runs periodically.
- It is designed specifically to find and break these reference cycles.
- It identifies groups of objects that are only reachable from within the group itself and cleans them up.
- You can interact with it via the `gc` module (`gc.collect()`, `gc.disable()`, etc.), but you rarely need to.

### 4. Memory Leaks in Python

Even with automatic memory management, memory leaks can still happen. A memory leak occurs when memory is allocated but never released, causing the application's memory footprint to grow over time.

**Common Causes:**

- **Growing Global Objects**: Appending data to a global list or dictionary that never gets cleared. The references in the global object keep the data alive forever.
- **Unclosed Resources**: Forgetting to close files or network connections can leave memory buffers allocated. Using `with` statements helps prevent this.
- **Caches without Limits**: Caching objects (e.g., in a dictionary) to speed up lookups is common. If this cache can grow indefinitely, it becomes a memory leak.
- **Reference Cycles with `__del__`**: If an object in a reference cycle has a `__del__` method, the garbage collector cannot safely break the cycle and the objects may leak. This is a complex edge case.

### 5. `weakref`: Avoiding Unwanted References

Sometimes you need to refer to an object without increasing its reference count. This is useful for building caches or object graphs where you don't want the cache itself to keep an object alive.

The `weakref` module allows you to do this. A **weak reference** is a special reference that doesn't prevent the object from being garbage collected.

```python
import weakref

class MyObject:
    def __init__(self, name):
        self.name = name
    def __repr__(self):
        return f"MyObject({self.name})"

# Create a regular (strong) reference and a weak reference
obj = MyObject("my_obj")
weak_obj_ref = weakref.ref(obj)

# You can access the object by calling the weak reference
print(f"Object via weak ref: {weak_obj_ref()}") # Output: MyObject(my_obj)

# Now, delete the only strong reference
del obj

# The object is garbage collected. The weak reference now points to None.
print(f"Object via weak ref after del: {weak_obj_ref()}") # Output: None
```

`weakref.WeakValueDictionary` and `weakref.WeakKeyDictionary` are useful for building caches that don't keep their contents alive.

## 🔹 Quick Exercise

Consider the following code. After the last line is executed, will the `MyObject` instance be garbage collected? Why or why not?

```python
class MyObject:
    pass

my_global_cache = []

def create_and_cache():
    instance = MyObject()
    my_global_cache.append(instance)
    print("Instance created and cached.")

create_and_cache()
# Is the MyObject instance still in memory here?
```

**Answer:**
Yes, the `MyObject` instance is still in memory. Although the `instance` variable inside the function went out of scope, a strong reference to the object was added to the `my_global_cache` list. Since `my_global_cache` is a global variable, it (and the reference it holds) will persist for the life of the program, preventing the `MyObject` instance from being garbage collected. This is a classic example of a memory leak.

## 📝 Daily Assignment

**Goal**: Use `weakref` and `tracemalloc` to find and fix a memory leak.

1.  **The Leaky Code**: Create a file `leaky_app.py` with this code:

    ```python
    import tracemalloc
    import gc

    class Game:
        def __init__(self, name):
            self.name = name
            self.players = []

    class Player:
        def __init__(self, name, game):
            self.name = name
            # This creates a strong reference cycle!
            self.game = game

    def run_game():
        # Start tracking memory allocations
        tracemalloc.start()

        # Create a game and players
        game = Game("The Big Game")
        for i in range(5):
            player = Player(f"Player-{i}", game)
            game.players.append(player)

        print(f"Game '{game.name}' has {len(game.players)} players.")

        # Take a snapshot of memory usage
        snapshot1 = tracemalloc.take_snapshot()

        # The game should be cleaned up here, but it won't be...
        del game
        gc.collect() # Manually trigger the GC

        # Take a second snapshot to compare
        snapshot2 = tracemalloc.take_snapshot()

        # Compare the two snapshots
        top_stats = snapshot2.compare_to(snapshot1, 'lineno')

        print("\n[ Top 10 memory differences after del game ]")
        for stat in top_stats[:10]:
            print(stat)

    if __name__ == "__main__":
        run_game()
    ```

2.  **Run and Analyze**:

    - Run the script: `python leaky_app.py`.
    - Observe the output from `tracemalloc`. It will show that even after `del game` and `gc.collect()`, memory is still being used by the `Game` and `Player` objects. This is because the `Game` holds references to `Player`s, and each `Player` holds a reference back to the `Game`, creating a cycle.

3.  **Fix the Leak**:

    - Modify the `Player` class to use a **weak reference** to the game.
    - Import the `weakref` module.
    - Change `self.game = game` to `self.game = weakref.ref(game)`.
    - You may need to adjust how you access the game from the player (by calling the weak reference: `self.game()`).

4.  **Verify the Fix**:
    - Run the modified script again.
    - The `tracemalloc` output should now show little to no difference between the two snapshots, indicating that the `Game` and `Player` objects were successfully garbage collected after `del game` was called.

## 📖 Further Reading

- [Python Docs: `gc` — Garbage Collector interface](https://docs.python.org/3/library/gc.html)
- [Python Docs: `weakref` — Weak references](https://docs.python.org/3/library/weakref.html)
- [Python Docs: `tracemalloc` — Trace memory allocations](https://docs.python.org/3/library/tracemalloc.html)
- [Real Python: Memory Management in Python](https://realpython.com/python-memory-management/)
