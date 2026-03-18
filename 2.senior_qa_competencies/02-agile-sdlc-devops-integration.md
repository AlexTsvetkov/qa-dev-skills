# Course 02: Agile, SDLC & DevOps Integration

## 📖 What You Will Learn

- How QA participates in Agile and Scrum ceremonies
- Owning quality across the full SDLC
- DevOps culture and the continuous testing mindset
- Shift-left quality advocacy and its practical application
- CI/CD test integration with popular tools

---

## 1. Agile & Scrum Participation

### The Agile Manifesto — QA Perspective

The Agile Manifesto values individuals, working software, collaboration, and responding to change. For QA, this means:

| Agile Value | What It Means for QA |
|------------|---------------------|
| **Individuals and interactions** | QA is embedded in the team, not a separate gate |
| **Working software** | Quality is built in, not tested in at the end |
| **Customer collaboration** | QA advocates for the user's perspective |
| **Responding to change** | Test plans adapt as requirements evolve |

### Scrum Roles and QA

In Scrum, there is no explicit "QA role" — but a senior QA is deeply involved:

| Scrum Event | QA Participation |
|-------------|-----------------|
| **Sprint Planning** | Estimate testing effort, identify testability risks, clarify acceptance criteria |
| **Daily Standup** | Report testing progress, flag blockers, coordinate with devs on fixes |
| **Sprint Review / Demo** | Demonstrate tested features, share quality metrics, highlight known issues |
| **Sprint Retrospective** | Propose QA process improvements, discuss defect patterns, celebrate wins |
| **Backlog Refinement** | Review user stories for testability, suggest acceptance criteria, flag missing requirements |

### Definition of Done (DoD) — QA's Role

A senior QA should ensure the **Definition of Done** includes quality criteria:

```
Definition of Done — Example:
✅ Code complete and peer-reviewed
✅ Unit tests written and passing (≥80% coverage)
✅ Integration tests passing
✅ Functional testing complete (manual or automated)
✅ No open critical or blocker bugs
✅ Regression suite passing
✅ Test cases updated in TestRail/Zephyr
✅ Documentation updated (if applicable)
✅ Deployed to staging and smoke-tested
```

### Definition of Ready (DoR) — QA's Input

Before a story enters a sprint, QA should verify:

| Check | Why It Matters |
|-------|---------------|
| **Clear acceptance criteria** | Without them, you can't write test cases |
| **Testable requirements** | Vague requirements = untestable features |
| **Test data identified** | Know what data you need before testing starts |
| **Dependencies identified** | External APIs, other teams' features |
| **UI mockups available** | For UI testing, designs must exist |

### User Story and Acceptance Criteria

A well-written user story follows the format:

```
As a [user role],
I want to [action],
So that [benefit].

Acceptance Criteria:
- Given [precondition], When [action], Then [expected result]
- Given [precondition], When [action], Then [expected result]
```

**Example:**

```
As a registered user,
I want to reset my password via email,
So that I can regain access to my account.

Acceptance Criteria:
- Given I am on the login page, When I click "Forgot Password" and enter my email,
  Then I receive a password reset link within 5 minutes
- Given I have a valid reset link, When I click it and enter a new password,
  Then my password is updated and I can log in with it
- Given I have an expired reset link (>24 hours), When I click it,
  Then I see an error message asking me to request a new link
```

**QA's role**: Review acceptance criteria for completeness, add edge cases, and ensure they're testable.

---

## 2. SDLC Ownership Across Feature Delivery

### The Software Development Life Cycle (SDLC)

A senior QA is involved at **every** stage, not just the testing phase:

```
Requirements → Design → Development → Testing → Deployment → Maintenance
     ↑            ↑          ↑            ↑          ↑            ↑
   QA reviews   QA reviews  QA writes    QA         QA verifies  QA monitors
   for          test        tests in     executes   production   production
   testability  approach    parallel     & reports  deployment   quality
```

### QA Activities by SDLC Phase

| Phase | QA Activities | Deliverables |
|-------|--------------|-------------|
| **Requirements** | Review for testability, identify gaps, clarify ambiguities | Review comments, questions for PO |
| **Design** | Review architecture for testability, plan test approach | Test approach document |
| **Development** | Write test cases, prepare automation, set up test data | Test cases, automation scripts |
| **Testing** | Execute tests, report defects, retest fixes | Test results, defect reports |
| **Deployment** | Verify deployment, run smoke tests, validate configurations | Smoke test results |
| **Maintenance** | Monitor production, analyze escaped defects, update tests | Post-release analysis |

### Shift from "Testing Phase" to "Quality Throughout"

```
Traditional (Waterfall):
Requirements → Design → Code → [====TESTING====] → Deploy
                                   (all at end)

Modern (Agile/DevOps):
Requirements → Design → Code → Deploy
  [QA input]  [QA input] [QA testing in parallel] [QA verification]
```

The key shift: **Quality is everyone's responsibility**, but **QA is the advocate and guardian**.

---

## 3. DevOps Culture and Continuous Testing Mindset

### What is DevOps?

DevOps is a **culture and set of practices** that brings together development and operations to deliver software faster and more reliably.

```
     Dev                    Ops
   (Build)    →  DevOps  ← (Run)
              Collaboration
              Automation
              Measurement
              Sharing
```

### The Infinite Loop of DevOps

```
     Plan → Code → Build → Test → Release → Deploy → Operate → Monitor
       ↑                                                          ↓
       └──────────────────── Feedback ←──────────────────────────┘
```

### Where QA Fits in DevOps

| DevOps Stage | QA Contribution |
|-------------|----------------|
| **Plan** | Define quality criteria, risk assessment, test planning |
| **Code** | Unit test standards, code review participation, TDD advocacy |
| **Build** | Automated test execution in CI pipeline |
| **Test** | Functional, integration, regression, performance testing |
| **Release** | Release readiness assessment, go/no-go decision |
| **Deploy** | Smoke testing, deployment verification |
| **Operate** | Production monitoring, incident analysis |
| **Monitor** | Quality metrics tracking, escaped defect analysis |

### Continuous Testing

**Continuous Testing** = testing at every stage of the delivery pipeline, automatically.

```
Code Commit → Unit Tests → Build → Integration Tests → System Tests → Deploy to Staging → E2E Tests → Deploy to Prod → Smoke Tests → Monitoring
     ↓            ↓         ↓           ↓                   ↓                ↓               ↓             ↓              ↓            ↓
   seconds     seconds   minutes     minutes             minutes          minutes         minutes       minutes        seconds     ongoing
```

**Principles of Continuous Testing:**

1. **Automate everything possible** — manual gates slow the pipeline
2. **Fast feedback** — tests should fail fast so devs get immediate results
3. **Test in production-like environments** — avoid "works on my machine"
4. **Shift left** — catch defects as early as possible
5. **Shift right** — monitor production for issues that slip through
6. **Risk-based execution** — run the most critical tests first

### Testing in Production (Shift-Right)

| Technique | Description |
|-----------|-------------|
| **Canary Releases** | Deploy to a small % of users first, monitor for issues |
| **Feature Flags** | Toggle features on/off without deployment |
| **A/B Testing** | Compare two versions with real user traffic |
| **Synthetic Monitoring** | Automated scripts that simulate user actions in production |
| **Chaos Engineering** | Intentionally inject failures to test resilience |

---

## 4. Shift-Left Quality Advocacy

### What is Shift-Left?

**Shift-left** means moving testing and quality activities **earlier** in the development lifecycle. The earlier you find a bug, the cheaper it is to fix.

### Cost of Defects by Phase:

```
Cost to Fix ↑
            │                                          ████
            │                                    ████  ████
            │                              ████  ████  ████
            │                        ████  ████  ████  ████
            │                  ████  ████  ████  ████  ████
            │            ████  ████  ████  ████  ████  ████
            │      ████  ████  ████  ████  ████  ████  ████
            │████  ████  ████  ████  ████  ████  ████  ████
            └─────────────────────────────────────────────→
             Req  Design  Code  Unit  Int  System  UAT  Prod
                                Test  Test  Test
```

A bug found in production can cost **100x more** than the same bug found during requirements.

### Shift-Left Practices for Senior QA:

| Practice | How to Implement |
|----------|-----------------|
| **Requirements Reviews** | Participate in story refinement, challenge vague requirements |
| **Testability Advocacy** | Ensure features are designed to be testable (APIs, logging, data access) |
| **BDD/ATDD** | Write acceptance tests before development starts |
| **Pair Testing** | Work alongside developers during implementation |
| **Static Analysis** | Encourage linting, code analysis tools in the pipeline |
| **Unit Test Standards** | Advocate for developer-written unit tests with coverage targets |
| **Early Environment Access** | Get access to dev/integration environments early |
| **Prototype Testing** | Test designs and prototypes before development begins |

### BDD (Behavior-Driven Development) Example:

```gherkin
Feature: User Login

  Scenario: Successful login with valid credentials
    Given the user is on the login page
    And the user has a registered account with email "nastya@example.com"
    When the user enters email "nastya@example.com" and password "SecurePass123"
    And clicks the "Login" button
    Then the user should be redirected to the dashboard
    And should see a welcome message "Hello, Nastya!"

  Scenario: Failed login with wrong password
    Given the user is on the login page
    When the user enters email "nastya@example.com" and password "WrongPassword"
    And clicks the "Login" button
    Then the user should see an error message "Invalid email or password"
    And should remain on the login page
```

BDD scenarios serve as **living documentation**, **acceptance tests**, and **communication tools** between QA, dev, and product.

---

## 5. CI/CD Test Integration

### What is CI/CD?

| Term | Definition |
|------|-----------|
| **CI (Continuous Integration)** | Developers merge code frequently; each merge triggers automated builds and tests |
| **CD (Continuous Delivery)** | Code is always in a deployable state; release is a business decision |
| **CD (Continuous Deployment)** | Every change that passes the pipeline is automatically deployed to production |

### CI/CD Pipeline with Tests:

```
Developer pushes code
        ↓
┌──────────────────┐
│  CI Pipeline      │
│                   │
│  1. Checkout code │
│  2. Build         │
│  3. Unit Tests    │  ← Fast (seconds-minutes)
│  4. Lint / SAST   │
│  5. Publish       │
└──────────────────┘
        ↓ (on success)
┌──────────────────┐
│  CD Pipeline      │
│                   │
│  6. Deploy to Dev │
│  7. Integration   │  ← Medium (minutes)
│     Tests         │
│  8. Deploy to QA  │
│  9. E2E Tests     │  ← Slower (minutes-hours)
│ 10. Performance   │
│     Tests         │
│ 11. Deploy to     │
│     Staging       │
│ 12. Smoke Tests   │
│ 13. Deploy to     │  ← Continuous Deployment
│     Production    │
│ 14. Post-deploy   │
│     Verification  │
└──────────────────┘
```

### Popular CI/CD Tools

#### GitHub Actions

```yaml
# .github/workflows/test.yml
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
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run test:unit
      
  integration-tests:
    needs: unit-tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run test:integration
      
  e2e-tests:
    needs: integration-tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npx playwright install
      - run: npm run test:e2e
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: test-results
          path: test-results/
```

#### Jenkins (Declarative Pipeline)

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                sh 'npm ci'
                sh 'npm run build'
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh 'npm run test:unit'
            }
            post {
                always {
                    junit 'test-results/unit/*.xml'
                }
            }
        }
        
        stage('Integration Tests') {
            steps {
                sh 'npm run test:integration'
            }
        }
        
        stage('Deploy to QA') {
            steps {
                sh './deploy.sh qa'
            }
        }
        
        stage('E2E Tests') {
            steps {
                sh 'npm run test:e2e'
            }
            post {
                always {
                    publishHTML(target: [
                        reportName: 'E2E Test Report',
                        reportDir: 'test-results/e2e',
                        reportFiles: 'index.html'
                    ])
                }
            }
        }
    }
    
    post {
        failure {
            slackSend channel: '#qa-alerts',
                      message: "Pipeline failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
    }
}
```

#### GitLab CI/CD

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy
  - e2e

build:
  stage: build
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/

unit-tests:
  stage: test
  script:
    - npm run test:unit
  artifacts:
    reports:
      junit: test-results/unit.xml

integration-tests:
  stage: test
  script:
    - npm run test:integration

deploy-qa:
  stage: deploy
  script:
    - ./deploy.sh qa
  environment:
    name: qa

e2e-tests:
  stage: e2e
  script:
    - npx playwright install
    - npm run test:e2e
  artifacts:
    when: always
    paths:
      - test-results/
```

#### Azure DevOps

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
      - develop

pool:
  vmImage: 'ubuntu-latest'

stages:
  - stage: Build
    jobs:
      - job: BuildAndTest
        steps:
          - task: NodeTool@0
            inputs:
              versionSpec: '20.x'
          - script: npm ci
          - script: npm run build
          - script: npm run test:unit
          - task: PublishTestResults@2
            inputs:
              testResultsFormat: 'JUnit'
              testResultsFiles: 'test-results/*.xml'

  - stage: QA
    dependsOn: Build
    jobs:
      - job: E2ETests
        steps:
          - script: npm run test:e2e
          - task: PublishTestResults@2
            inputs:
              testResultsFormat: 'JUnit'
              testResultsFiles: 'e2e-results/*.xml'
```

### Test Pyramid in CI/CD

```
                    ╱╲
                   ╱  ╲        E2E Tests
                  ╱ UI ╲       (Few, slow, expensive)
                 ╱──────╲      Run on deploy to staging
                ╱        ╲
               ╱ Service  ╲    Integration/API Tests
              ╱  (API)     ╲   (Medium count, medium speed)
             ╱──────────────╲  Run on deploy to dev/QA
            ╱                ╲
           ╱     Unit Tests   ╲ Unit Tests
          ╱                    ╲(Many, fast, cheap)
         ╱______________________╲Run on every commit
```

### QA's Role in CI/CD:

| Responsibility | What You Do |
|---------------|-------------|
| **Pipeline Design** | Define which tests run at which stage |
| **Test Selection** | Determine the right test suite for each trigger |
| **Failure Analysis** | Investigate failed pipelines, distinguish real bugs from flaky tests |
| **Maintenance** | Keep test suites green, update tests when features change |
| **Metrics** | Track pipeline pass rates, test execution time, flaky test rates |
| **Optimization** | Parallelize tests, use test impact analysis, reduce execution time |

---

## 📝 Exercises

### Exercise 1: Sprint Planning Scenario

You're in sprint planning. The team is discussing these user stories:
1. User can filter products by category (5 story points)
2. Admin can export sales report to CSV (3 story points)
3. User can leave product reviews (8 story points)
4. Fix: Cart total doesn't update when changing quantity (2 story points)

Questions:
1. For each story, what testing concerns would you raise?
2. Which story has the highest testing risk? Why?
3. What would you add to the Definition of Done?

<details>
<summary>Click to see answers</summary>

**1. Testing concerns per story:**

| Story | Testing Concerns |
|-------|-----------------|
| Filter products | Performance with large catalogs, multiple filter combinations, empty results, URL state persistence |
| Export CSV | Large dataset handling, special characters in data, file format validation, timeout on big exports |
| Product reviews | Input validation (XSS), profanity filter, rating calculation, pagination, authentication requirement |
| Cart total fix | Regression risk — test all cart operations, concurrency issues, decimal precision, currency formatting |

**2. Highest testing risk: Product reviews (8 SP)**
- It's a new feature (no existing test base)
- Has security implications (user-generated content → XSS, injection)
- Touches multiple systems (auth, product catalog, notifications)
- Highest complexity (8 story points)

**3. Definition of Done additions:**
- Acceptance criteria verified by QA
- Automated regression tests updated
- Security review for user input features
- Performance tested for data-heavy features (CSV export, product filtering)
- Cross-browser verification for UI changes

</details>

---

### Exercise 2: Design a CI/CD Pipeline

Design a CI/CD pipeline for a web application with the following requirements:
- React frontend + Node.js backend
- PostgreSQL database
- Deployed to AWS

Specify: What tests run at each stage? What's the trigger? What happens on failure?

<details>
<summary>Click to see answer</summary>

```
Trigger: Push to any branch / PR to main
│
├── Stage 1: Build & Unit Tests (every commit)
│   ├── Frontend: npm run build, npm run test:unit
│   ├── Backend: npm run build, npm run test:unit
│   ├── Linting: eslint, prettier check
│   └── On failure: Block merge, notify developer
│
├── Stage 2: Integration Tests (every commit to develop/main)
│   ├── Start PostgreSQL container
│   ├── Run database migrations
│   ├── Backend API integration tests
│   └── On failure: Block pipeline, notify team
│
├── Stage 3: Deploy to Dev (on merge to develop)
│   ├── Deploy frontend + backend to dev environment
│   ├── Run smoke tests (health checks, critical paths)
│   └── On failure: Rollback, notify QA
│
├── Stage 4: E2E Tests (on deploy to dev)
│   ├── Playwright E2E test suite
│   ├── API contract tests
│   ├── Upload test reports + screenshots
│   └── On failure: Flag to QA team, don't promote
│
├── Stage 5: Deploy to Staging (on merge to main)
│   ├── Deploy to production-like environment
│   ├── Run full regression suite
│   ├── Performance tests (k6 load test)
│   └── On failure: Block release, create incident ticket
│
├── Stage 6: Deploy to Production (manual approval)
│   ├── Blue-green deployment
│   ├── Smoke tests against production
│   ├── Synthetic monitoring activated
│   └── On failure: Automatic rollback
│
└── Ongoing: Monitoring
    ├── Error rate monitoring (Datadog/CloudWatch)
    ├── Synthetic user journey monitoring
    └── Alerts → on-call team
```

</details>

---

### Exercise 3: Shift-Left Analysis

Your team has the following problem: 40% of bugs are found during the E2E testing phase, causing sprint delays. Propose a shift-left strategy to catch these bugs earlier.

<details>
<summary>Click to see answer</summary>

**Root Cause Analysis:**
If 40% of bugs are found in E2E testing, it likely means:
- Insufficient unit and integration testing
- Requirements are unclear or incomplete
- Developers aren't testing their own code adequately

**Shift-Left Strategy:**

| Action | Expected Impact | Timeline |
|--------|----------------|----------|
| **Strengthen DoR** — No story enters sprint without clear acceptance criteria | Prevents misunderstandings | Immediate |
| **BDD/ATDD adoption** — Write acceptance tests before development | Catches logic errors before coding | 1-2 sprints |
| **Developer testing standards** — Require ≥80% unit test coverage | Catches code-level bugs earlier | 2-3 sprints |
| **API-level test expansion** — More integration tests at the API layer | Catches integration bugs without slow UI tests | 2-3 sprints |
| **Pair testing sessions** — QA + Dev test together during development | Catches UI/UX issues before E2E phase | Immediate |
| **Static analysis in CI** — Add SonarQube/ESLint to pipeline | Catches code quality issues on commit | 1 sprint |
| **Story kickoffs** — QA meets with dev before story starts to discuss test approach | Aligns on edge cases early | Immediate |

**Expected Outcome:**
- Reduce E2E-phase bug discovery from 40% to <15%
- Increase unit/integration-phase bug discovery
- Reduce sprint delays caused by late-found bugs
- Faster feedback loops for developers

</details>

---

### Exercise 4: Agile Ceremonies Participation

Write down what you would say/do as a senior QA in each of these scenarios:

1. **Sprint Retrospective**: The team shipped a bug to production last sprint
2. **Backlog Refinement**: A user story has no acceptance criteria
3. **Daily Standup**: Your E2E tests have been failing for 2 days due to environment issues
4. **Sprint Review**: A stakeholder asks "How confident are we in this release?"

<details>
<summary>Click to see answers</summary>

**1. Sprint Retrospective — Production Bug:**
"Let's analyze the escaped defect. The bug was in the payment rounding logic. It wasn't caught because: (a) we didn't have a test case for amounts with 3+ decimal places, and (b) our test data didn't include edge-case amounts. **Action items**: I'll add boundary value test cases for all financial calculations, and I'll create a test data checklist for payment-related stories."

**2. Backlog Refinement — Missing Acceptance Criteria:**
"This story isn't ready for sprint yet. 'User can manage their profile' is too vague. We need acceptance criteria in Given/When/Then format. Specifically: What fields can be edited? What validation rules apply? Can users change their email (this affects authentication)? Are there any fields that require admin approval to change? I'll work with the PO to define these before the next planning."

**3. Daily Standup — Environment Issues:**
"Blocker: Our QA environment has been down for 2 days, blocking E2E test execution for stories X, Y, and Z. I've raised a ticket with DevOps (INFRA-456). In the meantime, I'm running what I can locally and focusing on API-level tests that don't depend on the QA environment. I need help escalating the environment issue — can the scrum master follow up with the DevOps team today?"

**4. Sprint Review — Confidence Question:**
"We executed 95% of planned tests with a 97% pass rate. All critical and blocker bugs are resolved. Two medium bugs remain — they're cosmetic issues in the admin panel and don't affect end users. Regression suite is 100% green. Performance testing shows response times within SLA. I'm confident this release is production-ready. The two remaining mediums are documented as known issues and scheduled for next sprint."

</details>

---

## 📚 Additional Resources

- [The Scrum Guide](https://scrumguides.org/) — Official Scrum framework documentation
- [Continuous Testing in DevOps by Dan Ashby](https://danashby.co.uk/) — Practical continuous testing concepts
- [Accelerate by Nicole Forsgren et al.](https://itrevolution.com/book/accelerate/) — Research on DevOps performance metrics
- [The DevOps Handbook](https://itrevolution.com/book/the-devops-handbook/) — Comprehensive DevOps practices guide
- [GitHub Actions Documentation](https://docs.github.com/en/actions) — CI/CD with GitHub Actions
- [Jenkins Documentation](https://www.jenkins.io/doc/) — Jenkins pipeline reference

---

## ✅ Self-Check Questions

After completing this course, you should be able to answer:

- [ ] What is the QA role in each Scrum ceremony?

<details>
<summary>Answer</summary>

**Sprint Planning**: Estimate testing effort, identify testability risks, clarify acceptance criteria, ensure stories are testable. **Daily Standup**: Report testing progress, flag blockers, coordinate with developers on bug fixes. **Sprint Review/Demo**: Demonstrate tested features, share quality metrics, highlight known issues and risks. **Sprint Retrospective**: Propose QA process improvements, analyze defect patterns, suggest shift-left practices. **Backlog Refinement**: Review user stories for testability, suggest acceptance criteria, flag missing requirements or edge cases.

</details>

- [ ] What is the Definition of Done and how should QA influence it?

<details>
<summary>Answer</summary>

The **Definition of Done (DoD)** is a shared agreement on what criteria must be met before a user story or increment is considered complete. QA should influence the DoD by ensuring it includes quality criteria such as: unit tests written and passing with coverage targets, functional testing complete, no open critical/blocker bugs, regression suite passing, test cases documented and updated, acceptance criteria verified, deployment to staging and smoke-tested. The DoD should be a team agreement, but QA advocates for the quality gates within it.

</details>

- [ ] What is shift-left testing and why is it important?

<details>
<summary>Answer</summary>

**Shift-left testing** means moving testing and quality activities earlier in the development lifecycle — from the traditional "test at the end" approach to testing throughout the process. It's important because the cost of fixing defects increases exponentially the later they're found (a bug found in production costs ~100x more than one found during requirements). Shift-left practices include: requirements reviews, BDD/ATDD (writing tests before code), developer unit testing standards, pair testing during development, static code analysis in CI, and story kickoffs where QA and dev align on test approach before coding begins.

</details>

- [ ] How does continuous testing differ from traditional testing?

<details>
<summary>Answer</summary>

**Traditional testing** happens in a distinct phase after development, typically as a gateway before release. **Continuous testing** integrates testing into every stage of the delivery pipeline automatically. Differences: (1) traditional is phase-based, continuous is ongoing; (2) traditional relies heavily on manual execution, continuous is primarily automated; (3) traditional provides feedback in days/weeks, continuous provides feedback in minutes; (4) traditional tests in isolated environments, continuous tests in production-like and production environments; (5) continuous testing includes both shift-left (earlier testing) and shift-right (production monitoring, canary releases, synthetic monitoring).

</details>

- [ ] What tests should run at each stage of a CI/CD pipeline?

<details>
<summary>Answer</summary>

**On every commit**: Unit tests, linting, static analysis — these are fast (seconds to minutes) and provide immediate feedback. **On merge/PR**: Integration tests, API contract tests — medium speed, validate component interactions. **On deploy to dev/QA**: End-to-end tests, functional tests — slower but validate full user workflows. **On deploy to staging**: Full regression suite, performance tests, security scans — comprehensive validation before production. **On deploy to production**: Smoke tests, health checks — quick verification that deployment succeeded. **Ongoing in production**: Synthetic monitoring, error rate tracking, alerting — continuous production validation.

</details>

- [ ] Name at least 3 CI/CD tools and their key features.

<details>
<summary>Answer</summary>

(1) **GitHub Actions** — YAML-based workflows integrated into GitHub, supports matrix builds, artifact management, and marketplace of reusable actions. Great for open-source and GitHub-centric teams. (2) **Jenkins** — Open-source, highly customizable with plugins, supports declarative and scripted pipelines. Best for complex enterprise setups with custom requirements. (3) **GitLab CI/CD** — Built into GitLab, YAML-based, supports Auto DevOps, container registry, and environments. Great for teams using GitLab for source control. (4) **Azure DevOps** — Microsoft's platform with build pipelines, release management, and tight Azure cloud integration. Best for Microsoft-stack teams.

</details>

- [ ] What is the DevOps infinite loop and where does QA fit?

<details>
<summary>Answer</summary>

The DevOps infinite loop consists of: **Plan → Code → Build → Test → Release → Deploy → Operate → Monitor → Plan** (continuous cycle). QA fits at every stage: during **Plan** (risk assessment, quality criteria), **Code** (unit test standards, code reviews), **Build** (automated test execution), **Test** (functional, integration, regression testing), **Release** (readiness assessment), **Deploy** (smoke testing, verification), **Operate** (production monitoring), and **Monitor** (quality metrics, escaped defect analysis). QA is not a separate phase but a cross-cutting concern throughout the entire DevOps lifecycle.

</details>

- [ ] How would you advocate for shift-left practices in a team that resists change?

<details>
<summary>Answer</summary>

Advocacy strategy: (1) **Use data** — track where bugs are found and show the cost of late detection (e.g., "40% of our bugs are found in E2E testing, causing X days of sprint delay"); (2) **Start small** — propose one practice at a time (e.g., story kickoffs before coding) rather than a complete overhaul; (3) **Demonstrate value** — run a pilot with one team/feature and measure the improvement; (4) **Make it easy** — provide templates, examples, and tooling (e.g., BDD scenario templates, pre-configured linting); (5) **Celebrate wins** — when shift-left catches a bug early, share the story; (6) **Involve leadership** — present the business case (faster releases, fewer production incidents, reduced rework costs).

</details>