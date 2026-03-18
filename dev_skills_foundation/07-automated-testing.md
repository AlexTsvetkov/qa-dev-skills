# Course 07: Automated Testing

## 📖 What You Will Learn

- What automated testing is and why it matters
- The testing pyramid: unit, integration, end-to-end tests
- Types of automated tests and when to use each
- Understanding test frameworks (JUnit, TestNG, REST Assured)
- How to read test code and test reports
- QA's role in test automation

---

## 1. Why Automate Tests?

### Manual testing problems:
- ⏰ **Slow** — takes hours/days to test everything
- 😴 **Error-prone** — humans miss things when repeating tasks
- 💰 **Expensive** — requires people's time for every release
- 🔄 **Not repeatable** — hard to run the exact same test every time

### Automated testing benefits:
- ⚡ **Fast** — run hundreds of tests in minutes
- 🎯 **Consistent** — same test, same way, every time
- 🔁 **Repeatable** — run on every code change in CI/CD
- 📊 **Measurable** — clear pass/fail reports
- 🛡️ **Safety net** — catch regressions immediately

---

## 2. The Testing Pyramid

```
            /\
           /  \        🔺 End-to-End (E2E) Tests
          / E2E\       Few, slow, expensive
         /──────\      Test the whole system through the UI
        /        \
       / Integr.  \    🔶 Integration Tests
      /────────────\   Medium amount, moderate speed
     /              \  Test components working together
    /   Unit Tests   \ 🟩 Unit Tests
   /──────────────────\ Many, fast, cheap
  /                    \ Test individual functions/methods
 /______________________\
```

| Level | What it tests | Speed | Quantity | Example |
|-------|--------------|-------|----------|---------|
| **Unit** | Single function/method | Milliseconds | Thousands | "Does `calculateTotal()` return correct sum?" |
| **Integration** | Components together | Seconds | Hundreds | "Does the API return user data from the database?" |
| **E2E** | Entire user workflow | Minutes | Tens | "Can a user log in, add to cart, and checkout?" |

---

## 3. Unit Tests

Test a single piece of code **in isolation**.

### Example (JUnit 5 — Java):

```java
class CalculatorTest {

    @Test
    void shouldAddTwoNumbers() {
        Calculator calc = new Calculator();
        int result = calc.add(2, 3);
        assertEquals(5, result);
    }

    @Test
    void shouldThrowExceptionForDivisionByZero() {
        Calculator calc = new Calculator();
        assertThrows(ArithmeticException.class, () -> calc.divide(10, 0));
    }

    @Test
    void shouldReturnEmptyListWhenNoUsersFound() {
        // Arrange
        UserRepository mockRepo = mock(UserRepository.class);
        when(mockRepo.findByRole("admin")).thenReturn(Collections.emptyList());
        UserService service = new UserService(mockRepo);

        // Act
        List<User> result = service.getUsersByRole("admin");

        // Assert
        assertTrue(result.isEmpty());
    }
}
```

### Test structure (AAA pattern):

```
Arrange  →  Set up test data and conditions
Act      →  Execute the code being tested
Assert   →  Verify the result is correct
```

---

## 4. Integration Tests

Test that **multiple components work together** correctly.

### API Integration Test (REST Assured — Java):

```java
class UserApiIntegrationTest {

    @Test
    void shouldCreateAndRetrieveUser() {
        // Create a user
        String userId =
            given()
                .contentType(ContentType.JSON)
                .body("""
                    {
                        "name": "Nastya",
                        "email": "nastya@test.com",
                        "role": "qa"
                    }
                    """)
            .when()
                .post("/api/users")
            .then()
                .statusCode(201)
                .body("name", equalTo("Nastya"))
                .extract().path("id").toString();

        // Retrieve the created user
        given()
            .pathParam("id", userId)
        .when()
            .get("/api/users/{id}")
        .then()
            .statusCode(200)
            .body("name", equalTo("Nastya"))
            .body("email", equalTo("nastya@test.com"))
            .body("role", equalTo("qa"));
    }

    @Test
    void shouldReturn404ForNonExistentUser() {
        given()
        .when()
            .get("/api/users/99999")
        .then()
            .statusCode(404)
            .body("error", equalTo("NOT_FOUND"));
    }

    @Test
    void shouldReturn400ForInvalidEmail() {
        given()
            .contentType(ContentType.JSON)
            .body("""
                {
                    "name": "Test",
                    "email": "not-an-email"
                }
                """)
        .when()
            .post("/api/users")
        .then()
            .statusCode(400);
    }
}
```

### REST Assured pattern:

```
given()   →  Set up request (headers, body, params)
when()    →  Send the request (GET, POST, etc.)
then()    →  Verify the response (status, body, headers)
```

---

## 5. End-to-End (E2E) Tests

Test complete user workflows through the **browser or API chain**.

### Selenium (Java) example:

```java
class LoginE2ETest {

    WebDriver driver;

    @BeforeEach
    void setUp() {
        driver = new ChromeDriver();
        driver.get("https://staging.example.com");
    }

    @Test
    void shouldLoginSuccessfully() {
        driver.findElement(By.id("email")).sendKeys("nastya@example.com");
        driver.findElement(By.id("password")).sendKeys("password123");
        driver.findElement(By.id("login-button")).click();

        WebElement welcome = new WebDriverWait(driver, Duration.ofSeconds(10))
                .until(ExpectedConditions.visibilityOfElementLocated(By.id("welcome-message")));

        assertEquals("Welcome, Nastya!", welcome.getText());
    }

    @AfterEach
    void tearDown() {
        driver.quit();
    }
}
```

---

## 6. Test Reports

### JUnit XML Report (standard format):

```xml
<testsuite name="UserApiTest" tests="5" failures="1" errors="0" time="2.345">
    <testcase name="shouldCreateUser" classname="UserApiTest" time="0.543"/>
    <testcase name="shouldGetUser" classname="UserApiTest" time="0.234"/>
    <testcase name="shouldUpdateUser" classname="UserApiTest" time="0.456"/>
    <testcase name="shouldDeleteUser" classname="UserApiTest" time="0.312">
        <failure message="Expected 204 but got 500">
            java.lang.AssertionError: Expected status 204 but got 500
            at UserApiTest.shouldDeleteUser(UserApiTest.java:78)
        </failure>
    </testcase>
    <testcase name="shouldReturn404" classname="UserApiTest" time="0.123"/>
</testsuite>
```

### Reading test reports:

| Field | Meaning |
|-------|---------|
| `tests="5"` | Total number of tests |
| `failures="1"` | Tests that ran but assertion failed |
| `errors="0"` | Tests that crashed (exception) |
| `time` | How long the test took |
| `<failure>` | Details of what went wrong |

---

## 7. Common Assertions

| Assertion | What it checks |
|-----------|---------------|
| `assertEquals(expected, actual)` | Values are equal |
| `assertNotEquals(a, b)` | Values are NOT equal |
| `assertTrue(condition)` | Condition is true |
| `assertFalse(condition)` | Condition is false |
| `assertNull(value)` | Value is null |
| `assertNotNull(value)` | Value is NOT null |
| `assertThrows(Exception.class, code)` | Code throws expected exception |
| `assertThat(list).hasSize(5)` | Collection has 5 elements |
| `assertThat(string).contains("text")` | String contains substring |

---

## 8. Test Best Practices

| Practice | Why |
|----------|-----|
| **One assertion per concept** | Know exactly what failed |
| **Descriptive test names** | `shouldReturn404WhenUserNotFound` not `test1` |
| **Independent tests** | Tests don't depend on each other's order |
| **Clean up test data** | Don't leave junk in the database |
| **Use test fixtures** | Reusable setup/teardown |
| **Don't test implementation** | Test behavior, not internal code |
| **Fast tests** | Slow tests won't be run |

---

## 📝 Exercises

### Exercise 1: Identify Test Types

Classify each test as Unit, Integration, or E2E:

1. Test that `calculateDiscount(100, 0.1)` returns `90`
2. Test that POST `/api/orders` creates an order in the database
3. Test that a user can browse products, add to cart, and complete checkout in the browser
4. Test that `validateEmail("bad")` returns `false`
5. Test that the login API returns a valid JWT token
6. Test that clicking "Add to Cart" shows the item count badge

### Exercise 2: Write Test Scenarios

For a `POST /api/users` endpoint, write test scenarios covering:
- Happy path (valid request)
- Missing required fields
- Invalid data formats
- Duplicate data
- Authentication/authorization

### Exercise 3: Read Test Report

Given this test report, answer the questions:
```
Tests run: 25, Failures: 3, Errors: 1, Skipped: 2
- FAILED: testCreateUserWithDuplicateEmail — Expected 409, got 500
- FAILED: testUpdateUserName — Expected "New Name", got "Old Name"
- FAILED: testDeleteUser — Expected 204, got 403
- ERROR: testGetUserOrders — NullPointerException at line 45
- SKIPPED: testAdminAccess — @Disabled("Auth not implemented yet")
- SKIPPED: testBulkImport — Condition not met
```

1. How many tests passed?
2. What's the difference between FAILED and ERROR?
3. Which failure suggests a bug vs. a missing feature?
4. Why were tests skipped?

---

## 📚 Additional Resources

- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/) — Java testing framework
- [REST Assured Documentation](https://rest-assured.io/) — API testing in Java
- [Selenium Documentation](https://www.selenium.dev/documentation/) — Browser automation
- [Testing Pyramid (Martin Fowler)](https://martinfowler.com/articles/practical-test-pyramid.html) — Testing strategy

---

## ✅ Self-Check

- [ ] Explain the testing pyramid and each level

<details>
<summary>Answer</summary>

The testing pyramid has three levels. **Unit tests** (base, largest layer) — test individual methods/functions in isolation, fast, cheap, run in milliseconds. **Integration tests** (middle layer) — test how components work together (e.g., service + database, API + external service), slower, need infrastructure. **E2E / UI tests** (top, smallest layer) — test the full application from the user's perspective through the UI, slowest, most brittle, most expensive. The pyramid shape means: write many unit tests, fewer integration tests, and fewest E2E tests.

</details>

- [ ] Describe the difference between unit, integration, and E2E tests

<details>
<summary>Answer</summary>

**Unit tests** test a single class/method in isolation — dependencies are mocked (e.g., test `UserService.createUser()` without a real database). They run in milliseconds and catch logic errors. **Integration tests** test multiple components together — real database, real HTTP calls (e.g., send a POST request to the API and verify it saves to the database). They take seconds and catch wiring/configuration errors. **E2E tests** test the entire system through the UI — a browser clicks buttons, fills forms, and verifies results. They take minutes and catch workflow/UI errors.

</details>

- [ ] Read a JUnit/REST Assured test and understand what it tests

<details>
<summary>Answer</summary>

A JUnit test has: `@Test` annotation marking the test method, a descriptive method name (e.g., `shouldReturn404WhenUserNotFound`), the **Arrange-Act-Assert** pattern — set up data, perform the action, check the result. REST Assured tests read like English: `given().contentType(JSON).body(request).when().post("/api/users").then().statusCode(201).body("name", equalTo("Nastya"))`. Key parts: `given()` — request setup, `when()` — the HTTP call, `then()` — assertions on the response.

</details>

- [ ] Understand common assertions (`assertEquals`, `assertThrows`, etc.)

<details>
<summary>Answer</summary>

- `assertEquals(expected, actual)` — checks two values are equal
- `assertNotNull(value)` — checks value is not null
- `assertTrue(condition)` / `assertFalse(condition)` — checks boolean conditions
- `assertThrows(Exception.class, () -> code)` — verifies an exception is thrown
- `assertThat(value, is(expected))` — Hamcrest matcher style, more readable
- REST Assured: `statusCode(200)`, `body("name", equalTo("Nastya"))`, `body("items", hasSize(3))`

When an assertion fails, the test fails and reports expected vs actual values.

</details>

- [ ] Read a test report and identify failures vs errors

<details>
<summary>Answer</summary>

A test report shows: **Passed** (green) — test ran and all assertions succeeded. **Failed** (red) — test ran but an assertion didn't match (e.g., expected 200 but got 500) — this usually means a bug in the application. **Error** (red/orange) — test couldn't run due to an unexpected exception (e.g., NullPointerException, connection timeout) — this might be a test infrastructure problem or a crash. **Skipped** (yellow/grey) — test was intentionally skipped (e.g., `@Disabled`). Focus on failures first (likely real bugs), then errors (might be environment issues).

</details>

- [ ] Write test scenarios for an API endpoint

<details>
<summary>Answer</summary>

For `POST /api/users` with body `{name, email, role}`, write scenarios covering: **Happy path** — valid data returns 201 with created user. **Validation errors** — missing name (400), invalid email (400), empty body (400). **Business logic** — duplicate email (409). **Not found** — GET/PUT/DELETE non-existent user (404). **Boundary values** — name at min/max length, special characters. **Security** — unauthorized request (401), forbidden action (403). **Edge cases** — extra fields in body, wrong Content-Type, concurrent requests. Each scenario should specify: input, expected status code, and expected response body.

</details>

- [ ] Explain why automated tests are important for CI/CD

<details>
<summary>Answer</summary>

Automated tests are the **safety net** that makes CI/CD possible. Without them, every code change would need manual verification before deployment — making continuous delivery impossible. Benefits: **fast feedback** (know within minutes if a change broke something), **regression prevention** (old features don't break when adding new ones), **confidence to deploy** (if all tests pass, the code is likely safe), **documentation** (tests show how the code should behave), and **enables frequent releases** (can deploy daily instead of monthly because testing is automated).

</details>
