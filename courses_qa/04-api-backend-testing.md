# Course 04: API & Backend Testing

## 📖 What You Will Learn

- REST API testing fundamentals and contract validation
- HTTP status code verification and what to expect
- Payload schema validation and error handling patterns
- Microservices testing approaches
- Using API tools: Postman, REST Assured, SoapUI
- Backend test data validation using SQL queries

---

## 1. REST API Testing and Contract Validation

### What is API Testing?

**API testing** verifies that application programming interfaces work correctly — they accept the right inputs, return the right outputs, handle errors gracefully, and perform within acceptable time limits.

### Why API Testing Matters:

```
UI Tests (slow, brittle)    →  Test THROUGH the UI
API Tests (fast, stable)    →  Test the LOGIC directly
Unit Tests (fastest)        →  Test individual FUNCTIONS
```

API tests sit in the **middle layer** of the test pyramid — they're faster and more stable than UI tests while covering business logic and integrations.

### REST API Fundamentals

REST (Representational State Transfer) uses standard HTTP methods on **resources**:

```
Resource: /api/users

GET    /api/users          → List all users
GET    /api/users/42       → Get user #42
POST   /api/users          → Create a new user
PUT    /api/users/42       → Replace user #42
PATCH  /api/users/42       → Partially update user #42
DELETE /api/users/42       → Delete user #42
```

### API Contract

An **API contract** defines the agreement between the API provider and consumer:

| Contract Element | Description | Example |
|-----------------|-------------|---------|
| **Endpoint URL** | Where to send the request | `POST /api/users` |
| **Request Method** | HTTP method | `POST` |
| **Request Headers** | Required headers | `Content-Type: application/json` |
| **Request Body** | Expected payload structure | `{ "name": string, "email": string }` |
| **Response Status** | Expected status code | `201 Created` |
| **Response Body** | Response payload structure | `{ "id": number, "name": string, ... }` |
| **Error Responses** | Error format and codes | `{ "error": string, "code": string }` |

### Contract Testing Example:

```typescript
// Verify the API contract for creating a user
test('POST /api/users should create user and return 201', async ({ request }) => {
  const response = await request.post('/api/users', {
    data: {
      name: 'Nastya',
      email: 'nastya@example.com',
      role: 'QA Engineer'
    }
  });

  // Verify status code
  expect(response.status()).toBe(201);

  // Verify response structure (contract)
  const body = await response.json();
  expect(body).toHaveProperty('id');
  expect(body).toHaveProperty('name', 'Nastya');
  expect(body).toHaveProperty('email', 'nastya@example.com');
  expect(body).toHaveProperty('role', 'QA Engineer');
  expect(body).toHaveProperty('createdAt');
  
  // Verify types
  expect(typeof body.id).toBe('number');
  expect(typeof body.createdAt).toBe('string');
  
  // Verify Location header
  expect(response.headers()['location']).toContain(`/api/users/${body.id}`);
});
```

### What to Test in API Testing:

| Category | What to Check |
|----------|-------------|
| **Functional** | Correct response for valid inputs, CRUD operations work |
| **Negative** | Invalid inputs, missing required fields, wrong data types |
| **Authentication** | Unauthorized access returns 401, forbidden returns 403 |
| **Authorization** | Users can only access their own resources |
| **Validation** | Input validation rules are enforced |
| **Idempotency** | PUT/DELETE produce same result on repeat calls |
| **Pagination** | Correct page size, total count, next/previous links |
| **Filtering/Sorting** | Query parameters work correctly |
| **Rate Limiting** | 429 returned when limit exceeded |
| **Performance** | Response time within SLA |

---

## 2. HTTP Status Code Verification

### Status Code Testing Strategy

Every API endpoint should be tested for **both success and error** status codes.

### Common Status Codes to Verify:

#### Success Cases (2xx):

```typescript
// 200 OK — GET successful
test('GET /api/users returns 200 with user list', async ({ request }) => {
  const response = await request.get('/api/users');
  expect(response.status()).toBe(200);
  const users = await response.json();
  expect(Array.isArray(users)).toBe(true);
});

// 201 Created — POST successful
test('POST /api/users returns 201 for new user', async ({ request }) => {
  const response = await request.post('/api/users', {
    data: { name: 'Test User', email: 'test@example.com' }
  });
  expect(response.status()).toBe(201);
});

// 204 No Content — DELETE successful
test('DELETE /api/users/42 returns 204', async ({ request }) => {
  const response = await request.delete('/api/users/42');
  expect(response.status()).toBe(204);
  expect(await response.text()).toBe('');
});
```

#### Client Error Cases (4xx):

```typescript
// 400 Bad Request — Invalid input
test('POST /api/users returns 400 for malformed JSON', async ({ request }) => {
  const response = await request.post('/api/users', {
    data: { email: 'not-an-email' } // missing required 'name', invalid email
  });
  expect(response.status()).toBe(400);
  const error = await response.json();
  expect(error).toHaveProperty('errors');
});

// 401 Unauthorized — No/invalid auth token
test('GET /api/admin returns 401 without auth token', async ({ request }) => {
  const response = await request.get('/api/admin');
  expect(response.status()).toBe(401);
});

// 403 Forbidden — Authenticated but not permitted
test('DELETE /api/users/1 returns 403 for non-admin', async ({ request }) => {
  const response = await request.delete('/api/users/1', {
    headers: { 'Authorization': `Bearer ${regularUserToken}` }
  });
  expect(response.status()).toBe(403);
});

// 404 Not Found — Resource doesn't exist
test('GET /api/users/999999 returns 404', async ({ request }) => {
  const response = await request.get('/api/users/999999');
  expect(response.status()).toBe(404);
});

// 409 Conflict — Duplicate resource
test('POST /api/users returns 409 for duplicate email', async ({ request }) => {
  // Create user first
  await request.post('/api/users', {
    data: { name: 'User', email: 'existing@example.com' }
  });
  // Try to create again with same email
  const response = await request.post('/api/users', {
    data: { name: 'Another User', email: 'existing@example.com' }
  });
  expect(response.status()).toBe(409);
});

// 422 Unprocessable Entity — Valid JSON but logic error
test('POST /api/orders returns 422 for zero quantity', async ({ request }) => {
  const response = await request.post('/api/orders', {
    data: { productId: 1, quantity: 0 }
  });
  expect(response.status()).toBe(422);
});
```

#### Server Error Cases (5xx):

```typescript
// Verify graceful error handling
test('API returns structured error response for server errors', async ({ request }) => {
  // This test assumes you can trigger a 500 (e.g., via a test endpoint)
  const response = await request.get('/api/test/trigger-error');
  expect(response.status()).toBe(500);
  const error = await response.json();
  expect(error).toHaveProperty('message');
  expect(error).not.toHaveProperty('stackTrace'); // Should not expose internals!
});
```

### Status Code Verification Checklist:

| Endpoint | 200 | 201 | 204 | 400 | 401 | 403 | 404 | 409 | 422 | 500 |
|----------|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|
| GET /users | ✅ | — | — | ✅ | ✅ | ✅ | — | — | — | ✅ |
| GET /users/:id | ✅ | — | — | ✅ | ✅ | ✅ | ✅ | — | — | ✅ |
| POST /users | — | ✅ | — | ✅ | ✅ | ✅ | — | ✅ | ✅ | ✅ |
| PUT /users/:id | ✅ | — | — | ✅ | ✅ | ✅ | ✅ | — | ✅ | ✅ |
| DELETE /users/:id | — | — | ✅ | — | ✅ | ✅ | ✅ | — | — | ✅ |

---

## 3. Payload Schema Validation and Error Handling

### JSON Schema Validation

Use JSON Schema to validate response structure automatically:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["id", "name", "email", "createdAt"],
  "properties": {
    "id": { "type": "integer", "minimum": 1 },
    "name": { "type": "string", "minLength": 1 },
    "email": { "type": "string", "format": "email" },
    "role": { "type": "string", "enum": ["admin", "user", "QA Engineer"] },
    "createdAt": { "type": "string", "format": "date-time" },
    "updatedAt": { "type": ["string", "null"], "format": "date-time" }
  },
  "additionalProperties": false
}
```

### Schema Validation in Tests:

```typescript
import Ajv from 'ajv';
import addFormats from 'ajv-formats';

const ajv = new Ajv();
addFormats(ajv);

const userSchema = {
  type: 'object',
  required: ['id', 'name', 'email', 'createdAt'],
  properties: {
    id: { type: 'integer', minimum: 1 },
    name: { type: 'string', minLength: 1 },
    email: { type: 'string', format: 'email' },
    createdAt: { type: 'string', format: 'date-time' }
  },
  additionalProperties: false
};

test('GET /api/users/:id should match user schema', async ({ request }) => {
  const response = await request.get('/api/users/1');
  const body = await response.json();
  
  const validate = ajv.compile(userSchema);
  const valid = validate(body);
  
  if (!valid) {
    console.log('Schema validation errors:', validate.errors);
  }
  expect(valid).toBe(true);
});
```

### Error Response Validation

APIs should return **consistent** error responses:

```json
// Standard error response format
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address"
      },
      {
        "field": "name",
        "message": "Name is required"
      }
    ]
  }
}
```

```typescript
// Verify error response structure
test('POST /api/users with invalid data returns structured error', async ({ request }) => {
  const response = await request.post('/api/users', {
    data: { email: 'invalid', name: '' }
  });
  
  expect(response.status()).toBe(400);
  const body = await response.json();
  
  // Verify error structure
  expect(body).toHaveProperty('error');
  expect(body.error).toHaveProperty('code');
  expect(body.error).toHaveProperty('message');
  expect(body.error).toHaveProperty('details');
  expect(Array.isArray(body.error.details)).toBe(true);
  
  // Verify specific field errors
  const fieldErrors = body.error.details.map(d => d.field);
  expect(fieldErrors).toContain('email');
  expect(fieldErrors).toContain('name');
});
```

### Input Validation Testing Checklist:

| Validation | Test Cases |
|-----------|-----------|
| **Required fields** | Send request without each required field |
| **Field types** | Send string where number expected, null where required |
| **String length** | Empty string, max length + 1, SQL injection attempt |
| **Number ranges** | Negative values, zero, max value + 1, decimal when integer expected |
| **Email format** | No @, no domain, special characters |
| **Date format** | Invalid format, past dates when future required, Feb 30 |
| **Enum values** | Value not in allowed list |
| **Array limits** | Empty array, too many items |
| **Nested objects** | Missing nested required fields, extra properties |

---

## 4. Microservices Testing Approaches

### Microservices Architecture

```
         ┌───────────┐
         │  API       │
         │  Gateway   │
         └─────┬─────┘
               │
     ┌─────────┼─────────┐
     │         │         │
┌────▼───┐ ┌──▼────┐ ┌──▼────┐
│ User   │ │ Order │ │ Payment│
│Service │ │Service│ │Service │
└────┬───┘ └──┬────┘ └──┬────┘
     │        │         │
┌────▼───┐ ┌──▼────┐ ┌──▼────┐
│User DB │ │OrderDB│ │PayDB  │
└────────┘ └───────┘ └───────┘
```

### Testing Levels in Microservices:

| Level | What It Tests | Example |
|-------|-------------|---------|
| **Unit** | Single service logic | User validation rules |
| **Integration** | Service + its database | User service creates record in DB |
| **Contract** | API agreement between services | Order service expects User service to return { id, name } |
| **Component** | One service end-to-end (mocking others) | Order service with mocked Payment service |
| **End-to-End** | Full flow across all services | User → Create Order → Process Payment → Confirm |

### Contract Testing with Pact:

Contract testing ensures services agree on their API contracts without needing to test them together.

```
Consumer (Order Service)          Provider (User Service)
         │                                │
         │  1. Define expectations         │
         │  "I expect GET /users/42        │
         │   to return { id, name }"       │
         │                                │
         │  2. Generate Pact file ──────> │
         │                                │
         │                    3. Verify   │
         │                    against     │
         │                    real API    │
         │                                │
         │  4. If verified → Contract     │
         │     is satisfied ✅             │
```

### Testing Strategies for Microservices:

| Strategy | Description | When to Use |
|----------|-------------|-------------|
| **Service Isolation** | Test each service independently with mocked dependencies | During development |
| **Contract Testing** | Verify API agreements between services | CI pipeline |
| **Integration Testing** | Test actual service-to-service communication | Dedicated test environment |
| **End-to-End Testing** | Test complete business flows | Pre-release |
| **Chaos Testing** | Kill services, inject latency, simulate failures | Resilience validation |

### Common Microservices Testing Challenges:

| Challenge | Solution |
|-----------|----------|
| **Service dependencies** | Mock external services, use contract tests |
| **Data consistency** | Test saga patterns, eventual consistency |
| **Test environment complexity** | Docker Compose for local, Kubernetes for CI |
| **Network failures** | Test timeouts, retries, circuit breakers |
| **Service discovery** | Verify health endpoints, readiness probes |
| **Distributed tracing** | Verify correlation IDs propagate across services |

---

## 5. API Testing Tools

### Postman

**Postman** is the most popular API testing tool for manual and automated API testing.

#### Postman Test Scripts:

```javascript
// Tests tab in Postman

// Verify status code
pm.test("Status code is 200", function() {
    pm.response.to.have.status(200);
});

// Verify response time
pm.test("Response time is less than 500ms", function() {
    pm.expect(pm.response.responseTime).to.be.below(500);
});

// Verify response body
pm.test("Response has correct structure", function() {
    const jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('id');
    pm.expect(jsonData).to.have.property('name');
    pm.expect(jsonData.name).to.be.a('string');
});

// Verify headers
pm.test("Content-Type is JSON", function() {
    pm.response.to.have.header("Content-Type", "application/json");
});

// Store values for next request
pm.test("Store user ID for subsequent tests", function() {
    const jsonData = pm.response.json();
    pm.environment.set("userId", jsonData.id);
    pm.environment.set("authToken", jsonData.token);
});

// JSON Schema validation
const schema = {
    type: "object",
    required: ["id", "name", "email"],
    properties: {
        id: { type: "number" },
        name: { type: "string" },
        email: { type: "string" }
    }
};

pm.test("Response matches schema", function() {
    pm.response.to.have.jsonSchema(schema);
});
```

#### Postman Collections for Automation:

```bash
# Run collection from command line with Newman
newman run my-collection.json \
  --environment staging.json \
  --reporters cli,junit \
  --reporter-junit-export results.xml
```

### REST Assured (Java)

**REST Assured** is a Java library for API testing, popular in enterprise environments.

```java
import static io.restassured.RestAssured.*;
import static org.hamcrest.Matchers.*;

public class UserApiTest {

    @Test
    public void getUserReturns200WithUserData() {
        given()
            .baseUri("https://api.example.com")
            .header("Authorization", "Bearer " + token)
        .when()
            .get("/api/users/42")
        .then()
            .statusCode(200)
            .contentType(ContentType.JSON)
            .body("id", equalTo(42))
            .body("name", notNullValue())
            .body("email", matchesPattern(".*@.*\\..*"))
            .time(lessThan(2000L)); // Response under 2 seconds
    }

    @Test
    public void createUserReturns201() {
        String requestBody = """
            {
                "name": "Nastya",
                "email": "nastya@example.com",
                "role": "QA Engineer"
            }
            """;

        given()
            .baseUri("https://api.example.com")
            .header("Authorization", "Bearer " + adminToken)
            .contentType(ContentType.JSON)
            .body(requestBody)
        .when()
            .post("/api/users")
        .then()
            .statusCode(201)
            .body("id", greaterThan(0))
            .body("name", equalTo("Nastya"))
            .header("Location", containsString("/api/users/"));
    }

    @Test
    public void deleteNonexistentUserReturns404() {
        given()
            .baseUri("https://api.example.com")
            .header("Authorization", "Bearer " + adminToken)
        .when()
            .delete("/api/users/999999")
        .then()
            .statusCode(404)
            .body("error.code", equalTo("NOT_FOUND"));
    }
}
```

### SoapUI

**SoapUI** is used for testing SOAP and REST APIs, especially in enterprise/legacy systems.

| Feature | SoapUI Free | SoapUI Pro |
|---------|------------|------------|
| SOAP Testing | ✅ | ✅ |
| REST Testing | ✅ | ✅ |
| Assertions | Basic | Advanced |
| Data-Driven | Manual | Excel/DB source |
| Reporting | Basic | Detailed |
| CI Integration | CLI runner | Full integration |

### Tool Comparison:

| Feature | Postman | REST Assured | SoapUI | Playwright API |
|---------|---------|-------------|--------|---------------|
| **Language** | JavaScript | Java | Groovy/Java | JS/TS/Python |
| **GUI** | ✅ Yes | ❌ Code only | ✅ Yes | ❌ Code only |
| **SOAP** | Limited | Limited | ✅ Full | ❌ No |
| **REST** | ✅ Full | ✅ Full | ✅ Full | ✅ Full |
| **CI/CD** | Newman CLI | JUnit/Maven | CLI runner | Native |
| **Collaboration** | Cloud sharing | Git (code) | Project files | Git (code) |
| **Best For** | Manual + automation | Java teams | SOAP/Enterprise | Full-stack testing |

---

## 6. Backend Test Data Validation Using SQL

### Why SQL Matters for QA

API responses show what the **frontend sees**, but SQL shows what **actually happened** in the database.

### Common SQL Queries for QA:

```sql
-- Verify a user was created correctly
SELECT id, name, email, role, created_at
FROM users
WHERE email = 'nastya@example.com';

-- Verify no duplicate entries
SELECT email, COUNT(*) as count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- Verify order total calculation
SELECT o.id, o.total_amount,
       SUM(oi.quantity * oi.unit_price) as calculated_total
FROM orders o
JOIN order_items oi ON o.id = oi.order_id
WHERE o.id = 42
GROUP BY o.id, o.total_amount
HAVING o.total_amount != SUM(oi.quantity * oi.unit_price);

-- Verify data after API DELETE (soft delete check)
SELECT id, email, deleted_at, is_active
FROM users
WHERE id = 42;

-- Check data integrity across tables
SELECT o.id as order_id, o.user_id, u.id as user_exists
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
WHERE u.id IS NULL;  -- Orders with non-existent users (orphaned records)

-- Verify payment status after transaction
SELECT p.id, p.order_id, p.amount, p.status, p.gateway_response
FROM payments p
WHERE p.order_id = 42
ORDER BY p.created_at DESC
LIMIT 1;
```

### API + SQL Verification Pattern:

```typescript
test('POST /api/users creates record in database', async ({ request }) => {
  // 1. Create user via API
  const response = await request.post('/api/users', {
    data: { name: 'Nastya', email: 'nastya-db-test@example.com' }
  });
  expect(response.status()).toBe(201);
  const apiUser = await response.json();
  
  // 2. Verify in database
  const dbResult = await db.query(
    'SELECT * FROM users WHERE id = $1',
    [apiUser.id]
  );
  
  expect(dbResult.rows).toHaveLength(1);
  const dbUser = dbResult.rows[0];
  expect(dbUser.name).toBe('Nastya');
  expect(dbUser.email).toBe('nastya-db-test@example.com');
  expect(dbUser.created_at).toBeDefined();
  
  // 3. Cleanup
  await db.query('DELETE FROM users WHERE id = $1', [apiUser.id]);
});
```

---

## 📝 Exercises

### Exercise 1: Design API Test Cases

For a `POST /api/orders` endpoint that creates an order, design test cases covering:
1. Happy path
2. Validation errors
3. Authentication/authorization
4. Business rule violations

<details>
<summary>Click to see answer</summary>

| # | Test Case | Method | Expected Status | Category |
|---|-----------|--------|----------------|----------|
| 1 | Create order with valid items | POST | 201 | Happy path |
| 2 | Create order with multiple items | POST | 201 | Happy path |
| 3 | Create order with missing productId | POST | 400 | Validation |
| 4 | Create order with quantity = 0 | POST | 422 | Validation |
| 5 | Create order with negative quantity | POST | 422 | Validation |
| 6 | Create order with non-existent product | POST | 404 | Business rule |
| 7 | Create order without auth token | POST | 401 | Auth |
| 8 | Create order with expired token | POST | 401 | Auth |
| 9 | Create order as banned user | POST | 403 | Authorization |
| 10 | Create order for out-of-stock item | POST | 409 | Business rule |
| 11 | Create order exceeding max quantity | POST | 422 | Business rule |
| 12 | Create order with invalid JSON body | POST | 400 | Validation |
| 13 | Create order — verify response schema | POST | 201 | Contract |
| 14 | Create order — verify DB record | POST | 201 | Data integrity |
| 15 | Create order — verify inventory decreased | POST | 201 | Side effects |

</details>

---

### Exercise 2: Write Postman Tests

Write Postman test scripts for a login endpoint that returns:
```json
{
  "token": "eyJ...",
  "user": {
    "id": 42,
    "name": "Nastya",
    "role": "admin"
  },
  "expiresIn": 3600
}
```

<details>
<summary>Click to see answer</summary>

```javascript
// Status verification
pm.test("Status code is 200", function() {
    pm.response.to.have.status(200);
});

// Performance
pm.test("Response time under 1 second", function() {
    pm.expect(pm.response.responseTime).to.be.below(1000);
});

// Token validation
pm.test("Token is present and is a JWT", function() {
    const data = pm.response.json();
    pm.expect(data.token).to.be.a('string');
    pm.expect(data.token.split('.')).to.have.lengthOf(3); // JWT has 3 parts
});

// User object validation
pm.test("User object has correct structure", function() {
    const user = pm.response.json().user;
    pm.expect(user).to.have.property('id');
    pm.expect(user).to.have.property('name');
    pm.expect(user).to.have.property('role');
    pm.expect(user.id).to.be.a('number');
    pm.expect(user.name).to.be.a('string');
});

// Expiration validation
pm.test("Token expiration is reasonable", function() {
    const data = pm.response.json();
    pm.expect(data.expiresIn).to.be.a('number');
    pm.expect(data.expiresIn).to.be.above(0);
    pm.expect(data.expiresIn).to.be.at.most(86400); // Max 24 hours
});

// Store for subsequent requests
pm.test("Store auth token", function() {
    const data = pm.response.json();
    pm.environment.set("authToken", data.token);
    pm.environment.set("userId", data.user.id);
});

// Security check
pm.test("Response does not contain sensitive data", function() {
    const text = pm.response.text();
    pm.expect(text).to.not.include('password');
    pm.expect(text).to.not.include('secret');
});

// Headers
pm.test("Correct content type", function() {
    pm.response.to.have.header("Content-Type", "application/json; charset=utf-8");
});
```

</details>

---

### Exercise 3: SQL Validation

An API endpoint `POST /api/orders` created an order. Write SQL queries to validate:
1. The order was created correctly
2. The order items are correct
3. The inventory was decreased
4. The payment record was created

<details>
<summary>Click to see answer</summary>

```sql
-- 1. Verify order was created
SELECT id, user_id, status, total_amount, created_at
FROM orders
WHERE id = 42;
-- Expected: status = 'pending', total matches expected amount

-- 2. Verify order items
SELECT oi.product_id, oi.quantity, oi.unit_price,
       (oi.quantity * oi.unit_price) as line_total,
       p.name as product_name
FROM order_items oi
JOIN products p ON oi.product_id = p.id
WHERE oi.order_id = 42;
-- Expected: correct products, quantities, and prices

-- 3. Verify inventory was decreased
-- Compare current stock with what it should be:
SELECT p.id, p.name, p.stock_quantity,
       oi.quantity as ordered_quantity
FROM products p
JOIN order_items oi ON p.id = oi.product_id
WHERE oi.order_id = 42;
-- Expected: stock_quantity = (original_stock - ordered_quantity)

-- 4. Verify payment record
SELECT p.id, p.order_id, p.amount, p.status,
       p.payment_method, p.transaction_id, p.created_at
FROM payments p
WHERE p.order_id = 42;
-- Expected: amount = order total, status = 'pending' or 'completed'

-- 5. Verify total consistency
SELECT o.total_amount as order_total,
       SUM(oi.quantity * oi.unit_price) as calculated_total,
       p.amount as payment_amount
FROM orders o
JOIN order_items oi ON o.id = oi.order_id
JOIN payments p ON o.id = p.order_id
WHERE o.id = 42
GROUP BY o.total_amount, p.amount;
-- Expected: all three amounts should match
```

</details>

---

### Exercise 4: Microservices Test Strategy

You have 3 microservices:
- **User Service** — manages user accounts
- **Order Service** — creates and manages orders
- **Notification Service** — sends emails and push notifications

Design a testing strategy that covers all testing levels.

<details>
<summary>Click to see answer</summary>

| Level | Service | Tests | Tools |
|-------|---------|-------|-------|
| **Unit** | User Service | Password hashing, email validation, role assignment | JUnit/Jest |
| **Unit** | Order Service | Price calculation, discount logic, tax computation | JUnit/Jest |
| **Unit** | Notification Service | Template rendering, recipient resolution | JUnit/Jest |
| **Integration** | User Service | CRUD operations against User DB | REST Assured + Test DB |
| **Integration** | Order Service | Order creation with DB, inventory check | REST Assured + Test DB |
| **Integration** | Notification Service | Email sending via mock SMTP | REST Assured + MailHog |
| **Contract** | Order → User | Order service expects user data format | Pact |
| **Contract** | Order → Notification | Notification service expects order event format | Pact |
| **Component** | Order Service | Full order flow with mocked User and Notification services | WireMock + REST Assured |
| **E2E** | All | User registers → Creates order → Receives confirmation email | Playwright + API |
| **Performance** | All | 100 concurrent orders, 1000 user lookups/sec | k6 |
| **Chaos** | All | Kill Notification Service — verify orders still complete | Chaos toolkit |

**Key Decisions:**
- Contract tests run in CI on every PR
- Integration tests use Docker Compose with isolated databases
- E2E tests run nightly and on release candidates
- Chaos tests run weekly

</details>

---

## 📚 Additional Resources

- [Postman Learning Center](https://learning.postman.com/) — Official Postman documentation and tutorials
- [REST Assured Documentation](https://rest-assured.io/) — Java API testing library
- [Pact Contract Testing](https://pact.io/) — Consumer-driven contract testing
- [JSON Schema](https://json-schema.org/) — JSON Schema specification and tools
- [OpenAPI/Swagger](https://swagger.io/) — API design and documentation standard

---

## ✅ Self-Check Questions

- [ ] What are the key components of an API contract?

<details>
<summary>Answer</summary>

An API contract includes: **Endpoint URL** (the path and method, e.g., `POST /api/users`), **Request headers** (required headers like Content-Type, Authorization), **Request body** (expected payload structure with field types and constraints), **Response status codes** (expected codes for success and error cases), **Response body** (structure of the returned data), and **Error responses** (consistent error format with codes and messages). The contract represents the agreement between API provider and consumer about how the API behaves.

</details>

- [ ] What HTTP status codes should you test for each CRUD operation?

<details>
<summary>Answer</summary>

**Create (POST)**: 201 (success), 400 (validation error), 401 (not authenticated), 403 (not authorized), 409 (duplicate/conflict), 422 (business logic error). **Read (GET)**: 200 (success), 401, 403, 404 (not found). **Update (PUT/PATCH)**: 200 (success), 400, 401, 403, 404, 409, 422. **Delete (DELETE)**: 204 (success, no content), 401, 403, 404. All operations should also handle 500 (server error) gracefully with structured error responses.

</details>

- [ ] How do you validate API response schemas?

<details>
<summary>Answer</summary>

Use **JSON Schema** validation to automatically verify response structure. Define a schema with required fields, data types, formats (email, date-time), enums, min/max values, and whether additional properties are allowed. In tests, use libraries like **Ajv** (JavaScript), **jsonschema** (Python), or built-in Postman schema validation to compare responses against the schema. This catches contract violations such as missing fields, wrong types, or unexpected properties without writing individual assertions for each field. Schema validation should be part of every API test to ensure backward compatibility.

</details>

- [ ] What are the testing levels for microservices?

<details>
<summary>Answer</summary>

Five levels: (1) **Unit** — test individual service logic in isolation (validation, calculations). (2) **Integration** — test a service with its actual database and dependencies. (3) **Contract** — verify API agreements between services using tools like Pact (consumer-driven contract testing). (4) **Component** — test one service end-to-end with mocked external services (using WireMock). (5) **End-to-End** — test complete business flows across all services in a test environment. Contract tests are particularly important because they catch integration issues without requiring all services to be running together.

</details>

- [ ] How do you use SQL to validate API behavior?

<details>
<summary>Answer</summary>

SQL validation verifies what **actually happened** in the database after an API call, beyond what the API response shows. Patterns: (1) After POST — verify the record was created with correct values (`SELECT * FROM users WHERE id = ?`). (2) After PUT/PATCH — verify fields were updated correctly. (3) After DELETE — check for soft delete (`deleted_at IS NOT NULL`) or actual removal. (4) **Data integrity** — verify related records (JOIN queries for order items, payments). (5) **Side effects** — verify inventory decreased, audit log created. (6) **No duplicates** — GROUP BY/HAVING to check for duplicates. This catches bugs where the API returns success but the database state is incorrect.

</details>

- [ ] Compare Postman, REST Assured, and Playwright for API testing.

<details>
<summary>Answer</summary>

**Postman**: GUI-based, great for manual exploration and quick automated scripts, uses JavaScript for tests, CI integration via Newman CLI. Best for teams needing a visual tool and collaboration features. **REST Assured**: Java library with a fluent BDD-style API (`given().when().then()`), integrates with JUnit/TestNG. Best for Java-based enterprise teams who want API tests in the same language as the application. **Playwright API Testing**: Built into the Playwright framework, uses the same test runner for UI and API tests. Best for teams already using Playwright for UI testing who want unified API + UI test suites. Choose based on your team's language, existing tools, and whether you need GUI exploration capabilities.

</details>