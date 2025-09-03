# Week 15, Day 3: Beautiful CLIs with Rich

## 🎯 Learning Objectives
- [ ] Understand how the Rich library can be used to create beautiful terminal interfaces.
- [ ] Learn to print richly formatted text and data structures.
- [ ] Display data in clean, formatted tables.
- [ ] Render beautiful, animated progress bars for long-running tasks.
- [ ] Learn to display other advanced elements like syntax-highlighted code and formatted tracebacks.

## 📚 Concepts

### 1. What is Rich?
**Rich** is a Python library for writing rich text and beautiful formatting to the terminal. It takes the pain out of creating CLI output that is easy to read and visually appealing. Rich can render colors, styles, tables, progress bars, markdown, syntax-highlighted code, and more.

It's the perfect companion to a CLI framework like Click or `argparse`.

First, install it: `poetry add rich`

### 2. The `Console` and `rich.print`
The main entry point for Rich is the `Console` object. You can use its `.print()` method as a powerful, style-aware replacement for the built-in `print()`.

Rich uses a BBCode-like syntax for inline styling.

```python
from rich.console import Console

# Create a console object
console = Console()

# Basic printing with styles
console.print("This is some regular text.")
console.print("This is bold.", style="bold")
console.print("This is bold red.", style="bold red")
console.print("This has a background.", style="white on blue")

# Using BBCode for inline styles
console.print("This text is [bold]bold[/bold] and this is [underline]underlined[/underline].")
console.print("[bold red]Alert![/bold red] [green]Everything is fine.[/green]")

# Rich's print can also beautifully format Python objects
my_dict = {"name": "Alice", "job": "Developer", "skills": ["Python", "SQL", "Docker"]}
console.print(my_dict)
```

### 3. Displaying Data with Tables
One of Rich's most powerful features is its ability to render beautiful tables directly in the terminal. This is fantastic for displaying structured data.

```python
from rich.console import Console
from rich.table import Table

console = Console()

# Create a table instance
table = Table(title="My User Data")

# Add columns
table.add_column("ID", justify="right", style="cyan", no_wrap=True)
table.add_column("Name", style="magenta")
table.add_column("Email", style="green")

# Add rows
table.add_row("1", "Alice", "alice@example.com")
table.add_row("2", "Bob", "bob@example.com")
table.add_row("3", "Charlie", "charlie@example.com")

# Print the table to the console
console.print(table)
```

### 4. Progress Bars
Rich makes creating progress bars for long-running tasks incredibly simple.

**Option 1: The `track` function (for simple loops)**
```python
import time
from rich.progress import track

for step in track(range(100), description="Processing..."):
    # Simulate some work
    time.sleep(0.1)
```
This will automatically display a description, a progress bar, percentage complete, and estimated time remaining.

**Option 2: The `Progress` context manager (for more control)**
```python
import time
from rich.progress import Progress

with Progress() as progress:
    task1 = progress.add_task("[red]Downloading...", total=1000)
    task2 = progress.add_task("[green]Processing...", total=1000)

    while not progress.finished:
        progress.update(task1, advance=1.5)
        progress.update(task2, advance=1.0)
        time.sleep(0.02)
```

### 5. Other Rich Renderables
Rich can render many other things beautifully:
-   **Markdown**: `console.print(Markdown("# This is a header"))`
-   **Syntax Highlighting**: `console.print(Syntax(my_code_string, "python", theme="monokai"))`
-   **Tracebacks**: Rich can be installed as the default traceback handler, which makes debugging exceptions much more pleasant.
    ```bash
    # Run this once to install the handler
    # python -m rich.traceback.install
    ```
    After this, any unhandled exception will be beautifully formatted with syntax highlighting and context.

## 🔹 Quick Exercise
Create a Rich `Table` to display a list of files and their sizes.
-   The table should have two columns: "Filename" and "Size (bytes)".
-   The "Size" column should be right-aligned.
-   Add at least two rows of fake file data.

```python
from rich.console import Console
from rich.table import Table

# --- Answer ---
console = Console()
table = Table(title="File Sizes")
table.add_column("Filename", style="cyan")
table.add_column("Size (bytes)", justify="right", style="green")

table.add_row("my_script.py", "1,234")
table.add_row("data.json", "5,678")
table.add_row("photo.jpg", "98,765")

console.print(table)
```

## 📝 Daily Assignment
**Goal**: Enhance your Click-based to-do list CLI from yesterday with a beautiful interface using Rich.

1.  **Integrate Rich**: Add `rich` to your project (`poetry add rich`) and create a `Console` object at the top of your `todo_click.py` script.
2.  **Prettify the `list` command**:
    -   Instead of printing plain text, use a Rich `Table` to display the to-do items.
    -   The table should have columns for "ID", "Description", and "Status".
    -   Use color and style to make it look good. For example, the "Status" could be "[green]Done[/green]" or "[yellow]Pending[/yellow]".
    -   Completed tasks (those starting with `[x]`) could have a strikethrough style applied to their description row.
3.  **Improve Feedback in `add` and `done`**:
    -   When a user adds a new task, print a confirmation message in color, e.g., `console.print("[green]✓[/green] Added new to-do: 'Buy milk'")`.
    -   When a user completes a task, print a similar confirmation message, e.g., `console.print("[bold green]✓ Marked 'Call the dentist' as done.[/bold green]")`.
4.  **Add a Progress Bar (Bonus)**:
    -   Create a new command called `simulate-work`.
    -   This command should just have a simple loop that runs for a few seconds.
    -   Wrap the loop with Rich's `track` function to show a progress bar while it's "working". This simulates how you would show progress for a long-running task in a real CLI.
5.  **Install the Traceback Handler**:
    -   Run `python -m rich.traceback.install` in your environment.
    -   Intentionally cause an error in your CLI (e.g., try to `done` an item number that doesn't exist and let it raise an `IndexError`) to see the beautifully formatted traceback.

## 📖 Further Reading
- [Rich Official Documentation](https://rich.readthedocs.io/en/latest/)
- [Rich on GitHub](https://github.com/Textualize/rich) (The README is a great gallery of features)
- [Building a Beautiful Python CLI with Rich](https://www.youtube.com/watch?v=4zbe_iI2g2Q) (Video Tutorial)
