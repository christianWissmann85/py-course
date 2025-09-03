# Week 7, Day 3: Publishing Packages

## 🎯 Learning Objectives

- [ ] Understand the roles of PyPI and TestPyPI in the Python ecosystem.
- [ ] Configure your package's metadata in `pyproject.toml`.
- [ ] Build your package into standard distribution formats (sdist and wheel).
- [ ] Upload your package to TestPyPI for practice.

## 📚 Concepts

### 1. PyPI and TestPyPI

- **PyPI (The Python Package Index)**: The official third-party software repository for Python. When you run `pip install some-package`, it downloads the package from PyPI. It's the central hub for sharing Python code with the world.

- **TestPyPI**: A separate instance of the package index that allows you to try out the distribution and registration process without affecting the real index. It's a sandbox for packaging. You should **always** upload to TestPyPI first to make sure everything works before publishing to the real PyPI.

### 2. Package Metadata in `pyproject.toml`

The `pyproject.toml` file is the heart of your package's configuration. We've used it for dependencies, but it also holds all the metadata needed for publishing.

```toml
[tool.poetry]
name = "my-awesome-app"
version = "0.1.0"
description = "A short, one-line description of your package."
authors = ["Your Name <you@example.com>"]
license = "MIT" # Good practice to include a license
readme = "README.md" # A file with a longer description
homepage = "https://github.com/your-username/my-awesome-app"
repository = "https://github.com/your-username/my-awesome-app"
keywords = ["utility", "cli", "example"]

# This section defines what is included in your package
[tool.poetry.packages]
include = "my_app" # Include the 'my_app' directory

[tool.poetry.dependencies]
python = "^3.12"
# ... your dependencies ...
```

This metadata is what users will see on the PyPI website.

### 3. Building Distributions

Before you can upload your package, you need to build it into a standard format. There are two main types:

- **sdist (Source Distribution)**: A tarball (`.tar.gz`) containing your source code, `pyproject.toml`, and other files needed to build the package on the user's machine.
- **bdist_wheel (Built Distribution or "Wheel")**: A `.whl` file. This is a pre-built version of your package. When a user installs a wheel, `pip` can just unzip it into the right place, which is much faster than running a build process. You should always provide wheels.

With Poetry, building is simple:

```bash
poetry build
```

This command creates both an `sdist` and a `wheel` in a new `dist/` directory.

### 4. Version Management

Package versions are crucial. The standard is **Semantic Versioning (SemVer)**: `MAJOR.MINOR.PATCH` (e.g., `1.2.5`).

- **`MAJOR`**: Increment for incompatible API changes.
- **`MINOR`**: Increment for adding functionality in a backward-compatible manner.
- **`PATCH`**: Increment for backward-compatible bug fixes.

Poetry helps you manage this with the `poetry version` command:

```bash
poetry version patch # 0.1.0 -> 0.1.1
poetry version minor # 0.1.1 -> 0.2.0
poetry version major # 0.2.0 -> 1.0.0
```

You must increment the version number before you can upload a new version to PyPI.

### 5. Uploading to PyPI/TestPyPI

1.  **Create accounts**: You'll need an account on [TestPyPI](https://test.pypi.org/) and the real [PyPI](https://pypi.org/).
2.  **Get an API Token**: From your account settings, create an API token. This is safer than using your username and password.
3.  **Configure Poetry**: Tell Poetry about your token.
    ```bash
    # For TestPyPI
    poetry config pypi-token.test-pypi pypi-AgEI...
    # For the real PyPI
    poetry config pypi-token.pypi pypi-AgEI...
    ```
4.  **Publish**:

    ```bash
    # First, build your package
    poetry build

    # Publish to TestPyPI
    poetry publish --repository testpypi

    # If that works, publish to the real PyPI
    poetry publish
    ```

## 🔹 Quick Exercise

In your `my-first-poetry-app` project, configure the `pyproject.toml` file with the necessary metadata for publishing.

1.  Open `pyproject.toml`.
2.  Under the `[tool.poetry]` section, add or complete the following keys:
    - `description`
    - `license` (use "MIT")
    - `readme` (use "README.md", and make sure you have one!)
    - `homepage` (use a dummy GitHub URL for now)
    - `repository`
    - `keywords` (a list of at least 3 relevant keywords)
3.  Run `poetry check`. This command validates your `pyproject.toml` file. It should report no errors.
4.  Run `poetry build`. Inspect the `dist/` directory to see the `.whl` and `.tar.gz` files it created.

## 📝 Daily Assignment

**Goal**: Prepare your utility package from yesterday and publish it to TestPyPI.

1.  **Prepare Your Package**:
    - In your `my-first-poetry-app` project, ensure your `pyproject.toml` is complete with all the metadata from the Quick Exercise.
    - Make sure you have a `README.md` file. Write a brief description of your package in it.
    - Ensure your `my_app/__init__.py` correctly defines the public API with `__all__`.
2.  **Create a TestPyPI Account**:
    - Go to `https://test.pypi.org/` and register an account.
3.  **Get an API Token**:
    - In your TestPyPI account settings, go to "API tokens" and create a new token. Copy it somewhere safe.
4.  **Configure Poetry**:
    - Run `poetry config pypi-token.test-pypi <your-token>` to configure Poetry with your token.
5.  **Build and Publish**:
    - Run `poetry version patch` to bump your package version.
    - Run `poetry build`.
    - Run `poetry publish --repository testpypi`.
6.  **Verify**:
    - Go to your package's new page on TestPyPI (e.g., `https://test.pypi.org/project/your-package-name/`).
    - Try installing it in a _separate_ temporary directory!
      ```bash
      mkdir /tmp/test-install
      cd /tmp/test-install
      python -m venv venv
      source venv/bin/activate
      pip install --index-url https://test.pypi.org/simple/ your-package-name
      # Now try to import it in a Python shell
      python -c "from my_app import some_function; print(some_function())"
      ```

## ⚠️ Common Mistakes

- **Name squatting**: Package names on PyPI are unique. If a name is taken, you can't use it. This is why TestPyPI is important—you can experiment without cluttering the real index or finding your desired name is already in use.
- **Forgetting to build before publishing**: If you edit your code, you must run `poetry build` again to create new distribution files with your changes.
- **Forgetting to increment the version**: You cannot upload two files with the same name and version. You must bump the version in `pyproject.toml` (with `poetry version`) before publishing an update.
- **Committing secrets**: Never commit API tokens or passwords to Git. `poetry config` stores them safely outside your project directory.

## 📖 Further Reading

- [Python Packaging User Guide](https://packaging.python.org/en/latest/) (The definitive guide)
- [PyPI - The Python Package Index](https://pypi.org/)
- [TestPyPI](https://test.pypi.org/)
- [Poetry Docs: Publishing](https://python-poetry.org/docs/basic-usage/#publishing-to-pypi)
