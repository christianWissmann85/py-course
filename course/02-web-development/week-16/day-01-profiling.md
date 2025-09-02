# Week 16, Day 1: Profiling Python Code

## 🎯 Learning Objectives
- [ ] Understand why profiling is the first and most crucial step in optimization.
- [ ] Learn to use Python's built-in `cProfile` module to identify performance bottlenecks.
- [ ] Analyze profiler output to understand where your program spends its time.
- [ ] Use `line_profiler` for a more granular, line-by-line performance analysis.
- [ ] Get an introduction to memory profiling with `memory_profiler`.

## 📚 Concepts

### 1. Why Profile? The First Rule of Optimization
The first rule of program optimization is: **don't do it.** The second rule (for experts only) is: **don't do it yet.** - *Michael A. Jackson*

This famous quote highlights a critical point: premature optimization is the root of many problems. Developers often guess where their code is slow, but their intuition is frequently wrong.

**Profiling** is the process of measuring a program's performance to identify which parts are taking the most time or consuming the most resources. You should *never* start optimizing without profiling first. The data will show you the actual **bottlenecks**, allowing you to focus your efforts where they will have the most impact.

### 2. `cProfile`: The Built-in Profiler
Python comes with a powerful built-in profiler called `cProfile`. It measures how much time is spent in each function call.

You can run it directly from the command line on a script:
```bash
python -m cProfile -o my_program.prof my_program.py
```
-   `-m cProfile`: Runs the `cProfile` module.
-   `-o my_program.prof`: Saves the profiling results to a binary file named `my_program.prof`.

The output file is not human-readable. You need to analyze it with the `pstats` module or a visualizer.

```python
import pstats
from pstats import SortKey

# Load the stats file
stats = pstats.Stats('my_program.prof')

# Sort the stats by cumulative time spent in each function and print the top 10
stats.sort_stats(SortKey.CUMULATIVE).print_stats(10)

# Sort by the total time spent *in* a function (excluding sub-calls)
stats.sort_stats(SortKey.TIME).print_stats(10)
```
**Understanding the Output:**
-   `ncalls`: The number of times the function was called.
-   `tottime`: Total time spent *in this function alone* (excluding time spent in functions it called).
-   `percall`: `tottime` divided by `ncalls`.
-   `cumtime`: **Cumulative time** spent in this function *plus all functions it called*. This is often the most useful metric for finding bottlenecks.
-   `percall`: `cumtime` divided by `ncalls`.

A great visualizer for `.prof` files is `snakeviz`.
```bash
# Install snakeviz
poetry add snakeviz
# Run the visualizer
snakeviz my_program.prof
```
This will open a web-based, interactive chart that helps you understand the call stack and where time is being spent.

### 3. `line_profiler`: Line-by-Line Analysis
`cProfile` tells you which *functions* are slow. `line_profiler` tells you which *lines inside a function* are slow. This is incredibly useful for finding the exact bottleneck within a complex function.

First, install it: `poetry add line_profiler`

You can't just run `line_profiler` on your whole script. You must explicitly mark the functions you want to profile with a `@profile` decorator.

```python
# Note: The @profile decorator is not imported.
# It's added to the built-ins by the line_profiler runner.
@profile
def my_slow_function():
    # ... some code ...
    very_slow_list_comprehension = [x*x for x in range(10**6)]
    # ... more code ...

# To run it, you use the 'kernprof' command that comes with the library:
# kernprof -l -v my_script.py
```
-   `kernprof`: The command-line tool for `line_profiler`.
-   `-l`: "line-by-line" mode.
-   `-v`: "verbose" mode, prints the results directly to the console.

The output will show your function's source code with timing information for each line, making it obvious where the time is being spent.

### 4. `memory_profiler`: Tracking Memory Usage
Sometimes the bottleneck isn't CPU time, but memory usage. `memory_profiler` works similarly to `line_profiler`, providing a line-by-line report of memory consumption.

First, install it: `poetry add memory-profiler`

Like `line_profiler`, you must decorate the functions you want to profile.
```python
from memory_profiler import profile

@profile
def my_memory_hungry_function():
    # This creates a huge list in memory
    big_list = list(range(10**7))
    # ... do something with it ...
    del big_list
```
**How to run it:**
```bash
python -m memory_profiler my_script.py
```
The output shows the memory usage at each line, including the "increment" (how much memory that line added) and the "memory usage" at that point.

## 🔹 Quick Exercise
You have the following function. How would you set it up to be profiled with `line_profiler`?

```python
def process_data(data: list[int]) -> list[int]:
    # Step 1
    processed = [i * 2 for i in data]
    # Step 2
    processed.sort()
    # Step 3
    return [i for i in processed if i % 3 == 0]
```

**Answer:**
You simply add the `@profile` decorator. You do not need to import it.

```python
# No import needed for the decorator itself
@profile
def process_data(data: list[int]) -> list[int]:
    # Step 1
    processed = [i * 2 for i in data]
    # Step 2
    processed.sort()
    # Step 3
    return [i for i in processed if i % 3 == 0]

# Then you would run from the command line:
# kernprof -l -v your_script.py
```

## 📝 Daily Assignment
**Goal**: Profile and identify bottlenecks in a provided piece of "slow" code.

1.  **Get the Code**: Create a file `slow_code.py` with the following content:
    ```python
    import time

    def process_data(data):
        time.sleep(0.5) # Simulates slow I/O
        result = [str(i) for i in data]
        return result

    def calculate_stats(data):
        # A deliberately inefficient calculation
        total = 0
        for i in range(len(data)):
            for j in range(len(data)):
                if i == j:
                    total += data[i] * data[j]
        return total

    def main():
        # Generate some data
        data = list(range(500))
        # Process the data
        processed = process_data(data)
        # Calculate stats on the original data
        stats = calculate_stats(data)
        print(f"Finished. Final stat: {stats}")

    if __name__ == "__main__":
        main()
    ```
2.  **Profile with `cProfile`**:
    -   Run `cProfile` on `slow_code.py` and save the output to a file.
    -   Use `pstats` or `snakeviz` to analyze the results.
    -   Write your analysis in a markdown file named `analysis.md`. Which function takes the most cumulative time (`cumtime`)? Which function takes the most time internally (`tottime`)?

3.  **Profile with `line_profiler`**:
    -   Add the `@profile` decorator to the `calculate_stats` function.
    -   Run `kernprof -l -v slow_code.py`.
    -   In your `analysis.md`, explain which specific line inside `calculate_stats` is the bottleneck.

4.  **Profile with `memory_profiler`**:
    -   Add the `@profile` decorator (from `memory_profiler`) to the `main` function.
    -   Run `python -m memory_profiler slow_code.py`.
    -   In your `analysis.md`, identify which line allocates the most memory.

This assignment will walk you through the full workflow of using different profiling tools to understand an application's performance from multiple angles.

## 📖 Further Reading
- [Python Docs: The Python Profilers](https://docs.python.org/3/library/profile.html)
- [Snakeviz - A web-based visualizer for `cProfile` output](https://jiffyclub.github.io/snakeviz/)
- [GitHub: `line_profiler`](https://github.com/pyutils/line_profiler)
- [GitHub: `memory_profiler`](https://github.com/pythonprofilers/memory_profiler)
