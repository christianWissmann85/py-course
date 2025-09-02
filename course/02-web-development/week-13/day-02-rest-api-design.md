# Week 13, Day 2: RESTful API Design

## 🎯 Learning Objectives
- [ ] Understand the core principles and constraints of REST (Representational State Transfer).
- [ ] Learn how to design and model API resources effectively.
- [ ] Understand the correct semantic use of HTTP methods (GET, POST, PUT, DELETE).
- [ ] Learn the standard meanings of common HTTP status codes.
- [ ] Explore different strategies for API versioning.

## 📚 Concepts

### 1. What is REST?
**REST (Representational State Transfer)** is an architectural style for designing networked applications. It's not a protocol or a standard, but a set of constraints that, if followed, lead to a scalable, maintainable, and easy-to-use API.

The key constraints of REST are:
1.  **Client-Server Architecture**: Separation of concerns. The client handles the user interface, and the server handles data storage and business logic.
2.  **Statelessness**: Each request from a client to a server must contain all the information needed to understand and complete the request. The server does not store any client context between requests.
3.  **Cacheability**: Responses must define themselves as cacheable or not. This helps improve performance and scalability.
4.  **Layered System**: A client cannot ordinarily tell whether it is connected directly to the end server or to an intermediary along the way (like a load balancer or cache).
5.  **Uniform Interface**: This is the most fundamental principle and has its own sub-constraints.

### 2. The Uniform Interface
This principle simplifies and decouples the architecture.
-   **Resource-Based**: Individual resources are identified in requests. For example, `/users/123` is the URI for a specific user resource.
-   **Manipulation of Resources Through Representations**: The client has a representation of a resource (e.g., a JSON object). It can modify that resource by sending this representation back to the server. The server then updates its state.
-   **Self-Descriptive Messages**: Each message includes enough information to describe how to process it (e.g., the `Content-Type` header tells the server it's receiving JSON).
-   **HATEOAS (Hypermedia as the Engine of Application State)**: The API response should include links to other related actions or resources. For example, a response for a user might include a link to `/users/123/posts`.

### 3. Resource Design & HTTP Methods
In REST, everything is a **resource**. A resource is an object with a type, associated data, relationships to other resources, and a set of methods that operate on it.

-   **Resources are Nouns, not Verbs**: Use `/users`, not `/getUsers`.
-   **Use HTTP Methods for Operations**: The HTTP method (verb) tells the server what to do with the resource (noun).

| Method  | Action                                     | Example Request          | Typical Response (Success) |
|---------|--------------------------------------------|--------------------------|----------------------------|
| `GET`   | Retrieve a resource or a collection        | `GET /users/123`         | `200 OK` + User data       |
| `POST`  | Create a new resource                      | `POST /users` + data     | `201 Created` + New User   |
| `PUT`   | Update/replace an existing resource entirely | `PUT /users/123` + data  | `200 OK` + Updated User    |
| `PATCH` | Partially update an existing resource      | `PATCH /users/123` + data| `200 OK` + Updated User    |
| `DELETE`| Delete a resource                          | `DELETE /users/123`      | `204 No Content`           |

### 4. HTTP Status Codes
Using standard status codes is crucial for a self-descriptive API.

**Common Success Codes:**
-   `200 OK`: General success for `GET`, `PUT`, `PATCH`.
-   `201 Created`: A new resource was successfully created (`POST`).
-   `204 No Content`: The action was successful, but there is no data to return (`DELETE`).

**Common Client Error Codes:**
-   `400 Bad Request`: The request was malformed (e.g., invalid JSON).
-   `401 Unauthorized`: The client is not authenticated.
-   `403 Forbidden`: The client is authenticated but does not have permission to access this resource.
-   `404 Not Found`: The requested resource does not exist.
-   `422 Unprocessable Entity`: The request was well-formed, but contained semantic errors (e.g., a required field was missing).

**Common Server Error Codes:**
-   `500 Internal Server Error`: A generic, unexpected error occurred on the server.

### 5. API Versioning
As your API evolves, you will need to make breaking changes. Versioning allows you to do this without breaking existing client integrations.

**Common Strategies:**
1.  **URI Versioning (Most Common)**: Put the version in the URL.
    -   `https://api.example.com/v1/users`
    -   `https://api.example.com/v2/users`
2.  **Header Versioning**: Use a custom request header.
    -   `Accept: application/vnd.example.api.v1+json`
3.  **Query Parameter Versioning**:
    -   `https://api.example.com/users?version=1`

URI versioning is the most straightforward and widely understood approach.

## 🔹 Quick Exercise

You are designing a simple API for a blog. What would be the appropriate RESTful endpoint (Method + URI) for the following actions?

1.  Get a list of all posts.
2.  Create a new post.
3.  Get a single post with an ID of `42`.
4.  Update the title of post `42`.
5.  Delete post `42`.
6.  Get all comments for post `42`.

**Answers:**
1.  `GET /posts`
2.  `POST /posts`
3.  `GET /posts/42`
4.  `PATCH /posts/42` (with a body like `{"title": "New Title"}`)
5.  `DELETE /posts/42`
6.  `GET /posts/42/comments` (This shows how to handle nested resources)

## 📝 Daily Assignment
**Goal**: Design a RESTful API for a simple "To-Do List" application and document it.

1.  **Define Resources**: Identify the main resources for your API. (Hint: You'll at least need a `Todo` resource).
2.  **Design Endpoints**: For each resource, define the endpoints for all CRUD (Create, Read, Update, Delete) operations. Specify the HTTP method, the URI, a description of what it does, and the expected success and error status codes.
    -   Think about how to list all todos, create one, get one, update one, and delete one.
    -   Consider sub-resources. Maybe a todo has sub-tasks?
3.  **Model the Data**: For each resource, define its data structure. What fields does a `Todo` item have? (e.g., `id`, `title`, `description`, `completed`, `created_at`).
4.  **Create Documentation**: Write your API design in a markdown file (`todo_api_design.md`). Use tables to clearly lay out the endpoints.

**Example Documentation Format:**

### Resource: `Todo`
**Data Model:**
- `id` (integer, read-only)
- `title` (string, required)
- `completed` (boolean, default: `false`)

### Endpoints
| Method | URI Path   | Description        | Success Response | Error Responses        |
|--------|------------|--------------------|------------------|------------------------|
| `GET`  | `/todos`   | Get all todo items | `200 OK`         | `500 Internal Error`   |
| `POST` | `/todos`   | Create a new todo  | `201 Created`    | `400 Bad Request`      |
| ...    | ...        | ...                | ...              | ...                    |

5.  **Implement a Client (Optional)**: Using the `APIClient` from yesterday, write a few functions that *would* interact with your designed API, even though the server doesn't exist. This helps you think about the design from a user's perspective.

## ⚠️ Common Mistakes
-   **Using Verbs in URIs**: `POST /createUser` is not RESTful. `POST /users` is. The action is in the HTTP method, not the URI.
-   **Not Using Plural Nouns**: Conventionally, collections are plural (`/users`), and a specific item is a member of that collection (`/users/123`).
-   **Ignoring Status Codes**: Returning `200 OK` for everything, even errors, makes the API difficult to use and debug.
-   **Putting everything in `POST` requests**: New developers sometimes avoid other methods and just use `POST` for everything. This violates the principle of using the correct semantic verb for the action.
-   **Leaking Server Details**: A `500` error should return a generic error message, not a full stack trace, which can be a security risk.

## 📖 Further Reading
- [Microsoft REST API design guide](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design)
- [Google Cloud API Design Guide](https://cloud.google.com/apis/design/)
- [What is REST - Red Hat](https://www.redhat.com/en/topics/api/what-is-rest-api)
