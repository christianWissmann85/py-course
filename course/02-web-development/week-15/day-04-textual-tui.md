# Week 15, Day 4: The Textual TUI Framework

## 🎯 Learning Objectives
- [ ] Understand what a Textual User Interface (TUI) is and when it's useful.
- [ ] Learn the basic architecture of a Textual application.
- [ ] Compose a user interface using Textual's built-in widgets (e.g., `Label`, `Button`, `Input`).
- [ ] Understand Textual's layout system (vertical, horizontal, and grid).
- [ ] Handle user input and widget interactions through the event system.

## 📚 Concepts

### 1. From CLI to TUI
While CLIs are powerful, they are often stateless—you run a command, it prints output, and it exits. A **Textual User Interface (TUI)** is a full-blown application that runs inside your terminal. It has a persistent state, interactive components (widgets), a layout, and responds to user events like key presses and mouse clicks.

**Textual** is a rapidly evolving framework by the author of Rich that makes building TUIs in Python incredibly easy and powerful. It uses Rich for rendering, so everything looks beautiful out of the box.

First, install it: `poetry add textual`

### 2. The Textual App
A Textual application is a class that inherits from `textual.app.App`.

```python
from textual.app import App, ComposeResult
from textual.widgets import Header, Footer, Label

class SimpleApp(App):
    """A very simple Textual app."""

    # The compose method is where you define the layout of your app.
    # It should yield the widgets you want to display.
    def compose(self) -> ComposeResult:
        yield Header()
        yield Label("Hello, World!")
        yield Footer()

if __name__ == "__main__":
    app = SimpleApp()
    app.run() # This starts the application.
```
-   `App`: The base class for all Textual apps.
-   `compose()`: This method is called by Textual to build the UI. You `yield` the widgets you want to appear on the screen.
-   `app.run()`: Starts the application, takes over the terminal, and runs the event loop.

### 3. Widgets
Widgets are the building blocks of a Textual UI. Textual comes with a rich library of them.
-   `Header`: A standard header with a title.
-   `Footer`: A footer that can display key bindings.
-   `Label`: A simple, non-interactive text widget.
-   `Button`: A clickable button.
-   `Input`: A text input field.
-   `Static`: A basic widget for displaying text or other renderables, often with more styling options than `Label`.
-   `DataTable`: A powerful table widget.
-   `Tree`: A tree view widget.

You can style widgets using CSS. Textual has its own CSS-like styling system that is incredibly powerful.

```python
from textual.app import App, ComposeResult
from textual.widgets import Static, Button

class ButtonApp(App):
    # This is Textual CSS. It styles the screen and the button.
    CSS = """
    Screen {
        align: center middle;
    }
    Button {
        width: 20;
    }
    """

    def compose(self) -> ComposeResult:
        yield Static("Press the button!", classes="header")
        yield Button("Press Me", variant="primary")
```

### 4. Layout System
Textual uses a modern CSS-like layout system. The primary layout modes are `vertical`, `horizontal`, and `grid`. You can group widgets inside **containers** to arrange them.

```python
from textual.app import App, ComposeResult
from textual.containers import Horizontal, Vertical
from textual.widgets import Static

class LayoutApp(App):
    CSS = """
    .box {
        border: solid green;
        height: auto;
        width: 1fr; /* 1 fraction of available space */
    }
    """
    def compose(self) -> ComposeResult:
        # A horizontal container holds two vertical containers.
        with Horizontal():
            with Vertical():
                yield Static("Left Pane - Item 1", classes="box")
                yield Static("Left Pane - Item 2", classes="box")
            with Vertical():
                yield Static("Right Pane", classes="box")
```

### 5. Event Handling
TUIs are interactive. Textual uses an event system to handle user input. You create methods named `on_<widget_type>_<event>()` to handle events.

```python
from textual.app import App, ComposeResult
from textual.widgets import Button, Static

class CounterApp(App):
    def __init__(self):
        super().__init__()
        self.counter = 0
        self.display = Static(f"Count: {self.counter}")

    def compose(self) -> ComposeResult:
        yield self.display
        yield Button("+1", id="plus")
        yield Button("-1", id="minus")

    # This method is automatically called when the button with id="plus" is pressed.
    def on_button_pressed(self, event: Button.Pressed) -> None:
        if event.button.id == "plus":
            self.counter += 1
        elif event.button.id == "minus":
            self.counter -= 1

        self.display.update(f"Count: {self.counter}")
```
-   **Event Handlers**: Methods like `on_button_pressed` are automatically discovered by Textual. The method name is derived from the event type.
-   **Event Object**: The `event` parameter contains information about the event, such as which button was pressed (`event.button`).
-   **Updating Widgets**: You can update a widget's content by calling its `.update()` method.

## 🔹 Quick Exercise
Create a simple Textual app that has a single `Input` widget and a `Label`. When you type something in the input field, the label below it should update to show what you've typed.

```python
from textual.app import App, ComposeResult
from textual.widgets import Input, Label

class InputMirrorApp(App):
    def compose(self) -> ComposeResult:
        yield Input(placeholder="Type here...")
        yield Label("...", id="mirror")

    # This event is fired every time the Input's value changes.
    def on_input_changed(self, event: Input.Changed) -> None:
        # Get the label widget using a CSS selector.
        mirror_label = self.query_one("#mirror", Label)
        # Update the label with the input's new value.
        mirror_label.update(f"You typed: {event.value}")

# if __name__ == "__main__":
#     app = InputMirrorApp()
#     app.run()
```

## 📝 Daily Assignment
**Goal**: Build a simple TUI front-end for your to-do list application.

You will create a Textual app that displays the to-do list from `todos.txt` and allows basic interaction.

1.  **Create the App**: Create a new file `todo_tui.py` and define a `TodoApp` class.
2.  **Display the List**:
    -   In the `compose` method, read the `todos.txt` file.
    -   For each to-do item, create a `Label` or `Static` widget and `yield` it. This will display the list vertically.
    -   Use styling to differentiate between completed and pending tasks (e.g., using Rich's BBCode and the `renderable` property of a `Static` widget).
3.  **Add an Input and Button**:
    -   Add an `Input` widget at the bottom for adding new tasks.
    -   Add a `Button` next to it labeled "Add".
4.  **Implement 'Add' Functionality**:
    -   When the "Add" button is pressed, get the text from the `Input` widget.
    -   Append this new text to your `todos.txt` file.
    -   **Crucially**, you need to refresh the display. A simple way to do this is to have a method that rebuilds the list of `Label`s and updates the screen. Textual has more advanced ways to do this (e.g., reactive variables), but a manual refresh is a good start.
5.  **Implement 'Done' Functionality (Bonus)**:
    -   Instead of `Label`s, use `Button`s for each to-do item.
    -   When a to-do item's button is pressed, mark that item as complete in the `todos.txt` file and refresh the screen to show the change in styling. You'll need a way to link each button to its corresponding line number.

## 📖 Further Reading
- [Textual Official Documentation](https://textual.textualize.io/)
- [Textual GitHub Repository](https://github.com/Textualize/textual)
- [Textual Tutorials](https://textual.textualize.io/tutorial/)
- [Building a Python TUI with Textual](https://www.youtube.com/watch?v=W_3_8W2L4sI) (Video)
