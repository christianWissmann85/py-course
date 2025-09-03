# Week 1 Project: Development Environment Documentation

## 🎯 Project Objective
The goal of this project is to solidify your understanding of the tools you've set up this week by creating a comprehensive, shareable guide for setting up a professional Python development environment from scratch. This is a common and incredibly valuable task for any development team.

## 📋 Requirements
Your final submission should be a single Markdown file named `DEVELOPMENT_GUIDE.md` inside a new GitHub repository. This guide should be clear enough for a new developer, unfamiliar with your setup, to get up and running.

### Core Tasks
1.  **Create a New GitHub Repository**: Create a new, public GitHub repository named `python-dev-environment-guide`.
2.  **Create the Guide**: Inside this repository, create the `DEVELOPMENT_GUIDE.md` file.
3.  **Write Comprehensive Documentation**: The guide must include the following sections:
    *   **Introduction**: Briefly explain the purpose of the guide and the "stack" it describes (e.g., Ubuntu/WSL2, pyenv, Poetry, VS Code).
    *   **Prerequisites**: A checklist of software the user should have before they start (e.g., a computer, internet access, administrative rights on their machine).
    *   **Part 1: WSL2/Ubuntu Setup**: Step-by-step instructions for installing and launching WSL2 and Ubuntu.
    *   **Part 2: Core Tools Installation**: Detailed steps for installing Git, PyEnv, and Poetry. Include the necessary build dependencies for `pyenv`.
    *   **Part 3: Python and Project Setup**: Instructions on how to use `pyenv` to install a specific Python version and how to use `poetry` to initialize a new project and add dependencies.
    *   **Part 4: VS Code Configuration**: How to install the recommended extensions (`Python`, `Ruff`, `MyPy Type Checker`) and how to set up the workspace `settings.json` for formatting and linting.
    *   **Part 5: Git & GitHub Workflow**: A summary of how to initialize a Git repository, connect it to GitHub, and the recommended branching strategy (e.g., `feature/branch-name`).
    *   **Troubleshooting**: A section for at least 3 common problems a user might encounter and their solutions. (e.g., "What to do if `pyenv install` fails?", "Why isn't `poetry` command found?").

### Bonus Tasks
For an extra challenge, you can add these to your project:

1.  **Automation Script**: Create a `setup.sh` shell script that automates as much of the installation process as possible. The script should be well-commented so users understand what it's doing.
2.  **Share with the Community**: Post a link to your finished guide on a social platform (like LinkedIn or Twitter/X), explaining what you built and what you learned. This is great practice for building a professional online presence.

## ✅ Definition of Done
- [ ] A public GitHub repository named `python-dev-environment-guide` exists.
- [ ] The repository contains a `DEVELOPMENT_GUIDE.md` file.
- [ ] The guide covers all the required sections listed under "Core Tasks".
- [ ] The instructions are clear, well-formatted, and easy to follow.
- [ ] All code blocks (for commands or configuration) are accurate and tested.
- [ ] (Bonus) The repository contains a functional `setup.sh` script.
- [ ] (Bonus) You have shared your work with others.

## 💡 Tips
-   Pretend you are writing this for your future self, six months from now, who has forgotten everything.
-   Ask a friend (even a non-technical one) to read parts of your guide. If they can roughly understand the steps, you're on the right track.
-   Use Markdown formatting (headings, lists, code blocks) to make the guide readable.

This project synthesizes everything you've learned in Week 1. Documentation is a critical, often-overlooked engineering skill. Mastering it will set you apart. Good luck!
