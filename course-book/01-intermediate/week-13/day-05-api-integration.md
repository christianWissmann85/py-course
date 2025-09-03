# Week 13, Day 5: API Integration Strategies

## 🎯 Learning Objectives

- [ ] Understand common challenges when integrating third-party APIs.
- [ ] Implement strategies for handling API rate limits.
- [ ] Use caching to improve performance and reduce API usage.
- [ ] Implement a retry mechanism with exponential backoff for handling transient errors.
- [ ] Learn how to mock APIs for reliable and fast testing.

## 📚 Concepts

### 1. The Reality of API Integration

Integrating with an external API involves more than just making HTTP requests. Production-quality integrations must be resilient to a wide range of issues: the API might be slow, it might be down, it might have usage quotas (rate limits), or it might return unexpected data. Building a robust client means anticipating and handling these issues gracefully.

### 2. Handling Rate Limits

Most public APIs enforce **rate limits** to prevent abuse and ensure fair usage. A rate limit is a cap on how many requests you can make in a given time period (e.g., 60 requests per minute).

If you exceed the limit, the API will typically respond with a `429 Too Many Requests` status code. The response headers often contain information about the limit and when you can try again (e.g., `Retry-After: 60`).

**Strategy: Simple Rate Limiting with a Token Bucket**
A common algorithm is the "token bucket." Imagine a bucket that can hold a certain number of tokens. The bucket is refilled with new tokens at a fixed rate. To make a request, you must take a token from the bucket. If the bucket is empty, you must wait.

```python
import time
from collections import deque

class TokenBucketRateLimiter:
    def __init__(self, capacity: int, refill_rate: float):
        """
        Args:
            capacity: The maximum number of tokens the bucket can hold.
            refill_rate: The number of tokens added to the bucket per second.
        """
        self.capacity = capacity
        self.refill_rate = refill_rate
        self._tokens = float(capacity)
        self._last_refill_time = time.monotonic()

    def _refill(self):
        now = time.monotonic()
        time_passed = now - self._last_refill_time
        new_tokens = time_passed * self.refill_rate
        self._tokens = min(self.capacity, self._tokens + new_tokens)
        self._last_refill_time = now

    def consume(self, tokens: int = 1) -> bool:
        """Consume tokens from the bucket. Returns True if successful."""
        self._refill()
        if self._tokens >= tokens:
            self._tokens -= tokens
            return True
        return False

# Usage
limiter = TokenBucketRateLimiter(capacity=10, refill_rate=1) # 1 request/sec, bursts up to 10

while True:
    if limiter.consume():
        print("Making an API call...")
        # make_api_call()
    else:
        print("Rate limit hit. Waiting...")
        time.sleep(1) # Wait before trying again
```

### 3. Caching API Responses

Caching is one of the most effective ways to improve performance and reduce your API call volume. If you request the same data frequently, store the result locally for a period of time instead of hitting the API every time.

**Strategy: In-Memory Cache with TTL (Time-To-Live)**
The `functools.lru_cache` is great for simple in-memory caching, but it doesn't have built-in time-based expiration. For that, we can build a simple decorator.

```python
import time
from functools import wraps
from typing import Callable, Any

def timed_cache(seconds: int):
    """A decorator to cache function results with a TTL."""
    cache = {}

    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args, **kwargs):
            key = (args, frozenset(kwargs.items()))

            if key in cache:
                result, timestamp = cache[key]
                if time.monotonic() - timestamp < seconds:
                    print(f"Cache HIT for key: {key}")
                    return result

            print(f"Cache MISS for key: {key}")
            result = func(*args, **kwargs)
            cache[key] = (result, time.monotonic())
            return result
        return wrapper
    return decorator

@timed_cache(seconds=60)
def get_user_data(user_id: int) -> dict:
    """A function that simulates a slow API call."""
    print(f"Fetching data for user {user_id} from API...")
    time.sleep(2)
    return {"id": user_id, "name": f"User {user_id}", "data": "..."}

# get_user_data(123) # Slow first call
# get_user_data(123) # Fast second call (from cache)
```

### 4. Retries with Exponential Backoff

Sometimes API calls fail due to transient network issues or temporary server problems. Instead of failing immediately, it's often better to **retry** the request.

**Exponential backoff** is a strategy where you wait progressively longer between each retry (e.g., 1s, 2s, 4s, 8s). This prevents a flood of retries from overwhelming a struggling server.

```python
import time
import httpx
import random

def get_with_retries(url: str, max_retries: int = 3, base_delay: float = 1.0):
    """Makes a GET request with exponential backoff."""
    for attempt in range(max_retries):
        try:
            with httpx.Client() as client:
                response = client.get(url, timeout=5.0)
                response.raise_for_status()
                return response.json()
        except (httpx.RequestError, httpx.HTTPStatusError) as e:
            print(f"Attempt {attempt + 1} failed: {e}")
            if attempt == max_retries - 1:
                print("Max retries reached. Failing.")
                raise

            # Calculate wait time with jitter
            delay = (base_delay * 2**attempt) + random.uniform(0, 1)
            print(f"Waiting {delay:.2f} seconds before retrying...")
            time.sleep(delay)
```

### 5. Mocking APIs for Testing

Your test suite should not depend on external APIs. It makes tests slow, unreliable, and potentially costly. The solution is to **mock** the API.

The `unittest.mock` library (and the `pytest-mock` plugin) allows you to replace the API-calling functions with a "mock" object that returns a predefined response.

```python
import pytest
from unittest.mock import Mock

# Assume we have this function in our application code (my_app/api.py)
def get_username(user_id: int) -> str:
    response = httpx.get(f"https://api.example.com/users/{user_id}")
    response.raise_for_status()
    return response.json()["name"]

# In our test file (tests/test_api.py)
def test_get_username(mocker):
    # Create a mock response object
    mock_response = Mock()
    mock_response.status_code = 200
    mock_response.json.return_value = {"id": 1, "name": "Jules"}

    # Use mocker to replace httpx.get with our mock
    mocker.patch("httpx.get", return_value=mock_response)

    # Call our application code
    username = get_username(1)

    # Assert that it behaved as expected
    assert username == "Jules"
    httpx.get.assert_called_once_with("https://api.example.com/users/1")
```

## 🔹 Quick Exercise

The `RateLimiter` class in the quick exercise from `CURRICULUM.md` is a bit complex. Let's try a simpler version.

Complete the `SimpleRateLimiter` class below. It should allow a certain number of calls within a fixed time window.

```python
import time
from collections import deque

class SimpleRateLimiter:
    def __init__(self, max_calls: int, period: float):
        self.max_calls = max_calls
        self.period = period
        self.timestamps = deque()

    def allow(self) -> bool:
        """Returns True if the request is allowed, False otherwise."""
        now = time.monotonic()

        # Remove timestamps that are outside the current time window
        while self.timestamps and self.timestamps[0] <= now - self.period:
            self.timestamps.popleft()

        # If we have space for another call, allow it
        if len(self.timestamps) < self.max_calls:
            self.timestamps.append(now)
            return True

        return False
```

## 📝 Daily Assignment

**Goal**: Make the "To-Do List" API client from Day 2 more robust by adding caching, retries, and rate limiting.

1.  **Create a Wrapper Class**: Create a new file `robust_client.py`. In it, create a class `RobustTodoClient` that wraps an instance of your `ApiClient` from Day 2.
2.  **Add Caching**:
    - Implement a caching mechanism for the `get_todo(todo_id)` method. Use a timed cache (like the one in the example) with a TTL of 30 seconds.
    - When you call `get_todo(1)` twice in a row, the second call should be a cache hit and should not perform a real (mocked) API call.
3.  **Add Retries**:
    - Wrap the API calls in a retry mechanism with exponential backoff.
    - To test this, you can use `pytest-mock` to make the mocked API call fail the first two times but succeed on the third.
4.  **Add Rate Limiting**:
    - Incorporate a rate limiter (like the `SimpleRateLimiter` from the exercise) into your client.
    - Ensure that if you try to make too many requests in a short period, the client will pause or prevent the call.
5.  **Write Tests**:
    - Write `pytest` tests for your `RobustTodoClient`.
    - Use `mocker` to simulate the underlying `ApiClient`.
    - Write a test that verifies caching works.
    - Write a test that verifies the retry logic works as expected.
    - Write a test to confirm that the rate limiter prevents excessive calls.

## 📖 Further Reading

- [Google Cloud: API Design - Retries](https://cloud.google.com/apis/design/errors#retries)
- [Stripe API: Rate Limiters](https://stripe.com/blog/rate-limiters)
- [Real Python: Caching in Python with `functools.lru_cache`](https://realpython.com/lru-cache-python/)
- [pytest-mock documentation](https://pytest-mock.readthedocs.io/en/latest/)
