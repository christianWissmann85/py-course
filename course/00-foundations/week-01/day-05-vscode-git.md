# Week 1, Day 5: VS Code & Git Setup

## 🎯 Learning Objectives
- [ ] Configure Visual Studio Code with essential extensions for a powerful Python development experience.
- [ ] Understand and configure workspace settings in VS Code for consistency.
- [ ] Learn the fundamentals of Git for version control.
- [ ] Set up a Git repository, connect it to GitHub, and understand collaborative workflows.

## 📚 Concepts

### 1. Essential VS Code Extensions
VS Code is a lightweight but powerful code editor. Its power comes from its vast ecosystem of extensions. For this course, these are essential:
- **Python (from Microsoft)**: The official extension. Provides IntelliSense (autocompletion), debugging, and environment selection.
- **Pylance (from Microsoft)**: A high-performance language server included with the Python extension, offering rich type information and smart suggestions.
- **Ruff (from Astral)**: Integrates the Ruff linter and formatter directly into the editor, showing you errors and formatting issues as you type.
- **MyPy Type Checker (from Matan Borenraout)**: A separate extension that runs MyPy in the background and displays type errors in your code.

### 2. Workspace Settings (`.vscode/settings.json`)
You can configure VS Code settings globally (for all projects) or locally (for the current project/workspace). Workspace settings are stored in a `.vscode/settings.json` file and should be committed to version control so that everyone on the team has the same editor configuration.

A good `settings.json` for our projects would be:
```json
{
    // --- General Editor Settings ---
    "files.trimTrailingWhitespace": true,

    // --- Python Specific Settings ---
    "python.testing.pytestEnabled": true,
    "python.testing.pytestArgs": [
        "tests"
    ],

    // --- Ruff & Formatting Settings ---
    "[python]": {
        "editor.defaultFormatter": "charliermarsh.ruff",
        "editor.formatOnSave": true,
        "editor.codeActionsOnSave": {
            "source.fixAll": "explicit",
            "source.organizeImports": "explicit"
        }
    },
    "ruff.importStrategy": "fromEnvironment",

    // --- MyPy Settings ---
    "mypy-type-checker.importStrategy": "fromEnvironment"
}
```
This configuration tells VS Code to use Ruff as the default formatter, to format your code every time you save, to enable pytest, and more.

### 3. Git Fundamentals
Git is a distributed version control system. It's the industry standard for tracking changes in source code. Key concepts:
- **Repository (repo)**: A project's folder, containing all files and their history.
- **Commit**: A snapshot of your files at a specific point in time.
- **Branch**: An independent line of development. The main branch is usually called `main` or `master`.
- **`git add <file>`**: Stages a file, marking it for inclusion in the next commit.
- **`git commit -m "Message"`**: Creates a new commit with the staged files.
- **`git push`**: Uploads your commits to a remote repository (like GitHub).
- **`git pull`**: Downloads changes from a remote repository.

### 4. GitHub Integration
GitHub is a web platform that hosts Git repositories. It provides tools for collaboration, code review (Pull Requests), issue tracking, and automation (GitHub Actions). The typical workflow is:
1. Create a repository on GitHub.
2. Clone it to your local machine.
3. Make changes locally on a new branch.
4. Push your branch to GitHub.
5. Open a Pull Request (PR) to merge your changes into the `main` branch.

### 5. `.gitignore`
This is a crucial file in any Git project. It tells Git which files or directories it should ignore and not track. This is used for:
- Virtual environment folders (Poetry creates them outside the project by default, which is good).
- Python bytecode files (`__pycache__/`, `*.pyc`).
- IDE configuration folders (`.vscode/` *can* be ignored, but `settings.json` is often useful to share).
- Secret files or environment variables (`.env`).

A good `.gitignore` for a Python project can be generated from [gitignore.io](https://www.toptal.com/developers/gitignore/api/python,vscode,linux).

## 💻 Code Examples

### Basic Git Workflow
```bash
# --- One-time setup ---
# Configure your name and email for Git
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# --- Starting a new project ---
# Navigate to your project directory (e.g., my-first-poetry-app)
cd my-first-poetry-app

# Initialize a new Git repository
git init

# Create a .gitignore file (use a template from gitignore.io)
touch .gitignore
# Add common Python patterns to it.

# Stage all files for the first commit
git add .

# Make your first commit
git commit -m "Initial commit: Setup Poetry project with basic config"

# --- Connecting to GitHub ---
# 1. Go to GitHub and create a new repository (e.g., my-first-poetry-app)
# 2. Copy the commands from the "…or push an existing repository from the command line" section. It will look like this:
git remote add origin https://github.com/your-username/my-first-poetry-app.git
git branch -M main
git push -u origin main
```

### Daily Git Workflow (Branching)
```bash
# Start on the main branch, make sure it's up to date
git checkout main
git pull origin main

# Create a new branch for your new feature
git checkout -b feature/add-user-login

# --- Do your work ---
# (Edit files, add new files)
# poetry add flask
# touch app/server.py

# --- Commit your changes ---
# Stage the changes
git add .

# Commit with a descriptive message
git commit -m "feat: Add initial Flask server setup for user login"

# Push your new branch to GitHub
git push origin feature/add-user-login

# Now go to GitHub to open a Pull Request.
```

## 🔹 Quick Exercise

1.  **Install VS Code Extensions**: Open VS Code, go to the Extensions view (Ctrl+Shift+X), and install `Python`, `Ruff`, and `MyPy Type Checker`.
2.  **Configure Format on Save**: Create a `.vscode/settings.json` file in your `my-first-poetry-app` project and add the configuration to enable "format on save" with Ruff.
3.  **Test Format on Save**: Open a Python file, make some formatting mistakes (e.g., messy indentation), and save the file. It should auto-format.
4.  **Initialize Git**: If you haven't already, run `git init` in your project.
5.  **Make First Commit**: Create a `.gitignore` file, then run `git add .` and `git commit -m "Initial commit"` to make your first commit.
6.  **Check Status**: Run `git status` and `git log` to see the state of your repository.

## 📝 Daily Assignment

1.  **Complete VS Code Setup**: Finalize your VS Code setup by creating a `.vscode/settings.json` file with all the recommended settings from the "Concepts" section.
2.  **Create GitHub Repository**: Go to GitHub.com, create a new public repository for your `my-first-poetry-app` project.
3.  **Push Your Code**: Follow the steps from the "Code Examples" to connect your local Git repository to the remote GitHub repository and push your `main` branch.
4.  **Branch Protection (Bonus)**: On your GitHub repository's settings page, go to "Branches" and add a branch protection rule for `main`. Require a pull request before merging. This is standard practice in professional teams.
5.  **Basic CI/CD (Bonus)**: Create a file `.github/workflows/ci.yml`. In this file, write a basic GitHub Actions workflow that:
    - Triggers on push to `main` and on pull requests.
    - Checks out the code.
    - Sets up Python and Poetry.
    - Runs `poetry install`.
    - Runs `poetry run ruff check .`.
    - Runs `poetry run mypy .`.
    Commit and push this file. Watch it run in the "Actions" tab on GitHub.

## 📦 Week 1 Project: Development Environment Documentation

This week, you have set up a complete, professional Python development environment. Your project is to document it.

1.  **Create a Guide**: In your `my-first-poetry-app` repository, create a new file called `DEVELOPMENT_GUIDE.md`.
2.  **Write Comprehensive Docs**: This guide should be so clear that a new developer could use it to set up their machine and start contributing to your project. Include sections on:
    - **Prerequisites**: What software is needed before starting (e.g., Git, VS Code).
    - **WSL/Ubuntu Setup**: Basic steps for setting up the OS.
    - **PyEnv & Python**: How to install `pyenv` and the correct Python version.
    - **Poetry**: How to install Poetry and project dependencies.
    - **VS Code**: Required extensions and a copy of the `settings.json`.
    - **Git Workflow**: A summary of your branching strategy (e.g., `feature/branch-name`).
3.  **Troubleshooting Section**: Add a section for common problems and their solutions (e.g., "What to do if `pyenv install` fails?").
4.  **Automation Script (Bonus)**: Create a shell script (`setup.sh`) that automates as much of the installation process as possible.

This project combines your learning from the entire week and produces a valuable artifact that is common in professional software projects.

## ⚠️ Common Mistakes

-   **Committing secrets**: Never commit files containing passwords, API keys, or other secrets. Use environment variables or a secrets management tool, and add the file paths (e.g., `.env`) to your `.gitignore`.
-   **Making large, unrelated commits**: A commit should be a small, logical unit of change. Don't mix a new feature, a bug fix, and a documentation update into one commit.
-   **Working directly on the `main` branch**: This is a major anti-pattern. Always create a new branch for your work. This keeps the `main` branch stable and allows for code review via Pull Requests.
-   **Not writing descriptive commit messages**: A message like "fix bug" is useless. A good message is in the imperative mood, e.g., "Fix: Prevent crash when user has no profile image."

## 📖 Further Reading

-   [Pro Git Book](https://git-scm.com/book/en/v2) (The definitive guide to Git)
-   [GitHub's "Hello World" Guide](https://docs.github.com/en/get-started/quickstart/hello-world)
-   [VS Code Docs for Python](https://code.visualstudio.com/docs/python/python-tutorial)
-   [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) (A popular specification for commit messages)
