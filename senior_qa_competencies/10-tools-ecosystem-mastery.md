# Course 10: Tools & Ecosystem Mastery

## 📖 What You Will Learn

- Issue & test management tools (Jira, TestRail, Zephyr)
- CI/CD tools (GitHub Actions, Jenkins, GitLab CI/CD, Azure DevOps)
- Source control tools (Git, Bitbucket)
- Automation IDEs and debuggers
- Cloud environments and containers (Docker, Kubernetes)
- Observability / monitoring integrations (logs, alerts, dashboards)

---

## 1. Issue & Test Management Tools

### Jira — The Industry Standard

| Concept | Description |
|---------|-------------|
| **Epic** | Large feature or initiative (e.g., "Checkout Redesign") |
| **Story** | User-facing feature ("As a user, I can pay with Apple Pay") |
| **Task** | Technical work item ("Set up Playwright framework") |
| **Bug** | Defect report with reproduction steps |
| **Sub-task** | Breakdown of a story/task into smaller pieces |

### Jira Workflows for QA:

```
Bug Lifecycle:
  Open → In Progress → In Review → QA Verification → Done
                                        ↓
                                    Reopened → In Progress

Story Testing:
  To Do → In Development → Code Review → QA Testing → Done
                                            ↓
                                      Failed QA → In Development
```

### JQL (Jira Query Language) for QA:

```sql
-- Find all open bugs assigned to you
project = PROJ AND type = Bug AND status != Done AND assignee = currentUser()

-- Bugs created this sprint
project = PROJ AND type = Bug AND sprint in openSprints()

-- Critical bugs without assignee
project = PROJ AND type = Bug AND priority in (Critical, Blocker) AND assignee is EMPTY

-- Issues updated in last 24 hours
project = PROJ AND updated >= -24h

-- Bugs by component
project = PROJ AND type = Bug AND component = "Checkout" ORDER BY priority DESC

-- Release readiness: unresolved issues in fix version
project = PROJ AND fixVersion = "3.2.0" AND resolution is EMPTY
```

### Test Management Tools:

| Tool | Strengths | Best For |
|------|-----------|----------|
| **TestRail** | Comprehensive test case management, reporting | Structured test management |
| **Zephyr** | Native Jira integration | Teams already in Jira |
| **qTest** | Enterprise features, requirements traceability | Large organizations |
| **Xray** | Jira-native, supports BDD | Agile teams using Jira |
| **Azure Test Plans** | Integrated with Azure DevOps | Microsoft ecosystem teams |

### Test Case Structure in TestRail:

```
Test Suite: Checkout
  └── Section: Payment Methods
       ├── TC-001: Pay with credit card (Visa)
       ├── TC-002: Pay with credit card (Mastercard)
       ├── TC-003: Pay with PayPal
       ├── TC-004: Pay with Apple Pay
       └── TC-005: Pay with gift card

Test Run: Sprint 24 Regression
  - Includes: All Checkout tests + User Management tests
  - Assigned to: Nastya
  - Due: March 15, 2025
  - Results: 45/50 Passed, 3 Failed, 2 Blocked
```

---

## 2. CI/CD Tools

### GitHub Actions:

```yaml
# .github/workflows/test.yml
name: Run Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm test

  api-tests:
    runs-on: ubuntu-latest
    needs: unit-tests  # Run after unit tests pass
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run test:api

  e2e-tests:
    runs-on: ubuntu-latest
    needs: api-tests
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

### Jenkins Pipeline:

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                sh 'npm ci'
            }
        }
        stage('Unit Tests') {
            steps {
                sh 'npm test'
            }
        }
        stage('API Tests') {
            steps {
                sh 'npm run test:api'
            }
        }
        stage('E2E Tests') {
            steps {
                sh 'npx playwright test'
            }
            post {
                always {
                    publishHTML(target: [
                        reportName: 'Playwright Report',
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html'
                    ])
                }
            }
        }
    }
    
    post {
        failure {
            slackSend channel: '#qa-alerts',
                      message: "Tests failed: ${env.BUILD_URL}"
        }
    }
}
```

### GitLab CI/CD:

```yaml
# .gitlab-ci.yml
stages:
  - test
  - e2e
  - deploy

unit-tests:
  stage: test
  image: node:20
  script:
    - npm ci
    - npm test
  artifacts:
    reports:
      junit: test-results.xml

api-tests:
  stage: test
  image: node:20
  script:
    - npm ci
    - npm run test:api

e2e-tests:
  stage: e2e
  image: mcr.microsoft.com/playwright:v1.40.0
  script:
    - npm ci
    - npx playwright test
  artifacts:
    when: always
    paths:
      - playwright-report/
    expire_in: 7 days
```

### CI/CD Comparison:

| Feature | GitHub Actions | Jenkins | GitLab CI | Azure DevOps |
|---------|---------------|---------|-----------|-------------|
| **Hosting** | Cloud | Self-hosted / Cloud | Cloud / Self-hosted | Cloud |
| **Config** | YAML | Groovy/YAML | YAML | YAML |
| **Free Tier** | 2000 min/month | Free (self-host) | 400 min/month | 1800 min/month |
| **Docker** | Native | Plugin | Native | Native |
| **Marketplace** | Large | Large plugin ecosystem | Moderate | Moderate |
| **Best For** | GitHub repos | Enterprise, complex pipelines | GitLab repos | Microsoft ecosystem |

---

## 3. Source Control with Git

### Essential Git Commands for QA:

```bash
# Clone a repository
git clone https://github.com/org/repo.git

# Create a feature branch for tests
git checkout -b feature/add-checkout-tests

# Check current status
git status

# Stage and commit changes
git add tests/checkout.spec.ts
git commit -m "Add checkout payment tests"

# Push to remote
git push origin feature/add-checkout-tests

# Update your branch with latest main
git fetch origin
git rebase origin/main

# View commit history
git log --oneline -10

# See what changed in a file
git diff tests/checkout.spec.ts

# Stash changes temporarily
git stash
git stash pop

# Cherry-pick a specific commit
git cherry-pick abc1234
```

### Branching Strategies:

```
GitFlow:
  main ─────────────────────────────── production
    └── develop ──────────────────── integration
         ├── feature/login-tests ── your work
         ├── feature/api-tests ──── team member
         └── release/3.2 ────────── release branch

Trunk-Based:
  main ─────────────────────────────── always deployable
    ├── short-lived-branch-1 (1-2 days)
    ├── short-lived-branch-2 (1-2 days)
    └── feature flags for incomplete features
```

### Pull Request Best Practices for QA:

```markdown
# PR: Add checkout payment method tests

## What
- Added 15 new E2E tests for payment methods
- Added API contract tests for /api/payments endpoint
- Fixed flaky test in cart module (replaced hard wait with locator)

## Test Coverage
- Credit card (Visa, MC, Amex): ✅
- PayPal: ✅
- Apple Pay: ✅ (mocked)
- Google Pay: ❌ (blocked by sandbox setup — tracked in JIRA-456)

## How to Review
1. Run: `npx playwright test tests/checkout/`
2. Check: All 15 tests pass
3. Note: Tests require staging environment to be up

## Related
- JIRA-123: Checkout payment testing
- Depends on: PR #45 (API mock setup)
```

---

## 4. Automation IDEs and Debuggers

### VS Code for Test Automation:

| Extension | Purpose |
|-----------|---------|
| **Playwright Test for VS Code** | Run/debug Playwright tests, pick locators |
| **REST Client** | Send HTTP requests from editor |
| **GitLens** | Git blame, history, comparisons |
| **ESLint/Prettier** | Code quality and formatting |
| **Thunder Client** | API testing within VS Code |
| **Database Client** | Query databases directly |

### Debugging Playwright Tests:

```typescript
// Method 1: Headed mode (see the browser)
npx playwright test --headed

// Method 2: Debug mode (step through)
npx playwright test --debug

// Method 3: Pause in code
test('checkout flow', async ({ page }) => {
  await page.goto('/checkout');
  await page.pause(); // Opens inspector — interact manually
  await page.fill('#email', 'test@example.com');
});

// Method 4: Trace viewer (post-mortem debugging)
npx playwright test --trace on
// Then: npx playwright show-trace trace.zip

// Method 5: VS Code breakpoints
// Set breakpoint in IDE → Run test in debug mode
```

### IntelliJ IDEA for QA:

| Feature | Use Case |
|---------|----------|
| **Database Tools** | Connect to DB, run queries, view data |
| **HTTP Client** | `.http` files for API testing |
| **Git Integration** | Visual diff, merge, blame |
| **Terminal** | Run tests, scripts |
| **Debugger** | Step through Java/Kotlin test code |

---

## 5. Cloud Environments and Containers

### Docker for QA:

```dockerfile
# Dockerfile for test environment
FROM mcr.microsoft.com/playwright:v1.40.0

WORKDIR /tests
COPY package*.json ./
RUN npm ci
COPY . .

CMD ["npx", "playwright", "test"]
```

```bash
# Build and run tests in Docker
docker build -t my-tests .
docker run --rm my-tests

# Run with environment variables
docker run --rm -e BASE_URL=https://staging.example.com my-tests

# Docker Compose for tests with dependencies
```

```yaml
# docker-compose.test.yml
version: '3.8'
services:
  app:
    image: my-app:latest
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgres://db:5432/test
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=test
      - POSTGRES_PASSWORD=test

  tests:
    build: ./tests
    environment:
      - BASE_URL=http://app:3000
    depends_on:
      - app
```

### Kubernetes Basics for QA:

```bash
# View pods (running containers)
kubectl get pods

# View logs of a specific pod
kubectl logs pod-name -f

# Check pod health
kubectl describe pod pod-name

# Port-forward to access a service locally
kubectl port-forward svc/my-service 8080:80

# View environment variables
kubectl exec pod-name -- env

# Common QA tasks:
# - Check which version is deployed
kubectl get deployment my-app -o jsonpath='{.spec.template.spec.containers[0].image}'

# - Restart a deployment (after config change)
kubectl rollout restart deployment/my-app

# - Check deployment status
kubectl rollout status deployment/my-app
```

### Environment Management:

| Environment | Purpose | Data |
|-------------|---------|------|
| **Local** | Developer testing | Mock/seed data |
| **Dev** | Integration testing | Shared test data |
| **Staging** | Pre-production testing | Production-like data |
| **Production** | Live users | Real data (read-only for QA) |

---

## 6. Observability and Monitoring Integrations

### Log Analysis for QA:

```bash
# Search logs in Kubernetes
kubectl logs pod-name | grep "ERROR"
kubectl logs pod-name --since=1h | grep "payment"

# Search with context (lines before/after match)
kubectl logs pod-name | grep -B 5 -A 5 "NullPointerException"

# ELK Stack query (Kibana)
# Find errors in checkout service in last hour:
# service: "checkout" AND level: "ERROR" AND @timestamp > now-1h
```

### Grafana Dashboard for QA:

```
Key Dashboards to Monitor:

1. APPLICATION HEALTH
   - Request rate (req/sec)
   - Error rate (% 5xx responses)
   - Response time (P50, P95, P99)
   - Active connections

2. INFRASTRUCTURE
   - CPU utilization per pod
   - Memory usage per pod
   - Disk I/O
   - Network throughput

3. BUSINESS METRICS
   - Orders per minute
   - Payment success rate
   - User registration rate
   - Cart abandonment rate

4. TEST HEALTH
   - CI pipeline pass rate
   - Test execution duration trend
   - Flaky test count
   - Code coverage trend
```

### Alert Configuration:

| Alert | Condition | Severity | Action |
|-------|-----------|----------|--------|
| High error rate | 5xx > 5% for 5 min | Critical | Page on-call, notify QA |
| Slow response | P95 > 2s for 10 min | Warning | Investigate, check recent deploys |
| Test failures | CI pass rate < 90% | Warning | Review failed tests, fix or quarantine |
| Deployment failed | Rollback triggered | Critical | Verify rollback successful |

---

## 📝 Exercises

### Exercise 1: JQL Queries

Write JQL queries for the following scenarios:

1. All bugs in the current sprint that are not resolved
2. High-priority issues created in the last 7 days
3. Issues assigned to you that are in QA status
4. Bugs in the "Checkout" component sorted by creation date

<details>
<summary>Click to see answers</summary>

```sql
-- 1. Unresolved bugs in current sprint
project = PROJ AND type = Bug AND sprint in openSprints() AND resolution is EMPTY

-- 2. High-priority issues from last 7 days
project = PROJ AND priority in (High, Critical, Blocker) AND created >= -7d

-- 3. My issues in QA status
project = PROJ AND assignee = currentUser() AND status = "QA Testing"

-- 4. Checkout bugs by creation date
project = PROJ AND type = Bug AND component = "Checkout" ORDER BY created DESC
```

</details>

### Exercise 2: CI/CD Pipeline Design

Design a GitHub Actions pipeline that runs unit tests, API tests, and E2E tests. E2E tests should only run on the main branch. All tests should upload reports on failure.

<details>
<summary>Click to see answer</summary>

```yaml
name: Test Pipeline
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npm test -- --reporter=junit --output=results.xml
      - uses: actions/upload-artifact@v4
        if: failure()
        with: { name: unit-test-report, path: results.xml }

  api-tests:
    runs-on: ubuntu-latest
    needs: unit-tests
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npm run test:api
      - uses: actions/upload-artifact@v4
        if: failure()
        with: { name: api-test-report, path: api-results/ }

  e2e-tests:
    runs-on: ubuntu-latest
    needs: api-tests
    if: github.ref == 'refs/heads/main'  # Only on main
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test
      - uses: actions/upload-artifact@v4
        if: always()
        with: { name: playwright-report, path: playwright-report/ }
```

</details>

---

## ✅ Self-Check Questions

- [ ] What JQL queries do you use most as a QA engineer?

<details>
<summary>Answer</summary>

Essential JQL for QA: (1) **Sprint bugs**: `type = Bug AND sprint in openSprints()` — track current sprint quality. (2) **My assignments**: `assignee = currentUser() AND status = "QA Testing"` — daily work queue. (3) **Release readiness**: `fixVersion = "X.Y" AND resolution is EMPTY` — what's still open for release. (4) **Bug trends**: `type = Bug AND created >= -30d` — monthly bug creation rate. (5) **Unassigned critical**: `type = Bug AND priority = Critical AND assignee is EMPTY` — urgent items needing attention.

</details>

- [ ] How do you integrate tests into a CI/CD pipeline?

<details>
<summary>Answer</summary>

Follow the test pyramid in CI: (1) **Unit tests** run on every push — fast feedback. (2) **API/integration tests** run after unit tests pass. (3) **E2E tests** run on main branch or before release — slower but comprehensive. Key practices: fail fast (stop pipeline on first failure), upload artifacts on failure (reports, screenshots), parallelize where possible, use caching for dependencies, set up notifications for failures. Tools: GitHub Actions (YAML), Jenkins (Groovy), GitLab CI (YAML).

</details>

- [ ] What Git workflows should a QA engineer know?

<details>
<summary>Answer</summary>

Essential Git skills: (1) **Branching** — create feature branches for test code. (2) **Pull requests** — submit test code for review. (3) **Rebasing** — keep your branch updated with main. (4) **Cherry-picking** — apply specific commits. (5) **Stashing** — temporarily save work. Understanding branching strategies (GitFlow vs trunk-based) helps QA know when and how code reaches each environment and when to test.

</details>

- [ ] How do Docker and Kubernetes help QA?

<details>
<summary>Answer</summary>

**Docker**: (1) Consistent test environments — "works on my machine" solved. (2) Run tests in containers matching CI. (3) Spin up dependencies (databases, services) for integration tests. (4) Docker Compose for multi-service test setups. **Kubernetes**: (1) Understand where the app runs in production. (2) Read logs from pods for debugging. (3) Port-forward to access services. (4) Verify deployments and rollbacks. (5) Check which version is deployed in each environment.

</details>