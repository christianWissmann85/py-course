# Week 15, Day 2: The Click Framework

## 🎯 Learning Objectives

- [ ] Understand the advantages of using the Click framework over `argparse`.
- [ ] Learn to build a CLI using Click's decorator-based approach.
- [ ] Create commands with options and arguments.
- [ ] Structure a complex CLI using command groups (subcommands).
- [ ] Learn how to use the `Context` object to pass state between commands.

## 📚 Concepts

### 1. Why Click? `argparse` vs. Click

While `argparse` is powerful and built-in, its API can be verbose. **Click** is a popular third-party library that aims to make the process of creating command-line tools more intuitive and less boilerplate-heavy by using decorators.

| Feature             | `argparse`                                              | `Click`                                             |
| ------------------- | ------------------------------------------------------- | --------------------------------------------------- |
| **API Style**       | Imperative (create parser, add arguments)               | Declarative (decorate functions)                    |
| **Code Structure**  | Logic is separate from parser definition.               | Logic is attached directly to the command function. |
| **Subcommands**     | Requires explicit subparser objects.                    | Handled elegantly with command groups.              |
| **Help Generation** | Good, but requires manual formatting for complex cases. | Excellent, automatically generated, and colorful.   |
| **Extensibility**   | Limited.                                                | Highly extensible and composable.                   |

Click's philosophy is that your command-line tool's logic should be encapsulated in functions, and Click simply exposes those functions to the command line.

First, install it: `poetry add click`

### 2. Basic Commands with `@click.command()`

In Click, a simple command is just a decorated function. The function's parameters become the CLI's arguments and options.

```python
import click

@click.command()
@click.option('--times', default=1, help='Number of times to greet.')
@click.argument('name')
def greet(times: int, name: str):
    """A simple program that greets the user."""
    for _ in range(times):
        click.echo(f"Hello, {name}!")

if __name__ == '__main__':
    greet()
```

- `@click.command()`: Marks the `greet` function as a CLI command.
- `@click.option()`: Defines an optional argument (`--times`). Click infers the type from the function's type hint (`int`) or the `default` value.
- `@click.argument()`: Defines a positional argument (`name`).
- `click.echo()`: A wrapper around `print()` that is more robust and supports color.
- The docstring of the function is automatically used in the `--help` message!

**How to run it:**

```bash
$ python my_script.py --help
# Usage: my_script.py [OPTIONS] NAME
#
#   A simple program that greets the user.
#
# Arguments:
#   NAME  [required]
#
# Options:
#   --times INTEGER  Number of times to greet.
#   --help           Show this message and exit.

$ python my_script.py Alice --times 2
# Hello, Alice!
# Hello, Alice!
```

### 3. Options and Arguments

- `@click.option()`:
  - Defined with a `--` prefix. A short version can be added: `@click.option('--verbose', '-v', ...)`.
  - `is_flag=True` is used for boolean flags like `--verbose`.
  - `prompt=True` will prompt the user for input if the option is not provided.
  - `type` can be specified explicitly, e.g., `type=click.Choice(['red', 'green', 'blue'])`.
- `@click.argument()`:
  - Defined by its name in uppercase.
  - Can have `nargs=-1` to accept a variable number of arguments (which will be passed as a tuple).

### 4. Command Groups for Subcommands

To create a tool with subcommands (like `git commit`), you use `@click.group()`. The main function becomes a "group" that other commands can attach to.

```python
import click

# 1. Create the main group
@click.group()
@click.option('--verbose', is_flag=True)
def cli(verbose: bool):
    """A fake version control system."""
    if verbose:
        click.echo("Verbose mode enabled.")

# 2. Add a command to the group
@cli.command()
@click.option('-m', '--message', required=True, help='Commit message.')
def commit(message: str):
    """Commit changes to the repository."""
    click.echo(f"Committing with message: '{message}'")

# 3. Add another command
@cli.command()
@click.argument('remote')
@click.option('--force', is_flag=True, help='Force the push.')
def push(remote: str, force: bool):
    """Push changes to a remote."""
    click.echo(f"Pushing to remote '{remote}'...")
    if force:
        click.echo("FORCING the push!")

if __name__ == '__main__':
    cli()
```

This structure is much more intuitive and readable than the `argparse` equivalent.

### 5. The Context Object

Sometimes, a parent command needs to pass information down to its subcommands. Click manages this through a **Context** object.

You can get the current context using `@click.pass_context`, and you can store arbitrary data in its `ctx.obj` attribute.

```python
import click

class AppState:
    """A simple class to hold our application's state."""
    def __init__(self):
        self.verbose = False

@click.group()
@click.option('--verbose', is_flag=True)
@click.pass_context
def cli(ctx, verbose: bool):
    """A CLI that passes state via the context."""
    # Create an AppState object and attach it to the context.
    ctx.obj = AppState()
    ctx.obj.verbose = verbose

@cli.command()
@click.pass_context
def status(ctx):
    """Check the status."""
    click.echo("Checking status...")
    # Access the state from the parent command.
    if ctx.obj.verbose:
        click.echo("Verbose status check enabled.")

if __name__ == '__main__':
    cli()
```

## 🔹 Quick Exercise

Convert the rectangle area calculator from yesterday's `argparse` exercise to use Click.

- It should be a single command.
- It should take `length` and `width` as required arguments.
- It should have an optional `--verbose` flag.

```python
import click

# --- Answer ---
@click.command()
@click.argument('length', type=float)
@click.argument('width', type=float)
@click.option('--verbose', '-v', is_flag=True, help="Enable verbose output.")
def area(length: float, width: float, verbose: bool):
    """Calculate the area of a rectangle."""
    if verbose:
        click.echo(f"Calculating area for a {length}x{width} rectangle.")

    result = length * width
    click.echo(f"The area is: {result}")

# if __name__ == '__main__':
#     area()
```

## 📝 Daily Assignment

**Goal**: Refactor the `argparse`-based to-do list CLI from yesterday to use Click.

1.  **Create a New Script**: Create `todo_click.py`.
2.  **Use a Command Group**: The main `cli` function should be a `@click.group()`.
3.  **Convert Subcommands**:
    - Convert the `add` subcommand into a function decorated with `@cli.command()`. It should take the description as a `@click.argument()`.
    - Convert the `list` subcommand. Use a `@click.option('--all')` flag.
    - Convert the `done` subcommand. It should take the item number as a `@click.argument()`.
4.  **Improve Output**: Use `click.echo()` for all output. Try using `click.secho()` to add some color (e.g., print completed items in green).
5.  **Type Hinting**: Ensure all your command functions and their parameters are fully type-hinted. Click uses this for automatic type conversion.
6.  **Context for File Path (Bonus)**: Use the `Context` object to pass the path to the `todos.txt` file from the main `cli` group down to the subcommands, so you don't have to hardcode the filename in each function.

Your final CLI should have the same functionality as yesterday's, but the code should be cleaner and more declarative.

## 📖 Further Reading

- [Click Official Documentation](https://click.palletsprojects.com/)
- [Building Command-Line APIs with Click](https://www.realpython.com/python-click/)
- [Typer](https://typer.tiangolo.com/): A library built on top of Click that uses type hints even more extensively to create CLIs with almost no boilerplate. It's a great next step after learning Click.
