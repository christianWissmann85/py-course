# Week 2 Project: Type-Safe CLI Calculator

## 🎯 Project Objective

To build a fully functional, type-safe command-line calculator. This project will synthesize everything you've learned in Week 2, including variables, type hints, control flow, loops, user input, and file I/O.

## 📋 Core Requirements

You will build a calculator that runs in a loop, asking the user for two numbers and an operator, performing the calculation, printing the result, and saving the operation to a history file.

### 1. Project Setup

- In your `my-first-poetry-app` project, create a new file: `my_first_poetry_app/cli_calculator.py`.
- Set up a main loop (e.g., `while True:`) that will continuously prompt the user for calculations. Include a way for the user to quit (e.g., by typing 'quit').

### 2. User Input Handling

- Prompt the user to enter a first number, an operator (+, -, \*, /), and a second number.
- Safely handle user input. If the user enters something that is not a number, print an error message and prompt again. Use a `try...except ValueError`.

### 3. Calculation Logic

- Create a central `calculate` function that is fully type-annotated.
- It should take two numbers (`float` or `int`) and an operator (`Literal["+", "-", "*", "/"]`) as arguments.
- It should return the result of the calculation.
- Handle the division by zero case gracefully. Instead of crashing, your function should return `None` or print an error message.

### 4. History Tracking (File I/O)

- Define a `HISTORY_FILE` constant using `Final` from `pathlib import Path`.
- After each successful calculation, append the full operation and result to the `history.txt` file. For example: `10.0 + 5.0 = 15.0`.
- When the program first starts, it should ask the user if they want to view the past calculation history. If they say yes, read the `history.txt` file and print its contents to the screen. Handle the case where the file doesn't exist yet.

### 5. Type Safety & Code Quality

- Your entire application must pass `mypy --strict`.
- Your code must be formatted with Ruff and pass all linter checks (`ruff check .`).
- Use `Final`, `Literal`, `Union`, and other tools from the `typing` module to make your types as precise as possible.

## ✅ Definition of Done

- [ ] The calculator runs in a loop and can be exited cleanly.
- [ ] The program accepts two numbers and an operator from the user.
- [ ] The program correctly performs addition, subtraction, multiplication, and division.
- [ ] The program handles invalid number inputs without crashing.
- [ ] The program handles division by zero without crashing.
- [ ] Each calculation is saved to a `history.txt` file.
- [ ] The user can view the history at the start of the program.
- [ ] All code is fully type-annotated.
- [ ] `mypy --strict .` reports no issues.
- [ ] `ruff check .` reports no issues.

## 🏆 Bonus Challenges

- **Add More Operators**: Extend the calculator to support more operations like exponentiation (`**`) or modulus (`%`).
- **Clear History**: Add a command (e.g., 'clear') that allows the user to wipe the calculation history file.
- **Use `pathlib`**: Use the `pathlib` module for all file path operations.
- **Refactor with Enums**: Instead of `Literal` for operators, use an `Enum` from the `enum` module. This is a more advanced and robust way to handle a fixed set of constants.

## 💡 Tips

- Break the problem down. Start with just getting user input. Then add the calculation logic. Then add the file history.
- Think about your `main` function and how you will structure the main loop.
- Write helper functions (e.g., `get_user_number`, `get_user_operator`) to keep your main loop clean.
- Run `mypy` and `ruff` frequently! Don't wait until the end to check your code.
