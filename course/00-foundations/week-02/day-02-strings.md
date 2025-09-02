# Week 2, Day 2: Strings & String Operations

## 🎯 Learning Objectives
- [ ] Master various methods for creating and manipulating strings.
- [ ] Use f-strings effectively for modern, readable string formatting.
- [ ] Understand the basics of character encoding (ASCII vs. UTF-8).
- [ ] Learn to use the `re` module for basic regular expression operations.

## 📚 Concepts

### 1. String Creation and Literals
Strings can be created with single (`'...'`), double (`"..."`), triple-single (`'''...'''`), or triple-double (`"""..."""`) quotes. Triple-quoted strings can span multiple lines.

```python
# All of these create the same string
s1: str = 'hello'
s2: str = "hello"

# Multi-line string
multi_line_s: str = """This is a string
that spans multiple
lines."""
```

### 2. F-strings for Formatting
Introduced in Python 3.6, f-strings are the modern way to format strings. They are more readable and faster than older methods like `%`-formatting or `.format()`. You prefix the string with an `f` and embed expressions directly inside curly braces `{}`.

```python
name: str = "Alice"
age: int = 30

# The f-string way
greeting: str = f"Hello, my name is {name} and I am {age} years old."
print(greeting)
# Output: Hello, my name is Alice and I am 30 years old.

# You can even put expressions inside
print(f"Next year, I will be {age + 1}.")
# Output: Next year, I will be 31.
```

### 3. String Methods
Strings are objects with many useful methods. Since strings are immutable, these methods always return a *new* string and do not modify the original.

Here are a few common ones:
- `.upper()`: Returns an uppercase version of the string.
- `.lower()`: Returns a lowercase version.
- `.strip()`: Removes leading/trailing whitespace.
- `.startswith(prefix)`: Returns `True` if the string starts with `prefix`.
- `.endswith(suffix)`: Returns `True` if the string ends with `suffix`.
- `.split(separator)`: Splits the string into a list of substrings.
- `.join(iterable)`: Joins elements of an iterable into a single string.

```python
raw_input: str = "  Hello, World!  "
clean_input: str = raw_input.strip()  # "Hello, World!"
upper_input: str = clean_input.upper() # "HELLO, WORLD!"

words: list[str] = clean_input.split(", ") # ['Hello', 'World!']
joined_words: str = "-".join(words)      # "Hello-World!"
```

### 4. Unicode and Encoding
Computers only understand numbers. Character encoding is a system that maps characters (like 'A', 'B', 'C') to numbers.
- **ASCII**: An early standard that could only represent 128 characters (English letters, numbers, punctuation).
- **Unicode**: A modern standard that can represent almost every character from every language in the world.
- **UTF-8**: The most common way to *encode* Unicode characters into bytes that can be stored or transmitted. It's a variable-width encoding, making it efficient for English text while still supporting all Unicode characters.

Python 3 strings are Unicode by default. When you write to a file or send data over a network, you must encode the string into bytes.

```python
my_string: str = "你好, world"
encoded_utf8: bytes = my_string.encode('utf-8')
decoded_string: str = encoded_utf8.decode('utf-8')

print(f"Original string: {my_string}")
print(f"Encoded as bytes: {encoded_utf8}")
print(f"Decoded back to string: {decoded_string}")
```

### 5. Regular Expressions (`re` module)
Regular expressions (regex) are a powerful mini-language for finding complex patterns in strings. Python's `re` module provides regex tools.

Common `re` functions:
- `re.search(pattern, string)`: Scans through the string looking for the first location where the pattern produces a match.
- `re.match(pattern, string)`: Tries to apply the pattern at the *start* of the string.
- `re.findall(pattern, string)`: Finds all substrings where the pattern matches and returns them as a list.
- `re.sub(pattern, repl, string)`: Replaces occurrences of the pattern with a replacement string.

```python
import re
from typing import Optional

text: str = "My email is example@domain.com, please contact me."
pattern: str = r"\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b"

match: Optional[re.Match[str]] = re.search(pattern, text)

if match:
    email: str = match.group(0)
    print(f"Found email: {email}") # Output: Found email: example@domain.com
else:
    print("No email found.")
```

## 🔹 Quick Exercise

Implement the following two functions. Use the `re` module for the second one.

```python
import re
from typing import Optional

def format_name(first: str, last: str) -> str:
    """
    Formats a name as 'Last, First'.
    Example: format_name("Jules", "Verne") -> "Verne, Jules"
    """
    # Implement here
    pass

def extract_email(text: str) -> Optional[str]:
    """
    Extracts the first email address found in the given text.
    Returns the email string if found, otherwise returns None.
    """
    # Use the regex pattern from the concepts section
    # Implement here
    pass

# --- Tests ---
assert format_name("John", "Doe") == "Doe, John"
assert extract_email("Contact me at test@example.org") == "test@example.org"
assert extract_email("No email here.") is None
```

## 📝 Daily Assignment

**Goal**: Build a versatile string processing utility.

1.  **Create Project File**: In your `my-first-poetry-app` project, create a new file: `my_first_poetry_app/string_processor.py`.
2.  **Implement 10 String Functions**: Create a file that contains at least 10 different functions that perform some kind of string operation. Each function must be fully type-annotated.
    - **Suggestions**:
        - `reverse_string(s: str) -> str`: Reverses a string.
        - `is_palindrome(s: str) -> bool`: Checks if a string is a palindrome.
        - `count_vowels(s: str) -> int`: Counts the number of vowels.
        - `censor_word(text: str, word: str) -> str`: Replaces a specific word with asterisks.
        - `word_count(text: str) -> int`: Counts the number of words.
        - `remove_whitespace(text: str) -> str`: Removes all whitespace characters.
        - `snake_to_camel(s: str) -> str`: Converts "snake_case" to "camelCase".
3.  **Add Regex Validation**:
    - Implement a function `is_valid_email(email: str) -> bool` that uses a regular expression to check if a string is a valid email format.
    - Implement a function `is_valid_phone_number(phone: str) -> bool` for a simple US phone number format (e.g., `XXX-XXX-XXXX`).
4.  **Create a `main` function**:
    - Write a `main()` function that demonstrates each of your functions working correctly.
    - Print the inputs and outputs clearly.
5.  **Run and Verify**:
    - Run your script: `poetry run python my_first_poetry_app/string_processor.py`.
    - Run the type checker: `poetry run mypy .` and fix all errors.
    - Run the linter: `poetry run ruff check . --fix` and fix all issues.

## ⚠️ Common Mistakes
- **Modifying strings in place**: Forgetting that string methods return a *new* string.
  ```python
  # Wrong
  my_str = "  hello  "
  my_str.strip() # This does nothing to my_str
  print(my_str)  # "  hello  "

  # Right
  my_str = "  hello  "
  my_str = my_str.strip()
  print(my_str) # "hello"
  ```
- **Confusing `re.search` and `re.match`**: `re.match` only looks at the very beginning of the string, while `re.search` looks anywhere in the string. `search` is usually what you want.
- **Raw strings for regex patterns**: Forgetting to use raw strings (`r"..."`) for regex patterns. Without the `r` prefix, backslashes in the pattern can be misinterpreted by Python as escape sequences.
- **Unicode errors**: Reading a file with the wrong encoding can lead to a `UnicodeDecodeError`. Always be explicit about encoding when dealing with files, e.g., `open('file.txt', 'r', encoding='utf-8')`.

## 📖 Further Reading
- [Python Docs: String Methods](https://docs.python.org/3/library/stdtypes.html#string-methods)
- [Python Docs: `re` — Regular expression operations](https://docs.python.org/3/library/re.html)
- [Real Python: F-Strings in Python](https://realpython.com/python-f-strings/)
- [The Absolute Minimum Every Software Developer Absolutely, Positively Must Know About Unicode and Character Sets](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/) (A classic and essential read)
