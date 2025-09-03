# Week 3, Day 3: Sets & Set Operations

## 🎯 Learning Objectives
- [ ] Master the fundamental set operations: union, intersection, difference, and symmetric difference.
- [ ] Understand the core properties of sets: uniqueness and unorderedness.
- [ ] Use set comprehensions for a concise way to create sets.
- [ ] Differentiate between `set` and `frozenset` and know when to use each.

## 📚 Concepts

### 1. Set Creation and Uniqueness
A `set` is an **unordered** collection of **unique** items. This makes them perfect for tasks like removing duplicates from a list or checking for membership. The items in a set must be immutable (or more accurately, hashable).

```python
# Create a set from a list to get unique items
numbers: list[int] = [1, 2, 2, 3, 4, 4, 4, 5]
unique_numbers: set[int] = set(numbers)
print(unique_numbers)  # {1, 2, 3, 4, 5} - order is not guaranteed

# Create a set with literals
vowels: set[str] = {'a', 'e', 'i', 'o', 'u'}
```
**Note**: To create an empty set, you must use `set()`, not `{}`. `{}` creates an empty dictionary.

### 2. Set Operations
Sets support powerful mathematical operations.
- **Union (`|`)**: All items from both sets.
- **Intersection (`&`)**: Only items present in both sets.
- **Difference (`-`)**: Items in the first set but *not* in the second.
- **Symmetric Difference (`^`)**: Items in either set, but *not* in both.

```python
set_a: set[int] = {1, 2, 3, 4}
set_b: set[int] = {3, 4, 5, 6}

# Union: {1, 2, 3, 4, 5, 6}
union_set: set[int] = set_a | set_b

# Intersection: {3, 4}
intersection_set: set[int] = set_a & set_b

# Difference: {1, 2}
difference_set: set[int] = set_a - set_b

# Symmetric Difference: {1, 2, 5, 6}
sym_diff_set: set[int] = set_a ^ set_b
```

### 3. Set Comprehensions
Similar to list comprehensions, set comprehensions provide a concise way to create sets.

```python
# Create a set of squares from 0 to 9
squares_set: set[int] = {x * x for x in range(10)}
# {0, 1, 64, 4, 36, 9, 16, 49, 81, 25} (order may vary)
```

### 4. `frozenset`: The Immutable Set
A `frozenset` is an immutable version of a `set`. Once created, you cannot add or remove items. Because they are immutable and hashable, you can use `frozenset`s as dictionary keys or as items within another set.

```python
fs: frozenset[int] = frozenset([1, 2, 3])

# This would raise an AttributeError:
# fs.add(4)

# You can use a frozenset as a dictionary key
permissions: dict[frozenset[str], str] = {
    frozenset(["read"]): "Reader",
    frozenset(["read", "write"]): "Editor"
}
```

### 5. Performance Advantages
The key advantage of sets is their performance for checking if an item is present.
- `item in my_set`: **O(1)** on average (very fast, constant time)
- `item in my_list`: **O(n)** on average (slow, requires scanning the list)

Use a set instead of a list when your primary need is to check for the existence of items, not to maintain order.

```python
# Suppose we have a large list of allowed user IDs
allowed_ids_list: list[int] = list(range(10_000_000))
# And a fast set version
allowed_ids_set: set[int] = set(allowed_ids_list)

# This check is very slow because it has to scan the list
# print(9_999_999 in allowed_ids_list)

# This check is extremely fast, regardless of the set's size
# print(9_999_999 in allowed_ids_set)
```

## 🔹 Quick Exercise

Implement the following two generic functions using set operations.

```python
from typing import TypeVar, List, Set

T = TypeVar('T')

def find_duplicates(items: list[T]) -> set[T]:
    """
    Finds all items that appear more than once in a list.
    Returns a set of the duplicate items.
    """
    # Hint: Iterate through the list. Keep track of items you've seen in one set
    # and items you've found to be duplicates in another set.
    pass

def symmetric_difference(a: set[T], b: set[T]) -> set[T]:
    """
    Calculates the symmetric difference between two sets without using the `^` operator.
    The symmetric difference is items that are in A or B, but not both.
    """
    # Hint: You can implement this using union, intersection, and difference.
    # (A union B) minus (A intersection B)
    pass

# --- Tests ---
assert find_duplicates([1, 2, 3, 2, 4, 5, 4, 4]) == {2, 4}
assert find_duplicates(['a', 'b', 'c']) == set()
set1 = {1, 2, 3}
set2 = {3, 4, 5}
assert symmetric_difference(set1, set2) == {1, 2, 4, 5}
```

## 📝 Daily Assignment
**Goal**: Build a small library for analyzing text using sets.

1.  **Create Project File**: In `my-first-poetry-app`, create a new file: `my_first_poetry_app/text_analyzer.py`.
2.  **Implement Text Analysis Functions**:
    - `get_unique_words(text: str) -> set[str]`: Takes a string of text, converts it to lowercase, splits it into words, and returns a set of the unique words. Punctuation should be handled simply (e.g., by removing it).
    - `compare_texts(text1: str, text2: str) -> dict[str, set[str]]`: This function should take two texts and return a dictionary summarizing their relationship. The dictionary should have three keys:
        - `"common_words"`: A set of words that appear in *both* texts.
        - `"unique_to_text1"`: A set of words that only appear in the first text.
        - `"unique_to_text2"`: A set of words that only appear in the second text.
    - `word_frequency(text: str) -> dict[str, int]`: Although not strictly a set operation, this is a related common task. Return a dictionary where keys are unique words and values are their frequencies.
3.  **Venn Diagram Simulation (Bonus)**:
    - Research how to represent a Venn diagram with sets.
    - Write a function `describe_venn_diagram(set_a: set, set_b: set, name_a: str, name_b: str) -> None` that prints a textual description of the relationship between two sets, e.g., "Items only in A: ...", "Items in both A and B: ...", etc.
4.  **`main()` function**:
    - Create two sample text strings.
    - Use your functions to analyze them and print the results clearly.
5.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes
- **Assuming sets are ordered**: The order of items in a set is arbitrary and can change. Never rely on it. If you need to preserve the order of unique items from a list, you need a different approach (like iterating and adding to a new list if not already seen).
- **Trying to put mutable items in a set**: `my_set.add([1, 2])` will raise a `TypeError` because lists are mutable and therefore not hashable. You can add tuples: `my_set.add((1, 2))`.
- **Creating an empty set with `{}`**: `empty = {}` creates an empty dictionary, not a set. Use `empty = set()`.
- **Inefficiently finding unique items**:
  ```python
  # Inefficient way to get unique items from a list
  unique_items = []
  for item in my_list:
      if item not in unique_items: # This "not in" check is O(n)
          unique_items.append(item)

  # Efficient Pythonic way
  unique_items = list(dict.fromkeys(my_list)) # Preserves order
  # Or if order doesn't matter:
  unique_items_set = set(my_list)
  ```

## 📖 Further Reading
- [Real Python: Python Sets](https://realpython.com/python-sets/)
- [Python Docs: Set Types — `set`, `frozenset`](https://docs.python.org/3/library/stdtypes.html#set-types-set-frozenset)
- [Python For Engineers: Sets](https://www.pythonforengineers.com/sets-in-python/) (Good examples of use cases)
