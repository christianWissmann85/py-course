# Week 16, Day 5: Cython & C Extensions

## 🎯 Learning Objectives
- [ ] Understand when and why to use Cython to accelerate Python code.
- [ ] Learn the basic syntax of Cython, including static type declarations.
- [ ] Compile a `.pyx` file into a C extension module.
- [ ] Integrate and use the compiled Cython module in a standard Python script.
- [ ] Benchmark the performance gains of Cython over pure Python.

## 📚 Concepts

### 1. When All Else Fails: Cython
You've profiled your code. You've chosen the best algorithms and data structures. You've vectorized operations with NumPy. But what if a specific, CPU-bound function is *still* the bottleneck?

This is where you reach for tools that bridge the gap between Python and C. **Cython** is the most popular and powerful tool for this.

**What is Cython?**
-   It's a programming language that is a **superset of Python**. This means any valid Python code is also valid Cython code.
-   It's a **compiler**. The Cython compiler takes your `.pyx` code and translates it into optimized C code.
-   This C code is then compiled into a standard Python **extension module** (`.so` on Linux/macOS, `.pyd` on Windows).
-   You can then `import` this compiled module into your Python code just like any other module.

The result is that your algorithm can run at near-C speeds, because it *is* C code, while still being callable from the high-level Python code you're used to.

### 2. Cython Basics: From `.py` to `.pyx`
Let's start with a simple Python function that we want to optimize.

**`my_math_pure.py`**
```python
def sum_of_squares(n):
    total = 0
    for i in range(n):
        total += i * i
    return total
```

To start with Cython, you can simply rename the file to `my_math_cython.pyx` and use the exact same code. Just by compiling this, you'll get a small speedup (perhaps 20-30%) because Cython compiles the Python bytecode into faster C instructions.

But the real magic comes from adding static types.

### 3. Static Typing with `cdef`
The biggest source of overhead in Python is its dynamic typing. Every time you do `a + b`, the interpreter has to look up the types of `a` and `b` and find the correct `__add__` method to call.

Cython lets you bypass this by declaring C-level static types for your variables using the `cdef` keyword.

**`my_math_cython.pyx` (Optimized)**
```cython
# We can add type hints for the Python-level function signature
def sum_of_squares(int n):
    # Use 'cdef' to declare C-level variables with static types.
    # These variables are much faster than Python objects.
    cdef int total = 0
    cdef int i

    for i in range(n):
        total += i * i

    return total
```
-   `def sum_of_squares(int n)`: We've typed the input `n` as a C integer. Cython will handle the conversion from a Python `int` automatically.
-   `cdef int total`: We declare `total` as a C integer. Operations on it will be pure C arithmetic, not slow Python object operations.
-   `cdef int i`: We even type the loop variable `i`.

This version can be **100x faster** or more than the pure Python version, because the entire loop runs in highly optimized, compiled C code with no Python interpreter overhead.

### 4. Compiling a Cython Module
To use your `.pyx` file, you need to compile it. This requires a C compiler to be installed on your system (e.g., GCC on Linux, Clang/XCode on macOS, MSVC on Windows).

The standard way to compile extensions is with a `setup.py` file.

**`setup.py`**
```python
from setuptools import setup
from Cython.Build import cythonize

setup(
    ext_modules=cythonize("my_math_cython.pyx")
)
```
You also need to add `cython` to your project: `poetry add cython`.

**Compilation Command:**
```bash
# This command tells setuptools to run the setup script and build the extension "in-place",
# meaning the compiled .so/.pyd file will be put in the current directory.
python setup.py build_ext --inplace
```
After running this, you will see a new file, such as `my_math_cython.cpython-312-x86_64-linux-gnu.so`. The exact name depends on your Python version and OS.

### 5. Using the Compiled Module
Now, you can import and use your super-fast function in any other Python script as if it were a regular Python module.

**`main.py`**
```python
from my_math_cython import sum_of_squares
import time

start_time = time.perf_counter()
result = sum_of_squares(200000) # This calls the fast, compiled C code
end_time = time.perf_counter()

print(f"Result: {result}")
print(f"Cython version took {end_time - start_time:.4f} seconds")
```

## 🔹 Quick Exercise
You are given a simple Python function. How would you convert it to Cython and add static C types for maximum performance?

**Python function:**
```python
def factorial(n):
    if n == 0:
        return 1
    result = 1
    for i in range(1, n + 1):
        result *= i
    return result
```
**Answer (Cython version):**
```cython
# In a .pyx file
def factorial(int n):
    cdef long long result = 1 # Use 'long long' to hold large numbers
    cdef int i

    if n == 0:
        return 1

    for i in range(1, n + 1):
        result *= i

    return result
```

## 📝 Daily Assignment
**Goal**: Optimize a CPU-bound image processing function using Cython.

The task is to convert a color image to grayscale. The standard formula for luminance is: `Y = 0.299*R + 0.587*G + 0.114*B`.

1.  **Get Setup**: You'll need NumPy and Pillow for this.
    `poetry add numpy pillow cython`
2.  **Create a Pure Python Version (`grayscale_python.py`)**:
    -   Write a function `grayscale(image_array: np.ndarray) -> np.ndarray`.
    -   The input will be a 3D NumPy array (height, width, 3 for RGB).
    -   The function should create a new 2D NumPy array (height, width) for the grayscale image.
    -   Iterate over every pixel in the input array using **two nested Python `for` loops** and apply the luminance formula to calculate the grayscale value.
    -   This will be **very slow**.
3.  **Create a Cython Version (`grayscale_cython.pyx`)**:
    -   Copy the pure Python function into your `.pyx` file.
    -   Use `cdef` to add static types for all your variables (loop counters, pointers to array data, etc.).
    -   **Hint**: Working with NumPy arrays in Cython is a common pattern. You'll need to import numpy (`cimport numpy as np`) and use typed memoryviews (`cdef unsigned char[:, :, :] view = image_array`) for efficient access. This is an advanced topic, so refer to the Cython documentation on NumPy.
4.  **Create `setup.py`**: Write the `setup.py` file needed to compile your `grayscale_cython.pyx` module.
5.  **Compile the Module**: Run `python setup.py build_ext --inplace`.
6.  **Benchmark**:
    -   Write a `main.py` script.
    -   Use Pillow to load an image (`Image.open(...)`) and convert it to a NumPy array (`np.array(...)`).
    -   Time how long the pure Python `grayscale` function takes.
    -   Time how long your compiled Cython `grayscale` function takes on the same data.
    -   Save the resulting grayscale image to disk (`Image.fromarray(...).save(...)`) to verify it's correct.
    -   Report your findings in a `cython_benchmark.md` file. The speedup should be dramatic.

## 📖 Further Reading
- [Cython Documentation - Basic Tutorial](https://cython.readthedocs.io/en/latest/src/tutorial/cython_tutorial.html)
- [Cython Documentation - Working with NumPy](https://cython.readthedocs.io/en/latest/src/tutorial/numpy.html)
- [Real Python: Speed Up Your Python With Cython](https://realpython.com/cython/)
