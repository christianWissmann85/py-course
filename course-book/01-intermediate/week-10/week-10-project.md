# Week 10 Project: Functional Data Pipeline

## 🎯 Project Objective

To build a complete, end-to-end data processing pipeline using a purely functional style. This project will challenge you to use all the concepts from this week: pure functions, immutable data structures, higher-order functions, function composition, and lazy evaluation with generators. You will build a system that is predictable, testable, and memory-efficient.

## 📋 Core Requirements

You will build a script that processes a collection of user records. The pipeline will filter for active users, calculate a "reputation score" based on their activity, and select the top N users, all without using traditional loops or mutable state.

### 1. Project Setup

- Create a new file for your project, e.g., `my_first_poetry_app/functional_project.py`.
- This project will rely heavily on the `typing` and `functools` modules.

### 2. The Data Model (Immutability)

- Define an immutable `User` data structure using `@dataclass(frozen=True)`.
- It should contain fields like: `user_id: int`, `username: str`, `posts: int`, `comments: int`, `is_active: bool`.

### 3. The Pipeline Functions (Purity and Composition)

You must implement the entire processing logic as a series of small, pure, and composable functions. **No `for` or `while` loops are allowed in your core processing logic.** Use `map`, `filter`, and generator expressions instead.

- **`is_active_user(user: User) -> bool`**: A simple predicate function.
- **`calculate_reputation(user: User) -> User`**: A pure function that calculates a reputation score. The formula can be simple, e.g., `(user.posts * 10) + (user.comments * 2)`. This function must return a _new_ `User` object with an added `reputation` field. (Hint: `dataclasses.replace` is your friend, but how do you add a new field? You may need a second, "enriched" User dataclass).
- **`select_active(users: Iterable[User]) -> Generator[User, None, None]`**: A generator that filters for active users.
- **`calculate_reputations(users: Iterable[User]) -> Generator[UserWithReputation, None, None]`**: A generator that maps users to users with their reputation calculated.
- **`sort_by_reputation(users: Iterable[UserWithReputation]) -> list[UserWithReputation]`**: A function that sorts the users by their reputation in descending order. `sorted()` with a `lambda` key is a good choice here.
- **`limit(items: Iterable[T], n: int) -> Generator[T, None, None]`**: A generic generator that yields the first `n` items from any iterable.

### 4. The `pipe` function

- Implement the `pipe` function we discussed in class to chain all your processing steps together. This will be the engine of your pipeline.

### 5. The Main Application Logic

- In a `main` function, create a list of sample `User` data.
- Construct the full pipeline using `pipe`:
  ```python
  # Pseudocode for the pipeline
  top_users = pipe(
      sample_users,
      select_active,
      calculate_reputations,
      sort_by_reputation,
      lambda users: limit(users, 3) # Use a lambda for the final step
  )
  ```
- The result of the pipeline should be an iterator. Convert it to a list and print the top 3 users.

### 6. Type Safety & Code Quality

- The entire application must be fully type-annotated and pass `mypy --strict`.
- All functions must be pure and have clear docstrings explaining what they do.
- The code must pass all `ruff check .` linter checks.

## ✅ Definition of Done

- [ ] An immutable `User` dataclass is defined.
- [ ] The data processing logic is implemented as a series of small, pure generator functions.
- [ ] A `pipe` function is used to compose the processing steps.
- [ ] The pipeline correctly identifies and ranks the top active users based on reputation.
- [ ] The solution is memory-efficient due to lazy evaluation (generators).
- [ ] The entire codebase is type-safe, documented, and passes the linter.

## 🏆 Bonus Challenges

- **Monadic Error Handling**: Refactor one of the pipeline steps to return a `Result` or `Maybe` type (from Day 2's lesson). For example, what if some user data is malformed? Create a `parse_user` function that returns a `Result[User, str]`. How does this change the structure of your pipeline? You'll need a `bind`-like function to handle it.
- **Curried Functions**: Refactor your pipeline functions (like `limit` or `sort_by_reputation`) into "curried" or higher-order functions. For example, `limit(n)` would return a _function_ that takes an iterable and returns the first `n` items. This makes the pipeline even more declarative: `pipe_lazy(..., limit(3), ...)`
- **Performance Profiling**: Use a decorator to time your pipeline. Process a very large list of users (e.g., 1 million) and compare the performance of a pipeline using generators vs. one that uses list comprehensions and materializes the list at each step.

## 💡 Tips

- **Think in terms of data flow**: Don't think about "how" to loop. Think about what you _have_ (a list of users) and what you _want_ (a list of top 3 users), and what transformations need to happen in between.
- **Test each function individually**: The beauty of pure functions is that they are easy to test in isolation.
- **Type hints are your guide**: A function `(Iterable[A]) -> Generator[B, None, None]` tells you exactly what it does: it transforms a stream of `A`'s into a stream of `B`'s. Use the types to help you compose the pipeline correctly.
