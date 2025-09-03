# Week 13, Day 3: WebSockets

## 🎯 Learning Objectives

- [ ] Understand the purpose of the WebSocket protocol and how it differs from HTTP.
- [ ] Learn to build a basic WebSocket server using the `websockets` library.
- [ ] Learn to build a WebSocket client to connect to a server.
- [ ] Manage connection lifecycles, including handling connections and disconnections.
- [ ] Implement simple real-time, bi-directional messaging patterns.

## 📚 Concepts

### 1. What are WebSockets?

HTTP is a request-response protocol. The client sends a request, the server sends a response, and the connection is typically closed. This is inefficient for real-time applications (like chat, live notifications, or multiplayer games) where the server might need to "push" data to the client at any time.

**WebSockets** provide a solution. The WebSocket protocol allows for a **full-duplex** (two-way) communication channel over a single, long-lived TCP connection.

- **Handshake**: A WebSocket connection starts as a standard HTTP `GET` request with special headers (`Upgrade: websocket`). If the server supports it, it "upgrades" the connection from HTTP to the WebSocket protocol.
- **Bi-directional**: Once the connection is established, both the client and server can send messages to each other at any time, independently.
- **Low Latency**: Because the connection is persistent, it avoids the overhead of establishing new HTTP connections for every message, resulting in lower latency.

### 2. The `websockets` Library

The `websockets` library is the leading choice for working with WebSockets in Python. It's built on top of `asyncio`, making it highly performant.

First, install it: `poetry add websockets`

### 3. Building a WebSocket Server

A server consists of a "handler" coroutine that is executed for each new client connection.

```python
import asyncio
import websockets

# This handler is called for each client that connects.
# The 'websocket' object is the connection to that specific client.
async def echo_server(websocket):
    print("Client connected.")
    try:
        # The server will wait here, listening for messages from the client.
        async for message in websocket:
            print(f"Received message: {message}")
            # Send a response back to the same client.
            response = f"Echo: {message}"
            await websocket.send(response)
            print(f"Sent response: {response}")
    except websockets.exceptions.ConnectionClosed as e:
        print(f"Client disconnected: {e}")
    finally:
        print("Connection handler finished.")

async def main():
    # Start the WebSocket server on localhost, port 8765
    # The first argument is the connection handler.
    async with websockets.serve(echo_server, "localhost", 8765):
        print("Server started on ws://localhost:8765")
        # The server will run forever until the program is stopped.
        await asyncio.Future()

if __name__ == "__main__":
    asyncio.run(main())
```

### 4. Building a WebSocket Client

A client connects to a server's URL (which starts with `ws://` or `wss://` for secure WebSockets).

```python
import asyncio
import websockets

async def client():
    # Connect to the server we built above.
    uri = "ws://localhost:8765"
    async with websockets.connect(uri) as websocket:
        # Send a message to the server.
        await websocket.send("Hello, Server!")
        print("Sent 'Hello, Server!'")

        # Wait for a response from the server.
        response = await websocket.recv()
        print(f"Received response: {response}")

if __name__ == "__main__":
    asyncio.run(client())
```

### 5. Managing Connections and State

In a real application, you need to manage multiple connections. A common pattern is to keep a set of all active client connections.

```python
import asyncio
import websockets

# A set to store all connected clients.
CONNECTED_CLIENTS = set()

async def broadcast(message: str):
    """Send a message to all connected clients."""
    # websockets.broadcast() is a convenient helper for this.
    # It sends the message to all websockets in the given set.
    if CONNECTED_CLIENTS:
        print(f"Broadcasting '{message}' to {len(CONNECTED_CLIENTS)} clients.")
        await websockets.broadcast(CONNECTED_CLIENTS, message)

async def chat_handler(websocket):
    """Handles new connections and messages."""
    # Add the new client to our set of connected clients.
    CONNECTED_CLIENTS.add(websocket)
    print(f"Client connected. Total clients: {len(CONNECTED_CLIENTS)}")
    try:
        # Listen for messages from this client.
        async for message in websocket:
            # When a message is received, broadcast it to everyone.
            await broadcast(f"User says: {message}")
    finally:
        # When the client disconnects, remove them from the set.
        CONNECTED_CLIENTS.remove(websocket)
        print(f"Client disconnected. Total clients: {len(CONNECTED_CLIENTS)}")

async def main():
    async with websockets.serve(chat_handler, "localhost", 8765):
        await asyncio.Future() # run forever

if __name__ == "__main__":
    asyncio.run(main())
```

This example forms the basis of a simple chat room.

## 🔹 Quick Exercise

Modify the `echo_server` from the first example. Instead of just echoing the message, the server should respond with the message in all uppercase letters.

```python
import asyncio
import websockets

async def uppercase_server(websocket):
    """Receives a message and responds with the uppercase version."""
    async for message in websocket:
        # Your code here: convert the message to uppercase
        response = message.upper()
        await websocket.send(response)

# The rest of the server setup code remains the same.
# async def main(): ...
# if __name__ == "__main__": ...
```

## 📝 Daily Assignment

**Goal**: Build a simple real-time chat application.

You will build two scripts: `chat_server.py` and `chat_client.py`.

### `chat_server.py`

1.  **Use the "Managing Connections and State" example** as your starting point. This server should be able to accept multiple client connections.
2.  **Track Users**: When a user connects, ask them for a username. The first message they send should be treated as their username.
3.  **Broadcast Messages**: When a user sends a message, the server should prepend the username to it (e.g., `"[Alice]: Hello everyone!"`) and then broadcast it to all _other_ connected clients. The user who sent the message should not receive their own message back.
4.  **Handle Disconnections**: When a user disconnects, broadcast a message to all remaining clients announcing their departure (e.g., `"[Server]: Alice has left the chat."`).

### `chat_client.py`

1.  **Connect to the Server**: The client should connect to `ws://localhost:8765`.
2.  **Send Messages**: The client should allow the user to type messages in the terminal and send them to the server. You'll need a way to handle user input without blocking the reception of messages. `asyncio.to_thread` can be useful for this, or you can create two separate tasks: one for sending and one for receiving.
3.  **Receive Messages**: The client must continuously listen for messages from the server and print them to the console.

**Putting it together:**
Run the server script in one terminal. Then, open several other terminals and run the client script in each. You should be able to send messages from any client and see them appear on all other clients.

## ⚠️ Common Mistakes

- **Blocking the Event Loop**: Using a blocking call (like `time.sleep()` or `input()`) inside an `async` function will freeze the entire server or client. Always use the `async` version (e.g., `await asyncio.sleep()`).
- **Not Handling Disconnections**: If you don't wrap your message loop in a `try...finally` or `try...except ConnectionClosed`, your server handler might crash when a client disconnects unexpectedly, and you won't be able to clean up resources (like removing them from the `CONNECTED_CLIENTS` set).
- **State Management Issues**: Forgetting to `add` a client on connection or `remove` them on disconnection can lead to memory leaks or trying to send messages to closed connections.

## 📖 Further Reading

- [websockets library documentation](https://websockets.readthedocs.io/en/stable/)
- [MDN: WebSockets API](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
- [Real Python: Python WebSockets](https://realpython.com/python-websockets/)
