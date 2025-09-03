# Week 13, Day 1: HTTP Clients

## 🎯 Learning Objectives

- [ ] Understand the fundamentals of the HTTP protocol (requests, responses, methods, status codes).
- [ ] Learn to make HTTP requests using the `requests` library for synchronous applications.
- [ ] Learn to use the `httpx` library for both synchronous and asynchronous HTTP requests.
- [ ] Implement common authentication patterns (e.g., headers, query parameters).
- [ ] Handle HTTP errors and network issues gracefully.

## 📚 Concepts

### 1. HTTP Basics

The **Hypertext Transfer Protocol (HTTP)** is the foundation of data communication for the World Wide Web. It's a client-server protocol: the client (e.g., your browser or Python script) sends an **HTTP request**, and the server sends back an **HTTP response**.

- **Request**: Consists of a method (verb), a URL (path), headers, and an optional body.
  - **Methods**: `GET` (retrieve data), `POST` (submit data), `PUT` (update data), `DELETE` (remove data), etc.
- **Response**: Consists of a status code, headers, and an optional body.
  - **Status Codes**:
    - `2xx` (e.g., `200 OK`): Success
    - `3xx` (e.g., `301 Moved Permanently`): Redirection
    - `4xx` (e.g., `404 Not Found`, `401 Unauthorized`): Client Error
    - `5xx` (e.g., `500 Internal Server Error`): Server Error

### 2. The `requests` Library (Synchronous)

The `requests` library is the de facto standard for making synchronous HTTP requests in Python. It provides a simple, elegant API.

First, you need to install it: `poetry add requests`

```python
import requests
from requests.exceptions import RequestException

try:
    # Make a GET request to an API
    response = requests.get("https://api.github.com/users/python")

    # Raise an HTTPError if the HTTP request returned an unsuccessful status code
    response.raise_for_status()

    # The response body is often JSON
    data = response.json()
    print(f"Python has {data.get('public_repos')} public repositories.")

    # You can also access headers
    print(f"Content-Type: {response.headers.get('Content-Type')}")

except RequestException as e:
    print(f"An error occurred: {e}")
```

### 3. The `httpx` Library (Sync and Async)

`httpx` is a modern HTTP client that provides an API very similar to `requests` but with support for both synchronous and asynchronous requests. This makes it a great choice for modern Python applications.

First, install it: `poetry add httpx`

**Synchronous Example (like `requests`):**

```python
import httpx

try:
    with httpx.Client() as client:
        response = client.get("https://api.github.com/users/python")
        response.raise_for_status()
        data = response.json()
        print(f"Python has {data.get('public_repos')} public repositories.")
except httpx.HTTPStatusError as e:
    print(f"HTTP error occurred: {e.response.status_code} - {e.response.text}")
except httpx.RequestError as e:
    print(f"An error occurred while requesting {e.request.url!r}.")
```

**Asynchronous Example (requires an `async` context):**

```python
import httpx
import asyncio

async def fetch_github_user(username: str):
    try:
        async with httpx.AsyncClient() as client:
            response = await client.get(f"https://api.github.com/users/{username}")
            response.raise_for_status()
            data = response.json()
            print(f"{username} has {data.get('public_repos')} public repositories.")
    except httpx.HTTPStatusError as e:
        print(f"HTTP error occurred: {e.response.status_code}")
    except httpx.RequestError as e:
        print(f"An error occurred while requesting {e.request.url!r}.")

# To run this, you'd use:
# asyncio.run(fetch_github_user("python"))
```

### 4. Authentication

Most APIs require authentication. This is typically done by sending a token or API key in the request headers.

```python
import httpx

# Example: Authenticating with a Bearer token
api_key = "YOUR_SUPER_SECRET_API_KEY"
headers = {"Authorization": f"Bearer {api_key}"}

with httpx.Client(headers=headers) as client:
    response = client.get("https://api.some-service.com/v1/me")
    # ...
```

Some APIs might expect the key in a custom header (e.g., `X-API-Key`) or as a query parameter (`?api_key=...`). Always check the API documentation.

### 5. Error Handling

Network requests can fail for many reasons. Robust code anticipates these failures.

- **Connection Errors**: The server is down or the domain doesn't exist. `httpx` raises a `RequestError` subclass (e.g., `ConnectError`).
- **HTTP Errors**: The server responded, but with an error status code (like 404 or 500). `response.raise_for_status()` is the easiest way to check for this. It raises an `HTTPStatusError`.
- **Timeouts**: The server is taking too long to respond. You can set a timeout: `httpx.get(url, timeout=5.0)`. This will raise a `TimeoutException`.

## 🔹 Quick Exercise

Complete the `APIClient` class below. It should initialize with a `base_url` and an `api_key`, and the `get` method should make an authenticated GET request to a specific endpoint.

```python
import httpx
from typing import Any, Optional

class APIClient:
    def __init__(self, base_url: str, api_key: str):
        self.base_url = base_url
        # Hint: Create headers for authentication
        self.headers = {"Authorization": f"Bearer {api_key}"}

    def get(self, endpoint: str) -> Optional[dict[str, Any]]:
        """Make an authenticated GET request."""
        full_url = f"{self.base_url}{endpoint}"
        try:
            with httpx.Client(headers=self.headers) as client:
                response = client.get(full_url)
                response.raise_for_status()
                return response.json()
        except httpx.RequestError as e:
            print(f"Request failed: {e}")
            return None
        except httpx.HTTPStatusError as e:
            print(f"HTTP Error: {e.response.status_code}")
            return None

# --- Test ---
# client = APIClient("https://api.example.com", "my-key")
# data = client.get("/v1/data")
```

## 📝 Daily Assignment

**Goal**: Build a flexible API client for a public API of your choice.

1.  **Choose an API**: Find a free public API that interests you. A good list can be found at [public-apis/public-apis](https://github.com/public-apis/public-apis). Some good choices are:
    - [The Dog API](https://thedogapi.com/)
    - [OpenWeatherMap](https://openweathermap.org/api) (requires a free key)
    - [REST Countries](https://restcountries.com/)
2.  **Create the Client**: In a new file `my_first_poetry_app/api_client.py`, create a class `ApiClient`.
3.  **Implement Methods**:
    - The `__init__` should handle the base URL and any necessary authentication (even if it's just a placeholder).
    - Implement methods for the main actions of the API. For example, for a countries API, you might have `get_all_countries()`, `get_country_by_name(name: str)`, etc.
    - Each method should handle making the request, checking for errors, and returning the parsed JSON data.
4.  **Create a Main Script**: Write a small `main.py` that imports your `ApiClient`, instantiates it, and uses it to fetch and print some data.
5.  **Error Handling**: Make sure your client is robust. What happens if you request a country that doesn't exist? What if the API is down? Your script should handle these cases gracefully without crashing.
6.  **Verify**: Run the linter and type checker on your code.

## ⚠️ Common Mistakes

- **Forgetting `raise_for_status()`**: Not checking the response status code is a common bug. A 404 or 500 error might still have a response body (e.g., an error message), and calling `.json()` on it might work, hiding the fact that the request actually failed.
- **Hardcoding URLs and Keys**: Don't put API keys or full URLs directly in your code. Use a configuration file or environment variables for keys, and construct URLs from a base URL and endpoints.
- **Ignoring Timeouts**: A request could hang forever if the server is unresponsive. Always set a reasonable timeout for production code.
- **Not Using a Session/Client Object**: For making multiple requests to the same host, using a `requests.Session()` or `httpx.Client()` object is much more efficient. It reuses the underlying TCP connection, which can significantly speed up your application.

## 📖 Further Reading

- [MDN: An overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)
- [Python `requests` library documentation](https://requests.readthedocs.io/en/latest/)
- [Python `httpx` library documentation](https://www.python-httpx.org/)
