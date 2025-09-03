# Week 15, Day 5: Building Interactive Textual Apps

## 🎯 Learning Objectives
- [ ] Learn advanced techniques for handling user input and actions.
- [ ] Understand and implement state management in a Textual app.
- [ ] Use reactive attributes and data binding to automatically update the UI when data changes.
- [ ] Learn how to create custom, reusable widgets.
- [ ] Explore Textual's "screen" system for managing different views in an application.

## 📚 Concepts

### 1. Advanced Event Handling: Actions
Yesterday we saw how to handle events with `on_*` methods. A more scalable way to handle events, especially from custom widgets, is to use **Actions**.

An action is a method on your App or Widget prefixed with `action_`. You can bind these actions to key presses or have a widget invoke them.

```python
from textual.app import App
from textual.widgets import Button

class ActionApp(App):
    # BINDINGS maps a key press to an action name and a description.
    # The description is shown in the Footer.
    BINDINGS = [
        ("q", "quit", "Quit"),
        ("a", "add_item('thing')", "Add a thing"),
    ]

    def action_quit(self) -> None:
        """Called when the user presses 'q'."""
        self.exit()

    def action_add_item(self, item: str) -> None:
        """Called when the user presses 'a'."""
        self.log(f"Added item: {item}")
```
-   `BINDINGS`: A class variable that defines key bindings. When the user presses 'q', Textual will look for and call a method named `action_quit`.
-   `action_*`: Any method with this prefix is an action that can be called.

### 2. State Management and Reactive Attributes
In our `CounterApp` yesterday, we manually updated the display widget whenever the `counter` variable changed. This can get tedious. Textual provides **reactive attributes** that automatically trigger an update when their value changes.

```python
from textual.app import App, ComposeResult
from textual.widgets import Button, Static
from textual.reactive import reactive

class ReactiveCounterApp(App):
    # 'counter' is now a reactive attribute.
    # The 'watch_counter' method will be called automatically whenever its value changes.
    counter = reactive(0)

    def compose(self) -> ComposeResult:
        yield Static(f"Count: {self.counter}", id="display")
        yield Button("+1", id="plus")

    # This is a "watch" method. It "watches" the 'counter' attribute.
    def watch_counter(self, new_value: int) -> None:
        """Called when self.counter changes."""
        self.query_one("#display", Static).update(f"Count: {new_value}")

    def on_button_pressed(self, event: Button.Pressed) -> None:
        self.counter += 1 # This will automatically trigger watch_counter!
```
-   `reactive()`: A decorator that turns a class attribute into a reactive one.
-   `watch_<attribute_name>()`: A special method that Textual calls whenever the corresponding reactive attribute is modified. This is the core of Textual's data binding system.

### 3. Creating Custom Widgets
For reusability, you can create your own widgets by subclassing `textual.widget.Widget` or another existing widget. This lets you encapsulate a piece of UI and its logic.

```python
from textual.app import App, ComposeResult
from textual.widgets import Static, Button
from textual.reactive import reactive

# --- Our Custom Widget ---
class Counter(Static):
    """A widget that displays a counter and has its own button."""
    counter = reactive(0)

    def compose(self) -> ComposeResult:
        yield Button(f"Press me! ({self.counter})")

    def watch_counter(self, new_value: int) -> None:
        # Update the button's label when the counter changes.
        self.query_one(Button).label = f"Press me! ({new_value})"

    def on_button_pressed(self, event: Button.Pressed) -> None:
        self.counter += 1

# --- The Main App ---
class CustomWidgetApp(App):
    def compose(self) -> ComposeResult:
        # We can now use our custom widget like any other.
        yield Counter()
        yield Counter() # Each instance has its own state.
```

### 4. Managing Views with Screens
For any non-trivial application, you'll need more than one "view" (e.g., a login screen, a main screen, a settings screen). Textual manages this with `Screen` objects.

A `Screen` is like a self-contained mini-app with its own `compose` method, event handlers, and actions.

```python
from textual.app import App, ComposeResult
from textual.screen import Screen
from textual.widgets import Button, Header, Footer

class QuitScreen(Screen):
    """A modal dialog screen to confirm quitting."""
    def compose(self) -> ComposeResult:
        yield Button("Yes, quit now", variant="error", id="quit")
        yield Button("No, go back", variant="primary", id="back")

    def on_button_pressed(self, event: Button.Pressed) -> None:
        if event.button.id == "quit":
            self.app.exit() # self.app refers to the main App instance
        else:
            self.app.pop_screen() # pop_screen returns to the previous screen

class MainApp(App):
    BINDINGS = [("q", "request_quit", "Quit")]

    def compose(self) -> ComposeResult:
        yield Header()
        yield Footer()

    def action_request_quit(self) -> None:
        # push_screen overlays the new screen on top of the current one.
        self.push_screen(QuitScreen())
```

## 🔹 Quick Exercise
Create a custom widget `WelcomeWidget` that contains a `Static` label and an `Input` widget. When the user types their name into the input, the static label should update to say "Welcome, [Name]!".

```python
from textual.app import App, ComposeResult
from textual.widgets import Input, Static
from textual.widget import Widget

# --- Answer ---
class WelcomeWidget(Widget):
    def compose(self) -> ComposeResult:
        yield Static("Please enter your name:", id="label")
        yield Input(placeholder="Your name")

    def on_input_changed(self, event: Input.Changed) -> None:
        # Update the static label within this widget.
        self.query_one("#label", Static).update(f"Welcome, {event.value}!")

class WelcomeApp(App):
    def compose(self) -> ComposeResult:
        yield WelcomeWidget()
```

## 📝 Daily Assignment
**Goal**: Evolve yesterday's TUI to-do list into a more robust and interactive application.

1.  **Create Custom Widgets**:
    -   Create a custom widget called `TodoItem` (e.g., inheriting from `Static`). It should be responsible for displaying a single to-do item's text.
    -   Create another custom widget called `TodoList` that is responsible for displaying a list of `TodoItem` widgets.

2.  **Use Reactive Attributes**:
    -   In your main `TodoApp`, create a reactive attribute `todos = reactive([])`. This list will be your single source of truth for the to-do items.
    -   Create a `watch_todos` method. When the `todos` list changes, this method should be responsible for clearing and re-populating the `TodoList` widget.

3.  **Refactor 'Add' Functionality**:
    -   When the "Add" button is pressed, instead of writing to the file and manually rebuilding the UI, you should simply append the new to-do text to the `self.todos` list. The reactive nature of the attribute will handle the UI update automatically.

4.  **Implement 'Done' Functionality using Actions**:
    -   Add a "Done" `Button` to your `TodoItem` custom widget.
    -   When this button is clicked, it should **post a message** (a custom event). For example: `self.post_message(self.TodoCompleted(item_text=self.text))`. You'll need to define `TodoCompleted` as a custom `Message` class.
    -   In your main `TodoApp`, create an `on_todo_item_todo_completed` event handler. This handler will receive the message, find the corresponding item in the `self.todos` list, mark it as complete, and the reactive system will update the UI. This decouples the child widget from the parent application logic.

5.  **Data Persistence (Bonus)**:
    -   Modify your app to load the to-do items from `todos.txt` when it starts up (`on_mount` event handler) and save the entire `self.todos` list back to the file whenever it changes (`watch_todos`).

## 📖 Further Reading
- [Textual Documentation: Reactive Attributes](https://textual.textualize.io/guide/reactivity/)
- [Textual Documentation: Custom Widgets](https://textual.textualize.io/guide/widgets/#custom-widgets)
- [Textual Documentation: Screens](https://textual.textualize.io/guide/screens/)
- [Textual Documentation: Actions](https://textual.textualize.io/guide/actions/)
