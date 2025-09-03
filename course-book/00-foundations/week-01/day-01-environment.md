# Week 1, Day 1: Development Environment

## 🎯 Learning Objectives
- [ ] Set up an Ubuntu/WSL2 environment for Python development.
- [ ] Install and configure PyEnv for managing multiple Python versions.
- [ ] Understand the importance of Python versioning in projects.
- [ ] Configure your shell for a productive development workflow.

## 📚 Concepts

### 1. Why Ubuntu/Linux for Python Development?
The professional Python world heavily relies on Linux-based environments. Tools, deployment pipelines, and servers almost exclusively run on Linux. Learning to work in a Unix-like environment from day one is crucial for a real-world workflow. It ensures that the environment you develop in matches the environment your code will run in.

### 2. WSL2: The Best of Both Worlds
For Windows users, the Windows Subsystem for Linux (WSL2) provides a full, native Linux kernel running directly within Windows. This isn't a slow virtual machine; it's a deeply integrated environment that gives you the power of a Linux command line and toolchain combined with the convenience of Windows applications like VS Code.

### 3. PyEnv for Python Version Management
You will often work on multiple projects that require different versions of Python. `pyenv` is the industry-standard tool to solve this. It allows you to:
- Install any Python version with a single command.
- Set a global default Python version for your user account.
- Set a local, per-project Python version.
- Switch between installed versions seamlessly.
This prevents version conflicts and makes your development environment robust and predictable.

### 4. Shell Configuration (`.bashrc` / `.zshrc`)
Your shell is your primary interface as a developer. Configuring it properly can dramatically boost your productivity. Your shell's configuration file (e.g., `.bashrc` for Bash, `.zshrc` for Zsh) is where you can:
- Add `pyenv` to your system's PATH.
- Create aliases (shortcuts) for long commands.
- Customize your command prompt.
- Set environment variables.

## 💻 Code Examples

### Initializing PyEnv in your Shell
To make `pyenv` work, you need to add these lines to your shell configuration file (`~/.bashrc` or `~/.zshrc`). This ensures `pyenv` is loaded every time you open a new terminal.

```bash
# Add these lines to the end of your ~/.bashrc or ~/.zshrc

export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```
*After adding these, you must restart your shell or run `source ~/.bashrc`.*

### Installing and Switching Python Versions

```bash
# Install a specific Python version
pyenv install 3.12.3

# Set the global default Python version
pyenv global 3.12.3

# Create a project folder
mkdir my-new-project
cd my-new-project

# Set a project-specific Python version
# This creates a .python-version file in the current directory
pyenv local 3.11.8

# Verify the active Python version
# Inside the project, this will show 3.11.8
# Outside the project, it will show 3.12.3
python --version
```

## 🔹 Quick Exercise

1.  **Install Python 3.12.x**: Use `pyenv` to install the latest available version of Python 3.12. (Hint: `pyenv install --list | grep 3.12`).
2.  **Set Global Version**: Set this new version as your global default.
3.  **Create a Test Directory**: Make a new directory called `~/pyenv-test`.
4.  **Set Local Version**: Inside `~/pyenv-test`, set a different local Python version, for example, `3.10.13`.
5.  **Verify Versions**:
    - Run `python --version` and `which python` inside `~/pyenv-test`.
    - `cd ..` to go back to your home directory.
    - Run `python --version` and `which python` again.
    - Observe how the version and the path to the Python executable change.

## 📝 Daily Assignment

1.  **Document Your Setup**: Create a new Markdown file in your personal notes called `my_dev_setup.md`.
2.  **Write Down Steps**: Document every step you took to set up your Ubuntu/WSL2, PyEnv, and shell configuration. Include the commands you ran and why you ran them. This is a critical skill for onboarding new team members.
3.  **Create Shell Aliases**: Add at least three useful aliases to your `.bashrc` or `.zshrc` file. Some ideas:
    - `gs` for `git status`
    - `ga` for `git add .`
    - `gc` for `git commit -m`
    - `update` for `sudo apt update && sudo apt upgrade -y`
4.  **Verify**: Open a new terminal window and test that your aliases work correctly.

## ⚠️ Common Mistakes

-   **Forgetting to restart the shell**: After modifying `.bashrc` or `.zshrc`, the changes are not applied until you open a new shell or run `source ~/.your_config_file`. Many beginners edit the file and wonder why nothing has changed.
-   **Installing build dependencies**: `pyenv` compiles Python from source, which requires build tools. On Ubuntu, you often need to run `sudo apt-get install -y build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python3-openssl` before `pyenv install` will work.
-   **Mixing `pyenv` with system Python**: Trying to use `sudo` with a `pyenv`-managed Python. `pyenv` is for your user account; system-wide tasks should use the system's Python. Avoid `sudo pip install`.

## 📖 Further Reading

-   [Official PyEnv GitHub Repository](https://github.com/pyenv/pyenv)
-   [WSL2 Installation Guide](https://learn.microsoft.com/en-us/windows/wsl/install)
-   [DigitalOcean: How To Install Python 3 and Set Up a Local Programming Environment on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-python-3-and-set-up-a-local-programming-environment-on-ubuntu-22-04) (A good general guide, but we prefer `pyenv` over using `apt` for Python versions).
