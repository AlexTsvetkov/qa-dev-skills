# Course 02: OpenAPI, Swagger & Postman

## 📖 What You Will Learn

- What an API is and why API specifications matter
- What OpenAPI specification is and how to read it
- What Swagger is and how to use Swagger UI
- What Postman is and how to make API requests with it
- How to go from API documentation to actual testing

---

## 1. What is an API?

**API (Application Programming Interface)** is a set of rules that allows different software systems to communicate with each other.

Think of it like a **restaurant menu**:
- The **menu** (API specification) tells you what dishes (endpoints) are available
- The **waiter** (HTTP) carries your order (request) to the kitchen (server) and brings back food (response)
- You don't need to know how the kitchen works — you just need to know the menu

### REST API

Most modern web APIs follow the **REST** (Representational State Transfer) style:

- Uses **HTTP methods** (GET, POST, PUT, DELETE) to perform operations
- Works with **resources** identified by URLs (e.g., `/api/users`, `/api/orders/42`)
- Uses **JSON** as the data format (usually)
- Is **stateless** — each request contains all information needed

```
Resource: /api/users

GET    /api/users       → List all users
POST   /api/users       → Create a new user
GET    /api/users/42    → Get user 42
PUT    /api/users/42    → Update user 42
DELETE /api/users/42    → Delete user 42
```

---

## 2. What is OpenAPI?

**OpenAPI Specification (OAS)** is a standard format for describing REST APIs. It's a document (usually in YAML or JSON) that describes:

- What endpoints are available
- What HTTP methods each endpoint supports
- What parameters each endpoint accepts
- What the request body should look like
- What responses to expect
- Authentication requirements

### Why it matters for QA:

- 📋 It's your **API contract** — what the API promises to do
- 🧪 You can **generate test cases** from it
- ✅ You can **validate** actual API responses against the specification
- 📝 It's always **up-to-date** documentation (when done right)

### OpenAPI document example (YAML):

```yaml
openapi: 3.0.3
info:
  title: User Management API
  description: API for managing users
  version: 1.0.0

servers:
  - url: https://api.example.com/v1
    description: Production server
  - url: https://staging-api.example.com/v1
    description: Staging server

paths:
  /users:
    get:
      summary: Get all users
      description: Returns a list of all users
      parameters:
        - name: role
          in: query
          required: false
          schema:
            type: string
            enum: [admin, user, qa]
          description: Filter users by role
        - name: page
          in: query
          required: false
          schema:
            type: integer
            default: 1
          description: Page number
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
        '401':
          description: Unauthorized
    
    post:
      summary: Create a new user
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
      responses:
        '201':
          description: User created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          description: Invalid request data
        '409':
          description: User with this email already exists

  /users/{id}:
    get:
      summary: Get user by ID
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
          description: User ID
      responses:
        '200':
          description: User found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          description: User not found

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
          example: 42
        name:
          type: string
          example: "Nastya"
        email:
          type: string
          format: email
          example: "nastya@example.com"
        role:
          type: string
          enum: [admin, user, qa]
          example: "qa"
        createdAt:
          type: string
          format: date-time
    
    CreateUserRequest:
      type: object
      required:
        - name
        - email
      properties:
        name:
          type: string
          minLength: 2
          maxLength: 100
        email:
          type: string
          format: email
        role:
          type: string
          enum: [admin, user, qa]
          default: user
```

### Reading the OpenAPI Spec — Key Sections:

| Section | What it describes |
|---------|-------------------|
| `info` | API name, version, description |
| `servers` | Base URLs for different environments |
| `paths` | All endpoints and their operations |
| `parameters` | Input data (query, path, header params) |
| `requestBody` | The body of POST/PUT/PATCH requests |
| `responses` | What the API returns (by status code) |
| `components/schemas` | Reusable data models (objects) |

### Parameter Locations:

| Location (`in`) | Example | Description |
|-----------------|---------|-------------|
| `path` | `/users/{id}` | Part of the URL path |
| `query` | `/users?role=qa` | After the `?` in the URL |
| `header` | `Authorization: Bearer token` | In the HTTP header |
| `cookie` | `session=abc123` | In a cookie |

---

## 3. What is Swagger?

**Swagger** is a set of tools built around the OpenAPI Specification:

| Tool | Purpose |
|------|---------|
| **Swagger UI** | Interactive API documentation in the browser |
| **Swagger Editor** | Write and edit OpenAPI specs online |
| **Swagger Codegen** | Generate client/server code from specs |

### Swagger UI

Swagger UI renders your OpenAPI specification as a beautiful, interactive web page where you can:

- 📖 **Read** API documentation
- 🧪 **Try out** API endpoints directly in the browser
- 📋 **See** request/response examples
- 🔒 **Authenticate** and test protected endpoints

### How to use Swagger UI:

1. Open the Swagger UI page (usually at `/swagger-ui.html` or `/api-docs`)
2. Browse the available endpoints
3. Click on an endpoint to expand it
4. Click **"Try it out"**
5. Fill in parameters and request body
6. Click **"Execute"**
7. See the response (status code, headers, body)

### 🧠 QA Tips for Swagger UI:

- **Check if Swagger matches the actual API behavior** — Swagger docs can be outdated!
- **Test edge cases** that Swagger might not show (empty strings, very long values, special characters)
- **Look at the schemas** — they tell you required fields, data types, and validation rules
- **Try different status codes** — can you trigger 400, 404, 409 responses?

---

## 4. What is Postman?

**Postman** is the most popular tool for API testing. It provides a user-friendly interface to:

- 📤 **Send HTTP requests** (GET, POST, PUT, DELETE, etc.)
- 📥 **Inspect responses** (status, headers, body)
- 📁 **Organize requests** into collections
- 🔄 **Automate tests** with test scripts
- 🌍 **Manage environments** (dev, staging, production)
- 📊 **Run collections** for automated testing

### Installing Postman

1. Go to [postman.com/downloads](https://www.postman.com/downloads/)
2. Download and install the desktop app
3. Create a free account (optional but recommended)

### Postman Interface Overview:

```
┌──────────────────────────────────────────────────────────┐
│  Collections  │  Request Tab                              │
│  ───────────  │  ┌──────────────────────────────────────┐ │
│  📁 My API    │  │ [GET ▼] [https://api.example.com/u.. │ │
│   ├─ Get Users│  │                          [Send]       │ │
│   ├─ Create   │  ├──────────────────────────────────────┤ │
│   └─ Delete   │  │ Params │ Auth │ Headers │ Body │ ... │ │
│               │  ├──────────────────────────────────────┤ │
│  📁 Another   │  │                                      │ │
│               │  │  (Request configuration area)         │ │
│               │  │                                      │ │
│  Environments │  ├──────────────────────────────────────┤ │
│  ───────────  │  │ Response                              │ │
│  🌍 Dev       │  │  Body │ Headers │ Test Results │      │ │
│  🌍 Staging   │  │  ┌────────────────────────────┐      │ │
│  🌍 Prod      │  │  │ { "id": 42, "name": ... }  │      │ │
│               │  │  └────────────────────────────┘      │ │
│               │  │  Status: 200 OK  Time: 142ms         │ │
└──────────────────────────────────────────────────────────┘
```

### Making Your First Request in Postman:

#### Step 1: Create a new request
- Click **"+"** to open a new tab
- Or click **"New" → "HTTP Request"**

#### Step 2: Configure the request

**GET Request:**
1. Select method: **GET**
2. Enter URL: `https://jsonplaceholder.typicode.com/users`
3. Click **Send**
4. See the response below

**POST Request:**
1. Select method: **POST**
2. Enter URL: `https://jsonplaceholder.typicode.com/posts`
3. Go to **Body** tab
4. Select **raw** and **JSON**
5. Enter the body:
```json
{
  "title": "My First Post",
  "body": "Hello from Postman!",
  "userId": 1
}
```
6. Click **Send**
7. Check the response — should be `201 Created`

### Request Configuration Tabs:

| Tab | Purpose |
|-----|---------|
| **Params** | Add query parameters (`?key=value`) |
| **Authorization** | Set up authentication (Bearer token, Basic Auth, etc.) |
| **Headers** | Add custom headers |
| **Body** | Set request body (for POST, PUT, PATCH) |
| **Pre-request Script** | JavaScript to run before the request |
| **Tests** | JavaScript to validate the response |
| **Settings** | Request-specific settings |

### Authentication in Postman:

Go to the **Authorization** tab and select the type:

| Auth Type | How it works |
|-----------|-------------|
| **No Auth** | No authentication |
| **API Key** | Sends a key as header or query param |
| **Bearer Token** | Sends `Authorization: Bearer <token>` header |
| **Basic Auth** | Sends username:password encoded in Base64 |
| **OAuth 2.0** | Complete OAuth flow |

### Writing Tests in Postman:

In the **Tests** tab, you can write JavaScript to validate responses:

```javascript
// Check status code
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Check response time
pm.test("Response time is less than 500ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(500);
});

// Check response body
pm.test("Response has correct structure", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData).to.be.an('array');
    pm.expect(jsonData.length).to.be.greaterThan(0);
});

// Check specific field
pm.test("First user has an email", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData[0]).to.have.property('email');
});

// Check Content-Type header
pm.test("Content-Type is JSON", function () {
    pm.response.to.have.header("Content-Type", "application/json; charset=utf-8");
});
```

---

## 5. From Swagger to Postman

You can import an OpenAPI/Swagger specification directly into Postman:

### How to import:

1. In Postman, click **"Import"** (top left)
2. Choose one of these options:
   - **File** — upload the OpenAPI YAML/JSON file
   - **Link** — paste the URL to the spec (e.g., `https://api.example.com/v3/api-docs`)
   - **Raw text** — paste the spec content
3. Postman will create a **collection** with all endpoints
4. Each endpoint becomes a ready-to-use request

### After importing:

- All endpoints are organized into folders
- Request parameters and bodies are pre-filled with examples
- You just need to set up the **environment** (base URL, auth tokens) and start testing!

---

## 📝 Exercises

### Exercise 1: Read an OpenAPI Spec

Look at the OpenAPI spec example in Section 2 and answer:

1. What is the base URL for the production server?
2. How many endpoints does this API have?
3. What parameters does `GET /users` accept?
4. What fields are required to create a user?
5. What status code does `POST /users` return when the email already exists?
6. What are the possible values for the `role` field?

<details>
<summary>Click to see answers</summary>

1. `https://api.example.com/v1`
2. 3 endpoints: `GET /users`, `POST /users`, `GET /users/{id}`
3. `role` (query, optional, enum: admin/user/qa) and `page` (query, optional, default: 1)
4. `name` and `email` are required
5. `409 Conflict`
6. `admin`, `user`, `qa`

</details>

---

### Exercise 2: Use a Public Swagger UI

1. Go to **[Petstore Swagger UI](https://petstore.swagger.io/)** — this is a demo API
2. Explore the available endpoints
3. Use "Try it out" to:
   - **GET** `/pet/findByStatus` — find pets with status "available"
   - **POST** `/pet` — create a new pet
   - **GET** `/pet/{petId}` — find the pet you just created
4. Pay attention to:
   - What status codes do you get?
   - What does the response body look like?
   - What happens if you try to GET a pet with a non-existent ID?

---

### Exercise 3: First Requests in Postman

Install Postman and complete these tasks:

**Using the free API at `https://jsonplaceholder.typicode.com`:**

1. **GET all users**: `GET https://jsonplaceholder.typicode.com/users`
   - How many users are there?
   - What fields does each user have?

2. **GET a specific user**: `GET https://jsonplaceholder.typicode.com/users/1`
   - What is the user's name?

3. **GET posts by a user**: `GET https://jsonplaceholder.typicode.com/posts?userId=1`
   - How many posts does user 1 have?

4. **Create a post**: `POST https://jsonplaceholder.typicode.com/posts`
   - Body:
   ```json
   {
     "title": "Test Post",
     "body": "This is my test post from Postman",
     "userId": 1
   }
   ```
   - What status code did you get?
   - What `id` was assigned to the new post?

5. **Update a post**: `PUT https://jsonplaceholder.typicode.com/posts/1`
   - Body:
   ```json
   {
     "title": "Updated Title",
     "body": "Updated body",
     "userId": 1
   }
   ```
   - What status code did you get?

6. **Delete a post**: `DELETE https://jsonplaceholder.typicode.com/posts/1`
   - What status code did you get?
   - What's in the response body?

---

### Exercise 4: Write Postman Tests

For the requests from Exercise 3, add tests in the **Tests** tab:

1. For **GET all users**:
```javascript
pm.test("Status is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Returns 10 users", function () {
    pm.expect(pm.response.json().length).to.eql(10);
});

pm.test("Each user has required fields", function () {
    const users = pm.response.json();
    users.forEach(user => {
        pm.expect(user).to.have.property('id');
        pm.expect(user).to.have.property('name');
        pm.expect(user).to.have.property('email');
    });
});
```

2. Write similar tests for the other requests. Test:
   - Status code is correct
   - Response body has expected structure
   - Specific field values match what you expect

---

### Exercise 5: Import Swagger into Postman

1. Go to Postman → **Import**
2. Use this URL: `https://petstore.swagger.io/v2/swagger.json`
3. Import the collection
4. Explore the generated requests
5. Set up the base URL and make a few requests
6. Compare: Is it easier to test from Swagger UI or Postman? Why?

---

## 📚 Additional Resources

- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html) — Official spec documentation
- [Swagger Editor](https://editor.swagger.io/) — Write and preview OpenAPI specs online
- [Swagger Petstore](https://petstore.swagger.io/) — Practice API with Swagger UI
- [Postman Learning Center](https://learning.postman.com/) — Official Postman tutorials
- [JSONPlaceholder](https://jsonplaceholder.typicode.com/) — Free fake API for testing

---

## ✅ Self-Check

After completing this course, you should be able to:

- [ ] Explain what REST API is and how resources/endpoints work

<details>
<summary>Answer</summary>

A REST API is a web service that follows REST (Representational State Transfer) principles. It uses **HTTP methods** (GET, POST, PUT, DELETE) to perform operations on **resources** identified by URLs. For example, `/api/users` is a resource representing all users, and `/api/users/42` is a specific user. REST APIs are **stateless** (each request is self-contained) and typically use **JSON** for data exchange. Endpoints are the specific URL + method combinations the API exposes (e.g., `GET /api/users`, `POST /api/users`).

</details>

- [ ] Read an OpenAPI specification and understand its structure

<details>
<summary>Answer</summary>

An OpenAPI spec is a YAML/JSON document describing a REST API. Key sections: **`info`** (API name, version, description), **`servers`** (base URLs for different environments), **`paths`** (all endpoints with their methods, parameters, request bodies, and responses), and **`components/schemas`** (reusable data models). Parameters can be in the **path** (`/users/{id}`), **query** (`?role=qa`), **header**, or **cookie**. The `required` field indicates mandatory parameters, and `enum` lists allowed values.

</details>

- [ ] Use Swagger UI to explore and test API endpoints

<details>
<summary>Answer</summary>

Swagger UI renders an OpenAPI spec as an interactive web page (usually at `/swagger-ui.html` or `/api-docs`). To test an endpoint: browse the available endpoints, click to expand one, click **"Try it out"**, fill in parameters and request body, click **"Execute"**, and inspect the response (status code, headers, body). As QA, always verify that Swagger documentation matches actual API behavior, test edge cases not shown in examples, and check schemas for required fields and validation rules.

</details>

- [ ] Make GET, POST, PUT, DELETE requests in Postman

<details>
<summary>Answer</summary>

In Postman: click **"+"** for a new request tab, select the HTTP method from the dropdown, enter the URL, configure headers/body as needed, and click **Send**. For GET, you typically just need the URL. For POST/PUT, go to the **Body** tab, select **raw** and **JSON**, then enter the JSON payload. The response appears below with status code, time, and body. Use the **Params** tab to add query parameters.

</details>

- [ ] Set up authentication in Postman

<details>
<summary>Answer</summary>

Go to the **Authorization** tab of a request and select the auth type. Common types: **Bearer Token** (sends `Authorization: Bearer <token>` header — paste your JWT or API token), **Basic Auth** (enter username and password, Postman encodes them in Base64), **API Key** (sends a key as a header or query parameter). You can also set auth at the collection level so all requests inherit it.

</details>

- [ ] Write basic test scripts in Postman

<details>
<summary>Answer</summary>

In the **Tests** tab, write JavaScript assertions using `pm.test()`. Examples: `pm.test("Status is 200", () => { pm.response.to.have.status(200); });` checks the status code. `pm.expect(pm.response.json()).to.have.property('id');` checks for a field. `pm.expect(pm.response.responseTime).to.be.below(500);` checks performance. `pm.response.to.have.header("Content-Type", "application/json; charset=utf-8");` checks headers.

</details>

- [ ] Import a Swagger/OpenAPI spec into Postman

<details>
<summary>Answer</summary>

In Postman, click **"Import"** (top left), then choose **File** (upload a YAML/JSON file), **Link** (paste the URL to the spec, e.g., `https://api.example.com/v3/api-docs`), or **Raw text** (paste the spec content). Postman creates a collection with all endpoints pre-configured, including parameters and example bodies. You then set up the environment (base URL, auth tokens) and start testing.

</details>

- [ ] Understand the relationship between OpenAPI spec, Swagger UI, and Postman

<details>
<summary>Answer</summary>

The **OpenAPI specification** is the single source of truth — a machine-readable document describing the API contract. **Swagger UI** is a visualization tool that renders the spec as interactive documentation in a browser, allowing quick exploration and testing. **Postman** is a full-featured API testing tool where you can import the spec, organize requests into collections, write test scripts, manage environments, and run automated test suites. The spec feeds both tools: Swagger UI for documentation/exploration, Postman for systematic testing.

</details>
