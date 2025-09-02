# Week 3, Day 1: Lists & List Operations

## 🎯 Learning Objectives
- [ ] Master the creation, indexing, and slicing of lists.
- [ ] Understand the concept of mutability and how it applies to lists.
- [ ] Use common list methods to modify lists in place.
- [ ] Write type-safe list comprehensions and generic list functions.

## 📚 Concepts

### 1. List Creation and Indexing
A list is an ordered, mutable collection of items. You can create lists using square brackets `[]`. Lists are 0-indexed, meaning the first item is at index 0.

```python
# A list of integers
numbers: list[int] = [10, 20, 30, 40, 50]

# Accessing items by index
first_item: int = numbers[0]  # 10
third_item: int = numbers[2]  # 30

# Negative indexing starts from the end
last_item: int = numbers[-1] # 50
```

### 2. Mutability
Lists are **mutable**, which means you can change their content after they are created. This is a key difference from immutable types like `str` or `tuple`.

```python
numbers: list[int] = [1, 2, 3]
print(f"Original list ID: {id(numbers)}")

# Modify the list in place
numbers[1] = 99  # changes the item at index 1
numbers.append(4) # adds an item to the end

print(f"Modified list: {numbers}") # [1, 99, 3, 4]
print(f"Modified list ID: {id(numbers)}") # The ID is the same!
```
Because lists are mutable, if you assign a list to a new variable, both variables point to the *same* list object in memory.

### 3. Slicing
Slicing lets you get a sub-list from a list. The syntax is `my_list[start:stop:step]`. The `stop` index is exclusive. Slicing creates a *new* list (a shallow copy).

```python
numbers: list[int] = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# Get items from index 2 up to (but not including) index 5
sub_list: list[int] = numbers[2:5]  # [2, 3, 4]

# Get the first 3 items
first_three: list[int] = numbers[:3] # [0, 1, 2]

# Get all items from index 5 to the end
from_five: list[int] = numbers[5:] # [5, 6, 7, 8, 9]

# Get every other item
evens: list[int] = numbers[::2] # [0, 2, 4, 6, 8]

# A clever way to create a copy of a list
numbers_copy: list[int] = numbers[:]
```

### 4. Common List Methods
These methods modify the list in-place.
- `.append(item)`: Adds a single item to the end of the list.
- `.extend(iterable)`: Adds all items from an iterable to the end.
- `.insert(index, item)`: Inserts an item at a specific index.
- `.remove(item)`: Removes the *first* occurrence of an item. Raises `ValueError` if not found.
- `.pop(index=-1)`: Removes and returns the item at an index (defaults to the last item).
- `.sort()`: Sorts the list in place.

### 5. List Comprehensions
List comprehensions provide a concise and readable way to create lists. They are often more efficient than creating an empty list and using a `for` loop with `.append()`.

```python
# --- Standard for loop ---
squares: list[int] = []
for i in range(10):
    squares.append(i * i)

# --- List comprehension (preferred) ---
squares_comp: list[int] = [i * i for i in range(10)]

# You can also add conditions
even_squares: list[int] = [i * i for i in range(10) if i % 2 == 0]
```

### 6. Generic List Functions with `TypeVar`
What if you want to write a function that works on a list of *any* type, but you want the type checker to know that the output list has the same type as the input? You use `TypeVar`.

```python
from typing import TypeVar, List

# T is a "Type Variable". It can stand for any type.
T = TypeVar('T')

def first_item(items: List[T]) -> T | None:
    """Returns the first item of a list, or None if empty."""
    if not items:
        return None
    return items[0]

# MyPy knows that if you pass a list[str], the result is str | None.
# If you pass a list[int], the result is int | None.
names: list[str] = ["Alice", "Bob"]
first_name: str | None = first_item(names) # Type is str | None
```

## 🔹 Quick Exercise

Implement the following two generic functions. Use the `TypeVar` `T` we defined above.

```python
from typing import TypeVar, List

T = TypeVar('T')

def rotate_list(items: List[T], n: int) -> List[T]:
    """
    Rotates a list by n positions to the right.
    Example: rotate_list([1, 2, 3, 4], 1) -> [4, 1, 2, 3]
    """
    # Hint: Use slicing. A positive n rotates right, negative n rotates left.
    # The modulo operator (%) might be useful to handle large n.
    pass

def flatten(nested_list: List[List[T]]) -> List[T]:
    """
    Flattens a list of lists into a single list.
    Example: flatten([[1, 2], [3, 4]]) -> [1, 2, 3, 4]
    """
    # Hint: Use a list comprehension with a nested for loop.
    pass


# --- Tests ---
assert rotate_list([1, 2, 3, 4, 5], 2) == [4, 5, 1, 2, 3]
assert rotate_list(['a', 'b', 'c'], -1) == ['b', 'c', 'a']
assert flatten([[1], [2, 3], [4]]) == [1, 2, 3, 4]
assert flatten([['a', 'b'], ['c']]) == ['a', 'b', 'c']
```

## 📝 Daily Assignment
**Goal**: Implement several list-based algorithms and a simple custom list-like class.

1.  **Create Project File**: In `my-first-poetry-app`, create a new file: `my_first_poetry_app/list_utils.py`.
2.  **Implement List Algorithms**:
    - `running_sum(numbers: list[float]) -> list[float]`: Returns a new list where each element is the sum of all previous elements in the original list. E.g., `[1, 2, 3]` -> `[1, 3, 6]`.
    - `remove_duplicates(items: list[T]) -> list[T]`: Returns a new list with duplicate items removed, preserving the original order. (Hint: you can't use a `set` directly if you need to preserve order).
    - `pairwise_sum(list1: list[int], list2: list[int]) -> list[int]`: Returns a new list where each element is the sum of the elements at the same index in the two input lists. The output list should be the length of the shorter input list. Use `zip`.
3.  **Performance Considerations**:
    - Write two functions to find common items between two lists:
      - `find_common_brute_force(list1: list, list2: list) -> list`: Uses nested `for` loops.
      - `find_common_optimized(list1: list, list2: list) -> list`: Uses a `set` for faster lookups.
    - Write a `main` function to time both functions on large lists (e.g., 10,000 items each) using the `time` module. Print the performance difference.
4.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes
- **`my_list.sort()` vs `sorted(my_list)`**:
  - `my_list.sort()` sorts the list *in-place* and returns `None`. A common bug is `new_list = my_list.sort()`, which makes `new_list` `None`.
  - `sorted(my_list)` returns a *new*, sorted list and leaves the original unchanged.
- **Shallow Copies**: Assigning `new_list = old_list` does not create a copy. Both variables point to the same list. Slicing (`new_list = old_list[:]`) creates a *shallow copy*. If the list contains other mutable objects (like other lists), those inner objects are *not* copied.
- **`append` vs `extend`**: `append` adds its argument as a single element. `extend` iterates over its argument and adds each item.
  ```python
  x = [1, 2]
  x.append([3, 4]) # x is now [1, 2, [3, 4]]

  y = [1, 2]
  y.extend([3, 4]) # y is now [1, 2, 3, 4]
  ```

## 📖 Further Reading
- [Python Docs: More on Lists](https://docs.python.org/3/tutorial/datastructures.html#more-on-lists)
- [Real Python: Python Lists and Tuples](https://realpython.com/python-lists-tuples/)
- [Python Docs: `typing.TypeVar`](https://docs.python.org/3/library/typing.html#typing.TypeVar)
- [Real Python: Python List Comprehension](https://realpython.com/list-comprehension-python/)
