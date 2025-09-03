# Week 4, Day 2: *args and **kwargs

## 🎯 Learning Objectives
- [ ] Write functions that can accept an arbitrary number of positional arguments (`*args`).
- [ ] Write functions that can accept an arbitrary number of keyword arguments (`**kwargs`).
- [ ] Understand how to use the unpacking operators (`*` and `**`) to pass arguments to other functions.
- [ ] Correctly type `*args` and `**kwargs` in function signatures.

## 📚 Concepts

### 1. `*args`: Arbitrary Positional Arguments
Sometimes you want a function to accept any number of positional arguments. You can do this with the `*args` syntax. Inside the function, `args` will be a **tuple** containing all the positional arguments that were passed.

```python
# The type hint for *args is for the individual elements.
# So `*args: int` means args will be a tuple[int, ...].
def sum_all(*numbers: int) -> int:
    """Sums all numbers passed as arguments."""
    print(f"The 'numbers' variable is a tuple: {numbers}")
    total = 0
    for num in numbers:
        total += num
    return total

# You can call it with any number of arguments
print(sum_all(1, 2, 3))       # Output: 6
print(sum_all(10, 20, 30, 40)) # Output: 100
```
The name `args` is just a convention; `*my_numbers` would also work. The `*` is the important part.

### 2. `**kwargs`: Arbitrary Keyword Arguments
Similarly, `**kwargs` allows a function to accept any number of keyword arguments. Inside the function, `kwargs` will be a **dictionary** containing the argument names as keys and their values.

```python
# The type hint for **kwargs is for the values.
# So `**kwargs: str` means kwargs will be a dict[str, str].
def display_attributes(**attributes: str) -> None:
    """Displays all keyword attributes passed to it."""
    print(f"The 'attributes' variable is a dict: {attributes}")
    for key, value in attributes.items():
        print(f" - {key}: {value}")

display_attributes(name="Laptop", price="1200", status="In Stock")
# Output:
#  - name: Laptop
#  - price: 1200
#  - status: In Stock
```
The name `kwargs` is also a convention. The `**` is what makes it work.

### 3. The Unpacking Operators (`*` and `**`)
The `*` and `**` symbols can also be used when *calling* a function. This is known as unpacking.
- `*` unpacks a sequence (like a list or tuple) into positional arguments.
- `**` unpacks a dictionary into keyword arguments.

```python
def describe_item(name: str, price: float, quantity: int) -> None:
    print(f"Item: {name}, Price: ${price:.2f}, Quantity: {quantity}")

# --- Unpacking a list with * ---
item_details_list: list[any] = ["Bread", 1.99, 5]
describe_item(*item_details_list) # Same as describe_item("Bread", 1.99, 5)

# --- Unpacking a dictionary with ** ---
item_details_dict: dict[str, any] = {
    "name": "Milk",
    "price": 3.50,
    "quantity": 2
}
describe_item(**item_details_dict) # Same as describe_item(name="Milk", price=3.50, quantity=2)
```

### 4. Forwarding Arguments
A common use case for `*args` and `**kwargs` is to write a wrapper function that "forwards" all its arguments to another function. This is the core principle behind decorators, which we'll learn about tomorrow.

```python
def logger(func, *args, **kwargs):
    """A function that logs when another function is called."""
    print(f"Calling function '{func.__name__}' with:")
    print(f"  Positional args: {args}")
    print(f"  Keyword args: {kwargs}")

    # Forward all collected arguments to the original function
    result = func(*args, **kwargs)

    print(f"Function '{func.__name__}' returned: {result}")
    return result

# Use the logger
logger(describe_item, "Cheese", price=5.00, quantity=1)
```

### 5. Advanced Typing with `TypeVarTuple` (Python 3.11+)
For very advanced cases, Python 3.11 introduced `TypeVarTuple` which allows you to type the arguments in `*args` with more precision. This is beyond the scope of our foundations but is good to be aware of.

## 🔹 Quick Exercise

Implement the following two functions.

```python
from typing import Any

def sum_all(*numbers: float) -> float:
    """
    Accepts any number of positional float arguments and returns their sum.
    """
    # Python's built-in sum() function works on any iterable (like a tuple).
    pass

def create_dict(**kwargs: Any) -> dict[str, Any]:
    """
    Accepts any number of keyword arguments and returns them as a dictionary.
    """
    # The **kwargs syntax already gives you a dictionary!
    pass

# --- Tests ---
assert sum_all(1.5, 2.5, 3.0) == 7.0
assert sum_all(10.0) == 10.0
assert sum_all() == 0.0

created = create_dict(a=1, b="hello", c=True)
assert created == {"a": 1, "b": "hello", "c": True}
assert create_dict() == {}
```

## 📝 Daily Assignment
**Goal**: Build a flexible API-like function that can create different types of objects.

1.  **Create Project File**: In `my-first-poetry-app`, create a new file: `my_first_poetry_app/flexible_api.py`.
2.  **Define Models with `TypedDict`**:
    - Create a `User` `TypedDict` with fields `username: str` and `email: str`.
    - Create a `Product` `TypedDict` with fields `product_name: str` and `price: float`.
3.  **Implement a Factory Function**:
    - Create a single function `create_entity(entity_type: str, **kwargs: Any) -> User | Product | None`.
    - This function takes a string `entity_type` (`"user"` or `"product"`) and arbitrary keyword arguments (`**kwargs`).
    - **Inside the function**:
        - If `entity_type` is `"user"`, it should expect `username` and `email` in `kwargs` and return a `User` dictionary.
        - If `entity_type` is `"product"`, it should expect `product_name` and `price` in `kwargs` and return a `Product` dictionary.
        - If `entity_type` is unknown, it should return `None`.
        - Use `.get()` to safely access keys from `kwargs` to avoid errors if they are missing.
4.  **Implement a "Client" Function**:
    - Create a function `register_new_user(user_details: dict[str, Any]) -> User | None`.
    - This function takes a dictionary of user details.
    - It should use the `**` unpacking operator to call `create_entity`, passing `"user"` as the type and the `user_details` dictionary as the keyword arguments.
5.  **`main()` function**:
    - Demonstrate your functions:
        - Call `create_entity` directly to create a product.
        - Create a user details dictionary and pass it to `register_new_user` to create a user.
        - Call `create_entity` with an invalid type to show it returns `None`.
    - Print all the created entities.
6.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes
- **Order of parameters**: A function's parameters must be in this order: standard positional, `*args`, keyword-only, `**kwargs`.
  ```python
  # Correct order
  def f(a, b, *args, kw_only, **kwargs): ...

  # Incorrect order (e.g., *args after **kwargs)
  # def f(a, **kwargs, *args): ... # SyntaxError
  ```
- **Confusing `*` on definition vs. call**:
  - In `def f(*args):`, `*` *gathers* arguments into a tuple.
  - In `f(*my_list)`, `*` *unpacks* the list into arguments.
- **Using `args` or `kwargs` directly**: Remember `args` is a tuple and `kwargs` is a dict. To access the first positional argument, you use `args[0]`. To access a keyword argument named `user`, you use `kwargs['user']`.
- **Poor typing**: Typing `*args` as `Any` is easy but provides no type safety. Be as specific as you can, e.g., `*args: int`.

## 📖 Further Reading
- [Real Python: Python `*args` and `**kwargs`](https://realpython.com/python-args-kwargs/)
- [Trey Hunner: `*args` and `**kwargs` in Python Explained](https://treyhunner.com/2018/04/keyword-arguments-in-python/)
- [Python Docs: Unpacking Argument Lists](https://docs.python.org/3/tutorial/controlflow.html#unpacking-argument-lists)
