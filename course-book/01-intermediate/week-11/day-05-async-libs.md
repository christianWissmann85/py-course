# Week 11, Day 5: Async Libraries & HTTP

## 🎯 Learning Objectives

- [ ] Use the `httpx` library to make asynchronous HTTP requests.
- [ ] Understand how to interact with the filesystem asynchronously using `aiofiles`.
- [ ] Get an overview of libraries for working with databases and WebSockets asynchronously.
- [ ] Consolidate best practices for real-world asynchronous applications.

## 📚 Concepts

### 1. `httpx` for Asynchronous HTTP

The popular `requests` library is synchronous. For async applications, the modern choice is `httpx`. It offers a very similar API to `requests`, but with `async` and `await` support.

To perform multiple requests concurrently, you should use an `AsyncClient` as an async context manager. This manages the connection pool for you efficiently.

```python
import httpx
import asyncio

async def main():
    urls = [
        "https://www.google.com",
        "https://www.bing.com",
        "https://www.duckduckgo.com"
    ]
    async with httpx.AsyncClient() as client:
        tasks = [client.get(url) for url in urls]
        responses = await asyncio.gather(*tasks)

        for response in responses:
            print(f"URL: {response.url}, Status: {response.status_code}")
```

### 2. `aiofiles` for Asynchronous File I/O

Python's built-in `open()` function is blocking. If you read a large file, it will block the entire event loop. The `aiofiles` library provides an asynchronous version of the file API that you can use in your coroutines.

```python
# poetry add aiofiles
import aiofiles
import asyncio

async def main():
    # async with and aiofiles.open()
    async with aiofiles.open('my_async_file.txt', mode='w') as f:
        # await the write operation
        await f.write("This was written asynchronously!")

    async with aiofiles.open('my_async_file.txt', mode='r') as f:
        content = await f.read()
        print(content)
```

### 3. Asynchronous Databases

Just like files and network requests, database operations are I/O-bound and can block the event loop. Most major database libraries now have asynchronous counterparts:

- **PostgreSQL**: `asyncpg` is a very fast, low-level library. `psycopg` (the most popular Postgres driver) also has async support.
- **SQLite**: `aiosqlite` provides an async wrapper around the built-in `sqlite3` module.
- **MySQL**: `aiomysql` provides an async driver.
- **ORMs**: SQLAlchemy 2.0 has a fully integrated `asyncio` interface. Tortoise ORM and Piccolo are other popular async-native ORMs.

### 4. WebSockets

WebSockets provide a persistent, two-way communication channel between a client and a server. This is perfect for real-time applications like chat apps, live notifications, or multiplayer games.

The `websockets` library is the standard choice for this in Python.

```python
# poetry add websockets
import asyncio
import websockets

# A simple echo server
async def echo(websocket):
    async for message in websocket:
        await websocket.send(f"Echo: {message}")

async def main():
    async with websockets.serve(echo, "localhost", 8765):
        await asyncio.Future() # run forever

# asyncio.run(main())
```

### 5. Best Practices for Real-World Async

- **Choose the right libraries**: Use libraries designed for `asyncio`. A single blocking call can ruin the performance of an async app.
- **Structure with a main coroutine**: Have a single top-level `main` async function that is started with `asyncio.run()`. This function orchestrates all other tasks.
- **Separate I/O and CPU work**: Keep your I/O-bound async code separate from CPU-bound logic. If you need to run a heavy CPU task, use `loop.run_in_executor()` to run it in a separate thread or process pool without blocking the event loop.
- **Use context managers**: Use `async with` for any resource that needs to be set up and torn down (clients, database connections, etc.).

## 🔹 Quick Exercise

Complete the `fetch_json` function below. It should use `httpx` to fetch data from a public JSON API and return it as a dictionary.

```python
import httpx
import asyncio

async def fetch_json(url: str) -> dict:
    """
    Fetches JSON data from a URL asynchronously.
    """
    async with httpx.AsyncClient() as client:
        response = await client.get(url)
        response.raise_for_status() # Raise an exception for bad status codes
        return response.json()

async def main():
    # A public test API
    url = "https://jsonplaceholder.typicode.com/todos/1"
    data = await fetch_json(url)
    assert data["userId"] == 1
    assert data["title"] == "delectus aut autem"
    print("Successfully fetched and parsed JSON.")

# --- Run the main coroutine ---
asyncio.run(main())
```

## 📝 Daily Assignment

**Goal**: Build a simple, asynchronous API client for a public API.

1.  **Choose an API**: Find a simple, free, public API that doesn't require authentication. Good options:
    - [The PokeAPI](https://pokeapi.co/) (for Pokémon data)
    - [JSONPlaceholder](https://jsonplaceholder.typicode.com/) (for fake user/post data)
    - [The Dog API](https://dog.ceo/dog-api/) (for random dog pictures)
2.  **Create Project File**: Create `my_first_poetry_app/api_client.py`. Add `httpx`.
3.  **Create an `APIClient` class**:
    - The `__init__` should take a `base_url` for the API.
    - Implement at least three `async` methods that correspond to different API endpoints. For example, for PokeAPI:
      - `async def get_pokemon(self, name: str) -> dict | None:`
      - `async def get_ability(self, id: int) -> dict | None:`
      - `async def get_pokemon_concurrently(self, names: list[str]) -> list[dict | None]:`
4.  **Implement the methods**:
    - Each method should construct the correct URL from the `base_url` and the arguments.
    - Use an `httpx.AsyncClient` to make the requests.
    - Handle potential errors (e.g., a 404 Not Found) gracefully, perhaps by returning `None`.
    - The `get_pokemon_concurrently` method **must** use `asyncio.gather` to fetch all Pokémon at the same time.
5.  **`main()` coroutine**:
    - Instantiate your `APIClient`.
    - Call your methods to demonstrate they work. Fetch a single Pokémon, and then fetch a list of 10 Pokémon concurrently.
6.  **Verify**: Run the script, type checker, and linter.

## 📦 Week 11 Project: Async Web Scraper

This week's project is to build a high-performance web scraper that can fetch and parse many pages concurrently. It will combine everything from this week: creating and gathering tasks, rate limiting with semaphores, and using libraries like `httpx` and `BeautifulSoup`.

See `week-11-project.md` for a more detailed breakdown.

## ⚠️ Common Mistakes

- **Not using a client "session"**: Creating a new `httpx.AsyncClient()` for every single request is inefficient. It's much better to create one client and reuse it for all requests to the same host, as it can reuse connections. The `async with` block makes this easy.
- **Forgetting to install async versions of libraries**: If you have `psycopg2` installed, you need to install `psycopg[binary,pool]` to get async support. Always check the library's documentation for its `asyncio` interface.
- **Not handling errors in `gather`**: If one task in a `gather` call fails, it can bring down the whole group. Use `return_exceptions=True` if you want to allow other tasks to complete and then inspect the errors later.

## 📖 Further Reading

- [`aiofiles` on PyPI](https://pypi.org/project/aiofiles/)
- [`aiosqlite` on PyPI](https://pypi.org/project/aiosqlite/)
- [`asyncpg` on PyPI](https://pypi.org/project/asyncpg/)
- [`websockets` Documentation](https://websockets.readthedocs.io/en/stable/)
