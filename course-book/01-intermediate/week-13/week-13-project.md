# Week 13 Project: Build an API Gateway

## 🎯 Project Goal

To build a simple but robust API Gateway that acts as a single entry point for multiple downstream microservices. The gateway will provide centralized functionalities like rate limiting, caching, and request routing for both HTTP and WebSocket traffic.

## 📚 Background

In a microservices architecture, an **API Gateway** is a server that sits between the client applications and the backend services. Instead of clients talking to dozens of different services, they talk to the gateway. The gateway then intelligently routes requests to the appropriate service.

This pattern is extremely powerful because it allows you to implement cross-cutting concerns in a single place. These include:

- **Authentication & Authorization**: Authenticate users once at the gateway.
- **Rate Limiting**: Protect all your services from abuse.
- **Caching**: Cache frequent responses to reduce load on downstream services.
- **Request/Response Transformation**: Modify requests and responses as they pass through.
- **Service Discovery**: Route to services without the client needing to know their actual addresses.

For this project, we will use a modern Python web framework to build our gateway. **FastAPI** is an excellent choice because it's built on top of `asyncio` and has great support for both HTTP and WebSockets.

```bash
# Make sure you are in your project directory
poetry add fastapi uvicorn httpx
```

## 📋 Project Requirements

### 1. The Microservices

You can't have a gateway without services to route to. Create two very simple "microservices" that will run alongside your gateway.

**`service_a.py` (HTTP Service)**

- A simple FastAPI app running on `http://localhost:8001`.
- It should have a few endpoints, like:
  - `GET /items/{item_id}`: Returns some dummy item data.
  - `GET /users`: Returns a list of users.

**`service_b.py` (WebSocket Service)**

- A simple FastAPI app running on `http://localhost:8002`.
- It should have a WebSocket endpoint at `/ws` that acts as an echo server (sending back any message it receives).

### 2. The Gateway (`gateway.py`)

This is the main application, which will run on `http://localhost:8000`.

**A. Basic Routing**

- The gateway must inspect the incoming URL and route the request to the correct service.
- Requests to `/service-a/...` should be forwarded to `service_a` (`http://localhost:8001`).
- Requests to `/service-b/...` should be forwarded to `service_b` (`http://localhost:8002`).
- Use `httpx.AsyncClient` to make the downstream requests.
- The gateway should forward the original request's method, headers, query parameters, and body. It must then stream the response from the downstream service back to the original client.

**B. Rate Limiting**

- Implement a rate limiter (you can use the `SimpleRateLimiter` from Day 5's exercise).
- The rate limit should be applied per client IP address.
- If a client exceeds the rate limit (e.g., >10 requests in 60 seconds), the gateway should immediately respond with a `429 Too Many Requests` status code without forwarding the request.

**C. Caching**

- Implement a caching layer (you can use the `timed_cache` decorator from Day 5's exercise or a more robust dictionary-based cache).
- `GET` requests to the gateway should be cached. For example, a request to `GET /service-a/items/123` should be cached using its full path as the key.
- The cache should have a Time-To-Live (TTL) of, for example, 30 seconds.
- The first time a request is made, it should be a "CACHE MISS," and the gateway should forward it. The second time (within the TTL), it should be a "CACHE HIT," and the gateway should return the cached response directly without contacting the downstream service.

**D. WebSocket Proxy**

- The gateway must be able to handle WebSocket connections.
- A WebSocket connection to `ws://localhost:8000/service-b/ws` should be proxied to the downstream WebSocket service at `ws://localhost:8002/ws`.
- The gateway should establish a WebSocket connection with both the client and the downstream server, and then pass messages back and forth between them in both directions.

### 3. Running the System

You will need to run all three services simultaneously in different terminal windows:

```bash
# Terminal 1: Run Service A
uvicorn service_a:app --port 8001

# Terminal 2: Run Service B
uvicorn service_b:app --port 8002

# Terminal 3: Run the Gateway
uvicorn gateway:app --port 8000
```

## 🏆 Bonus Challenges

1.  **Centralized Authentication**: Add a simple authentication check at the gateway. Require a specific header (e.g., `X-API-Key: my-secret-key`). If the header is missing or incorrect, the gateway should return a `401 Unauthorized` error.
2.  **Configuration**: Instead of hardcoding the downstream service URLs (`http://localhost:8001`), load them from a simple configuration file (e.g., `config.json`).
3.  **Circuit Breaker**: Implement a "circuit breaker" pattern. If a downstream service fails (e.g., returns 5xx errors) multiple times in a row, the gateway should "trip the breaker" and immediately fail subsequent requests to that service for a short period, giving it time to recover.
4.  **Request/Response Logging**: Add structured logging to the gateway to log key details of each request and response (method, path, status code, duration).

## 💡 Tips

- Tackle the requirements one at a time. Start with basic HTTP routing. Once that works, add caching. Then rate limiting. Finally, tackle the WebSocket proxy, which is the most complex part.
- FastAPI's `Request` object gives you access to all the details you need (method, headers, body).
- FastAPI's `StreamingResponse` is perfect for sending the raw response from a downstream service back to the client.
- For the WebSocket proxy, you will need to create two `asyncio` tasks running concurrently: one to read from the client and write to the server, and another to read from the server and write to the client. `asyncio.gather` will be your friend here.
- Remember that a single client might have multiple simultaneous connections. Your rate limiter should be ableto handle this.
