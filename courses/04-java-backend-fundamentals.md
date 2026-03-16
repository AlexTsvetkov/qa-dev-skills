# Course 04: Java Backend Fundamentals

## 📖 What You Will Learn

- How a web application is structured (frontend vs backend)
- What a Java backend application is and what frameworks are used
- How an HTTP request travels through a Java backend (step by step)
- Key backend concepts: controllers, services, repositories, databases
- How to read backend logs and understand what happened
- What QA should know about the backend to be more effective

---

## 1. Frontend vs Backend

Every web application has two main parts:

```
┌─────────────┐         HTTP          ┌─────────────────┐         SQL          ┌──────────┐
│  FRONTEND   │  ←──── Request ────→  │    BACKEND       │  ←──── Query ────→  │ DATABASE │
│             │        Response       │                  │        Result       │          │
│ (Browser)   │                       │ (Java/Spring)    │                     │(PostgreSQL│
│ HTML/CSS/JS │                       │                  │                     │ MySQL...) │
└─────────────┘                       └─────────────────┘                     └──────────┘
```

| Part | What it does | Technologies |
|------|-------------|-------------|
| **Frontend** | What the user sees and interacts with | HTML, CSS, JavaScript, React, Angular |
| **Backend** | Processes requests, business logic, data storage | Java, Spring Boot, Node.js, Python |
| **Database** | Stores data permanently | PostgreSQL, MySQL, MongoDB |

### Why QA needs to understand the backend:

- 🐛 **Find bugs faster** — understand where a bug might be
- 📝 **Write better bug reports** — include backend error details
- 🔍 **Test more effectively** — know what the backend validates vs what it doesn't
- 💬 **Communicate with developers** — speak the same language
- 🧪 **API testing** — test the backend directly, without the frontend

---

## 2. Java Backend Stack

Most enterprise Java applications use these technologies:

### Core Technologies:

| Technology | Purpose |
|-----------|---------|
| **Java** | The programming language |
| **Spring Boot** | Framework for building web applications quickly |
| **Spring MVC** | Handles HTTP requests and responses |
| **Spring Data JPA** | Simplifies database operations |
| **Hibernate** | Maps Java objects to database tables (ORM) |
| **Maven / Gradle** | Build tools — manage dependencies and build the project |

### Project Structure (typical Spring Boot app):

```
my-application/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/myapp/
│   │   │       ├── MyApplication.java          ← Entry point
│   │   │       ├── controller/                  ← HTTP request handlers
│   │   │       │   └── UserController.java
│   │   │       ├── service/                     ← Business logic
│   │   │       │   └── UserService.java
│   │   │       ├── repository/                  ← Database access
│   │   │       │   └── UserRepository.java
│   │   │       ├── model/                       ← Data objects
│   │   │       │   └── User.java
│   │   │       ├── dto/                         ← Data Transfer Objects
│   │   │       │   ├── CreateUserRequest.java
│   │   │       │   └── UserResponse.java
│   │   │       └── exception/                   ← Error handling
│   │   │           └── UserNotFoundException.java
│   │   └── resources/
│   │       ├── application.properties           ← Configuration
│   │       └── application.yml                  ← Configuration (YAML format)
│   └── test/
│       └── java/                                ← Tests
├── pom.xml                                      ← Maven dependencies
└── build.gradle                                 ← Gradle dependencies (alternative)
```

---

## 3. How a Request Travels Through the Backend

Let's trace what happens when you send `GET /api/users/42`:

```
                    HTTP Request: GET /api/users/42
                              │
                              ▼
┌──────────────────────────────────────────────────┐
│  1. DISPATCHER SERVLET                            │
│     Spring's front controller — routes the        │
│     request to the right controller               │
└──────────────────┬───────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────┐
│  2. CONTROLLER (UserController)                   │
│     @GetMapping("/api/users/{id}")                │
│     - Receives the HTTP request                   │
│     - Extracts parameters (id = 42)               │
│     - Calls the service layer                     │
│     - Returns HTTP response                       │
└──────────────────┬───────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────┐
│  3. SERVICE (UserService)                         │
│     - Contains business logic                     │
│     - Validates data                              │
│     - Calls the repository to access database     │
│     - Transforms data if needed                   │
└──────────────────┬───────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────┐
│  4. REPOSITORY (UserRepository)                   │
│     - Communicates with the database              │
│     - Translates Java calls into SQL queries      │
│     - Returns data as Java objects                │
└──────────────────┬───────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────┐
│  5. DATABASE                                      │
│     SELECT * FROM users WHERE id = 42             │
│     → Returns the row from the users table        │
└──────────────────────────────────────────────────┘
```

The response travels back the same way:
```
Database → Repository → Service → Controller → HTTP Response → Client
```

---

## 4. Understanding Each Layer

### Layer 1: Controller

The **controller** is the entry point for HTTP requests. It maps URLs to Java methods.

```java
@RestController                          // This class handles HTTP requests
@RequestMapping("/api/users")            // Base URL for all methods in this class
public class UserController {

    private final UserService userService;

    // Constructor injection — Spring provides the UserService
    public UserController(UserService userService) {
        this.userService = userService;
    }

    // GET /api/users — get all users
    @GetMapping
    public List<UserResponse> getAllUsers() {
        return userService.getAllUsers();
    }

    // GET /api/users/42 — get user by ID
    @GetMapping("/{id}")
    public UserResponse getUserById(@PathVariable Long id) {
        return userService.getUserById(id);
    }

    // POST /api/users — create a new user
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)   // Returns 201 instead of 200
    public UserResponse createUser(@RequestBody CreateUserRequest request) {
        return userService.createUser(request);
    }

    // PUT /api/users/42 — update user
    @PutMapping("/{id}")
    public UserResponse updateUser(@PathVariable Long id,
                                    @RequestBody CreateUserRequest request) {
        return userService.updateUser(id, request);
    }

    // DELETE /api/users/42 — delete user
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)  // Returns 204
    public void deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
    }

    // GET /api/users?role=qa&page=1 — with query parameters
    @GetMapping
    public List<UserResponse> getUsersByRole(
            @RequestParam(required = false) String role,
            @RequestParam(defaultValue = "1") int page) {
        return userService.getUsersByRole(role, page);
    }
}
```

**Key annotations to know:**

| Annotation | Purpose |
|-----------|---------|
| `@RestController` | Marks a class as a REST controller |
| `@RequestMapping` | Sets the base URL path |
| `@GetMapping` | Handles GET requests |
| `@PostMapping` | Handles POST requests |
| `@PutMapping` | Handles PUT requests |
| `@DeleteMapping` | Handles DELETE requests |
| `@PathVariable` | Extracts value from the URL path (`/users/{id}`) |
| `@RequestParam` | Extracts query parameter (`?role=qa`) |
| `@RequestBody` | Reads the JSON body into a Java object |
| `@ResponseStatus` | Sets the HTTP status code for the response |

### Layer 2: Service

The **service** contains business logic — the "brain" of the application.

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public List<UserResponse> getAllUsers() {
        List<User> users = userRepository.findAll();
        return users.stream()
                .map(this::toResponse)    // Convert each User to UserResponse
                .collect(Collectors.toList());
    }

    public UserResponse getUserById(Long id) {
        User user = userRepository.findById(id)
                .orElseThrow(() -> new UserNotFoundException("User not found: " + id));
        return toResponse(user);
    }

    public UserResponse createUser(CreateUserRequest request) {
        // Business logic: check if email already exists
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new EmailAlreadyExistsException("Email already in use");
        }

        // Create new user
        User user = new User();
        user.setName(request.getName());
        user.setEmail(request.getEmail());
        user.setRole(request.getRole() != null ? request.getRole() : "user");
        user.setCreatedAt(LocalDateTime.now());

        // Save to database
        User savedUser = userRepository.save(user);
        return toResponse(savedUser);
    }

    public void deleteUser(Long id) {
        if (!userRepository.existsById(id)) {
            throw new UserNotFoundException("User not found: " + id);
        }
        userRepository.deleteById(id);
    }

    // Helper: convert User entity to UserResponse DTO
    private UserResponse toResponse(User user) {
        return new UserResponse(
            user.getId(),
            user.getName(),
            user.getEmail(),
            user.getRole(),
            user.getCreatedAt()
        );
    }
}
```

**What the service does:**
- ✅ Validates business rules (e.g., "email must be unique")
- ✅ Transforms data between layers
- ✅ Handles "what if" scenarios (user not found, duplicate email)
- ✅ Coordinates between different repositories if needed

### Layer 3: Repository

The **repository** talks to the database. Spring Data JPA generates most of the code automatically.

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    // Spring generates the SQL automatically from the method name!
    // SELECT * FROM users WHERE email = ?
    Optional<User> findByEmail(String email);

    // SELECT COUNT(*) FROM users WHERE email = ? > 0
    boolean existsByEmail(String email);

    // SELECT * FROM users WHERE role = ?
    List<User> findByRole(String role);

    // SELECT * FROM users WHERE name LIKE '%keyword%'
    List<User> findByNameContaining(String keyword);

    // Custom query when method names get too complex
    @Query("SELECT u FROM User u WHERE u.role = :role AND u.createdAt > :date")
    List<User> findActiveUsersByRole(@Param("role") String role,
                                      @Param("date") LocalDateTime date);
}
```

**Built-in methods from JpaRepository:**

| Method | SQL Equivalent |
|--------|----------------|
| `findAll()` | `SELECT * FROM users` |
| `findById(42)` | `SELECT * FROM users WHERE id = 42` |
| `save(user)` | `INSERT INTO users ...` or `UPDATE users ...` |
| `deleteById(42)` | `DELETE FROM users WHERE id = 42` |
| `existsById(42)` | `SELECT COUNT(*) FROM users WHERE id = 42 > 0` |
| `count()` | `SELECT COUNT(*) FROM users` |

### Layer 4: Model (Entity)

The **model** (or entity) represents a database table as a Java class.

```java
@Entity                          // This class maps to a database table
@Table(name = "users")           // Table name in the database
public class User {

    @Id                          // Primary key
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // Auto-increment
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    @Column(nullable = false, unique = true)
    private String email;

    @Column(nullable = false)
    private String role;

    @Column(name = "created_at")
    private LocalDateTime createdAt;

    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    // ... etc
}
```

**This maps to a database table:**

```sql
CREATE TABLE users (
    id          BIGINT PRIMARY KEY AUTO_INCREMENT,
    name        VARCHAR(100) NOT NULL,
    email       VARCHAR(255) NOT NULL UNIQUE,
    role        VARCHAR(255) NOT NULL,
    created_at  TIMESTAMP
);
```

### DTOs (Data Transfer Objects)

DTOs are simple objects used to transfer data between layers. They control what data is exposed to the client.

```java
// What the client sends (request body)
public class CreateUserRequest {
    private String name;      // Required
    private String email;     // Required
    private String role;      // Optional

    // Getters and setters
}

// What the client receives (response body)
public class UserResponse {
    private Long id;
    private String name;
    private String email;
    private String role;
    private LocalDateTime createdAt;

    // Constructor, getters
}
```

**Why use DTOs instead of returning the entity directly?**
- 🔒 **Security** — don't expose internal fields (like password hash)
- 📋 **Control** — choose exactly what data to return
- 🔄 **Flexibility** — change the database without breaking the API

---

## 5. Error Handling

When something goes wrong, the backend needs to return proper error responses.

### Exception Handler:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    // Handle "User not found" → 404
    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleUserNotFound(UserNotFoundException ex) {
        return new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            "NOT_FOUND",
            ex.getMessage()
        );
    }

    // Handle "Email already exists" → 409
    @ExceptionHandler(EmailAlreadyExistsException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public ErrorResponse handleDuplicateEmail(EmailAlreadyExistsException ex) {
        return new ErrorResponse(
            HttpStatus.CONFLICT.value(),
            "CONFLICT",
            ex.getMessage()
        );
    }

    // Handle validation errors → 400
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex) {
        String message = ex.getBindingResult().getFieldErrors().stream()
                .map(e -> e.getField() + ": " + e.getDefaultMessage())
                .collect(Collectors.joining(", "));
        return new ErrorResponse(400, "BAD_REQUEST", message);
    }

    // Handle everything else → 500
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneral(Exception ex) {
        return new ErrorResponse(500, "INTERNAL_ERROR", "An unexpected error occurred");
    }
}
```

### Error Response format:

```json
{
  "status": 404,
  "error": "NOT_FOUND",
  "message": "User not found: 42"
}
```

### 🧠 QA Perspective: Error handling checklist

- [ ] Does the API return correct status codes for different errors?
- [ ] Are error messages helpful but not exposing internal details?
- [ ] What happens when you send invalid JSON?
- [ ] What happens when required fields are missing?
- [ ] Does a 500 error leak stack traces or internal information?

---

## 6. Request Validation

The backend validates incoming data before processing it:

```java
public class CreateUserRequest {

    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be 2-100 characters")
    private String name;

    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    private String email;

    @Pattern(regexp = "^(admin|user|qa)$", message = "Role must be admin, user, or qa")
    private String role;
}
```

**Common validation annotations:**

| Annotation | What it validates |
|-----------|-------------------|
| `@NotBlank` | Not null, not empty, not just whitespace |
| `@NotNull` | Not null (but can be empty) |
| `@Size(min, max)` | String length or collection size |
| `@Email` | Valid email format |
| `@Min(value)` | Minimum numeric value |
| `@Max(value)` | Maximum numeric value |
| `@Pattern(regexp)` | Matches a regex pattern |
| `@Past` | Date must be in the past |
| `@Future` | Date must be in the future |

### 🧠 QA Test Ideas from Validation:

When you see `@NotBlank` on the `name` field, test with:
- `null` → should fail
- `""` (empty string) → should fail
- `"  "` (whitespace only) → should fail
- `"A"` (too short if `@Size(min=2)`) → should fail
- Valid name → should pass

---

## 7. Configuration

Backend applications have configuration files that control behavior:

### application.properties:
```properties
# Server port
server.port=8080

# Database connection
spring.datasource.url=jdbc:postgresql://localhost:5432/myapp
spring.datasource.username=postgres
spring.datasource.password=secret

# Show SQL queries in logs
spring.jpa.show-sql=true

# Logging level
logging.level.com.example=DEBUG
logging.level.org.springframework.web=INFO

# API prefix
server.servlet.context-path=/api/v1
```

### application.yml (same thing, different format):
```yaml
server:
  port: 8080
  servlet:
    context-path: /api/v1

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/myapp
    username: postgres
    password: secret
  jpa:
    show-sql: true

logging:
  level:
    com.example: DEBUG
    org.springframework.web: INFO
```

### Profiles (different configs for different environments):

```
application.properties          ← Default config
application-dev.properties      ← Development overrides
application-staging.properties  ← Staging overrides
application-prod.properties     ← Production overrides
```

Run with a specific profile:
```bash
java -jar myapp.jar --spring.profiles.active=staging
```

---

## 8. Reading Backend Logs

Logs are your **best friend** when debugging. Here's how to read them:

### Log levels (from most to least detail):

| Level | Purpose | Example |
|-------|---------|---------|
| **TRACE** | Very detailed tracing | Every SQL parameter value |
| **DEBUG** | Debugging information | Request/response details |
| **INFO** | Normal operations | "Application started", "User created" |
| **WARN** | Potential problems | "Slow query detected", "Deprecated API used" |
| **ERROR** | Something broke | "Database connection failed", "NullPointerException" |

### Example log output:

```
2024-01-15 10:30:00.123 INFO  [main] c.e.MyApplication : Starting MyApplication
2024-01-15 10:30:02.456 INFO  [main] c.e.MyApplication : Application started on port 8080

2024-01-15 10:30:15.789 DEBUG [http-nio-8080-exec-1] c.e.controller.UserController : GET /api/users/42
2024-01-15 10:30:15.790 DEBUG [http-nio-8080-exec-1] c.e.service.UserService : Finding user by ID: 42
2024-01-15 10:30:15.795 DEBUG [http-nio-8080-exec-1] org.hibernate.SQL : select u from users u where u.id=?
2024-01-15 10:30:15.798 DEBUG [http-nio-8080-exec-1] c.e.service.UserService : User found: Nastya
2024-01-15 10:30:15.800 DEBUG [http-nio-8080-exec-1] c.e.controller.UserController : Returning user: 42

2024-01-15 10:30:20.100 WARN  [http-nio-8080-exec-2] c.e.service.UserService : User not found: 999
2024-01-15 10:30:20.101 ERROR [http-nio-8080-exec-2] c.e.exception.GlobalExceptionHandler : UserNotFoundException: User not found: 999
```

### Reading the log format:

```
2024-01-15 10:30:15.789 DEBUG [http-nio-8080-exec-1] c.e.controller.UserController : GET /api/users/42
│          │              │     │                      │                               │
│          │              │     │                      │                               └─ Message
│          │              │     │                      └─ Class that logged this
│          │              │     └─ Thread (which request is being processed)
│          │              └─ Log level
│          └─ Time
└─ Date
```

### 🧠 QA Tips for Logs:

- **Ask developers** to set log level to DEBUG for your testing environment
- **Search for ERROR** lines when a request fails
- **Look for SQL queries** to understand what data the backend is fetching
- **Check thread names** to trace a specific request through the logs
- **Use timestamps** to correlate your test action with log entries

---

## 📝 Exercises

### Exercise 1: Trace the Request

Given this code, trace what happens when you send `POST /api/users` with this body:
```json
{
  "name": "Nastya",
  "email": "nastya@example.com",
  "role": "qa"
}
```

Write down each step:
1. Which controller method receives the request?
2. What does the service do?
3. What SQL query does the repository execute?
4. What does the response look like?

<details>
<summary>Click to see answer</summary>

1. `UserController.createUser()` receives the request, `@RequestBody` converts JSON to `CreateUserRequest`
2. `UserService.createUser()`:
   - Checks if email exists (`userRepository.existsByEmail("nastya@example.com")`)
   - Creates a new `User` object and sets fields
   - Saves to database (`userRepository.save(user)`)
   - Converts to `UserResponse` and returns
3. `INSERT INTO users (name, email, role, created_at) VALUES ('Nastya', 'nastya@example.com', 'qa', '2024-01-15 10:30:00')`
4. Response:
```json
{
  "id": 43,
  "name": "Nastya",
  "email": "nastya@example.com",
  "role": "qa",
  "createdAt": "2024-01-15T10:30:00"
}
```
Status: `201 Created`

</details>

---

### Exercise 2: What Goes Wrong?

For each scenario, identify:
- What error will occur?
- In which layer (controller, service, repository)?
- What status code should the client receive?

**Scenarios:**

1. `POST /api/users` with `{"name": "", "email": "nastya@example.com"}`
2. `POST /api/users` with `{"name": "Nastya", "email": "nastya@example.com"}` but that email already exists
3. `GET /api/users/999` but user 999 doesn't exist
4. `POST /api/users` with `{"name": "Nastya"}` (missing email)
5. `DELETE /api/users/42` but the database is temporarily down

<details>
<summary>Click to see answers</summary>

1. **Validation error** in **controller** layer → `400 Bad Request` (name is blank, violates `@NotBlank`)
2. **Business logic error** in **service** layer → `409 Conflict` (service checks `existsByEmail`)
3. **Not found error** in **service** layer → `404 Not Found` (service throws `UserNotFoundException`)
4. **Validation error** in **controller** layer → `400 Bad Request` (email is required, violates `@NotBlank`)
5. **Infrastructure error** in **repository** layer → `500 Internal Server Error` (database connection failed)

</details>

---

### Exercise 3: Design Test Cases

Based on the `CreateUserRequest` validation rules:
```java
@NotBlank @Size(min = 2, max = 100) private String name;
@NotBlank @Email private String email;
@Pattern(regexp = "^(admin|user|qa)$") private String role;
```

Design test cases for the `POST /api/users` endpoint:

| # | Test Case | Body | Expected Status |
|---|-----------|------|-----------------|
| 1 | Valid user with all fields | ? | ? |
| 2 | Valid user without role | ? | ? |
| 3 | Empty name | ? | ? |
| 4 | ... | ... | ... |

Create at least **10 test cases** covering positive and negative scenarios.

<details>
<summary>Click to see example test cases</summary>

| # | Test Case | Body | Expected Status |
|---|-----------|------|-----------------|
| 1 | Valid user, all fields | `{"name":"Nastya","email":"n@test.com","role":"qa"}` | 201 |
| 2 | Valid user, no role (should default) | `{"name":"Nastya","email":"n@test.com"}` | 201 |
| 3 | Empty name | `{"name":"","email":"n@test.com"}` | 400 |
| 4 | Name too short (1 char) | `{"name":"N","email":"n@test.com"}` | 400 |
| 5 | Name too long (101 chars) | `{"name":"AAAA...101 chars","email":"n@test.com"}` | 400 |
| 6 | Missing name | `{"email":"n@test.com"}` | 400 |
| 7 | Invalid email | `{"name":"Nastya","email":"not-an-email"}` | 400 |
| 8 | Empty email | `{"name":"Nastya","email":""}` | 400 |
| 9 | Invalid role | `{"name":"Nastya","email":"n@test.com","role":"superadmin"}` | 400 |
| 10 | Duplicate email | `{"name":"New","email":"existing@test.com"}` | 409 |
| 11 | Name with special characters | `{"name":"O'Brien","email":"o@test.com"}` | 201 |
| 12 | Empty body | `{}` | 400 |
| 13 | Null body | (empty request) | 400 |

</details>

---

### Exercise 4: Read the Logs

Given these logs, answer the questions:

```
10:30:00.100 INFO  c.e.controller.UserController : POST /api/users
10:30:00.102 DEBUG c.e.service.UserService : Creating user: name=Test, email=test@example.com
10:30:00.105 DEBUG c.e.service.UserService : Checking email uniqueness: test@example.com
10:30:00.150 DEBUG org.hibernate.SQL : select count(*) from users where email=?
10:30:00.155 DEBUG c.e.service.UserService : Email is unique, proceeding
10:30:00.160 DEBUG org.hibernate.SQL : insert into users (name, email, role, created_at) values (?, ?, ?, ?)
10:30:00.200 ERROR c.e.exception.GlobalExceptionHandler : DataIntegrityViolationException: Unique constraint violation
10:30:00.201 ERROR c.e.controller.UserController : Request failed with status 500
```

**Questions:**
1. What request was made?
2. Did the email uniqueness check pass?
3. What went wrong?
4. In which layer did the error occur?
5. Why might the uniqueness check have passed but the INSERT still failed?
6. Is the 500 status code correct, or should it be something else?

<details>
<summary>Click to see answers</summary>

1. `POST /api/users` to create a user with email `test@example.com`
2. Yes, the check passed ("Email is unique, proceeding")
3. A unique constraint violation occurred when trying to insert the record
4. The error occurred in the **repository** layer (during the SQL INSERT)
5. **Race condition**: Another request created a user with the same email between the check (SELECT) and the insert (INSERT). This is a concurrency bug!
6. It should probably be `409 Conflict` — the error handler should catch `DataIntegrityViolationException` and return a more appropriate status

</details>

---

### Exercise 5: Explore a Real Application

If you have access to a Spring Boot application:

1. **Find the controller** for one of the API endpoints you test
2. **Trace the request** through controller → service → repository
3. **Look at the entity** — what database table does it map to?
4. **Check the validation** — what validation annotations are on the request DTO?
5. **Look at error handling** — how does the app handle exceptions?
6. **Check the configuration** — what's in `application.properties`?

If you don't have access to a real application, use the [Spring Petclinic](https://github.com/spring-projects/spring-petclinic) sample project to explore.

---

## 📚 Additional Resources

- [Spring Boot Getting Started](https://spring.io/guides/gs/rest-service/) — Build a REST service step by step
- [Spring Petclinic](https://github.com/spring-projects/spring-petclinic) — Sample Spring Boot application
- [Baeldung](https://www.baeldung.com/) — Excellent Java/Spring tutorials
- [Spring Boot Reference](https://docs.spring.io/spring-boot/docs/current/reference/html/) — Official documentation

---

## ✅ Self-Check

After completing this course, you should be able to:

- [ ] Explain the difference between frontend and backend
- [ ] Describe the layers: Controller → Service → Repository → Database
- [ ] Read a Spring Boot controller and understand what endpoints it exposes
- [ ] Understand what annotations like `@GetMapping`, `@RequestBody`, `@PathVariable` do
- [ ] Trace how an HTTP request travels through the backend
- [ ] Understand how validation works and design test cases from validation rules
- [ ] Read backend log output and identify errors
- [ ] Explain what DTOs are and why they're used
- [ ] Know what `application.properties` / `application.yml` configures
- [ ] Communicate more effectively with backend developers