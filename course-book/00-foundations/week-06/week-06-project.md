# Week 6 Project: Robust CLI Application

## 🎯 Project Objective
To build a robust, production-quality command-line interface (CLI) application. This project synthesizes everything from this week: you will define custom exceptions, implement a comprehensive test suite using Test-Driven Development (TDD), and integrate a logging system to create an application that is reliable and easy to debug.

For this project, we will build a simple contact book application.

## 📋 Core Requirements
You will build a CLI that allows a user to add, find, delete, and list contacts, all stored in a JSON file.

### 1. Project Setup
-   In your `my-first-poetry-app` project, create a new file: `my_first_poetry_app/contact_book.py`.
-   Create a corresponding test file: `tests/test_contact_book.py`.
-   Your contact book data will be stored in a JSON file (e.g., `contacts.json`).

### 2. The `Contact` Model
-   Define a `Contact` data model. A `dataclass` is a great choice for this. It should contain `name: str`, `email: str`, and `phone: str`.

### 3. Custom Exception Hierarchy
-   In a new `exceptions.py` file, define a hierarchy for your application's errors:
    -   `class ContactBookError(Exception)`: A base class for all your app's errors.
    -   `class ContactNotFoundError(ContactBookError)`: Raised when trying to find a contact that doesn't exist.
    -   `class DuplicateContactError(ContactBookError)`: Raised when trying to add a contact that already exists (e.g., same email).
    -   `class InvalidDataError(ContactBookError)`: Raised when the data for a contact is invalid (e.g., bad email format).

### 4. The `ContactManager` Class (The Core Logic)
This class will handle all the logic for loading, saving, and managing contacts. **You should develop this class using TDD.**

-   `__init__(self, storage_file: Path)`: Takes a `pathlib.Path` object for the JSON file where contacts are stored.
-   `load_contacts(self) -> None`: Loads contacts from the JSON file. Should handle `FileNotFoundError` gracefully (by starting with an empty contact book) and `json.JSONDecodeError` (by logging an error and starting fresh).
-   `save_contacts(self) -> None`: Saves the current list of contacts to the JSON file.
-   `add_contact(self, contact: Contact) -> None`: Adds a new contact. Raises `DuplicateContactError` if a contact with the same email already exists.
-   `find_contact(self, email: str) -> Contact`: Finds a contact by email. Raises `ContactNotFoundError` if not found.
-   `delete_contact(self, email: str) -> None`: Deletes a contact by email. Raises `ContactNotFoundError` if not found.
-   `list_contacts(self) -> list[Contact]`: Returns a list of all contacts.

### 5. The CLI (User Interface)
-   This part of the code will be in a `main` function or a separate `cli.py` file.
-   It should present a simple menu to the user: `(A)dd, (F)ind, (D)elete, (L)ist, (Q)uit`.
-   It will create an instance of `ContactManager` and call the appropriate methods based on user input.
-   **Crucially**, this UI layer is responsible for catching exceptions from the `ContactManager` and printing friendly error messages to the user. For example, if `add_contact` raises a `DuplicateContactError`, the CLI should catch it and print `"Error: A contact with that email already exists."`

### 6. Logging
-   Integrate the `logging` module.
-   Configure it to log `INFO` level messages to the console and `DEBUG` level messages to a file (`contact_book.log`).
-   Log important events:
    -   `INFO`: Application start/stop, contact added/deleted.
    -   `DEBUG`: The specific data being added, which contact is being searched for.
    -   `ERROR`: Any exceptions that are caught (e.g., "Failed to load contacts from file due to corrupted data.").

## ✅ Definition of Done
- [ ] The application allows users to add, find, delete, and list contacts.
- [ ] Contact data is persisted to a JSON file.
- [ ] A custom exception hierarchy is defined and used.
- [ ] The core `ContactManager` logic is developed using TDD and has high test coverage.
- [ ] The CLI layer gracefully handles all custom exceptions from the logic layer.
- [ ] A logging system is in place, logging to both console and a file at different levels.
- [ ] The entire codebase is type-safe, documented, and passes the linter.

## 🏆 Bonus Challenges
-   **Use a Real CLI Library**: Instead of a simple `input()` loop, use a library like `typer` or `click` to build a more professional CLI with subcommands (e.g., `python contact_book.py add --name "Alice" ...`).
-   **Mocking for Tests**: In your tests for the `ContactManager`, the `load_contacts` and `save_contacts` methods interact with the filesystem. This can be slow and unreliable. Use `unittest.mock.patch` to "mock" the `open` function and test the logic without actually touching the disk.
-   **Update Functionality**: Add a method `update_contact(self, email: str, **updates)` that allows changing the name or phone number of an existing contact. Write tests for it first!

## 💡 Tips
-   **Separate Concerns**: Keep your core logic (`ContactManager`) separate from your UI (`cli`). The `ContactManager` should raise exceptions, and the `cli` should handle them. This makes your core logic reusable.
-   **TDD Workflow**: For each method in `ContactManager`, write the test first. Watch it fail. Write the code to make it pass. Then refactor.
-   **Think About User Experience**: What's the most helpful error message to show a user if they try to find a contact that doesn't exist? Design your CLI outputs to be clear and helpful.
