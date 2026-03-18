# Course 01: HTTP Fundamentals

## 📖 What You Will Learn

- What HTTP is and how it works
- The structure of HTTP requests and responses
- HTTP methods (GET, POST, PUT, DELETE, PATCH, etc.)
- HTTP headers and their purpose
- HTTP status codes and what they mean
- How the browser communicates with a server

---

## 1. What is HTTP?

**HTTP (HyperText Transfer Protocol)** is the foundation of data communication on the web. It defines how messages are formatted and transmitted between a **client** (usually a browser or an app) and a **server**.

### How it works (simplified):

```
Client (Browser)                    Server
     |                                |
     |  ---- HTTP Request --->        |
     |                                |  (processes the request)
     |  <--- HTTP Response ----       |
     |                                |
```

1. The **client** sends an **HTTP Request** to the server (e.g., "Give me the homepage")
2. The **server** processes the request and sends back an **HTTP Response** (e.g., the HTML of the page)

### Key characteristics of HTTP:
- **Stateless** — each request is independent; the server doesn't remember previous requests
- **Text-based** — messages are human-readable (at least in HTTP/1.1)
- **Request-Response model** — client always initiates, server always responds

### HTTP vs HTTPS

**HTTPS** = HTTP + **TLS/SSL encryption**. The data is encrypted during transmission, so nobody in between can read it. Today, almost all websites use HTTPS.

---

## 2. Structure of an HTTP Request

Every HTTP request has these parts:

```
[Method] [URL] [HTTP Version]
[Headers]

[Body (optional)]
```

### Real example:

```http
POST /api/users HTTP/1.1
Host: example.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...

{
  "name": "Nastya",
  "role": "QA Engineer"
}
```

Let's break this down:

| Part | Example | Description |
|------|---------|-------------|
| **Method** | `POST` | What action to perform |
| **URL/Path** | `/api/users` | Where to send the request |
| **HTTP Version** | `HTTP/1.1` | Protocol version |
| **Headers** | `Content-Type: application/json` | Metadata about the request |
| **Body** | `{"name": "Nastya", ...}` | Data sent with the request |

---

## 3. Structure of an HTTP Response

```
[HTTP Version] [Status Code] [Status Message]
[Headers]

[Body]
```

### Real example:

```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/users/42

{
  "id": 42,
  "name": "Nastya",
  "role": "QA Engineer",
  "createdAt": "2024-01-15T10:30:00Z"
}
```

| Part | Example | Description |
|------|---------|-------------|
| **Status Code** | `201` | Numeric result of the request |
| **Status Message** | `Created` | Human-readable status description |
| **Headers** | `Content-Type: application/json` | Metadata about the response |
| **Body** | `{"id": 42, ...}` | The data returned by the server |

---

## 4. HTTP Methods

HTTP methods (also called "verbs") tell the server **what action** to perform.

### Main Methods:

| Method | Purpose | Has Body? | Safe? | Idempotent? |
|--------|---------|-----------|-------|-------------|
| **GET** | Retrieve data | No | ✅ Yes | ✅ Yes |
| **POST** | Create new data | Yes | ❌ No | ❌ No |
| **PUT** | Replace data completely | Yes | ❌ No | ✅ Yes |
| **PATCH** | Update data partially | Yes | ❌ No | ❌ No |
| **DELETE** | Remove data | Optional | ❌ No | ✅ Yes |
| **HEAD** | Same as GET but without body | No | ✅ Yes | ✅ Yes |
| **OPTIONS** | Check what methods are available | No | ✅ Yes | ✅ Yes |

### Important terms:

- **Safe** — the request does NOT modify data on the server (GET, HEAD, OPTIONS)
- **Idempotent** — calling it once or multiple times has the same result (GET, PUT, DELETE)

### Examples in practice:

```
GET    /api/users          → Get list of all users
GET    /api/users/42       → Get user with ID 42
POST   /api/users          → Create a new user
PUT    /api/users/42       → Replace user 42 completely
PATCH  /api/users/42       → Update some fields of user 42
DELETE /api/users/42       → Delete user 42
```

### 🤔 PUT vs PATCH — What's the difference?

**PUT** replaces the entire resource:
```json
// PUT /api/users/42
// You must send ALL fields
{
  "name": "Nastya",
  "role": "Senior QA",
  "email": "nastya@example.com"
}
```

**PATCH** updates only specific fields:
```json
// PATCH /api/users/42
// Send only what you want to change
{
  "role": "Senior QA"
}
```

---

## 5. HTTP Headers

Headers are **key-value pairs** that carry metadata about the request or response. They are like labels on a package — they describe what's inside and how to handle it.

### Common Request Headers:

| Header | Purpose | Example |
|--------|---------|---------|
| `Host` | The domain name of the server | `Host: api.example.com` |
| `Content-Type` | Format of the body data | `Content-Type: application/json` |
| `Accept` | What format the client wants back | `Accept: application/json` |
| `Authorization` | Authentication credentials | `Authorization: Bearer <token>` |
| `User-Agent` | Information about the client | `User-Agent: Mozilla/5.0...` |
| `Cookie` | Cookies sent to the server | `Cookie: session=abc123` |
| `Cache-Control` | Caching instructions | `Cache-Control: no-cache` |

### Common Response Headers:

| Header | Purpose | Example |
|--------|---------|---------|
| `Content-Type` | Format of the response body | `Content-Type: application/json` |
| `Content-Length` | Size of the response body in bytes | `Content-Length: 348` |
| `Set-Cookie` | Set a cookie on the client | `Set-Cookie: session=abc123` |
| `Location` | URL for redirects or new resources | `Location: /api/users/42` |
| `Access-Control-Allow-Origin` | CORS — who can access this | `Access-Control-Allow-Origin: *` |
| `X-Request-Id` | Unique ID for tracking | `X-Request-Id: abc-123-def` |

### Content-Type Values You'll See Often:

| Content-Type | Description |
|-------------|-------------|
| `application/json` | JSON data (most common for APIs) |
| `application/xml` | XML data |
| `text/html` | HTML page |
| `text/plain` | Plain text |
| `multipart/form-data` | File uploads |
| `application/x-www-form-urlencoded` | HTML form data |

---

## 6. HTTP Status Codes

Status codes tell you **what happened** with your request. They are grouped into 5 categories:

### Categories:

| Range | Category | Meaning |
|-------|----------|---------|
| **1xx** | Informational | Request received, continuing... |
| **2xx** | Success | Request was successful |
| **3xx** | Redirection | You need to go somewhere else |
| **4xx** | Client Error | You (the client) made a mistake |
| **5xx** | Server Error | The server failed |

### Most Important Status Codes:

#### ✅ 2xx — Success

| Code | Name | Meaning |
|------|------|---------|
| **200** | OK | Request succeeded (general success) |
| **201** | Created | A new resource was created (after POST) |
| **204** | No Content | Success, but nothing to return (often after DELETE) |

#### 🔀 3xx — Redirection

| Code | Name | Meaning |
|------|------|---------|
| **301** | Moved Permanently | Resource moved to a new URL forever |
| **302** | Found | Resource temporarily at a different URL |
| **304** | Not Modified | Cached version is still valid |

#### ❌ 4xx — Client Errors

| Code | Name | Meaning |
|------|------|---------|
| **400** | Bad Request | Invalid request (e.g., malformed JSON) |
| **401** | Unauthorized | Not authenticated (need to log in) |
| **403** | Forbidden | Authenticated but not allowed |
| **404** | Not Found | Resource doesn't exist |
| **405** | Method Not Allowed | HTTP method not supported for this URL |
| **409** | Conflict | Conflicts with current state (e.g., duplicate) |
| **422** | Unprocessable Entity | Request is valid but logically wrong |
| **429** | Too Many Requests | Rate limited — slow down! |

#### 💥 5xx — Server Errors

| Code | Name | Meaning |
|------|------|---------|
| **500** | Internal Server Error | Generic server error |
| **502** | Bad Gateway | Server got an invalid response from upstream |
| **503** | Service Unavailable | Server is temporarily down |
| **504** | Gateway Timeout | Upstream server didn't respond in time |

### 🧠 QA Perspective: What to check

As a QA engineer, pay attention to:
- **Does the API return the correct status code?** (e.g., 201 for creation, not 200)
- **Are error responses consistent?** (e.g., always return a JSON error object)
- **Does 401 vs 403 work correctly?** (unauthorized vs forbidden are different!)
- **What happens on 5xx?** — Does the client handle server errors gracefully?

---

## 7. URL Structure

Understanding URLs is essential for API testing:

```
https://api.example.com:443/api/v2/users?role=qa&active=true#section1
\___/   \______________/ \_/ \_____________/ \___________________/ \______/
  |           |           |        |                  |               |
scheme      host        port     path          query params       fragment
```

| Part | Description | Example |
|------|-------------|---------|
| **Scheme** | Protocol | `https` |
| **Host** | Server address | `api.example.com` |
| **Port** | Network port (optional) | `443` (default for HTTPS) |
| **Path** | Resource location | `/api/v2/users` |
| **Query Parameters** | Filters/options | `?role=qa&active=true` |
| **Fragment** | Client-side anchor | `#section1` |

### Query Parameters

Query parameters are used to filter, sort, or paginate data:

```
GET /api/users?role=qa&active=true&page=1&limit=20&sort=name
```

| Parameter | Purpose |
|-----------|---------|
| `role=qa` | Filter by role |
| `active=true` | Filter by status |
| `page=1` | Pagination — which page |
| `limit=20` | Pagination — how many per page |
| `sort=name` | Sort by name |

---

## 📝 Exercises

### Exercise 1: Identify the Parts

Look at this HTTP request and identify all parts:

```http
GET /api/products?category=electronics&inStock=true HTTP/1.1
Host: shop.example.com
Accept: application/json
Authorization: Bearer abc123token
```

**Questions:**
1. What is the HTTP method?
2. What is the path?
3. What are the query parameters?
4. What response format does the client expect?
5. Is this request authenticated?

---

### Exercise 2: Match Status Codes

Match each scenario with the correct status code:

| Scenario | Status Code |
|----------|-------------|
| User successfully logged in | ? |
| User sent invalid email format | ? |
| User tries to access admin page without admin role | ? |
| New order was created | ? |
| The server database is down | ? |
| User tries to find a product that was deleted | ? |
| User is not logged in and tries to access protected resource | ? |

<details>
<summary>Click to see answers</summary>

| Scenario | Status Code |
|----------|-------------|
| User successfully logged in | **200 OK** |
| User sent invalid email format | **400 Bad Request** (or **422 Unprocessable Entity**) |
| User tries to access admin page without admin role | **403 Forbidden** |
| New order was created | **201 Created** |
| The server database is down | **500 Internal Server Error** (or **503 Service Unavailable**) |
| User tries to find a product that was deleted | **404 Not Found** |
| User is not logged in and tries to access protected resource | **401 Unauthorized** |

</details>

---

### Exercise 3: Construct HTTP Requests

Write the HTTP requests for each scenario:

1. **Get a list of all orders for user 42**
2. **Create a new product** with name "Laptop" and price 999.99
3. **Update only the price** of product 7 to 899.99
4. **Delete** order 15
5. **Get all products** that cost less than 500, sorted by price, page 2, 10 items per page

<details>
<summary>Click to see answers</summary>

```http
# 1. Get orders for user 42
GET /api/users/42/orders HTTP/1.1
Host: api.example.com
Accept: application/json

# 2. Create a new product
POST /api/products HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "name": "Laptop",
  "price": 999.99
}

# 3. Update only the price (PATCH, not PUT!)
PATCH /api/products/7 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "price": 899.99
}

# 4. Delete order 15
DELETE /api/orders/15 HTTP/1.1
Host: api.example.com

# 5. Get filtered, sorted, paginated products
GET /api/products?maxPrice=500&sort=price&page=2&limit=10 HTTP/1.1
Host: api.example.com
Accept: application/json
```

</details>

---

### Exercise 4: Hands-on with `curl`

Use the command line to make real HTTP requests:

```bash
# 1. Simple GET request — see a response from a public API
curl -v https://httpbin.org/get

# 2. POST request with JSON body
curl -v -X POST https://httpbin.org/post \
  -H "Content-Type: application/json" \
  -d '{"name": "Nastya", "skill": "QA"}'

# 3. See only response headers
curl -I https://httpbin.org/get

# 4. PUT request
curl -v -X PUT https://httpbin.org/put \
  -H "Content-Type: application/json" \
  -d '{"name": "Nastya", "skill": "Senior QA"}'

# 5. DELETE request
curl -v -X DELETE https://httpbin.org/delete
```

**The `-v` flag** (verbose) shows you the full request and response, including headers. This is very useful for learning!

**Study the output and identify:**
- Request headers sent by curl
- Response status code
- Response headers
- Response body

---

### Exercise 5: Real-World QA Scenarios

Think about these situations and answer:

1. You're testing a login API. The user enters wrong credentials. What status code should the API return? Why?
2. You send a POST request to create a user, but the email already exists. What status code would you expect?
3. You make a GET request and receive a 200 status, but the body is empty. Is this a bug? Why or why not?
4. The frontend shows a "Something went wrong" error. You check the network tab and see a 502 status code. What does this tell you about where the problem is?
5. You notice the API returns 200 for everything, even errors (with error details in the body). Is this a good practice? What problems could this cause?

---

## 📚 Additional Resources

- [MDN Web Docs — HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP) — The best reference for HTTP
- [HTTP Status Codes Reference](https://httpstatuses.com/) — Quick status code lookup
- [httpbin.org](https://httpbin.org/) — Free service for testing HTTP requests
- [curl documentation](https://curl.se/docs/manpage.html) — Master the curl command

---

## ✅ Self-Check

After completing this course, you should be able to:

- [ ] Explain what HTTP is and how request-response works

<details>
<summary>Answer</summary>

HTTP (HyperText Transfer Protocol) is the foundation of data communication on the web. It works using a **request-response model**: a client (browser or app) sends an **HTTP request** to a server, the server processes it and sends back an **HTTP response**. Key characteristics: HTTP is **stateless** (each request is independent), **text-based** (human-readable in HTTP/1.1), and the client always initiates communication. HTTPS adds TLS/SSL encryption on top of HTTP to protect data in transit.

</details>

- [ ] Name all major HTTP methods and when to use each

<details>
<summary>Answer</summary>

- **GET** — Retrieve data (e.g., get a user profile). Safe & idempotent.
- **POST** — Create new data (e.g., create a new user). Not safe, not idempotent.
- **PUT** — Replace a resource completely (e.g., update all fields of a user). Idempotent.
- **PATCH** — Partially update a resource (e.g., change only the user's email). Not idempotent.
- **DELETE** — Remove a resource (e.g., delete a user). Idempotent.
- **HEAD** — Same as GET but returns only headers, no body. Safe & idempotent.
- **OPTIONS** — Check what methods are available for a resource. Safe & idempotent.

</details>

- [ ] Distinguish between PUT and PATCH

<details>
<summary>Answer</summary>

**PUT** replaces the **entire** resource — you must send all fields, even those that haven't changed. If you omit a field, it may be set to null. **PATCH** updates only the **specific fields** you include in the request body, leaving everything else unchanged. Use PUT when you want to overwrite the whole object; use PATCH when you want to modify just one or a few fields.

</details>

- [ ] Read and understand HTTP headers

<details>
<summary>Answer</summary>

HTTP headers are **key-value pairs** that carry metadata about the request or response. Common request headers include: `Content-Type` (format of the body, e.g., `application/json`), `Authorization` (authentication credentials, e.g., `Bearer <token>`), `Accept` (what format the client wants back), `Host` (the server's domain), and `User-Agent` (info about the client). Common response headers include: `Content-Type`, `Content-Length`, `Set-Cookie`, `Location` (for redirects), and `Access-Control-Allow-Origin` (CORS).

</details>

- [ ] Know the main status codes by heart (200, 201, 400, 401, 403, 404, 500)

<details>
<summary>Answer</summary>

- **200 OK** — Request succeeded (general success)
- **201 Created** — A new resource was successfully created (typically after POST)
- **400 Bad Request** — The client sent an invalid request (e.g., malformed JSON, missing required fields)
- **401 Unauthorized** — The client is not authenticated (needs to log in)
- **403 Forbidden** — The client is authenticated but doesn't have permission to access the resource
- **404 Not Found** — The requested resource doesn't exist
- **500 Internal Server Error** — Something went wrong on the server side

</details>

- [ ] Explain the difference between 401 and 403

<details>
<summary>Answer</summary>

**401 Unauthorized** means the client is **not authenticated** — the server doesn't know who you are (no token, expired token, or invalid credentials). The solution is to log in or provide valid credentials. **403 Forbidden** means the client **is authenticated** (the server knows who you are) but you **don't have permission** to access the requested resource. For example, a regular user trying to access an admin-only endpoint would get 403.

</details>

- [ ] Construct HTTP requests for common API operations

<details>
<summary>Answer</summary>

Examples of common API requests:
- **List resources**: `GET /api/users` with `Accept: application/json`
- **Get single resource**: `GET /api/users/42`
- **Create resource**: `POST /api/users` with `Content-Type: application/json` and a JSON body containing the new resource's data
- **Update resource**: `PUT /api/users/42` (full replacement) or `PATCH /api/users/42` (partial update) with JSON body
- **Delete resource**: `DELETE /api/users/42`
- **Filter/paginate**: `GET /api/users?role=qa&page=1&limit=20&sort=name`

</details>

- [ ] Use `curl` to make HTTP requests from the terminal

<details>
<summary>Answer</summary>

Basic curl examples:
- **GET**: `curl -v https://api.example.com/users`
- **POST with JSON**: `curl -v -X POST https://api.example.com/users -H "Content-Type: application/json" -d '{"name":"Nastya"}'`
- **With auth**: `curl -v -H "Authorization: Bearer <token>" https://api.example.com/users`
- **Headers only**: `curl -I https://api.example.com/users`
- **PUT**: `curl -v -X PUT https://api.example.com/users/42 -H "Content-Type: application/json" -d '{"name":"Updated"}'`
- **DELETE**: `curl -v -X DELETE https://api.example.com/users/42`

The `-v` (verbose) flag shows full request and response details including headers.

</details>

- [ ] Parse a URL into its components (scheme, host, path, query params)

<details>
<summary>Answer</summary>

Given `https://api.example.com:443/api/v2/users?role=qa&active=true#section1`:
- **Scheme**: `https` — the protocol
- **Host**: `api.example.com` — the server address
- **Port**: `443` — network port (default for HTTPS)
- **Path**: `/api/v2/users` — the resource location
- **Query Parameters**: `role=qa&active=true` — filters/options passed as key-value pairs after `?`, separated by `&`
- **Fragment**: `section1` — client-side anchor (not sent to server)

</details>
