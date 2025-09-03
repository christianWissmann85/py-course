# Week 16, Day 4: NumPy & Vectorization

## 🎯 Learning Objectives
- [ ] Understand the role of NumPy as the foundation for numerical computing in Python.
- [ ] Learn to create and manipulate NumPy's core data structure, the `ndarray`.
- [ ] Grasp the concept of "vectorization" and how it avoids slow Python loops.
- [ ] Use broadcasting to perform operations on arrays of different shapes.
- [ ] See a practical demonstration of the performance gains from using NumPy.

## 📚 Concepts

### 1. What is NumPy?
**NumPy (Numerical Python)** is the fundamental package for numerical computation in Python. It provides:
-   A powerful N-dimensional array object (`ndarray`).
-   Sophisticated "broadcasting" functions.
-   Tools for integrating C/C++ and Fortran code.
-   Useful linear algebra, Fourier transform, and random number capabilities.

The key to NumPy's power is that its arrays are densely packed and homogeneous (all elements are of the same type). The core operations are implemented in highly optimized, pre-compiled C code. This allows NumPy to perform mathematical operations on entire arrays at once, a process called **vectorization**.

First, install it: `poetry add numpy`

### 2. The `ndarray`
The NumPy array, or `ndarray`, is a grid of values, all of the same type.

```python
import numpy as np

# Create an array from a Python list
a = np.array([1, 2, 3, 4])
print(a)
# [1 2 3 4]

# Create a 2D array (a matrix)
b = np.array([[1, 2, 3], [4, 5, 6]])
print(b)
# [[1 2 3]
#  [4 5 6]]

# Get information about the array
print(f"Shape: {a.shape}")     # (4,)
print(f"Shape: {b.shape}")     # (2, 3)
print(f"Data type: {a.dtype}") # int64

# Create arrays with placeholder data
zeros = np.zeros((3, 4)) # 3x4 array of zeros
ones = np.ones((2, 5))   # 2x5 array of ones
rng = np.arange(10)      # Like Python's range: [0 1 2 3 4 5 6 7 8 9]
```

### 3. Vectorization: The Magic of NumPy
Vectorization is the process of applying an operation to an entire array at once, rather than element by element in a Python loop. This is **dramatically faster** because the loop happens in optimized C code, not in the Python interpreter.

**Slow Python Loop:**
```python
python_list = list(range(1_000_000))
result_list = []
for i in python_list:
    result_list.append(i * 2)
```

**Fast NumPy Vectorization:**
```python
numpy_array = np.arange(1_000_000)
# The multiplication is applied to every element at once in C.
result_array = numpy_array * 2
```
The NumPy version can be 100x faster or more. **The rule of thumb for performance with NumPy is: avoid Python loops whenever possible.** Use vectorized operations instead.

### 4. Array Operations and Indexing
You can perform all standard mathematical operations on arrays. You can also access elements using a rich indexing system similar to Python lists, but more powerful.

```python
a = np.array([10, 20, 30, 40])
b = np.array([1, 2, 3, 4])

# Element-wise operations
c = a + b  # [11 22 33 44]
d = a * b  # [10 40 90 160]

# Universal Functions (ufuncs)
e = np.sin(a) # Applies the sin function to each element

# Indexing and Slicing
matrix = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
print(matrix[0, 1]) # Get element at row 0, column 1 -> 2

# Boolean Indexing (very powerful)
# Get all elements greater than 5
print(matrix[matrix > 5]) # [6 7 8 9]
```

### 5. Broadcasting
Broadcasting is a powerful mechanism that allows NumPy to work with arrays of different shapes when performing arithmetic operations. It "broadcasts" the smaller array across the larger array so that they have compatible shapes.

```python
matrix = np.array([[1, 2, 3], [4, 5, 6]])
scalar = 10

# The scalar is "broadcast" across the entire matrix
result = matrix + scalar
# [[11 12 13]
#  [14 15 16]]

# A more complex example
row_vector = np.array([10, 20, 30])
result = matrix + row_vector # The row is added to EACH row of the matrix
# [[11 22 33]
#  [14 25 36]]
```
Broadcasting avoids the need to create extra copies of your data to make shapes match and is a cornerstone of writing efficient, vectorized NumPy code.

## 🔹 Quick Exercise
You have two NumPy arrays representing 2D vectors. Write a function to calculate the Euclidean distance between them.
The formula is `sqrt((x1-x2)² + (y1-y2)²)`.

```python
import numpy as np

def euclidean_distance(v1: np.ndarray, v2: np.ndarray) -> float:
    """Calculates the Euclidean distance between two 2D vectors."""
    # --- Answer ---
    # Use vectorized operations, not Python loops or math.sqrt
    diff = v1 - v2
    squared_diff = diff ** 2
    sum_of_squares = np.sum(squared_diff)
    return np.sqrt(sum_of_squares)

# Example
p1 = np.array([1, 1])
p2 = np.array([4, 5])
# distance = euclidean_distance(p1, p2) # Should be 5.0
```

## 📝 Daily Assignment
**Goal**: Analyze a dataset of temperatures and demonstrate the performance benefits of NumPy.

You will compare the speed of pure Python vs. NumPy for a common data analysis task.

1.  **Generate Data**: Create a function `generate_data(num_points: int) -> list[float]` that generates a Python list of random temperature readings (e.g., floating-point numbers between -10.0 and 40.0). Generate a dataset with at least 1 million data points.

2.  **Implement Pure Python Version**:
    -   Write a function `calculate_stats_python(data: list[float]) -> dict`.
    -   This function should iterate through the list to calculate the mean, standard deviation, and the number of "hot days" (temperatures > 30.0).
    -   Do not use any libraries other than `math` for standard deviation.

3.  **Implement NumPy Version**:
    -   Write a function `calculate_stats_numpy(data: np.ndarray) -> dict`.
    -   This function should take a NumPy array as input.
    -   Use NumPy's built-in functions to calculate the statistics: `np.mean()`, `np.std()`.
    -   Use boolean indexing to efficiently count the number of "hot days" (`np.sum(data > 30.0)`).

4.  **Benchmark**:
    -   Use the `time` module to benchmark both functions on the same large dataset.
    -   Run each function several times and take the average to get a stable result.
    -   In a markdown file `numpy_benchmark.md`, present your results in a table.
    -   Include a conclusion explaining the performance difference. Why is the NumPy version so much faster? Relate it back to the concept of vectorization.

**Example `numpy_benchmark.md` table:**

| Calculation       | Pure Python (seconds) | NumPy (seconds) | Speedup Factor |
|-------------------|-----------------------|-----------------|----------------|
| Mean              | 0.123                 | 0.001           | 123x           |
| Standard Deviation| 0.245                 | 0.002           | 122.5x         |
| Hot Day Count     | 0.098                 | 0.003           | 32.7x          |

## 📖 Further Reading
- [NumPy: the absolute basics for beginners](https://numpy.org/doc/stable/user/absolute_beginners.html) (Official Guide)
- [Real Python: Look Ma, No `for` Loops: Array Programming With NumPy](https://realpython.com/numpy-array-programming/)
- [From Python to NumPy](https://www.labri.fr/perso/nrougier/from-python-to-numpy/) (A great resource for vectorization patterns)
