# Course 03: Automation & Framework Expertise

## 📖 What You Will Learn

- How to design and maintain automation frameworks from scratch
- Test automation strategy across UI, API, E2E, integration, and regression
- Modern automation tools: Selenium, Playwright, Cypress
- Code review practices for automation code
- Automation best practices and design patterns
- Monitoring and reducing flaky tests
- Version control with Git and branching strategies

---

## 1. Designing & Maintaining Automation Frameworks

### What is a Test Automation Framework?

A test automation framework is a **structured set of guidelines, tools, and practices** for writing, organizing, and running automated tests efficiently and maintainably.

### Key Framework Components:

```
automation-framework/
├── config/                    # Environment configs, test settings
│   ├── dev.config.ts
│   ├── staging.config.ts
│   └── prod.config.ts
├── src/
│   ├── pages/                 # Page Object Models
│   │   ├── LoginPage.ts
│   │   ├── DashboardPage.ts
│   │   └── BasePage.ts
│   ├── api/                   # API client wrappers
│   │   ├── UserApi.ts
│   │   └── BaseApi.ts
│   ├── helpers/               # Utility functions
│   │   ├── DataGenerator.ts
│   │   ├── WaitHelpers.ts
│   │   └── DateUtils.ts
│   ├── fixtures/              # Test data
│   │   ├── users.json
│   │   └── products.json
│   └── types/                 # Type definitions
│       └── index.ts
├── tests/
│   ├── ui/                    # UI/E2E tests
│   │   ├── login.spec.ts
│   │   └── checkout.spec.ts
│   ├── api/                   # API tests
│   │   ├── users.api.spec.ts
│   │   └── orders.api.spec.ts
│   └── integration/           # Integration tests
│       └── cart-payment.spec.ts
├── reports/                   # Test reports output
├── .github/workflows/         # CI/CD pipeline
├── package.json
├── tsconfig.json
├── playwright.config.ts       # (or cypress.config.ts)
└── README.md
```

### Framework Design Principles:

| Principle | Description | Example |
|-----------|-------------|---------|
| **Separation of Concerns** | Tests, page objects, data, and config are separate | Tests don't contain selectors; page objects do |
| **DRY (Don't Repeat Yourself)** | Common actions are reusable | Login helper used across all tests |
| **Maintainability** | Easy to update when the app changes | Change a selector in one place, not 50 tests |
| **Readability** | Tests read like documentation | `loginPage.loginAs(user)` not `driver.findElement(...)` |
| **Scalability** | Framework grows with the application | New features = new page objects, not restructuring |
| **Reliability** | Tests produce consistent results | Proper waits, test isolation, clean data |

### Page Object Model (POM)

The **Page Object Model** is the most widely used design pattern in test automation. Each page (or component) of the application is represented by a class.

```typescript
// pages/LoginPage.ts
import { Page } from '@playwright/test';

export class LoginPage {
  private page: Page;
  
  // Selectors
  private emailInput = '[data-testid="email-input"]';
  private passwordInput = '[data-testid="password-input"]';
  private loginButton = '[data-testid="login-button"]';
  private errorMessage = '[data-testid="error-message"]';
  
  constructor(page: Page) {
    this.page = page;
  }
  
  async navigate() {
    await this.page.goto('/login');
  }
  
  async login(email: string, password: string) {
    await this.page.fill(this.emailInput, email);
    await this.page.fill(this.passwordInput, password);
    await this.page.click(this.loginButton);
  }
  
  async getErrorMessage(): Promise<string> {
    return await this.page.textContent(this.errorMessage) ?? '';
  }
  
  async isLoginButtonVisible(): Promise<boolean> {
    return await this.page.isVisible(this.loginButton);
  }
}
```

```typescript
// tests/ui/login.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from '../../src/pages/LoginPage';

test.describe('Login Feature', () => {
  let loginPage: LoginPage;
  
  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    await loginPage.navigate();
  });
  
  test('should login successfully with valid credentials', async ({ page }) => {
    await loginPage.login('nastya@example.com', 'SecurePass123');
    await expect(page).toHaveURL('/dashboard');
  });
  
  test('should show error with invalid credentials', async () => {
    await loginPage.login('nastya@example.com', 'WrongPassword');
    const error = await loginPage.getErrorMessage();
    expect(error).toContain('Invalid email or password');
  });
});
```

### Other Design Patterns:

| Pattern | Description | When to Use |
|---------|-------------|-------------|
| **Page Object Model** | One class per page/component with selectors and actions | UI tests (most common) |
| **Screenplay Pattern** | Actor-based: "Actor performs Task on Target" | Complex workflows, BDD |
| **Factory Pattern** | Create test data objects dynamically | Test data generation |
| **Builder Pattern** | Step-by-step object construction | Complex test configurations |
| **Singleton Pattern** | Single instance of shared resources | Database connections, API clients |
| **Strategy Pattern** | Swap algorithms at runtime | Different browser/environment configs |

---

## 2. Test Automation Strategy

### The Test Automation Pyramid

```
                    ╱╲
                   ╱  ╲        Manual / Exploratory
                  ╱    ╲       (Few, human judgment)
                 ╱──────╲
                ╱        ╲     E2E / UI Tests
               ╱  E2E     ╲   (10-20%, slow, brittle)
              ╱─────────────╲
             ╱               ╲  Integration / API Tests
            ╱  Integration    ╲ (20-40%, medium speed)
           ╱───────────────────╲
          ╱                     ╲ Unit Tests
         ╱     Unit Tests        ╲(40-60%, fast, stable)
        ╱_________________________╲
```

### Test Types and Automation Strategy:

| Test Type | What It Tests | Speed | Stability | Tools |
|-----------|--------------|-------|-----------|-------|
| **Unit** | Individual functions/methods | ⚡ Very fast | 🟢 Very stable | Jest, JUnit, pytest |
| **Integration** | Component interactions, APIs | 🏃 Fast | 🟢 Stable | REST Assured, Supertest |
| **E2E** | Full user workflows | 🐢 Slow | 🟡 Can be flaky | Playwright, Cypress, Selenium |
| **Visual** | UI appearance/layout | 🏃 Medium | 🟡 Requires baselines | Percy, Applitools, Playwright |
| **Performance** | Speed, load, scalability | 🐢 Slow | 🟢 Stable | k6, JMeter, Gatling |
| **Accessibility** | A11y compliance | 🏃 Fast | 🟢 Stable | axe-core, Lighthouse |

### What to Automate vs. What to Test Manually:

| Automate ✅ | Keep Manual 🖐️ |
|------------|----------------|
| Regression tests (run every release) | Exploratory testing (creative, unscripted) |
| Smoke tests (critical paths) | Usability testing (subjective assessment) |
| Data-driven tests (many inputs) | Visual design review (aesthetic judgment) |
| API contract validation | Ad-hoc investigation of reported bugs |
| Cross-browser/cross-platform checks | One-time setup verification |
| Performance benchmarks | Complex business scenario first-run |

### Automation Strategy Document:

```markdown
# Test Automation Strategy

## Scope
- Automated: Regression, smoke, API, performance
- Manual: Exploratory, usability, complex business scenarios

## Tool Selection
- UI: Playwright (cross-browser, fast, reliable)
- API: Playwright API testing + REST Assured
- Performance: k6
- Visual: Playwright visual comparison

## Coverage Targets
- Unit: 80%+ code coverage (developer-owned)
- API: 100% of endpoints with happy + error paths
- E2E: Top 20 critical user journeys
- Regression: Full suite runs nightly, smoke on every deploy

## Execution Strategy
- CI: Unit + lint on every commit
- CD: Integration tests on merge, E2E on deploy
- Nightly: Full regression + performance
- Weekly: Full cross-browser matrix

## Maintenance Plan
- Flaky test budget: <5% of total tests
- Review and update tests with every feature change
- Quarterly automation health review
```

---

## 3. Modern Automation Tools

### Selenium

The **original** browser automation tool. Still widely used in enterprise.

```java
// Selenium WebDriver (Java)
WebDriver driver = new ChromeDriver();
driver.get("https://example.com/login");

driver.findElement(By.id("email")).sendKeys("nastya@example.com");
driver.findElement(By.id("password")).sendKeys("SecurePass123");
driver.findElement(By.id("login-btn")).click();

WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(ExpectedConditions.urlContains("/dashboard"));

String title = driver.getTitle();
assertEquals("Dashboard", title);

driver.quit();
```

**Pros:** Multi-language support, large community, W3C standard
**Cons:** Requires separate driver management, slower, verbose API

### Playwright

**Modern** browser automation by Microsoft. Gaining rapid adoption.

```typescript
// Playwright (TypeScript)
import { test, expect } from '@playwright/test';

test('user can login and see dashboard', async ({ page }) => {
  await page.goto('/login');
  
  await page.fill('[data-testid="email"]', 'nastya@example.com');
  await page.fill('[data-testid="password"]', 'SecurePass123');
  await page.click('[data-testid="login-btn"]');
  
  await expect(page).toHaveURL(/.*dashboard/);
  await expect(page.locator('h1')).toHaveText('Welcome, Nastya!');
});
```

**Pros:** Auto-wait, multi-browser, fast, built-in API testing, trace viewer
**Cons:** Newer (smaller community than Selenium), TypeScript/JS/Python/Java/.NET only

### Cypress

**Developer-friendly** testing framework for web applications.

```javascript
// Cypress (JavaScript)
describe('Login', () => {
  it('should login successfully', () => {
    cy.visit('/login');
    
    cy.get('[data-testid="email"]').type('nastya@example.com');
    cy.get('[data-testid="password"]').type('SecurePass123');
    cy.get('[data-testid="login-btn"]').click();
    
    cy.url().should('include', '/dashboard');
    cy.get('h1').should('contain', 'Welcome, Nastya!');
  });
});
```

**Pros:** Great DX, time-travel debugging, automatic waiting, network stubbing
**Cons:** Only Chromium/Firefox/Edge (no Safari/WebKit), runs in-browser (limitations)

### Tool Comparison:

| Feature | Selenium | Playwright | Cypress |
|---------|----------|-----------|---------|
| **Languages** | Java, Python, C#, JS, Ruby | JS/TS, Python, Java, .NET | JS/TS only |
| **Browsers** | All | Chromium, Firefox, WebKit | Chromium, Firefox, Edge |
| **Speed** | Slow | Fast | Fast |
| **Auto-wait** | No (manual waits) | Yes | Yes |
| **API Testing** | No (separate tool) | Yes (built-in) | Yes (built-in) |
| **Debugging** | Difficult | Trace viewer, inspector | Time-travel, snapshots |
| **Parallel** | Selenium Grid | Built-in | Built-in (paid for cloud) |
| **Mobile** | Appium integration | Android/Mobile emulation | Limited |
| **Community** | Massive | Growing fast | Large |
| **Best For** | Enterprise, legacy | Modern web, cross-browser | Developer testing |

---

## 4. Code Review Practices for Automation Code

### Why Review Automation Code?

Automation code is **production code** — it needs the same quality standards. Bad test code leads to:
- Flaky tests that nobody trusts
- High maintenance costs
- Tests that don't actually test anything meaningful

### Automation Code Review Checklist:

| Category | What to Check |
|----------|-------------|
| **Test Design** | Does the test verify one specific behavior? Is it independent? |
| **Naming** | Is the test name descriptive? Can you understand what failed without reading the code? |
| **Selectors** | Are selectors stable (data-testid) or brittle (CSS classes, XPath positions)? |
| **Assertions** | Are assertions specific and meaningful? Not just "page loaded"? |
| **Test Data** | Is test data created/cleaned properly? No shared state between tests? |
| **Waits** | Are waits explicit and appropriate? No `sleep()` or `Thread.sleep()`? |
| **Error Handling** | Will the test provide useful information on failure? |
| **DRY** | Is there code duplication that should be a helper or page object method? |
| **Performance** | Does the test run in reasonable time? Can it be parallelized? |

### Good vs. Bad Test Examples:

❌ **Bad test:**
```typescript
test('test1', async ({ page }) => {
  await page.goto('https://staging.example.com/login');
  await page.waitForTimeout(3000); // Hard-coded wait!
  await page.click('#app > div > form > div:nth-child(1) > input'); // Brittle selector!
  await page.keyboard.type('admin');
  await page.click('#app > div > form > div:nth-child(2) > input');
  await page.keyboard.type('admin123');
  await page.click('#app > div > form > button');
  await page.waitForTimeout(5000); // Another hard wait!
  const url = page.url();
  expect(url).toBeTruthy(); // Meaningless assertion!
});
```

✅ **Good test:**
```typescript
test('should redirect to dashboard after successful login', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.navigate();
  
  await loginPage.login(testUsers.validUser.email, testUsers.validUser.password);
  
  await expect(page).toHaveURL('/dashboard');
  await expect(page.getByRole('heading', { name: 'Welcome' })).toBeVisible();
});
```

### Selector Strategy Priority:

```
Best → data-testid="login-button"     (stable, test-specific)
       role-based: getByRole('button', { name: 'Login' })
       text-based: getByText('Login')
       id: #login-button               (stable but not test-specific)
       CSS class: .btn-primary          (can change with design updates)
Worst → XPath: //div[3]/form/button    (extremely brittle)
```

---

## 5. Automation Best Practices and Patterns

### The FIRST Principles for Good Tests:

| Principle | Description |
|-----------|-------------|
| **F — Fast** | Tests should run quickly to provide fast feedback |
| **I — Isolated** | Each test is independent, no shared state |
| **R — Repeatable** | Same result every time, regardless of environment |
| **S — Self-validating** | Tests clearly pass or fail, no manual interpretation |
| **T — Timely** | Tests are written close to when the feature is developed |

### Test Isolation Strategies:

```typescript
// ✅ Good: Each test creates its own data
test.beforeEach(async ({ request }) => {
  // Create fresh user for this test
  const response = await request.post('/api/test/users', {
    data: { email: `user-${Date.now()}@test.com`, password: 'Test123!' }
  });
  testUser = await response.json();
});

test.afterEach(async ({ request }) => {
  // Clean up after test
  await request.delete(`/api/test/users/${testUser.id}`);
});

// ❌ Bad: Tests depend on shared data
// "This test only works if test2 runs first"
```

### Handling Asynchronous Operations:

```typescript
// ❌ Bad: Hard-coded waits
await page.waitForTimeout(5000);

// ✅ Good: Wait for specific conditions
await page.waitForSelector('[data-testid="results-list"]');
await expect(page.locator('.loading')).not.toBeVisible();
await page.waitForResponse(resp => resp.url().includes('/api/search'));
```

### Data-Driven Testing:

```typescript
// Run the same test with multiple data sets
const testCases = [
  { email: 'valid@test.com', password: 'Valid123!', expected: 'success' },
  { email: 'invalid-email', password: 'Valid123!', expected: 'Invalid email format' },
  { email: 'valid@test.com', password: 'short', expected: 'Password too short' },
  { email: '', password: 'Valid123!', expected: 'Email is required' },
];

for (const tc of testCases) {
  test(`login with ${tc.email} should ${tc.expected}`, async ({ page }) => {
    const loginPage = new LoginPage(page);
    await loginPage.navigate();
    await loginPage.login(tc.email, tc.password);
    
    if (tc.expected === 'success') {
      await expect(page).toHaveURL('/dashboard');
    } else {
      await expect(loginPage.errorMessage).toHaveText(tc.expected);
    }
  });
}
```

### Retry and Recovery:

```typescript
// playwright.config.ts
export default defineConfig({
  retries: process.env.CI ? 2 : 0,  // Retry failed tests in CI
  use: {
    actionTimeout: 10000,
    navigationTimeout: 30000,
    screenshot: 'only-on-failure',
    trace: 'on-first-retry',
    video: 'on-first-retry',
  },
});
```

---

## 6. Monitoring and Reducing Flaky Tests

### What is a Flaky Test?

A **flaky test** is a test that sometimes passes and sometimes fails without any code changes. Flaky tests destroy confidence in the test suite.

### Common Causes of Flakiness:

| Cause | Example | Solution |
|-------|---------|----------|
| **Timing issues** | Element not ready when test clicks it | Use auto-wait, explicit waits for conditions |
| **Test order dependency** | Test B fails if Test A didn't run first | Ensure test isolation, independent setup |
| **Shared state** | Tests modify same database records | Unique test data per test, cleanup |
| **Environment instability** | Staging server is slow or down | Health checks before test run, retry logic |
| **Animation/transition** | Click happens during CSS animation | Wait for animations to complete |
| **Race conditions** | API response arrives after assertion | Wait for API responses before asserting |
| **Date/time dependency** | Test fails at midnight or month-end | Mock time, use relative dates |

### Flaky Test Detection:

```typescript
// Track test stability over time
// Run the same test suite multiple times and compare results

// Metrics to track:
// - Flaky test rate = (tests that flipped pass/fail) / total tests
// - Target: < 2% flaky rate
// - Quarantine threshold: if a test fails > 3 times in 10 runs without code changes
```

### Flaky Test Management Strategy:

```
1. DETECT: Run tests multiple times, track pass/fail patterns
2. QUARANTINE: Move confirmed flaky tests to a separate suite
3. FIX: Prioritize fixing quarantined tests (treat as tech debt)
4. PREVENT: Code reviews focused on flakiness patterns
5. MONITOR: Dashboard showing flaky test trends over time
```

### Anti-Patterns That Cause Flakiness:

```typescript
// ❌ Anti-pattern: sleep/wait instead of condition
await page.waitForTimeout(5000);

// ❌ Anti-pattern: global test data
const USER_EMAIL = 'test@example.com'; // Same email in all tests

// ❌ Anti-pattern: order-dependent tests  
test('create order', async () => { /* creates order #1 */ });
test('verify order', async () => { /* assumes order #1 exists */ });

// ❌ Anti-pattern: non-deterministic assertions
expect(results.length).toBeGreaterThan(0); // How many? Depends on data

// ❌ Anti-pattern: relying on external services
await page.waitForSelector('.weather-widget'); // Third-party API might be down
```

---

## 7. Version Control with Git

### Essential Git Commands for QA:

```bash
# Clone a repository
git clone https://github.com/team/automation-tests.git

# Create and switch to a new branch
git checkout -b feature/add-login-tests

# Check current status
git status

# Stage changes
git add tests/login.spec.ts
git add .   # Stage all changes

# Commit with descriptive message
git commit -m "Add login page E2E tests for valid and invalid credentials"

# Push to remote
git push origin feature/add-login-tests

# Pull latest changes from main
git checkout main
git pull origin main

# Merge main into your branch (keep up to date)
git checkout feature/add-login-tests
git merge main

# Rebase (alternative to merge, cleaner history)
git checkout feature/add-login-tests
git rebase main

# View commit history
git log --oneline --graph

# Stash changes temporarily
git stash
git stash pop
```

### Branching Strategies:

#### Git Flow:
```
main ─────────●─────────────────●────────── (production releases)
               \               /
develop ────●───●───●───●───●──────●──── (integration branch)
             \     / \     /
feature/     ●───●   ●───●              (feature branches)
login-tests  bugfix/
             cart-fix
```

#### GitHub Flow (simpler):
```
main ─────────────●───────●───────●──── (always deployable)
                 / \     / \     /
feature/        ●───●   ●   ●──●       (short-lived branches)
add-tests            bugfix/
                     fix-flaky
```

#### Trunk-Based Development:
```
main ────●──●──●──●──●──●──●──●──── (single branch, small commits)
          │  │  │  │  │  │  │  │
          feature flags / short branches
```

### Git Best Practices for Test Automation:

| Practice | Description |
|----------|-------------|
| **Descriptive branch names** | `feature/add-checkout-e2e-tests`, not `fix` |
| **Small, focused commits** | One logical change per commit |
| **Meaningful commit messages** | Describe what and why, not how |
| **Pull requests with reviews** | All test code goes through code review |
| **Keep branches short-lived** | Merge within 1-3 days to avoid conflicts |
| **Don't commit secrets** | No passwords, API keys, or tokens in code |
| **Use .gitignore** | Exclude reports, node_modules, screenshots |

### .gitignore for Test Automation:

```gitignore
# Dependencies
node_modules/

# Test artifacts
test-results/
playwright-report/
screenshots/
videos/

# Environment files
.env
.env.local

# IDE files
.idea/
.vscode/

# OS files
.DS_Store
Thumbs.db

# Build output
dist/
build/
```

---

## 📝 Exercises

### Exercise 1: Design a Framework Structure

You're starting a new automation project for an e-commerce website. Design the framework structure including:
1. Folder structure
2. Key page objects needed
3. Test categories
4. Configuration approach

<details>
<summary>Click to see answer</summary>

```
ecommerce-automation/
├── config/
│   ├── base.config.ts         # Shared settings
│   ├── dev.config.ts          # Dev environment URLs
│   ├── staging.config.ts      # Staging environment
│   └── test-data.config.ts    # Data generation settings
├── src/
│   ├── pages/                 # Page Object Models
│   │   ├── BasePage.ts        # Common page methods
│   │   ├── HomePage.ts
│   │   ├── LoginPage.ts
│   │   ├── ProductListPage.ts
│   │   ├── ProductDetailPage.ts
│   │   ├── CartPage.ts
│   │   ├── CheckoutPage.ts
│   │   ├── PaymentPage.ts
│   │   ├── OrderConfirmPage.ts
│   │   └── AdminDashboard.ts
│   ├── components/            # Reusable UI components
│   │   ├── Header.ts
│   │   ├── Footer.ts
│   │   ├── SearchBar.ts
│   │   └── ProductCard.ts
│   ├── api/
│   │   ├── BaseApi.ts
│   │   ├── AuthApi.ts
│   │   ├── ProductApi.ts
│   │   └── OrderApi.ts
│   ├── helpers/
│   │   ├── TestDataFactory.ts
│   │   ├── DatabaseHelper.ts
│   │   └── EmailHelper.ts
│   └── fixtures/
│       ├── users.json
│       ├── products.json
│       └── addresses.json
├── tests/
│   ├── smoke/                 # Critical path tests (run on every deploy)
│   │   ├── login.smoke.spec.ts
│   │   └── purchase.smoke.spec.ts
│   ├── regression/            # Full regression suite (nightly)
│   │   ├── auth/
│   │   ├── catalog/
│   │   ├── cart/
│   │   ├── checkout/
│   │   └── admin/
│   ├── api/                   # API tests
│   │   ├── auth.api.spec.ts
│   │   ├── products.api.spec.ts
│   │   └── orders.api.spec.ts
│   └── visual/                # Visual regression tests
│       └── homepage.visual.spec.ts
├── playwright.config.ts
├── package.json
├── tsconfig.json
├── .gitignore
├── .github/workflows/ci.yml
└── README.md
```

</details>

---

### Exercise 2: Refactor a Bad Test

Refactor this poorly written test into a good one using Page Object Model and best practices:

```typescript
test('test', async ({ page }) => {
  await page.goto('http://localhost:3000/login');
  await page.waitForTimeout(2000);
  await page.click('#app > div > form > input:nth-child(1)');
  await page.keyboard.type('admin@test.com');
  await page.click('#app > div > form > input:nth-child(2)');
  await page.keyboard.type('password123');
  await page.click('#app > div > form > button');
  await page.waitForTimeout(3000);
  await page.click('#app > div > nav > a:nth-child(3)');
  await page.waitForTimeout(2000);
  const text = await page.textContent('#app > div > main > h1');
  expect(text).not.toBeNull();
});
```

<details>
<summary>Click to see answer</summary>

**Step 1: Create Page Objects**

```typescript
// pages/LoginPage.ts
export class LoginPage {
  constructor(private page: Page) {}
  
  async navigate() {
    await this.page.goto('/login');
  }
  
  async login(email: string, password: string) {
    await this.page.fill('[data-testid="email-input"]', email);
    await this.page.fill('[data-testid="password-input"]', password);
    await this.page.click('[data-testid="login-button"]');
  }
}

// pages/DashboardPage.ts
export class DashboardPage {
  constructor(private page: Page) {}
  
  get heading() {
    return this.page.getByRole('heading', { level: 1 });
  }
  
  async navigateToProducts() {
    await this.page.click('[data-testid="nav-products"]');
  }
}

// pages/ProductsPage.ts
export class ProductsPage {
  constructor(private page: Page) {}
  
  get pageTitle() {
    return this.page.getByRole('heading', { name: 'Products' });
  }
}
```

**Step 2: Refactored Test**

```typescript
test('should navigate to products page after login', async ({ page }) => {
  // Arrange
  const loginPage = new LoginPage(page);
  const dashboardPage = new DashboardPage(page);
  const productsPage = new ProductsPage(page);
  
  // Act
  await loginPage.navigate();
  await loginPage.login('admin@test.com', 'password123');
  
  await expect(page).toHaveURL('/dashboard');
  
  await dashboardPage.navigateToProducts();
  
  // Assert
  await expect(productsPage.pageTitle).toBeVisible();
  await expect(productsPage.pageTitle).toHaveText('Products');
});
```

**What improved:**
- ✅ Descriptive test name
- ✅ Page Object Model (selectors isolated)
- ✅ Stable selectors (data-testid, roles)
- ✅ No hard-coded waits
- ✅ Meaningful assertions
- ✅ Relative URLs (configurable base URL)
- ✅ Arrange-Act-Assert structure

</details>

---

### Exercise 3: Flaky Test Analysis

These tests fail intermittently. Identify the cause and propose fixes:

1. Test fails 20% of the time: `expect(page.locator('.notification')).toBeVisible()`
2. Test fails on Monday mornings: `expect(order.deliveryDate).toBe('2024-03-15')`
3. Test fails when run in parallel: `expect(userCount).toBe(5)`
4. Test passes locally but fails in CI: `await page.click('.dropdown-item')`

<details>
<summary>Click to see answers</summary>

**1. Notification visibility (20% failure rate)**
- **Cause:** Notification appears with animation/delay and test asserts too early
- **Fix:**
```typescript
// Wait for notification with timeout
await expect(page.locator('.notification')).toBeVisible({ timeout: 10000 });
// Or wait for the API call that triggers the notification
await page.waitForResponse(resp => resp.url().includes('/api/notify'));
await expect(page.locator('.notification')).toBeVisible();
```

**2. Monday morning failures (hardcoded date)**
- **Cause:** Hardcoded date becomes stale, or delivery date calculation includes weekends
- **Fix:**
```typescript
// Use relative dates
const expectedDate = calculateNextBusinessDay(new Date());
expect(order.deliveryDate).toBe(formatDate(expectedDate));
// Or use date matchers
expect(new Date(order.deliveryDate)).toBeAfter(new Date());
```

**3. Parallel execution failure (shared data)**
- **Cause:** Multiple tests read/write to the same user table simultaneously
- **Fix:**
```typescript
// Create isolated data per test
const uniquePrefix = `test-${Date.now()}-${Math.random().toString(36).slice(2)}`;
await createTestUsers(5, { emailPrefix: uniquePrefix });
const userCount = await countUsers({ emailPrefix: uniquePrefix });
expect(userCount).toBe(5);
```

**4. CI-only failure (dropdown click)**
- **Cause:** In CI, the screen size may be different (no visible dropdown), or animations are slower
- **Fix:**
```typescript
// Set consistent viewport
await page.setViewportSize({ width: 1280, height: 720 });
// Ensure dropdown is open before clicking item
await page.click('.dropdown-trigger');
await expect(page.locator('.dropdown-menu')).toBeVisible();
await page.click('.dropdown-item');
```

</details>

---

### Exercise 4: Git Workflow

You're working on adding checkout tests. Describe the complete Git workflow from start to finish.

<details>
<summary>Click to see answer</summary>

```bash
# 1. Start from latest main
git checkout main
git pull origin main

# 2. Create feature branch
git checkout -b feature/checkout-e2e-tests

# 3. Write your tests (commit as you go)
# ... write CartPage.ts
git add src/pages/CartPage.ts
git commit -m "Add CartPage page object with item management methods"

# ... write CheckoutPage.ts
git add src/pages/CheckoutPage.ts
git commit -m "Add CheckoutPage page object with form and payment methods"

# ... write test file
git add tests/regression/checkout/checkout-flow.spec.ts
git commit -m "Add E2E tests for standard checkout flow (happy path + error cases)"

# 4. Keep up to date with main
git fetch origin
git rebase origin/main
# (resolve any conflicts if needed)

# 5. Push to remote
git push origin feature/checkout-e2e-tests

# 6. Create Pull Request on GitHub
# - Title: "Add E2E tests for checkout flow"
# - Description: List what tests were added, what they cover
# - Request review from team members
# - CI pipeline runs automatically on PR

# 7. Address review comments
# ... make changes
git add .
git commit -m "Address PR feedback: improve wait strategies and add payment error test"
git push origin feature/checkout-e2e-tests

# 8. After approval, merge via GitHub (squash merge preferred)
# 9. Delete the feature branch
git branch -d feature/checkout-e2e-tests
```

</details>

---

## 📚 Additional Resources

- [Playwright Documentation](https://playwright.dev/) — Official Playwright docs and guides
- [Cypress Documentation](https://docs.cypress.io/) — Cypress getting started and API reference
- [Selenium Documentation](https://www.selenium.dev/documentation/) — Selenium WebDriver docs
- [Page Object Model — Martin Fowler](https://martinfowler.com/bliki/PageObject.html) — Original POM concept
- [Pro Git Book](https://git-scm.com/book/en/v2) — Comprehensive Git reference (free)
- [Test Automation University](https://testautomationu.applitools.com/) — Free courses on test automation

---

## ✅ Self-Check Questions

After completing this course, you should be able to answer:

- [ ] What is the Page Object Model and why is it important?

<details>
<summary>Answer</summary>

The **Page Object Model (POM)** is a design pattern where each page or component of the application is represented by a class. The class encapsulates the page's **selectors** and **actions** (methods), separating them from test logic. It's important because: (1) **Maintainability** — when a UI element changes, you update one page object, not dozens of tests; (2) **Readability** — tests read like business actions (`loginPage.login(user)`) not technical steps; (3) **Reusability** — page methods are shared across multiple tests; (4) **Separation of concerns** — tests define what to verify, page objects define how to interact with the UI.

</details>

- [ ] What is the test automation pyramid and what should each layer contain?

<details>
<summary>Answer</summary>

The test automation pyramid has three main layers: **Base — Unit Tests** (40-60% of tests): test individual functions/methods, very fast, very stable, developer-owned. **Middle — Integration/API Tests** (20-40%): test component interactions and API contracts, medium speed, stable, owned by QA + devs. **Top — E2E/UI Tests** (10-20%): test full user workflows through the browser, slow, can be flaky, QA-owned. The pyramid shape means you should have many more unit tests than E2E tests. Above the pyramid sits **Manual/Exploratory testing** for creative, judgment-based testing that can't be automated.

</details>

- [ ] How do you decide what to automate and what to keep manual?

<details>
<summary>Answer</summary>

**Automate**: repetitive regression tests, smoke tests for critical paths, data-driven tests with many input combinations, API contract validation, cross-browser/cross-platform checks, performance benchmarks — anything that runs frequently and has deterministic expected results. **Keep manual**: exploratory testing (requires creativity and human judgment), usability testing (subjective assessment), visual design review (aesthetic judgment), ad-hoc bug investigation, one-time setup verification, complex business scenarios being tested for the first time. The rule of thumb: if you'll run it more than 3 times, automate it.

</details>

- [ ] What causes flaky tests and how do you fix them?

<details>
<summary>Answer</summary>

Common causes: (1) **Timing issues** — element not ready when test interacts; fix with auto-wait and explicit condition waits instead of `sleep()`. (2) **Test order dependency** — test B needs test A's data; fix with independent setup/teardown per test. (3) **Shared state** — tests modify same database records; fix with unique test data and cleanup. (4) **Environment instability** — slow servers; fix with health checks and retry logic. (5) **Race conditions** — API response arrives after assertion; fix by waiting for API responses. (6) **Date/time dependency** — fails at midnight or weekends; fix with mocked time or relative dates. Management strategy: detect → quarantine → fix → prevent → monitor.

</details>

- [ ] What are the key Git branching strategies and which is best for test automation?

<details>
<summary>Answer</summary>

(1) **Git Flow** — long-lived develop and main branches with feature branches; good for large teams with scheduled releases. (2) **GitHub Flow** — single main branch with short-lived feature branches and PRs; simpler, good for continuous delivery. (3) **Trunk-Based Development** — everyone commits to main with very short branches or feature flags; fastest but requires strong CI/CD. For test automation, **GitHub Flow** is typically the best balance: it's simple, encourages small PRs with code reviews, and works well with CI/CD pipelines. Feature branches keep work isolated while PRs ensure quality through reviews.

</details>

- [ ] Compare Selenium, Playwright, and Cypress — when would you choose each?

<details>
<summary>Answer</summary>

**Selenium**: Choose for enterprise environments with multi-language requirements (Java, C#, Python), existing Selenium infrastructure, or when you need broad browser support including legacy browsers. It's the industry standard with the largest community. **Playwright**: Choose for modern web testing with cross-browser needs (including WebKit/Safari), built-in API testing, excellent debugging (trace viewer), and when you want the best auto-wait reliability. Best for new projects prioritizing quality and speed. **Cypress**: Choose for developer-centric teams writing tests alongside code, when you value DX (time-travel debugging, hot reload), and when you don't need WebKit/Safari support. Best for single-page applications where developers own testing.

</details>

- [ ] What should an automation code review check for?

<details>
<summary>Answer</summary>

Key checks: (1) **Test design** — does each test verify one specific behavior? Is it independent? (2) **Naming** — is the test name descriptive enough to understand what failed? (3) **Selectors** — stable (data-testid, role-based) vs brittle (nth-child, CSS classes)? (4) **Assertions** — specific and meaningful, not just "page loaded"? (5) **Test data** — isolated per test, properly cleaned up? (6) **Waits** — explicit condition waits, no `sleep()` or `waitForTimeout()`? (7) **DRY** — duplicated code that should be a helper or page object? (8) **Error messages** — will failure output be useful for debugging? (9) **Performance** — reasonable execution time, parallelizable?

</details>