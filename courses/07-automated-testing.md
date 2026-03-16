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
- [ ] Describe the difference between unit, integration, and E2E tests
- [ ] Read a JUnit/REST Assured test and understand what it tests
- [ ] Understand common assertions (`assertEquals`, `assertThrows`, etc.)
- [ ] Read a test report and identify failures vs errors
- [ ] Write test scenarios for an API endpoint
- [ ] Explain why automated tests are important for CI/CD