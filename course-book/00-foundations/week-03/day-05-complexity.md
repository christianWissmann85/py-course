# Week 3, Day 5: Algorithm Complexity

## 🎯 Learning Objectives
- [ ] Understand what Big O notation is and why it's important for analyzing algorithms.
- [ ] Learn to analyze the time and space complexity of simple algorithms.
- [ ] Recognize common complexity classes (e.g., O(1), O(n), O(n²)).
- [ ] Understand how choosing the right data structure can drastically improve performance.

## 📚 Concepts

### 1. What is Algorithm Complexity? (Big O Notation)
When we write code, we want it to be fast and efficient. Algorithm complexity is a way to measure how the performance of an algorithm scales with the size of its input. We use **Big O notation** to describe this.

Big O notation describes the **worst-case scenario**. It tells us the upper bound of an algorithm's runtime or memory usage. For example, if an algorithm is `O(n)`, its runtime grows linearly with the size of the input, `n`. If the input doubles, the runtime roughly doubles.

We focus on the dominant term and ignore constants. `O(2n + 10)` simplifies to `O(n)`. We care about the overall growth trend, not the exact millisecond count.

### 2. Time vs. Space Complexity
- **Time Complexity**: How the runtime of an algorithm changes with the input size. This is usually what people mean when they say "complexity".
- **Space Complexity**: How the amount of memory (or space) an algorithm uses changes with the input size.

Often, there's a trade-off. You might be able to make an algorithm faster by using more memory, or save memory by accepting a slower runtime.

### 3. Common Complexity Classes
Here are the most common classes, from best to worst:
- **`O(1)` - Constant Time**: The runtime is the same, regardless of the input size.
  - *Example*: Accessing an item in a list by index (`my_list[5]`) or a dictionary by key.
- **`O(log n)` - Logarithmic Time**: The runtime grows very slowly. The algorithm halves the input size with each step.
  - *Example*: Binary search in a sorted list.
- **`O(n)` - Linear Time**: The runtime grows linearly with the input size.
  - *Example*: Looping through all items in a list once.
- **`O(n log n)` - Log-Linear Time**: A very common and efficient complexity for sorting algorithms.
  - *Example*: Merge Sort and Tim Sort (Python's default `.sort()`).
- **`O(n²)` - Quadratic Time**: The runtime grows quadratically. Often involves nested loops over the same input.
  - *Example*: A simple bubble sort, or finding pairs in a list with two nested loops.
- **`O(2ⁿ)` - Exponential Time**: The runtime doubles with each addition to the input. Becomes unusable very quickly.
  - *Example*: A recursive calculation of Fibonacci numbers without memoization.

### 4. How to Analyze Algorithms
- A simple loop that iterates `n` times is `O(n)`.
- A nested loop where both loops go up to `n` is `O(n * n) = O(n²)`.
- If you have two separate loops, you add their complexities: `O(n) + O(n) = O(2n)`, which simplifies to `O(n)`.
- The complexity of a function is determined by its most dominant part. A function with an `O(n²)` part and an `O(n)` part is `O(n²)`.

### 5. Optimization through Data Structures
Choosing the right data structure is the easiest way to improve performance.
- **Problem**: Find if a value exists in a collection.
  - Using a `list`: `value in my_list` is `O(n)`.
  - Using a `set`: `value in my_set` is `O(1)`.
- **Problem**: Find an item by its ID.
  - Using a `list` of objects: You have to loop through, which is `O(n)`.
  - Using a `dict` where keys are IDs: `my_dict[id]` is `O(1)`.

This is why we spent the last few days learning about these different data structures!

## 🔹 Quick Exercise

The "two-sum" problem is a classic interview question. A naive solution would use two nested loops, giving it `O(n²)` time complexity. Your task is to implement it with `O(n)` time complexity.

**Hint**: As you iterate through the list, what data structure would allow you to check for the existence of the `complement` (`target - current_number`) in `O(1)` time?

```python
from typing import List, Tuple, Optional

def find_pair_sum(nums: list[int], target: int) -> tuple[int, int] | None:
    """
    Finds a pair of two numbers in a list that sum up to a given target.
    This implementation should have O(n) time complexity.
    """
    # Create a set to store numbers we've seen so far.
    seen_numbers: set[int] = set()

    for num in nums:
        complement = target - num
        if complement in seen_numbers:
            return (complement, num)
        seen_numbers.add(num)

    return None

# --- Tests ---
assert find_pair_sum([1, 2, 3, 4, 5], 9) == (4, 5)
assert find_pair_sum([10, 20, 35, 50], 70) == (20, 50)
assert find_pair_sum([5, 5, 10], 10) == (5, 5)
assert find_pair_sum([1, 2, 3], 7) is None
```

## 📝 Daily Assignment
**Goal**: Implement and analyze two different sorting algorithms to see complexity in action.

1.  **Create Project File**: In `my-first-poetry-app`, create a new file: `my_first_poetry_app/sorters.py`.
2.  **Implement Bubble Sort (`O(n²)`):**
    - Write a function `bubble_sort(numbers: list[int]) -> list[int]`.
    - Bubble sort works by repeatedly swapping adjacent elements if they are in the wrong order. This requires nested loops.
    - Research the algorithm if you're not familiar with it.
3.  **Implement Merge Sort (`O(n log n)`):**
    - Write a function `merge_sort(numbers: list[int]) -> list[int]`.
    - Merge sort is a recursive "divide and conquer" algorithm.
        - **Divide**: Split the list into two halves.
        - **Conquer**: Recursively call `merge_sort` on each half.
        - **Combine**: Merge the two sorted halves back into one sorted list.
    - You will need a helper function, `merge(left: list[int], right: list[int]) -> list[int]`, to do the combining step.
4.  **Analyze and Benchmark**:
    - In a `main` function:
        - Create a large, randomly ordered list of numbers (e.g., 1000 items).
        - Use the `time` module to measure how long each sorting function takes to sort the list.
        - Print the results. You should see a significant performance difference.
        - Add comments to your code explaining the time and space complexity of each function.
5.  **Verify**: Run the script, type checker, and linter.

## 📦 Week 3 Project: Data Structure Library
This week's project is to implement some of these core data structures from scratch to deeply understand their mechanics and performance characteristics. You will create your own simplified, type-safe versions of a List, a Stack (using your list), and a Queue.

See `week-03-project.md` for a more detailed breakdown.

## ⚠️ Common Mistakes
- **Premature Optimization**: Don't waste time optimizing code that isn't a bottleneck. Write clean, readable code first. Only optimize when you have identified a performance problem. As Donald Knuth said, "Premature optimization is the root of all evil."
- **Ignoring Space Complexity**: An algorithm might be fast but use a huge amount of memory. For example, creating a copy of a massive list might be fast, but it doubles the memory requirement.
- **Misinterpreting Big O**: Big O describes growth rate, not absolute speed. An `O(n²)` algorithm might be faster than an `O(n)` one for very small `n`, but the `O(n)` algorithm will always win as `n` grows.
- **Analyzing best-case instead of worst-case**: Big O is about the worst-case guarantee. While an algorithm might be fast sometimes, we analyze its complexity based on the input that makes it work the hardest.

## 📖 Further Reading
- [Big O Cheat Sheet](https://www.bigocheatsheet.com/) (An excellent reference for the complexity of common algorithms and data structures)
- [Real Python: Big O Notation and Algorithm Analysis](https://realpython.com/big-o-notation-python/)
- [MIT OpenCourseWare: Introduction to Algorithms](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-fall-2011/) (For a deep, academic dive)
