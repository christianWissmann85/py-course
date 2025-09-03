# Week 13, Day 4: Low-Level Network Programming

## 🎯 Learning Objectives

- [ ] Understand the role of the `socket` module for low-level networking.
- [ ] Learn the fundamental differences between TCP (Transmission Control Protocol) and UDP (User Datagram Protocol).
- [ ] Build a basic TCP client and server.
- [ ] Design a simple application-layer protocol for communication.
- [ ] Handle common network errors and connection issues.

## 📚 Concepts

### 1. Sockets: The Foundation of Networking

HTTP, WebSockets, and most other network protocols are built on top of **sockets**. A socket is an endpoint for sending or receiving data across a computer network. The `socket` module in Python provides a low-level interface to the operating system's network stack.

Working with sockets directly gives you maximum control, but it also means you are responsible for many details that higher-level libraries like `requests` or `websockets` handle for you (like formatting requests, handling connection state, etc.).

### 2. TCP vs. UDP

There are two primary transport protocols used on the internet:

**TCP (Transmission Control Protocol)**

- **Connection-Oriented**: A connection (like a phone call) must be established before data is sent.
- **Reliable**: Guarantees that all data arrives in the correct order and without errors. If a packet is lost, it's re-transmitted.
- **Stream-Based**: Data is seen as a continuous stream of bytes.
- **Use Cases**: Web browsing (HTTP), file transfers (FTP), email (SMTP). Anything where data integrity is paramount.

**UDP (User Datagram Protocol)**

- **Connectionless**: No connection is established. You just send packets (datagrams) to a destination.
- **Unreliable**: No guarantee of delivery, order, or error-checking. Packets might be lost, duplicated, or arrive out of order.
- **Datagram-Based**: Data is sent in discrete packets.
- **Fast and Low-Overhead**: Because it does less work, it's much faster than TCP.
- **Use Cases**: Video streaming, online gaming, DNS. Anything where speed is more important than perfect reliability.

### 3. Building a TCP Server

A TCP server listens for incoming connections on a specific address and port.

```python
import socket

def run_server(host: str = '127.0.0.1', port: int = 65432):
    # AF_INET specifies the IPv4 address family.
    # SOCK_STREAM specifies that this is a TCP socket.
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        # Bind the socket to the address and port.
        s.bind((host, port))

        # Start listening for incoming connections. The '1' is the backlog size.
        s.listen(1)
        print(f"Server listening on {host}:{port}")

        # Accept a new connection. This is a blocking call.
        # It returns a new socket object for this connection and the client's address.
        conn, addr = s.accept()

        with conn:
            print(f"Connected by {addr}")
            while True:
                # Receive data from the client (up to 1024 bytes).
                # This is also a blocking call.
                data = conn.recv(1024)
                if not data:
                    # If recv() returns an empty bytes object, the client has closed the connection.
                    break
                print(f"Received from client: {data.decode('utf-8')}")

                # Send the data back to the client, prepended with "Echo: ".
                conn.sendall(b"Echo: " + data)

if __name__ == "__main__":
    run_server()
```

### 4. Building a TCP Client

The client creates a socket and connects to the server's address and port.

```python
import socket

def run_client(host: str = '127.0.0.1', port: int = 65432):
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        # Connect to the server.
        s.connect((host, port))

        # Send a message to the server.
        message = "Hello, world!"
        s.sendall(message.encode('utf-8'))
        print(f"Sent to server: {message}")

        # Receive the response from the server.
        data = s.recv(1024)

        print(f"Received from server: {data.decode('utf-8')}")

if __name__ == "__main__":
    run_client()
```

### 5. Designing a Simple Protocol

When using sockets, you're just sending bytes. Both sides need to agree on what those bytes mean. This agreement is your **application-layer protocol**.

A simple protocol might be:

- All messages are UTF-8 encoded text.
- Messages are terminated by a newline character (`\n`).

A more robust protocol might involve sending a fixed-size header that specifies the length of the message body that follows. This avoids issues with needing to know how much data to `recv()`.

```python
# Protocol Idea: First 4 bytes are an integer (in network byte order)
# representing the length of the message that follows.

# Client sending
message = "My long message"
encoded_message = message.encode('utf-8')
header = len(encoded_message).to_bytes(4, 'big') # 'big' is network byte order
s.sendall(header + encoded_message)

# Server receiving
header_data = conn.recv(4)
message_length = int.from_bytes(header_data, 'big')
message_data = conn.recv(message_length)
print(message_data.decode('utf-8'))
```

## 🔹 Quick Exercise

Create a simple TCP server that, upon connection, sends the current date and time to the client and then immediately closes the connection.

```python
import socket
import datetime

def time_server(host: str = '127.0.0.1', port: int = 65431):
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.bind((host, port))
        s.listen()
        print("Time server listening...")
        conn, addr = s.accept()
        with conn:
            print(f"Connected to {addr}")
            # Get the current time as a string.
            current_time = datetime.datetime.now().isoformat()
            # Encode it and send it to the client.
            conn.sendall(current_time.encode('utf-8'))
            # The 'with' statement will automatically close the connection.

# if __name__ == "__main__":
#     time_server()
```

You can test this with a simple client or even a tool like `netcat` (`nc localhost 65431`).

## 📝 Daily Assignment

**Goal**: Build a simple key-value store using a TCP server.

You will build a server that acts like a simple dictionary. Clients can connect and send commands to set, get, or delete keys.

1.  **Design the Protocol**:

    - Decide on a simple text-based protocol. For example:
      - `SET key value\n`
      - `GET key\n`
      - `DELETE key\n`
    - The server should respond with `OK\n`, the value for `GET` requests, or `ERROR\n` if something goes wrong.

2.  **Implement the Server (`kv_server.py`)**:

    - The server should have a global Python dictionary to store the key-value pairs.
    - It should listen for connections in a loop, handling one client at a time.
    - For each client, it should read a command, parse it, perform the action on the dictionary, and send back the appropriate response.
    - Handle potential errors (e.g., `GET` for a key that doesn't exist).

3.  **Implement the Client (`kv_client.py`)**:

    - The client should take commands from the command line (e.g., `python kv_client.py GET mykey`).
    - It should connect to the server, send the formatted command, wait for the response, print it, and then exit.

4.  **Test It**:
    - Run the server.
    - In other terminals, use the client to set a few keys, get them back, and then delete them.

## ⚠️ Common Mistakes

- **Assuming `recv()` gets the whole message**: `recv(1024)` will read _at most_ 1024 bytes. If the sender sent more, you need to call `recv()` in a loop until you have the full message. This is why length-prefixing protocols are important.
- **Forgetting to `encode()`/`decode()`**: Sockets send `bytes`, not `str`. You must encode strings before sending and decode bytes after receiving.
- **Not Closing Sockets**: Forgetting to call `close()` (or not using a `with` statement) can leave sockets open, consuming system resources.
- **Ignoring Network Byte Order**: When sending multi-byte data like integers, different computer architectures may store the bytes in a different order (little-endian vs. big-endian). The standard is to use "network byte order," which is big-endian.

## 📖 Further Reading

- [Python Docs: `socket` — Low-level networking interface](https://docs.python.org/3/library/socket.html)
- [Real Python: Socket Programming in Python (Guide)](https://realpython.com/python-sockets/)
- [Socket Programming HOWTO](https://docs.python.org/3/howto/sockets.html)
