# Course 2: Methodologies & DevOps

## 📖 What You Will Learn

- Agile methodology understanding
- Testing methodologies: Waterfall, Scrum, V-Model, DevOps
- CI/CD test integration and automation

---

## 1. Software Development Methodologies

### Overview Comparison

```
┌─────────────┬───────────────┬──────────────┬──────────────┬──────────────┐
│ Aspect      │ Waterfall     │ V-Model      │ Scrum        │ DevOps       │
├─────────────┼───────────────┼──────────────┼──────────────┼──────────────┤
│ Approach    │ Sequential    │ Sequential + │ Iterative    │ Continuous   │
│             │               │ Verification │              │              │
│ Testing     │ After dev     │ Each phase   │ Every sprint │ Continuous   │
│ Flexibility │ Very low      │ Low          │ High         │ Very high    │
│ Feedback    │ Late          │ Medium       │ Fast         │ Immediate    │
│ Release     │ One big       │ One big      │ Incremental  │ Continuous   │
│ Risk        │ High (late)   │ Medium       │ Low          │ Very low     │
│ Best for    │ Fixed scope   │ Regulated    │ Evolving     │ Fast-paced   │
│             │ projects      │ industries   │ products     │ products     │
└─────────────┴───────────────┴──────────────┴──────────────┴──────────────┘
```

### 1.1 Waterfall Model

```
Requirements → Design → Implementation → Testing → Deployment → Maintenance
     │                                       │
     └───────── No going back ───────────────┘

QA Role in Waterfall:
  - Receives completed requirements upfront
  - Creates comprehensive test documentation
  - Testing is a distinct phase AFTER development
  - All bugs found late = expensive to fix

Pros:
  ✓ Clear documentation
  ✓ Well-defined phases
  ✓ Easy to manage and track

Cons:
  ✗ Late feedback
  ✗ Expensive bug fixes
  ✗ No room for changing requirements
  ✗ Customer sees product only at the end
```

### 1.2 V-Model (Verification & Validation)

```
Requirements  ←─────────────────────→  Acceptance Testing
     │                                        │
  System Design  ←──────────────→  System Testing
       │                                  │
    Architecture  ←────────→  Integration Testing
         │                            │
      Module Design  ←──→  Unit Testing
              │                │
           Coding ─────────────┘

Left side: Verification (building the product right)
Right side: Validation (building the right product)

Key Principle: Each development phase has a corresponding test phase
  - Requirements → Acceptance tests
  - System design → System tests
  - Architecture → Integration tests
  - Module design → Unit tests
```

### 1.3 Scrum (Agile)

```
Product    Sprint     Sprint      Daily    Sprint    Sprint
Backlog → Planning → Backlog → Standups → Review → Retro
                                  │
                              Development
                              + Testing
                              (2-4 weeks)

Scrum Roles:
  Product Owner:  Prioritizes backlog, defines acceptance criteria
  Scrum Master:   Facilitates process, removes blockers
  Dev Team:       Developers + QA (cross-functional)

QA in Scrum:
  ┌──────────────────────────────────────────────────────┐
  │ Sprint Planning: QA estimates testing effort         │
  │ Refinement:      QA reviews stories, writes AC       │
  │ During Sprint:   Test as stories are developed       │
  │ Daily Standup:   QA reports testing status           │
  │ Sprint Review:   QA demonstrates tested features     │
  │ Retrospective:   QA suggests process improvements    │
  └──────────────────────────────────────────────────────┘

Definition of Done (DoD) — QA Checklist:
  □ All acceptance criteria met
  □ Test cases executed and passed
  □ No open critical/high bugs
  □ Regression tests passed
  □ Code reviewed
  □ Documentation updated
```

### 1.4 DevOps

```
                 ┌──── CONTINUOUS ────┐
                 │                    │
    Plan → Code → Build → Test → Release → Deploy → Operate → Monitor
     ↑                                                            │
     └────────────────── FEEDBACK LOOP ───────────────────────────┘

DevOps = Development + Operations

Key Practices:
  - Continuous Integration (CI): merge + build + test frequently
  - Continuous Delivery (CD): always deployable
  - Continuous Deployment: auto-deploy to production
  - Infrastructure as Code (IaC)
  - Monitoring and observability
```

---

## 2. Agile Methodology Deep Dive

### Agile Manifesto Values

```
Individuals and interactions    OVER   Processes and tools
Working software                OVER   Comprehensive documentation
Customer collaboration          OVER   Contract negotiation
Responding to change            OVER   Following a plan
```

### Agile Principles for QA

```
1. Early and continuous delivery of valuable software
   → QA: Test early, provide fast feedback

2. Welcome changing requirements, even late
   → QA: Adaptive test cases, exploratory testing

3. Deliver working software frequently
   → QA: Testing within each sprint, no testing phase

4. Business and developers work together daily
   → QA: 3 Amigos sessions, daily standups

5. Build projects around motivated individuals
   → QA: Empowered to make quality decisions

6. Face-to-face conversation is most effective
   → QA: Direct communication over written specs

7. Working software is the primary measure of progress
   → QA: Focus on testable, shippable increments
```

### Agile Testing Practices

```
Practice             │ Description
─────────────────────┼──────────────────────────────────
3 Amigos             │ PO + Dev + QA discuss story together
BDD/ATDD             │ Tests define behavior before coding
TDD                  │ Developer writes tests first
Exploratory Testing  │ Session-based, charter-driven
Pair Testing         │ QA + Dev test together
Sprint Testing       │ Test within the sprint, not after
Continuous Testing   │ Automated tests in CI/CD pipeline
```

---

## 3. CI/CD Test Integration

### CI/CD Pipeline with Testing

```
Developer commits code
         │
         ▼
┌─────────────────┐
│  Source Control  │  (Git: push/PR)
│  (GitHub/GitLab) │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  CI Server      │  (Jenkins/GitHub Actions/GitLab CI)
│  Triggers Build │
└────────┬────────┘
         │
    ┌────▼─────┐
    │  Build   │  Compile, package
    └────┬─────┘
         │
    ┌────▼─────────────┐
    │  Unit Tests      │  Fast (<5 min)
    │  (JUnit, pytest) │
    └────┬─────────────┘
         │
    ┌────▼─────────────┐
    │  Integration     │  API tests (<10 min)
    │  Tests           │
    └────┬─────────────┘
         │
    ┌────▼─────────────┐
    │  Static Analysis │  Linting, security scans
    └────┬─────────────┘
         │
    ┌────▼─────────────┐
    │  Deploy to QA    │  Automated deployment
    └────┬─────────────┘
         │
    ┌────▼─────────────┐
    │  E2E Tests       │  UI tests (<30 min)
    │  (Selenium, PW)  │
    └────┬─────────────┘
         │
    ┌────▼─────────────┐
    │  Performance     │  Load tests (optional)
    └────┬─────────────┘
         │
    ┌────▼─────────────┐
    │  Deploy to Prod  │  Manual gate or auto
    └──────────────────┘
```

### CI/CD Tools Comparison

| Tool | Type | Config File | Key Feature |
|------|------|-------------|-------------|
| Jenkins | Self-hosted | Jenkinsfile | Highly extensible plugins |
| GitHub Actions | Cloud | .github/workflows/*.yml | Native GitHub integration |
| GitLab CI/CD | Cloud/Self | .gitlab-ci.yml | Built-in DevOps platform |
| Azure DevOps | Cloud | azure-pipelines.yml | Microsoft ecosystem |

### GitHub Actions Example

```yaml
name: Test Pipeline
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run unit tests
        run: npm test
      
      - name: Run API tests
        run: npm run test:api
      
      - name: Run E2E tests
        run: npx playwright test
      
      - name: Upload test report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-report
          path: test-results/
```

### Jenkins Pipeline Example

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps { sh 'mvn compile' }
        }
        stage('Unit Tests') {
            steps { sh 'mvn test' }
        }
        stage('Integration Tests') {
            steps { sh 'mvn verify -Pintegration' }
        }
        stage('Deploy to QA') {
            steps { sh './deploy.sh qa' }
        }
        stage('E2E Tests') {
            steps { sh 'npx playwright test' }
        }
    }
    post {
        always {
            junit '**/target/surefire-reports/*.xml'
            publishHTML(target: [reportDir: 'test-results'])
        }
        failure {
            slackSend channel: '#qa-alerts', message: "Build FAILED"
        }
    }
}
```

### Test Integration Best Practices

```
1. Fast Feedback First
   Unit tests (seconds) → API tests (minutes) → E2E tests (minutes)

2. Fail Fast
   If unit tests fail, don't run E2E tests — save time

3. Parallel Execution
   Run test suites in parallel to reduce total time

4. Test Environment Isolation
   Each pipeline run gets a clean environment

5. Artifact Preservation
   Always save test reports, screenshots, logs

6. Notification
   Alert team on Slack/Teams when tests fail

7. Merge Blocking
   Failed tests = cannot merge PR

8. Flaky Test Monitoring
   Track and quarantine unstable tests
```

---

## ✅ Self-Check Questions

1. Compare Waterfall and Scrum from a QA perspective.

<details>
<summary>Answer</summary>

**Waterfall QA:** Testing is a separate phase after development is complete. QA creates extensive documentation upfront but finds bugs late (expensive to fix). No flexibility for requirement changes. **Scrum QA:** Testing happens continuously within each sprint. QA participates from planning through review. Lightweight test cases that adapt to changing requirements. Fast feedback loops. 3 Amigos sessions ensure shared understanding. Definition of Done includes testing criteria.

</details>

2. What is the V-Model and how does it relate testing to development?

<details>
<summary>Answer</summary>

The V-Model pairs each development phase with a corresponding testing phase: Requirements → Acceptance Testing, System Design → System Testing, Architecture → Integration Testing, Module Design → Unit Testing. The left side is "Verification" (building the product right) and the right side is "Validation" (building the right product). Test planning starts early — acceptance test cases are designed during requirements, not after coding.

</details>

3. What tests should run in a CI/CD pipeline and in what order?

<details>
<summary>Answer</summary>

Order by speed and scope: (1) **Lint/Static Analysis** — instant feedback on code quality, (2) **Unit Tests** — fast, run in seconds, (3) **Integration/API Tests** — run in minutes, verify component interactions, (4) **E2E/UI Tests** — slower, run after deployment to QA, (5) **Performance Tests** — optional, run periodically or before releases. The principle is "fail fast" — if unit tests fail, skip the slower stages.

</details>

4. What is DevOps and how does it change the QA role?

<details>
<summary>Answer</summary>

DevOps combines Development and Operations into a continuous loop: Plan → Code → Build → Test → Release → Deploy → Operate → Monitor. For QA, this means: (1) testing is continuous, not a phase, (2) automation is essential for CI/CD pipelines, (3) QA monitors production quality (observability), (4) shift-left approach — test earlier, (5) infrastructure knowledge needed (containers, cloud), (6) quality gates are automated, not manual checkpoints.

</details>

5. Write a basic GitHub Actions workflow that runs tests on every pull request.

<details>
<summary>Answer</summary>

```yaml
name: PR Tests
on: [pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm test
      - run: npm run test:e2e
```

Key elements: trigger on `pull_request`, checkout code, setup runtime, install dependencies, run tests. Add `if: always()` for artifact upload to preserve reports even on failure.

</details>