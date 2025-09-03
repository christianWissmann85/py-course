# Week 7, Day 5: Dependency Management

## 🎯 Learning Objectives

- [ ] Understand and use Semantic Versioning (SemVer) to specify dependency constraints.
- [ ] Read and interpret a `poetry.lock` file and understand its importance for reproducible builds.
- [ ] Use Poetry to manage and update dependencies safely.
- [ ] Understand the importance of auditing dependencies for security vulnerabilities.

## 📚 Concepts

### 1. Semantic Versioning (SemVer)

Most Python packages follow a versioning scheme called Semantic Versioning. A version number is given in the format `MAJOR.MINOR.PATCH` (e.g., `2.31.0`).

- **`MAJOR` (2)**: Incremented for incompatible API changes. Upgrading this will likely break your code.
- **`MINOR` (31)**: Incremented for adding functionality in a backward-compatible way. Upgrading this should be safe.
- **`PATCH` (0)**: Incremented for making backward-compatible bug fixes. Upgrading this is almost always safe.

### 2. Dependency Specification

In `pyproject.toml`, you specify your dependencies and their allowed version ranges. Poetry uses special symbols for this:

- **Caret (`^`)**: `^2.31.0` means "allow updates to any version that is greater than or equal to `2.31.0` but less than `3.0.0`". This is the most common and recommended specifier, as it allows safe minor and patch updates.
- **Tilde (`~`)**: `~2.31.0` is more restrictive. It means "allow updates greater than or equal to `2.31.0` but less than `2.32.0`". It only allows patch updates.
- **Greater/Less than (`>`, `<`, `>=`, `<=`)**: For explicit bounds, e.g., `requests = ">=2.0.0, <3.0.0"`.
- **Asterisk (`*`)**: `*` means "any version". This is dangerous and should be avoided.

```toml
[tool.poetry.dependencies]
python = "^3.12"
requests = "^2.31.0" # Allows 2.31.0 up to (but not including) 3.0.0
pydantic = "~2.5.0"  # Allows 2.5.0 up to (but not including) 2.6.0
```

### 3. The Lock File (`poetry.lock`)

The `pyproject.toml` file specifies a _range_ of allowed versions. This can still lead to problems if a new minor version of a dependency introduces a subtle bug.

To solve this, Poetry creates a `poetry.lock` file. This file records the **exact** version of every package and sub-package that was installed. When you run `poetry install`, Poetry will always install the exact versions from the lock file, ensuring that every developer on the team and your production server has the exact same environment.

**The `poetry.lock` file should always be committed to version control.**

### 4. Update Strategies

- `poetry install`: Installs the exact versions from `poetry.lock`. If the lock file doesn't exist, it resolves dependencies from `pyproject.toml` and creates one. This is what you run when first checking out a project.
- `poetry update`: This command tells Poetry to ignore the lock file, look for the latest versions that match your constraints in `pyproject.toml`, and then write a new, updated `poetry.lock` file. You run this when you intentionally want to upgrade your dependencies.
- `poetry update <package_name>`: Updates only a single package and its dependencies.

### 5. Security Scanning

Your application is only as secure as its weakest dependency. A vulnerability in a library you use is a vulnerability in your application. It is crucial to audit your dependencies for known security issues.

Poetry has a built-in command for this:

```bash
poetry check
# This checks pyproject.toml for errors

# For security, you can integrate a tool like 'pip-audit' or use GitHub's Dependabot.
# Poetry has a plugin for this as well.
poetry self add poetry-plugin-check
poetry check
```

GitHub's **Dependabot** is a fantastic tool that automatically scans your repository's dependencies and creates pull requests to update any that have known security vulnerabilities.

## 🔹 Quick Exercise

Practice specifying dependency versions in your `my-first-poetry-app`'s `pyproject.toml` file.

1.  Open `pyproject.toml`.
2.  Use `poetry add` to add the following packages with specific version constraints:
    - `rich` with a caret constraint: `poetry add "rich^13.0"`
    - `typer` with a tilde constraint: `poetry add "typer~0.9.0"`
    - `requests` with an exact version: `poetry add "requests==2.31.0"`
3.  Inspect your `pyproject.toml` to see how Poetry recorded these constraints.
4.  Inspect your `poetry.lock` file. Find the entries for `rich`, `typer`, and `requests` and see that it has locked their exact versions.
5.  Run `poetry show` to see a tree of all your dependencies.

## 📝 Daily Assignment

**Goal**: Perform a dependency audit and create an update plan for your project.

1.  **Examine Dependencies**: In your `my-first-poetry-app` project, run `poetry show --tree` to see a full list of your main dependencies and the sub-dependencies they rely on.
2.  **Check for Outdated Packages**: Run `poetry update --dry-run`. The `--dry-run` flag will show you what packages _would be_ updated if you ran the command, without actually changing anything.
3.  **Create an Update Plan**:
    - Create a new markdown file in your project called `DEPENDENCY_AUDIT.md`.
    - In this file, list the packages that the `--dry-run` command showed could be updated.
    - For each package, write down its current version and the new version.
    - Go to PyPI and look up the release history for one or two of the packages. Is the update a MAJOR, MINOR, or PATCH release? What new features or bugfixes does it include?
4.  **Document Dependencies**:
    - In the same `DEPENDENCY_AUDIT.md` file, create a table.
    - List each of your project's _direct_ dependencies (the ones you added).
    - For each one, write a one-sentence explanation of what it's used for in your project. This is a crucial documentation practice.
5.  **Enable Dependabot (Bonus)**: If your project is on GitHub, go to the repository settings -> Security & Analysis. Enable Dependabot alerts and security updates. This is a real-world best practice for maintaining a secure project.

## 📦 Week 7 Project: Published Package

This week's project is to take the package you created and prepare it for a real release. You will add documentation, set up a CI/CD pipeline to automate testing and publishing, and publish it to TestPyPI.

**Key Requirements**:

- A complete `pyproject.toml` with all metadata.
- A well-written `README.md` and other documentation.
- A GitHub Actions workflow that automatically runs tests on every push.
- A separate GitHub Actions workflow that automatically builds and publishes your package to TestPyPI whenever you create a new "release" on GitHub.

See `week-07-project.md` for a more detailed breakdown.

## ⚠️ Common Mistakes

- **Committing a vague `pyproject.toml`**: Not specifying version constraints (e.g., using `*`) can lead to your application breaking when a dependency releases a new major version.
- **Not committing the `poetry.lock` file**: This is the most common mistake. The lock file is the key to reproducible builds. It **must** be in version control.
- **Running `poetry update` instead of `poetry install`**: In a team or CI/CD environment, you should always run `poetry install` to respect the lock file. Only run `poetry update` when you are consciously making the decision to upgrade dependencies.
- **Ignoring security vulnerabilities**: An unpatched vulnerability in a dependency is a major security risk. Automated tools like Dependabot should be considered essential.

## 📖 Further Reading

- [Semantic Versioning 2.0.0](https://semver.org/)
- [Poetry Docs: Versioning](https://python-poetry.org/docs/dependency-specification/)
- [Poetry Docs: `lock` command](https://python-poetry.org/docs/cli/#lock)
- [GitHub: About Dependabot security updates](https://docs.github.com/en/code-security/dependabot/dependabot-security-updates/about-dependabot-security-updates)
