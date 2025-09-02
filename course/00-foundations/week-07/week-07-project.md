# Week 7 Project: Published Package

## 🎯 Project Objective
To take a Python project from a local collection of scripts to a fully-fledged, documented, tested, and published package on TestPyPI. This project combines everything we've learned about modules, packages, testing, and dependency management into a single, real-world workflow.

You will prepare the `contact_book` application from Week 6 for release.

## 📋 Core Requirements
You will add documentation and a CI/CD pipeline to your `contact_book` project and publish it.

### 1. Project Preparation
-   Locate your `contact_book` project from last week. If you didn't complete it, start with a simplified version containing the `ContactManager` and `Contact` classes.
-   Ensure the project uses Poetry and has a clear package structure (e.g., all source code inside a `contact_book/` directory).
-   Make sure your `pyproject.toml` is complete with all the necessary metadata (`name`, `version`, `description`, `authors`, `readme`, `license`, etc.).

### 2. Documentation
-   **`README.md`**: Write a high-quality `README.md` file. It should include:
    -   A short description of the project.
    -   Installation instructions (how to install from TestPyPI).
    -   Usage examples (how to run the CLI).
-   **Docstrings**: Ensure every class and function has a clear, well-formatted docstring.

### 3. CI Pipeline (Testing)
-   Create a GitHub Actions workflow file at `.github/workflows/tests.yml`.
-   This workflow should trigger on every `push` to the `main` branch and on every `pull_request`.
-   The workflow should have a job that:
    1.  Checks out the code.
    2.  Sets up Python.
    3.  Installs Poetry and the project dependencies (`poetry install`).
    4.  Runs your linter (`poetry run ruff check .`).
    5.  Runs your type checker (`poetry run mypy .`).
    6.  Runs your test suite with coverage (`poetry run pytest --cov`).

### 4. CD Pipeline (Publishing)
-   Create a *second* GitHub Actions workflow file at `.github/workflows/publish.yml`.
-   This workflow should only trigger when you create a **new release** on GitHub.
-   The workflow should have a job that:
    1.  Checks out the code.
    2.  Sets up Python and Poetry.
    3.  Configures Poetry with an API token for **TestPyPI**. This token should be stored as a **secret** in your GitHub repository settings, not in the file itself.
    4.  Runs `poetry build`.
    5.  Runs `poetry publish --repository testpypi`.

### 5. Publish to TestPyPI
-   Once your workflows are set up and your tests are passing, create a new release on your project's GitHub page (e.g., `v0.1.0`).
-   This should trigger your `publish.yml` workflow.
-   Go to the "Actions" tab on GitHub and confirm that the workflow ran successfully.
-   Go to TestPyPI and find your newly published package.

## ✅ Definition of Done
- [ ] The project has a complete `pyproject.toml` and a high-quality `README.md`.
- [ ] A `tests.yml` workflow is in place and successfully runs tests on every push.
- [ ] A `publish.yml` workflow is in place.
- [ ] The package is successfully published to TestPyPI by creating a GitHub release.
- [ ] You can successfully `pip install` your package from TestPyPI in a new virtual environment.

## 🏆 Bonus Challenges
-   **Publish to PyPI**: Once you are confident everything works on TestPyPI, create a real PyPI account and publish a version there. Be careful—versions published to the real PyPI cannot be easily removed.
-   **Code Coverage Badge**: Use a service like `codecov.io` and add a step to your `tests.yml` to upload the coverage report. Add a coverage badge to your `README.md`.
-   **Automate Version Bumping**: Investigate tools that can automatically bump the version number in `pyproject.toml` and create a git tag as part of your release process.

## 💡 Tips
-   **GitHub Secrets**: To store your TestPyPI token, go to your repository on GitHub -> Settings -> Secrets and variables -> Actions. Add a new repository secret (e.g., `TEST_PYPI_TOKEN`). In your workflow file, you can then access it with `${{ secrets.TEST_PYPI_TOKEN }}`.
-   **Workflow Triggers**: The `on:` key in your workflow files is very important. Use `on: [push, pull_request]` for testing and `on: release: types: [created]` for publishing.
-   **Start Simple**: Get your tests running in CI first. Then, tackle the publishing workflow.
-   **Test Your Published Package**: The final proof is always to create a fresh virtual environment and try to install and use your package just like a real user would.
