# Week 12 Project: Parallel Web Crawler

## 🎯 Project Goal

Build a multi-threaded web crawler that can explore a website from a starting URL, discover all reachable internal links, and do so concurrently to improve performance.

## 📚 Background

A web crawler is a bot that systematically browses the web for the purpose of web indexing. Search engines like Google use extremely sophisticated crawlers to discover and index the content of the internet.

Our crawler will be much simpler. It will start at a single URL, download the page, parse it for new links, and add those new links to a queue of pages to visit. The key challenge is to do this efficiently using multiple threads, while safely managing the shared data structures (the queue of pages to visit and the set of pages that have already been visited).

This project will require you to use `requests` to download web pages and `BeautifulSoup4` to parse the HTML.

```bash
# Make sure you are in your project directory
poetry add requests beautifulsoup4
```

## 📋 Project Requirements

### 1. The Crawler Function

Create a core function, let's call it `parse_links(url: str, base_domain: str) -> set[str]`. This function will be the main task for each of your threads.

- It takes a `url` to crawl and the `base_domain` of the website (e.g., "python.org") to ensure you don't crawl external sites.
- It should handle network errors gracefully (e.g., using a `try...except` block for `requests.exceptions.RequestException`).
- It uses `requests.get()` to download the HTML content of the URL.
- It uses `BeautifulSoup` to parse the HTML.
- It finds all anchor tags (`<a href="...">`).
- For each link found, it must be "normalized":
  - Relative links (e.g., `/about`, `contact.html`) must be converted to absolute links (e.g., `https://www.python.org/about`). The `urllib.parse.urljoin` function is perfect for this.
  - Links that go to a different domain should be ignored.
  - Links to fragments (e.g., `#section-2`) can be ignored.
- The function should return a `set` of all valid, absolute, internal URLs found on the page.

### 2. The Concurrent Manager

This is the main part of your application that orchestrates the crawling process.

- **Shared State**: You will need two data structures that will be accessed by multiple threads:
  1.  `urls_to_crawl`: A queue of URLs that are waiting to be processed. A `collections.deque` is highly recommended for its efficient `popleft()` operation.
  2.  `visited_urls`: A `set` containing all URLs that have been added to the queue, to prevent processing the same URL multiple times.
- **Synchronization**: Because multiple threads will be reading from and writing to these shared data structures, you **must** protect them with a `threading.Lock`. Every time you access `urls_to_crawl` or `visited_urls`, it must be inside a `with lock:` block.
- **Executor**: Use a `concurrent.futures.ThreadPoolExecutor` to manage a pool of worker threads.

### 3. The Main Execution Loop

Your main function should implement the following logic:

1.  Initialize your shared data structures: `urls_to_crawl` (with the starting URL), `visited_urls` (also with the starting URL), and the `lock`.
2.  Set a limit, for example, `max_pages_to_crawl = 50`.
3.  Use a `with ThreadPoolExecutor(...) as executor:` block.
4.  The main loop should continue as long as there are URLs in `urls_to_crawl` and you haven't reached your `max_pages_to_crawl` limit.
5.  Inside the loop:
    - Acquire the lock.
    - Pop a URL from the `urls_to_crawl` deque.
    - Release the lock.
    - Submit a task to the executor to run your `parse_links` function on that URL.
    - Attach a callback function to the `Future` object using `future.add_done_callback()`.
6.  The callback function is where you handle the results. It will receive the `future` object as an argument.
    - Inside the callback, get the result using `future.result()`. This will be the set of new links.
    - Acquire the lock.
    - For each `new_link` in the result set:
      - If the `new_link` is not already in `visited_urls`:
        - Add it to `visited_urls`.
        - Add it to the `urls_to_crawl` deque.
    - Release the lock.
7.  The main thread needs to wait for the crawling to finish. A simple way is to just let the `with executor:` block finish, but you might need a more sophisticated way to handle shutdown, ensuring all tasks are complete. An `Event` or `Barrier` could be useful here to signal when the work is truly "done" (i.e., the queue is empty and all workers are finished).

### 4. Output

Your program should print out information as it runs, such as:

- Which URL is currently being crawled.
- How many new links were found on a page.
- The total number of unique pages found so far.

At the end, print a summary of the crawl, including the total number of unique pages visited and the list of all URLs found.

## 🏆 Bonus Challenges

1.  **Respect `robots.txt`**: Before crawling a domain, download and parse its `robots.txt` file to see which paths you are not allowed to crawl. Python's `urllib.robotparser` can help with this.
2.  **Configurable Depth**: Add an option to limit the crawl depth. A URL found on the start page is at depth 1, a URL found on one of those pages is at depth 2, and so on.
3.  **Performance Tuning**: Experiment with the number of worker threads (`max_workers`). Is more always better? Use `time.perf_counter()` to profile your crawler and find the optimal number of threads for a given website.
4.  **Error Handling**: Improve the error handling. What happens on a 404 Not Found error? What about a 301 Redirect? Your crawler should handle these cases robustly.

## 💡 Tips

- Start by writing the `parse_links` function and testing it on a single URL before you add any concurrency.
- Use the `logging` module to have threads log their actions. This is much safer than `print` in a multi-threaded context and will help you debug race conditions.
- The logic for managing the queue and visited set is the trickiest part. Think carefully about what needs to be protected by the lock.
- Choose a simple, well-structured website for your initial tests (e.g., `toscrape.com`). Avoid very large and complex sites like Wikipedia until you are confident your crawler works correctly.
