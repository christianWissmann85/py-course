# Week 7, Day 4: Virtual Environments

## 🎯 Learning Objectives
- [ ] Understand why virtual environments are essential for Python development.
- [ ] Create and manage virtual environments using Python's built-in `venv` module.
- [ ] Understand how Poetry automates virtual environment management.
- [ ] Use environment variables for configuration.

## 📚 Concepts

### 1. Why Virtual Environments?
Imagine you have two projects:
- Project A requires version 1.0 of a library.
- Project B requires version 2.0 of the *same* library.

If you install libraries globally on your system, you can't satisfy both requirements. You would have a version conflict.

A **virtual environment** is an isolated Python environment that has its own installation directories and dependencies. This allows every project to have its own set of dependencies, independent of other projects. It's one of the most crucial tools for professional Python development.

### 2. Creating Environments with `venv`
Python comes with a built-in module, `venv`, for creating virtual environments.

```bash
# 1. Navigate to your project directory
mkdir my-venv-project
cd my-venv-project

# 2. Create a virtual environment named 'venv'
# This creates a 'venv/' directory in your project.
python -m venv venv

# 3. Activate the environment
# On Linux/macOS:
source venv/bin/activate
# On Windows:
# venv\Scripts\activate

# Your shell prompt will change to show you're in the venv.
# (venv) $

# 4. Now, any 'pip install' will only affect this environment
pip install requests

# 5. Deactivate when you're done
deactivate
```
**Important**: You should add your virtual environment directory (`venv/`) to your `.gitignore` file.

### 3. Poetry and Virtual Environments
We've been using virtual environments all along without thinking about it, because **Poetry handles them for us automatically!**

When you run `poetry install` or `poetry add`, Poetry:
1.  Checks if a virtual environment exists for the project.
2.  If not, it creates one for you (usually in a central cache directory, not in your project folder).
3.  It installs the dependencies into that specific virtual environment.

When you run a command with `poetry run`, Poetry automatically activates the correct virtual environment for that command and then deactivates it. This is why we don't need to run `source venv/bin/activate` ourselves.

```bash
# To find out where Poetry created the virtual environment for your project:
poetry env info

# To manually activate it (rarely needed):
poetry shell

# To run a single command within it:
poetry run python my_script.py
```

### 4. Environment Variables
Environment variables are variables that live outside your Python code, in your operating system's shell. They are an excellent way to configure your application's behavior without changing the code itself. This is especially important for sensitive data like API keys or database passwords.

You can access environment variables in Python using the `os` module.

```python
import os

# Get an environment variable. .get() is safe and returns None if not found.
api_key = os.environ.get("MY_API_KEY")

if api_key:
    print("API Key found!")
else:
    print("API Key not set. Please set the MY_API_KEY environment variable.")
```
To set an environment variable for a single command:
```bash
MY_API_KEY="12345-secret" poetry run python my_app.py
```
To set it for your entire shell session:
```bash
export MY_API_KEY="12345-secret"
```
You can add this `export` line to your shell's startup file (`~/.bashrc` or `~/.zshrc`) to make it permanent.

For project-specific variables, a common practice is to use a `.env` file and a library like `python-dotenv` to load them.

## 🔹 Quick Exercise

Practice creating a virtual environment manually with `venv`.

1.  Create a new directory `~/temp-venv-test` and `cd` into it.
2.  Create a virtual environment inside it named `my_env`.
3.  Activate the environment. Your shell prompt should change.
4.  Run `which python`. You should see that it points to the Python executable inside your `my_env` directory, not your system or pyenv Python.
5.  Install a package, e.g., `pip install cowsay`.
6.  Run `cowsay "I'm in a venv!"`.
7.  Deactivate the environment.
8.  Try to run `cowsay` again. It should fail, because it was only installed in the virtual environment.

## 📝 Daily Assignment
**Goal**: Set up a project that uses environment variables for configuration.

1.  **Create a New Project**: Create a new Poetry project called `config-app`.
2.  **Add `python-dotenv`**: Add the `python-dotenv` library to your project: `poetry add python-dotenv`.
3.  **Create a `.env` file**: In the root of your `config-app` project, create a file named `.env`. **Add `.env` to your `.gitignore` file immediately!**
4.  **Add Configuration to `.env`**:
    ```
    APP_NAME="My Awesome App"
    LOG_LEVEL="DEBUG"
    API_TOKEN="super-secret-token-123"
    ```
5.  **Write the Application**:
    -   Create a `config_app/main.py` file.
    -   In this file, import `os` and `load_dotenv` from `dotenv`.
    -   Call `load_dotenv()` at the very top of the script. This will load the variables from your `.env` file into the environment.
    -   Write a `main` function that reads these three environment variables using `os.environ.get()`.
    -   Print a welcome message that uses the `APP_NAME`.
    -   "Configure" a logger using the `LOG_LEVEL`.
    -   Print the API token to show that it was loaded (in a real app, you would *never* print a secret token).
6.  **Verify**: Run `poetry run python -m config_app.main` and see that it prints the configuration values from your `.env` file.

## ⚠️ Common Mistakes
- **Forgetting to activate the venv**: When using `venv` manually, forgetting to `source .../activate` means you are still using your global `pip`, not the isolated one.
- **Committing the `venv` directory to Git**: This directory can be large and contains machine-specific paths. It should always be in `.gitignore`.
- **Committing `.env` files**: `.env` files are designed to hold secrets and machine-specific configuration. They must **never** be committed to version control.
- **Relying on Poetry's venv location**: Poetry may change where it stores virtual environments. Don't hardcode paths to it. Use `poetry run` or `poetry shell` to let Poetry manage it for you.

## 📖 Further Reading
- [Python Docs: `venv` — Creation of virtual environments](https://docs.python.org/3/library/venv.html)
- [Real Python: Python Virtual Environments: A Primer](https://realpython.com/python-virtual-environments-a-primer/)
- [Poetry Docs: Managing Environments](https://python-poetry.org/docs/managing-environments/)
- [The Twelve-Factor App: Config](https://12factor.net/config) (A classic guide to application configuration)
