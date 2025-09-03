# Week 1, Day 2: Poetry & Dependency Management

## 🎯 Learning Objectives

- [ ] Understand the core problems of dependency management in Python.
- [ ] Master essential Poetry commands for project initialization and dependency management.
- [ ] Create your first project managed by Poetry from scratch.
- [ ] Understand the structure and importance of the `pyproject.toml` file.

## 📚 Concepts

### 1. Why Poetry over pip/venv?

Traditional Python dependency management involves using `pip` to install packages and `venv` to create isolated environments. This often leads to a `requirements.txt` file that doesn't lock down sub-dependencies, causing the "it works on my machine" problem.

Poetry is a modern, all-in-one tool that solves these issues by:

- **Combining virtual environment and package management**: No more separate `venv` and `pip` commands.
- **Deterministic builds**: Poetry uses a `poetry.lock` file to ensure that the exact same versions of all dependencies (and their dependencies) are installed every time.
- **Clear dependency separation**: It distinguishes between main dependencies and development-only dependencies (like `pytest` or `mypy`).
- **Standardized configuration**: It uses the official `pyproject.toml` file, which is the new standard for configuring Python projects.

### 2. Poetry Installation and Configuration

Poetry is installed separately from your Python projects. It's a command-line tool that manages your projects for you. The official installer is recommended to avoid conflicts with other Python tools.

### 3. Creating New Projects

`poetry new <project_name>` scaffolds a new project with a sensible directory structure, a `pyproject.toml`, and a `README`. `poetry init` can be used to start using Poetry in an existing project.

### 4. Managing Dependencies

- `poetry add <package>`: Adds a dependency to your project and installs it.
- `poetry add <package> --group dev`: Adds a development-only dependency.
- `poetry remove <package>`: Removes a dependency.
- `poetry install`: Installs all dependencies listed in `poetry.lock` (or `pyproject.toml` if no lock file exists).
- `poetry update`: Updates dependencies to the latest allowed versions and regenerates the lock file.

### 5. Virtual Environments with Poetry

Poetry automatically creates and manages a virtual environment for each of your projects. You don't need to create or activate it manually. To run a script within the project's virtual environment, you use `poetry run <command>`. For example, `poetry run python my_script.py`.

## 💻 Code Examples

### Creating a New Poetry Project

```bash
# This creates a new directory called 'my-awesome-project'
poetry new my-awesome-project

cd my-awesome-project

# Let's look at the created files
ls
# Expected output:
# pyproject.toml  README.md  my_awesome_project/  tests/
```

### The `pyproject.toml` file

This file is the heart of your project's configuration.

```toml
[tool.poetry]
name = "my-awesome-project"
version = "0.1.0"
description = ""
authors = ["Your Name <you@example.com>"]
readme = "README.md"

[tool.poetry.dependencies]
python = "^3.12"

[tool.poetry.group.dev.dependencies]
# Development dependencies go here

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

### Adding and Running Dependencies

```bash
# Add the popular 'requests' library as a dependency
poetry add requests

# Add 'pytest' as a development dependency
poetry add pytest --group dev

# Now, let's run a python script that uses our new dependency
# Create a file: my_awesome_project/main.py
# with content:
# import requests
# print(requests.get("https://python-poetry.org").status_code)

# Run the script using Poetry's virtual environment
poetry run python my_awesome_project/main.py
# Expected output: 200
```

## 🔹 Quick Exercise

1.  **Create a Project**: Create a new Poetry project called `quick-poetry-demo`.
2.  **Add Dependencies**:
    - Add `httpx` as a main dependency.
    - Add `mypy` and `ruff` as development dependencies.
3.  **Inspect Files**: Look at `pyproject.toml` and `poetry.lock`. See how they changed after you added the dependencies.
4.  **Remove a Dependency**: Decide you don't want `mypy` after all and remove it.
5.  **Update Dependencies**: Run `poetry update` to see if any new versions are available.
6.  **Generate `requirements.txt`**: Although you don't need it for Poetry projects, some platforms require a `requirements.txt`. Generate one with `poetry export -f requirements.txt --output requirements.txt`. Check its contents.

## 📝 Daily Assignment

1.  **Create a New Project**: Initialize a new Poetry project named `my-first-poetry-app`.
2.  **Add Dependencies**: Add the following five or more libraries as main dependencies. Feel free to choose your own, but here are some suggestions:
    - `rich`
    - `pydantic`
    - `loguru`
    - `typer`
    - `requests`
3.  **Add Development Dependencies**: Add `pytest`, `pytest-cov`, and `black` as development dependencies.
4.  **Configure Pre-Commit Hooks (Bonus)**: This is an advanced but extremely useful step.
    - Research what pre-commit hooks are.
    - Add `pre-commit` as a development dependency (`poetry add pre-commit --group dev`).
    - Create a `.pre-commit-config.yaml` file in your project root.
    - Configure it to run `ruff` and `black` automatically before each commit.
    - Run `poetry run pre-commit install` to set it up.

## ⚠️ Common Mistakes

- **Forgetting `poetry run`**: Running `python my_script.py` directly will use your global Python, not the project's virtual environment where your dependencies are installed. This leads to `ModuleNotFoundError`. Always use `poetry run`.
- **Manually editing `poetry.lock`**: The lock file is meant to be machine-generated. Never edit it by hand. Use `poetry add/remove/update` to manage it.
- **Committing without the lock file**: The `poetry.lock` file is essential for reproducible builds. It **must** be committed to your version control (e.g., Git).
- **Confusing `poetry install` and `poetry update`**:
  - `poetry install` installs the exact versions from `poetry.lock`. Use this for CI or when a new developer clones the repo.
  - `poetry update` looks for newer versions allowed by your `pyproject.toml` constraints and updates `poetry.lock`. Use this when you intentionally want to upgrade packages.

## 📖 Further Reading

- [Poetry Official Documentation](https://python-poetry.org/docs/)
- [What is pyproject.toml?](https://snarky.ca/what-the-heck-is-pyproject-toml/)
- [Pre-commit framework](https://pre-commit.com/)
