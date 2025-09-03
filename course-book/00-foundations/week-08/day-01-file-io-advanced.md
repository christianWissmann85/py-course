# Week 8, Day 1: Advanced File I/O

## 🎯 Learning Objectives
- [ ] Master the different file modes for reading, writing, and appending.
- [ ] Understand how context managers (`with` statement) ensure files are properly closed.
- [ ] Correctly handle text file encodings to prevent errors.
- [ ] Read and write binary files, such as images or executables.

## 📚 Concepts

### 1. File Modes
When you `open()` a file, you specify a mode. We've used `"r"` (read) and `"w"` (write), but there are others:
- **`"r"`**: Read (default). Raises an error if the file doesn't exist.
- **`"w"`**: Write. Creates the file if it doesn't exist, **truncates (erases) the file if it exists**.
- **`"a"`**: Append. Creates the file if it doesn't exist, and adds new data to the end of the file if it does.
- **`"x"`**: Exclusive creation. Creates the file, but raises a `FileExistsError` if it already exists.
- **`"b"`**: Binary mode. Add this to other modes (e.g., `"rb"`, `"wb"`) to read/write files as bytes instead of text.
- **`"+"`**: Update (reading and writing). Add this to other modes (e.g., `"r+"`, `"w+"`).

### 2. Context Managers (`with` statement)
As we've seen, the `with open(...) as f:` syntax is the standard way to work with files. The `with` statement creates a **context manager**.

The primary benefit is that it **guarantees the file will be closed** (`f.close()` is called automatically) when the block is exited, even if an exception occurs inside the block. This prevents resource leaks.

```python
# The with statement handles closing the file for you.
with open("my_file.txt", "w") as f:
    f.write("This is safe.")
    # f.close() is called automatically here.

# The old, unsafe way (don't do this):
f = open("my_file.txt", "w")
try:
    f.write("This is less safe.")
finally:
    # You must remember to close it yourself, even on error.
    f.close()
```

### 3. Encodings
Text files are stored as bytes. An **encoding** is the rule that maps text characters to bytes.
- `utf-8` is the modern standard and can represent any Unicode character.
- `ascii` is an older, 7-bit encoding for English characters only.
- `cp1252` is a common legacy encoding on Windows.

If you read a file with the wrong encoding, you'll get a `UnicodeDecodeError` or garbled text. **You should always explicitly specify the encoding.**

```python
text = "你好, world! 🌍"

# Write the text to a file using UTF-8 encoding
with open("encoded_text.txt", "w", encoding="utf-8") as f:
    f.write(text)

# Read it back correctly
with open("encoded_text.txt", "r", encoding="utf-8") as f:
    content = f.read()
    assert content == text

# Trying to read it with the wrong encoding will fail
try:
    with open("encoded_text.txt", "r", encoding="ascii") as f:
        f.read()
except UnicodeDecodeError as e:
    print(f"Caught expected error: {e}")
```

### 4. Binary Operations
To work with non-text files like images, sounds, or executables, you must use binary mode. You do this by adding `'b'` to the mode string (e.g., `'rb'` for read-binary, `'wb'` for write-binary).

In binary mode, read operations return `bytes` objects, and you must write `bytes` objects.

```python
# Let's create a simple binary file with 4 bytes
with open("data.bin", "wb") as f:
    f.write(b'\x01\x02\x03\x04')

# Now read the bytes back
with open("data.bin", "rb") as f:
    binary_data: bytes = f.read()
    print(binary_data)        # b'\x01\x02\x03\x04'
    print(list(binary_data)) # [1, 2, 3, 4]
```

### 5. File Seeking
You can move the "cursor" within a file using the `.seek()` method. This is useful for jumping to specific parts of a file without reading everything in between.
- `f.seek(offset, whence)`
  - `offset`: The number of bytes to move.
  - `whence`: The reference point. `0` (default) for the start of the file, `1` for the current position, `2` for the end of the file.

```python
with open("my_file.txt", "w") as f:
    f.write("0123456789")

with open("my_file.txt", "r") as f:
    f.seek(5) # Move to the 6th byte (index 5)
    char = f.read(1) # Read 1 character
    print(char) # '5'

    print(f.tell()) # 6, tells you the current position
```

## 🔹 Quick Exercise

Complete the `read_chunks` generator function. This function should read a large binary file in fixed-size chunks, which is a memory-efficient way to process large files.

```python
from typing import Generator

def read_chunks(filename: str, chunk_size: int = 1024) -> Generator[bytes, None, None]:
    """
    Reads a binary file in chunks of a given size and yields each chunk.
    """
    # Use a 'with' statement and open the file in binary read mode ('rb').
    # Use a while loop with the "walrus operator" (:=) for a concise solution.
    # The loop should continue as long as f.read(chunk_size) returns a non-empty bytes object.
    # Yield each chunk.
    with open(filename, "rb") as f:
        while chunk := f.read(chunk_size):
            yield chunk

# --- Test Setup ---
# Create a dummy binary file for testing
with open("dummy.bin", "wb") as f:
    f.write(b'\x00' * 3000) # Write 3000 zero-bytes

# --- Tests ---
gen = read_chunks("dummy.bin", 1024)
assert len(next(gen)) == 1024
assert len(next(gen)) == 1024
assert len(next(gen)) == 952 # The last chunk is smaller
try:
    next(gen)
    assert False, "Generator should be exhausted"
except StopIteration:
    pass # This is expected
```

## 📝 Daily Assignment
**Goal**: Build a simple file utility that can copy a file, handling text and binary modes correctly.

1.  **Create Project File**: In your project, create a new file: `my_first_poetry_app/file_copier.py`.
2.  **Implement the `copy_file` function**:
    -   `copy_file(source_path: str, dest_path: str, is_binary: bool = False) -> None`.
    -   The function should determine the correct mode (`"r"`/`"w"` or `"rb"`/`"wb"`) based on the `is_binary` flag.
    -   It should open the source file for reading and the destination file for writing.
    -   To be memory-efficient, it should read the source file in chunks (like in the Quick Exercise) and write each chunk to the destination file.
3.  **Add Error Handling**:
    -   Use `try...except` to handle `FileNotFoundError` if the source file doesn't exist.
    -   Handle any other potential `IOError`.
    -   Use logging to report success, errors, and which mode (text/binary) is being used.
4.  **Create a `main` function for demonstration**:
    -   Create a sample text file (`source.txt`).
    -   Call `copy_file` to copy it to `dest.txt`. Verify the contents.
    -   Create a sample binary file (e.g., `source.bin` with some random bytes: `os.urandom(100)`).
    -   Call `copy_file` with `is_binary=True` to copy it to `dest.bin`. Verify the contents.
    -   Demonstrate your error handling by trying to copy a file that doesn't exist.
5.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes
- **Forgetting binary mode for non-text files**: Reading an image or executable in text mode (`"r"`) will result in a `UnicodeDecodeError` or corrupted data because the file's bytes don't conform to a valid text encoding.
- **Reading large files into memory**: `file.read()` reads the *entire* file into memory. This is fine for small files, but for large ones (gigabytes), it will crash your program. Always process large files line-by-line or chunk-by-chunk.
- **Hardcoding file encodings**: While `utf-8` is the best default, sometimes you have to work with legacy files. If so, make the encoding a parameter of your function rather than hardcoding it.
- **Path manipulation with strings**: Using `+` to join path components (`dir + "/" + filename`) is not cross-platform. Always use `pathlib.Path` and the `/` operator.

## 📖 Further Reading
- [Real Python: Reading and Writing Files in Python](https://realpython.com/read-write-files-python/)
- [Python Docs: `open()` built-in function](https://docs.python.org/3/library/functions.html#open)
- [Python Docs: `pathlib` module](https://docs.python.org/3/library/pathlib.html)
