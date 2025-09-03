# Week 15 Project: Terminal Dashboard

## 🎯 Project Goal
To build a functional and visually appealing real-time monitoring dashboard that runs entirely inside the terminal, using the Textual framework. This project will synthesize everything you've learned this week about CLIs, Rich, and TUIs.

## 📚 Background
Dashboards are a critical tool for monitoring systems, applications, and data. While web-based dashboards are common, terminal-based dashboards are lightweight, fast, and can be run anywhere you have an SSH connection. Tools like `htop`, `gtop`, and `bpytop` are popular examples of what's possible in a TUI.

You will build your own dashboard, combining custom widgets, real-time updates, and a clean layout to present information effectively. This project will heavily rely on Textual's features for layout, widgets, and asynchronous updates.

```bash
# Make sure you are in your project directory
poetry add textual
# Optional, for getting real system data in the bonus challenges
poetry add psutil
```

## 📋 Project Requirements

### 1. Application Layout
Your dashboard should have a clear, multi-pane layout. A good starting point would be:
-   A `Header` widget at the top with the title of your dashboard.
-   A `Footer` widget at the bottom to show key bindings (e.g., `q` to quit).
-   A main content area split into several sections. You can use Textual's layout containers (`Horizontal`, `Vertical`, `Grid`) to achieve this. A 2x2 grid is a great starting point.

### 2. Core Widgets
You need to create several "monitoring" widgets. For the core requirement, you can use simulated data that updates on a timer.

-   **CPU Usage Widget**:
    -   Displays a "graph" of CPU usage over time. You can simulate this with a list of random numbers.
    -   Use a `Sparkline` widget from Rich, embedded in a `Static` widget, to display the data.
    -   The widget should have a border and a title (e.g., "CPU Usage (%)").
-   **Memory Usage Widget**:
    -   Displays current RAM and Swap usage with progress bars.
    -   Use Textual's `ProgressBar` widget.
    -   Update the values periodically.
-   **Log Viewer Widget**:
    -   A simple scrolling panel that displays fake log messages.
    -   Use a `Log` widget for this, which is optimized for scrolling text.
    -   Add new log lines on a timer to simulate a live log feed.
-   **Process Table Widget**:
    -   Displays a list of fake processes in a table.
    -   Use Textual's `DataTable` widget.
    -   Columns should include PID, Process Name, CPU %, and Memory %.
    -   The table should be sortable by clicking on the column headers.

### 3. Real-Time Updates
The dashboard must feel "live."
-   Use Textual's timers (`self.set_interval`) in your `on_mount` method to trigger data updates every 1-2 seconds.
-   The timer callbacks should update the data, and reactive attributes (`watch_` methods) should update the widgets on screen.

### 4. Interactivity
-   The application must close cleanly when the user presses `q`. Use Textual's `BINDINGS` for this.
-   The `DataTable` for processes must be interactive, allowing the user to sort by different columns.

## 🏆 Bonus Challenges
1.  **Use Real Data**:
    -   Integrate the `psutil` library to fetch real system data.
    -   Replace your simulated CPU and memory data with actual values from `psutil.cpu_percent()` and `psutil.virtual_memory()`.
    -   Populate the process table with real running processes from `psutil.process_iter()`.
2.  **CLI Entry Point with Click**:
    -   Create a `cli.py` file that uses Click to launch your TUI.
    -   Add a command-line option `--update-interval` to allow the user to specify how frequently the dashboard should refresh.
    -   Pass this value from the Click command into your Textual `App`'s constructor.
3.  **Custom Widget Theming**:
    -   Use Textual's CSS capabilities to create a custom theme for your dashboard.
    -   Define custom colors for borders, titles, and different states (e.g., make the CPU graph turn red if usage is high).
4.  **Configuration from a File**:
    -   Allow users to configure the dashboard via a TOML or YAML file.
    -   The configuration could specify the update interval, color themes, or which widgets to display.

## 💡 Tips
-   **Build Widgets Incrementally**: Create each widget as a separate custom class (e.g., `CPUWidget(Static):`, `MemoryWidget(Static):`). Get each one working on its own before trying to combine them into a single layout.
-   **Use Reactive Variables for State**: For each widget, store its data in reactive variables (e.g., `cpu_data = reactive([])`). This will make updating the UI much cleaner. The timer will update the data, and the `watch_` method will update the display.
-   **Start with a Static Layout**: Don't worry about resizing and responsiveness at first. Get your widgets arranged in a fixed grid or layout.
-   **Read the Docs**: The Textual documentation is excellent. The guides on custom widgets, timers, and the `DataTable` will be especially helpful.
-   **Logging is Your Friend**: When things go wrong in a TUI, `print()` won't work. Use Textual's built-in logging by calling `self.log(...)` from anywhere in your app. You can view the logs by running your app with the `textual run --dev my_app.py` command.
