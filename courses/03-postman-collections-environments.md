# Course 03: Postman Collections & Environments

## 📖 What You Will Learn

- How to organize requests into collections
- How to use variables (collection, environment, global)
- How to create and switch between environments
- How to chain requests together
- How to run collections with the Collection Runner
- How to use pre-request scripts and test scripts together

---

## 1. What Are Collections?

A **collection** is a group of related API requests organized into folders. Think of it as a project folder for your API tests.

### Why use collections?

- 📁 **Organization** — group related requests together
- 🔄 **Reusability** — share collections with your team
- 🧪 **Automation** — run all requests automatically with the Collection Runner
- 📝 **Documentation** — collections can serve as living API documentation
- 🏗️ **Workflows** — chain requests together (e.g., login → create → verify)

### Collection structure example:

```
📁 User Management API
├── 📁 Authentication
│   ├── POST Login
│   ├── POST Register
│   └── POST Logout
├── 📁 Users
│   ├── GET All Users
│   ├── GET User by ID
│   ├── POST Create User
│   ├── PUT Update User
│   └── DELETE Delete User
└── 📁 User Orders
    ├── GET User Orders
    └── POST Create Order
```

### Creating a collection:

1. Click **"New"** → **"Collection"**
2. Give it a name (e.g., "User Management API")
3. Optionally add a description
4. Click **"Create"**

### Adding requests to a collection:

1. Create a new request (click **"+"**)
2. Configure the request (method, URL, headers, body)
3. Click **"Save"** (Ctrl+S / Cmd+S)
4. Choose the collection and folder
5. Give the request a descriptive name

### Creating folders in a collection:

1. Right-click on the collection → **"Add Folder"**
2. Name the folder (e.g., "Authentication", "Users")
3. Drag requests into folders to organize them

---

## 2. Variables in Postman

Variables are **placeholders** that store values you can reuse across requests. Instead of hardcoding URLs, tokens, and IDs, you use variables.

### Why use variables?

Without variables (❌ Bad):
```
GET https://api.staging.example.com/v1/users/42
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjo0Mn0.abc123
```

With variables (✅ Good):
```
GET {{baseUrl}}/users/{{userId}}
Authorization: Bearer {{authToken}}
```

### Variable syntax:

Variables are referenced using **double curly braces**: `{{variableName}}`

You can use them in:
- **URL**: `{{baseUrl}}/api/users`
- **Headers**: `Authorization: Bearer {{token}}`
- **Body**: `{"userId": {{userId}}}`
- **Query params**: `role={{userRole}}`
- **Tests/Scripts**: `pm.variables.get("token")`

### Variable Scopes (Priority Order):

Postman has different scopes for variables, from most specific to most general:

```
┌─────────────────────────────────────────┐
│  1. Local (highest priority)             │ ← Set in scripts, lives only during execution
│  ┌─────────────────────────────────────┐ │
│  │  2. Data                             │ │ ← From external data files (CSV/JSON)
│  │  ┌─────────────────────────────────┐ │ │
│  │  │  3. Environment                  │ │ │ ← Specific to selected environment
│  │  │  ┌─────────────────────────────┐ │ │ │
│  │  │  │  4. Collection               │ │ │ │ ← Shared across all requests in collection
│  │  │  │  ┌─────────────────────────┐ │ │ │ │
│  │  │  │  │  5. Global (lowest)      │ │ │ │ │ ← Available everywhere
│  │  │  │  └─────────────────────────┘ │ │ │ │
│  │  │  └─────────────────────────────┘ │ │ │
│  │  └─────────────────────────────────┘ │ │
│  └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

If the same variable name exists in multiple scopes, the **most specific** scope wins.

### Setting Variables:

#### In the Postman UI:
- **Collection variables**: Click on collection → **Variables** tab
- **Environment variables**: Click the **eye icon** (👁️) next to the environment dropdown
- **Global variables**: Click the **eye icon** → **Globals**

#### In Scripts (Pre-request or Tests):

```javascript
// Set variables at different scopes
pm.globals.set("apiVersion", "v1");                    // Global
pm.collectionVariables.set("baseUrl", "https://api.example.com"); // Collection
pm.environment.set("authToken", "eyJhbG...");          // Environment
pm.variables.set("tempValue", "123");                   // Local

// Get variables
const token = pm.environment.get("authToken");
const baseUrl = pm.collectionVariables.get("baseUrl");

// Unset variables
pm.environment.unset("authToken");

// Get a variable from any scope (uses priority order)
const value = pm.variables.get("someVariable");
```

---

## 3. Environments

An **environment** is a set of variables that define a context for your API testing. Typically, you have different environments for different servers.

### Common environment setup:

#### 🟢 Development Environment:
| Variable | Value |
|----------|-------|
| `baseUrl` | `http://localhost:8080/api` |
| `authToken` | `dev-token-123` |
| `dbName` | `app_dev` |

#### 🟡 Staging Environment:
| Variable | Value |
|----------|-------|
| `baseUrl` | `https://staging.example.com/api` |
| `authToken` | `staging-token-456` |
| `dbName` | `app_staging` |

#### 🔴 Production Environment:
| Variable | Value |
|----------|-------|
| `baseUrl` | `https://api.example.com/api` |
| `authToken` | `prod-token-789` |
| `dbName` | `app_prod` |

### Creating an environment:

1. Click the **environment dropdown** (top right) → **"Manage Environments"** (⚙️)
2. Click **"Add"**
3. Give it a name (e.g., "Development")
4. Add variables with their values
5. Click **"Save"**

### Switching environments:

- Use the **environment dropdown** in the top-right corner
- All `{{variableName}}` references will use the selected environment's values
- Your requests stay the same — only the variables change!

### Initial Value vs Current Value:

| Field | Purpose |
|-------|---------|
| **Initial Value** | Shared when you export/share the environment. Safe for team sharing. |
| **Current Value** | Your local value. Only on your machine. Use for secrets/tokens. |

⚠️ **Security tip**: Never put real passwords or tokens in the **Initial Value**. Use **Current Value** for sensitive data — it won't be shared when you export the environment.

---

## 4. Chaining Requests

**Request chaining** means using the output of one request as the input for another. This is essential for testing workflows like:

1. **Login** → get auth token
2. **Create a user** → get the new user's ID
3. **Get the user** → verify the user was created
4. **Delete the user** → clean up

### How to chain requests:

#### Step 1: Save a value from a response

In the **Tests** tab of the first request (e.g., Login):

```javascript
// Parse the response
const response = pm.response.json();

// Save the token to environment variables
pm.environment.set("authToken", response.token);

// Save other useful values
pm.environment.set("userId", response.user.id);

console.log("Token saved:", response.token);
```

#### Step 2: Use the saved value in the next request

In the next request (e.g., Get User), use the variable:
- **URL**: `{{baseUrl}}/users/{{userId}}`
- **Header**: `Authorization: Bearer {{authToken}}`

### Complete chaining example:

**Request 1: Login**
```
POST {{baseUrl}}/auth/login
Body: { "email": "nastya@example.com", "password": "secret123" }

Tests:
  const response = pm.response.json();
  pm.environment.set("authToken", response.token);
```

**Request 2: Create User**
```
POST {{baseUrl}}/users
Headers: Authorization: Bearer {{authToken}}
Body: { "name": "New User", "email": "new@example.com" }

Tests:
  const response = pm.response.json();
  pm.environment.set("createdUserId", response.id);
```

**Request 3: Get Created User**
```
GET {{baseUrl}}/users/{{createdUserId}}
Headers: Authorization: Bearer {{authToken}}

Tests:
  pm.test("User name matches", function () {
      const user = pm.response.json();
      pm.expect(user.name).to.eql("New User");
  });
```

**Request 4: Delete User (cleanup)**
```
DELETE {{baseUrl}}/users/{{createdUserId}}
Headers: Authorization: Bearer {{authToken}}

Tests:
  pm.test("User deleted", function () {
      pm.response.to.have.status(204);
  });
  // Clean up variables
  pm.environment.unset("createdUserId");
```

---

## 5. Pre-request Scripts

**Pre-request scripts** run JavaScript **before** the request is sent. Useful for:

- Generating dynamic data
- Setting up timestamps
- Computing authentication signatures
- Preparing test data

### Examples:

```javascript
// Generate a random email for testing
const randomEmail = "user_" + Math.random().toString(36).substring(7) + "@test.com";
pm.environment.set("randomEmail", randomEmail);

// Set current timestamp
pm.environment.set("timestamp", new Date().toISOString());

// Generate a random number
pm.environment.set("randomId", Math.floor(Math.random() * 1000));

// Compute a value based on other variables
const baseUrl = pm.environment.get("baseUrl");
const version = pm.collectionVariables.get("apiVersion");
pm.variables.set("fullUrl", baseUrl + "/" + version);
```

### Execution order:

```
1. Collection pre-request script     (runs first)
2. Folder pre-request script         (if request is in a folder)
3. Request pre-request script        (runs just before sending)
4. ──── REQUEST IS SENT ────
5. Collection test script            (runs first after response)
6. Folder test script                (if request is in a folder)
7. Request test script               (runs last)
```

---

## 6. Collection Runner

The **Collection Runner** lets you run all requests in a collection automatically, in sequence.

### How to use the Collection Runner:

1. Click on your collection
2. Click **"Run"** (▶️) button
3. Configure the run:
   - Select which requests to include
   - Set the number of **iterations**
   - Set a **delay** between requests (in milliseconds)
   - Optionally provide a **data file** (CSV/JSON)
4. Click **"Run Collection"**

### What you see in the results:

- ✅ Passed tests (green)
- ❌ Failed tests (red)
- Response times for each request
- Summary statistics

### Data-Driven Testing:

You can run the same collection with different data sets using a **CSV or JSON data file**.

**Example CSV file (`test-users.csv`):**
```csv
userName,userEmail,userRole
Alice,alice@test.com,admin
Bob,bob@test.com,user
Charlie,charlie@test.com,qa
```

**Example JSON file (`test-users.json`):**
```json
[
  {"userName": "Alice", "userEmail": "alice@test.com", "userRole": "admin"},
  {"userName": "Bob", "userEmail": "bob@test.com", "userRole": "user"},
  {"userName": "Charlie", "userEmail": "charlie@test.com", "userRole": "qa"}
]
```

**In your request body:**
```json
{
  "name": "{{userName}}",
  "email": "{{userEmail}}",
  "role": "{{userRole}}"
}
```

The runner will execute the collection **3 times** (once for each row), using different data each time.

---

## 7. Useful Postman Features

### Console (Debugging):

Open the **Postman Console** (View → Show Postman Console, or Ctrl+Alt+C / Cmd+Alt+C) to see:
- Full request details (URL, headers, body as actually sent)
- Full response details
- Console.log output from your scripts
- Network information

```javascript
// Log to Postman Console
console.log("Request URL:", pm.request.url.toString());
console.log("Response body:", pm.response.json());
console.log("Auth token:", pm.environment.get("authToken"));
```

### Dynamic Variables:

Postman provides built-in dynamic variables for generating test data:

| Variable | Description | Example Output |
|----------|-------------|----------------|
| `{{$randomFirstName}}` | Random first name | "Jessica" |
| `{{$randomLastName}}` | Random last name | "Smith" |
| `{{$randomEmail}}` | Random email | "jessica.smith@example.com" |
| `{{$randomUUID}}` | Random UUID | "6929bb52-3ab2-..." |
| `{{$timestamp}}` | Current Unix timestamp | "1614165600" |
| `{{$randomInt}}` | Random integer (0-1000) | "742" |
| `{{$randomBoolean}}` | Random boolean | "true" |
| `{{$randomPhoneNumber}}` | Random phone number | "+1-555-123-4567" |

### Sharing Collections:

You can share collections with your team:
1. **Export** — Save as JSON file, share via Git or email
2. **Team Workspace** — Share directly in Postman (requires account)
3. **Postman Link** — Generate a shareable link

### Collection Documentation:

Each collection, folder, and request can have **documentation**:
1. Click on the collection/request
2. Click the **documentation icon** (📝)
3. Write Markdown documentation
4. This documentation is visible to everyone who has the collection

---

## 📝 Exercises

### Exercise 1: Create a Collection

Create a collection called **"JSONPlaceholder Tests"** with the following structure:

```
📁 JSONPlaceholder Tests
├── 📁 Users
│   ├── GET All Users
│   ├── GET User by ID
│   └── GET User Posts
├── 📁 Posts
│   ├── GET All Posts
│   ├── GET Post by ID
│   ├── POST Create Post
│   ├── PUT Update Post
│   └── DELETE Delete Post
└── 📁 Comments
    └── GET Comments by Post
```

**Base URL**: `https://jsonplaceholder.typicode.com`

Set up a **collection variable** `baseUrl` with this value, and use `{{baseUrl}}` in all requests.

---

### Exercise 2: Create Environments

Create two environments:

**Environment 1: "JSONPlaceholder"**
| Variable | Value |
|----------|-------|
| `baseUrl` | `https://jsonplaceholder.typicode.com` |
| `testUserId` | `1` |

**Environment 2: "ReqRes"**
| Variable | Value |
|----------|-------|
| `baseUrl` | `https://reqres.in/api` |
| `testUserId` | `2` |

Now update your **"GET User by ID"** request to use:
- URL: `{{baseUrl}}/users/{{testUserId}}`

Switch between environments and verify that the request works with both APIs.

---

### Exercise 3: Chain Requests

Create a workflow in your collection:

1. **POST Create Post**
   - URL: `{{baseUrl}}/posts`
   - Body:
   ```json
   {
     "title": "Chain Test - {{$timestamp}}",
     "body": "This post was created by a chained request",
     "userId": 1
   }
   ```
   - **Tests** — save the created post's ID:
   ```javascript
   pm.test("Post created successfully", function () {
       pm.response.to.have.status(201);
   });
   
   const response = pm.response.json();
   pm.collectionVariables.set("createdPostId", response.id);
   console.log("Created post with ID:", response.id);
   ```

2. **GET Verify Post**
   - URL: `{{baseUrl}}/posts/{{createdPostId}}`
   - **Tests**:
   ```javascript
   pm.test("Post exists", function () {
       pm.response.to.have.status(200);
   });
   
   pm.test("Post has correct userId", function () {
       const post = pm.response.json();
       pm.expect(post.userId).to.eql(1);
   });
   ```

3. **PUT Update Post**
   - URL: `{{baseUrl}}/posts/{{createdPostId}}`
   - Body:
   ```json
   {
     "title": "Updated Chain Test",
     "body": "This post was updated",
     "userId": 1
   }
   ```
   - **Tests**:
   ```javascript
   pm.test("Post updated successfully", function () {
       pm.response.to.have.status(200);
       const post = pm.response.json();
       pm.expect(post.title).to.eql("Updated Chain Test");
   });
   ```

4. **DELETE Cleanup Post**
   - URL: `{{baseUrl}}/posts/{{createdPostId}}`
   - **Tests**:
   ```javascript
   pm.test("Post deleted", function () {
       pm.response.to.have.status(200);
   });
   pm.collectionVariables.unset("createdPostId");
   ```

Run the entire folder with the **Collection Runner** and verify all tests pass.

---

### Exercise 4: Data-Driven Testing

1. Create a CSV file called `test-posts.csv`:
```csv
title,body,userId
First Post,Content of first post,1
Second Post,Content of second post,2
Third Post,Content of third post,1
```

2. Create a **"POST Create Multiple Posts"** request:
   - URL: `{{baseUrl}}/posts`
   - Body:
   ```json
   {
     "title": "{{title}}",
     "body": "{{body}}",
     "userId": {{userId}}
   }
   ```
   - Tests:
   ```javascript
   pm.test("Post created with status 201", function () {
       pm.response.to.have.status(201);
   });
   
   pm.test("Title matches input", function () {
       const response = pm.response.json();
       const expectedTitle = pm.iterationData.get("title");
       pm.expect(response.title).to.eql(expectedTitle);
   });
   ```

3. Run with the Collection Runner:
   - Select only this request
   - Upload the CSV file as data
   - Set iterations to **3**
   - Run and verify all 3 iterations pass

---

### Exercise 5: Pre-request Scripts

Create a request that dynamically generates test data:

1. **POST Create Dynamic User**
   - **Pre-request Script**:
   ```javascript
   // Generate unique test data
   const timestamp = Date.now();
   const randomNum = Math.floor(Math.random() * 10000);
   
   pm.environment.set("testName", "TestUser_" + randomNum);
   pm.environment.set("testEmail", "test_" + timestamp + "@example.com");
   pm.environment.set("testJob", "QA Engineer");
   
   console.log("Generated test data:");
   console.log("  Name:", pm.environment.get("testName"));
   console.log("  Email:", pm.environment.get("testEmail"));
   ```

   - **URL**: `POST https://reqres.in/api/users`
   - **Body**:
   ```json
   {
     "name": "{{testName}}",
     "email": "{{testEmail}}",
     "job": "{{testJob}}"
   }
   ```

   - **Tests**:
   ```javascript
   pm.test("Status is 201", function () {
       pm.response.to.have.status(201);
   });
   
   pm.test("Name matches generated value", function () {
       const response = pm.response.json();
       const expectedName = pm.environment.get("testName");
       pm.expect(response.name).to.eql(expectedName);
   });
   
   pm.test("Response has an ID", function () {
       pm.expect(pm.response.json()).to.have.property('id');
   });
   ```

2. Run this request **5 times** with the Collection Runner (5 iterations) and verify each run creates a user with a unique name.

---

## 📚 Additional Resources

- [Postman Variables](https://learning.postman.com/docs/sending-requests/variables/) — Official docs on variables
- [Postman Collection Runner](https://learning.postman.com/docs/running-collections/intro-to-collection-runs/) — Running collections
- [Postman Scripting](https://learning.postman.com/docs/writing-scripts/intro-to-scripts/) — Writing test scripts
- [Postman Dynamic Variables](https://learning.postman.com/docs/writing-scripts/script-references/variables-list/) — Built-in random data generators
- [ReqRes API](https://reqres.in/) — Another free API for practicing

---

## ✅ Self-Check

After completing this course, you should be able to:

- [ ] Create and organize collections with folders
- [ ] Use variables in URLs, headers, and request bodies
- [ ] Explain the difference between global, environment, and collection variables
- [ ] Create and switch between environments (dev, staging, prod)
- [ ] Chain requests by saving response values into variables
- [ ] Write pre-request scripts to generate dynamic test data
- [ ] Run collections automatically with the Collection Runner
- [ ] Use data files (CSV/JSON) for data-driven testing
- [ ] Use the Postman Console for debugging
- [ ] Share collections with team members