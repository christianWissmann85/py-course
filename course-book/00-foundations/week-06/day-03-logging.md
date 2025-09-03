# Week 6, Day 3: Logging

## 🎯 Learning Objectives
- [ ] Understand why logging is superior to using `print()` for debugging and monitoring.
- [ ] Use Python's built-in `logging` module to record events.
- [ ] Differentiate between the five standard logging levels.
- [ ] Configure loggers with different handlers and formatters to control log output.

## 📚 Concepts

### 1. Why Logging?
While `print()` is great for quick debugging, it has major limitations in larger applications:
- You can't easily turn print statements on or off.
- You can't control the "severity" of a message (e.g., debug vs. critical error).
- You can't easily direct messages to different places (console, file, network).

The `logging` module solves all of these problems. It's a powerful and configurable system for recording events from your application.

### 2. Logging Levels
The logging module provides five standard levels of severity. When you configure a logger, you set a minimum level; any message with a severity below that level will be ignored.

- **`DEBUG`**: Detailed information, typically of interest only when diagnosing problems.
- **`INFO`**: Confirmation that things are working as expected.
- **`WARNING`**: An indication that something unexpected happened, or a hint of some problem in the near future (e.g., 'disk space low'). The software is still working as expected. (This is the default level).
- **`ERROR`**: Due to a more serious problem, the software has not been able to perform some function.
- **`CRITICAL`**: A very serious error, indicating that the program itself may be unable to continue running.

```python
import logging

logging.basicConfig(level=logging.DEBUG) # Configure the root logger

logging.debug("This is a debug message.")
logging.info("This is an info message.")
logging.warning("This is a warning message.")
logging.error("This is an error message.")
logging.critical("This is a critical message.")
```

### 3. Loggers, Handlers, and Formatters
The logging system is built on three main components:
- **Loggers**: The entry point into the logging system. You get a logger instance (`logging.getLogger(__name__)`) and call methods on it (`.info()`, `.warning()`, etc.). Using `__name__` creates a logger named after the current module, which is a best practice.
- **Handlers**: These send the log records to the appropriate destination.
  - `StreamHandler`: Sends logs to a stream, like the console (`sys.stdout`).
  - `FileHandler`: Sends logs to a file.
- **Formatters**: These specify the layout of the log records. You can include the timestamp, level name, logger name, and the message itself.

### 4. Basic Configuration
`logging.basicConfig()` is a quick way to set up the root logger, but for more control, you configure a logger object directly.

```python
import logging
import sys

# 1. Get a logger instance
log = logging.getLogger(__name__)
log.setLevel(logging.INFO) # Set the minimum level for this logger

# 2. Create a handler (e.g., to print to the console)
handler = logging.StreamHandler(sys.stdout)

# 3. Create a formatter
formatter = logging.Formatter(
    '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# 4. Set the formatter for the handler and add the handler to the logger
handler.setFormatter(formatter)
log.addHandler(handler)

# Now, use the logger
log.info("This is a well-configured informational message.")
log.warning("This is a warning.")
log.debug("This message will NOT appear, because the level is set to INFO.")
```

### 5. Best Practices
- **Use `logging.getLogger(__name__)`**: Always create a logger instance named after your module. This lets you configure logging for different parts of your application independently.
- **Log what, not how**: Log messages should describe the event, not the implementation detail. Good: `"User authentication failed for user 'alice'"`. Bad: `"Function 'authenticate' returned False"`.
- **Don't use string formatting**: Pass data as arguments to the logging methods. The logger will only format the string if the message is actually going to be processed, which is more efficient.
  ```python
  # Good, efficient way
  log.info("User %s logged in from IP %s", username, ip_address)

  # Bad, inefficient way (f-string is evaluated even if message is ignored)
  log.info(f"User {username} logged in from IP {ip_address}")
  ```

## 🔹 Quick Exercise

Complete the `setup_logger` function below. It should create and configure a logger that writes messages to both the console and a file.

```python
import logging
import sys

def setup_logger(name: str, log_file: str, level=logging.INFO) -> logging.Logger:
    """
    Sets up a logger that logs to both a file and the console.
    """
    logger = logging.getLogger(name)
    logger.setLevel(level)

    # Create a formatter
    formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')

    # Create a FileHandler
    file_handler = logging.FileHandler(log_file)
    file_handler.setFormatter(formatter)

    # Create a StreamHandler (for console)
    stream_handler = logging.StreamHandler(sys.stdout)
    stream_handler.setFormatter(formatter)

    # Add handlers to the logger
    logger.addHandler(file_handler)
    logger.addHandler(stream_handler)

    return logger

# --- Tests ---
my_app_logger = setup_logger("my_app", "app.log")
my_app_logger.info("This is an info message for the app.")
my_app_logger.warning("This is a warning.")

# Check the app.log file to see if the messages were written.
# You should also see them on the console.
```

## 📝 Daily Assignment
**Goal**: Integrate logging into the robust script you built on Day 1.

1.  **Refactor `robust_script.py`**: Open the script you created for Day 1's assignment (`robust_script.py`).
2.  **Set up Logging**:
    - At the top of the script, set up a logger.
    - Configure it to log `INFO` level messages and above to the console.
    - Configure it to also log `DEBUG` level messages and above to a file named `processing.log`. This means the file will have more detailed logs than the console.
3.  **Replace `print` with Logging**:
    - Go through your `process_user_data` function.
    - Replace all `print()` calls with appropriate logging calls:
        - Successful processing messages should be `log.info()`.
        - Error messages inside `except` blocks should be `log.error()` or `log.warning()`.
        - Add `log.debug()` messages to show more detail, like the specific user dictionary being processed.
4.  **Add Logging for Analysis**:
    - At the end of the script, after the loop is done, log a summary message at the `INFO` level. e.g., `f"Processed {total_records} records. {success_count} successful, {failure_count} failed."`
5.  **Run and Verify**:
    - Run the script. Check that the console output shows only `INFO` and `ERROR`/`WARNING` messages.
    - Open `processing.log`. Verify that it contains the `DEBUG` messages as well.

## ⚠️ Common Mistakes
- **Calling `basicConfig` after getting a logger**: `logging.basicConfig()` only works if the root logger has not already been configured. If you call `getLogger()` first, `basicConfig()` does nothing.
- **Adding handlers multiple times**: If you run a setup function multiple times, you might add the same handler to a logger repeatedly, which will cause your log messages to be duplicated. Check if a logger already has handlers before adding more: `if not logger.handlers: ...`.
- **Using the root logger directly**: Calling `logging.info()` uses the root logger. While fine for simple scripts, in libraries or larger applications, it's better to get your own logger with `logging.getLogger(__name__)` so that the application using your library can control its log output.

## 📖 Further Reading
- [Python Docs: Logging HOWTO](https://docs.python.org/3/howto/logging.html) (The essential guide)
- [Python Docs: `logging` module reference](https://docs.python.org/3/library/logging.html)
- [Real Python: Logging in Python](https://realpython.com/python-logging/)
