# Week 11 Project: Concurrent Web Scraper

## 🎯 Project Objective
To build a high-performance, concurrent web scraper using the `asyncio` framework. This project will combine all the concepts from this week: creating and gathering tasks, rate-limiting with semaphores, handling errors and timeouts, and using asynchronous libraries like `httpx` and `aiofiles`.

You will build a scraper that starts from a single URL, finds all the links on that page, and then concurrently scrapes those links for specific information.

## 📋 Core Requirements
The goal is to scrape the titles of all the blog posts from a target website, like the [Real Python](https://realpython.com/) blog, as quickly and responsibly as possible.

### 1. Project Setup
-   Create a new Poetry project: `poetry new async-scraper`.
-   Add the necessary dependencies: `poetry add httpx beautifulsoup4 aiofiles`.
-   Structure your code in a `scraper/` package.

### 2. The `scraper.py` Module
This will contain the core logic of your application.

-   **`fetch_html(client: httpx.AsyncClient, url: str) -> str | None`**:
    -   An `async` function that takes an `httpx` client and a URL.
    -   It should fetch the HTML content of the page.
    -   It must include error handling for network errors (`httpx.RequestError`) and timeouts (`asyncio.TimeoutError`), returning `None` on failure.

-   **`parse_links(html: str, base_url: str) -> set[str]`**:
    -   A **synchronous** function that takes HTML content and the page's base URL.
    -   It should use `BeautifulSoup` to parse the HTML and find all `<a>` tags with an `href` attribute.
    -   It should resolve relative URLs (e.g., `/tutorials/python-tricks/`) into absolute URLs (e.g., `https://realpython.com/tutorials/python-tricks/`).
    -   Return a `set` of unique, absolute URLs found on the page.

-   **`parse_title(html: str) -> str | None`**:
    -   A synchronous function that uses `BeautifulSoup` to find and return the text of the `<title>` tag from the HTML.

### 3. The `main` Coroutine (The Orchestrator)
This is where you will manage the concurrent execution.

-   **Concurrency Controls**:
    -   Create an `asyncio.Semaphore` to limit the number of concurrent requests to a responsible number (e.g., 10).
    -   Create an `asyncio.Queue` to hold the URLs that need to be scraped.
    -   Create a `set` to keep track of URLs that have already been visited to avoid re-scraping and getting into loops.

-   **The `worker` Coroutine**:
    -   `async def worker(queue: asyncio.Queue, ...)`: This will be your consumer.
    -   It should run in a loop, getting a URL from the queue.
    -   It must use the semaphore to rate-limit its network requests.
    -   It calls `fetch_html` to get the page content.
    -   If successful, it calls `parse_title` and `parse_links`.
    -   It should save the title and URL to a results file (using `aiofiles`).
    -   It should add all newly discovered links back into the queue for other workers to pick up.

-   **Orchestration Logic**:
    -   The `main` function should initialize the queue with a starting URL.
    -   It should create and start a number of worker tasks (e.g., 10 workers).
    -   It must have a mechanism to decide when to stop. A good approach is to stop after a certain number of pages have been successfully scraped, or after the queue has been empty for a certain amount of time. `await asyncio.wait_for(queue.join(), timeout=...)` can be a useful pattern here.
    -   Ensure all worker tasks are properly cancelled when the work is done.

### 4. Progress Tracking
-   Provide feedback to the user running the script.
-   Print out the URL being processed by each worker.
-   Keep a running count of successful scrapes and print a final summary.

## ✅ Definition of Done
- [ ] The scraper starts at a given URL and recursively follows links it finds.
- [ ] It fetches pages concurrently, respecting a rate limit set by a semaphore.
- [ ] It parses the title from each successfully fetched page.
- [ ] The scraped titles and their URLs are saved to a text file asynchronously.
- [ ] The application handles network errors and timeouts gracefully without crashing.
- [ ] The scraper avoids visiting the same URL more than once.
- [ ] The entire codebase is type-safe and passes `mypy --strict`.

## 🏆 Bonus Challenges
-   **Depth Limiting**: Modify the crawler to only go a certain number of "levels" deep from the starting page. (Hint: you can store `(url, depth)` tuples in the queue).
-   **Domain Filtering**: Ensure your scraper does not leave the original website (e.g., if it starts on `realpython.com`, it should not follow links to `google.com`).
-   **Robots.txt**: A very important real-world feature. Before scraping any site, you should check its `/robots.txt` file to see which paths you are allowed to scrape. Implement a function to fetch and parse this file, and make your workers respect its rules.
-   **Persistent State**: Save the `visited_urls` set to a file when the scraper stops, and load it when it starts, allowing you to resume a cancelled scrape.

## 💡 Tips
-   Start with a single worker to get the logic right before introducing concurrency.
-   Use `asyncio.create_task()` to start your worker coroutines so they run in the background.
-   The `queue.join()` and `queue.task_done()` pattern is your best friend for managing the workflow.
-   Be a responsible scraper! Always include a delay (`asyncio.sleep`) and a reasonable concurrency limit to avoid overwhelming the server you are scraping.
