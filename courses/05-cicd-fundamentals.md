# Course 05: CI/CD Fundamentals

## 📖 What You Will Learn

- What CI/CD is and why it matters
- The difference between Continuous Integration, Continuous Delivery, and Continuous Deployment
- How a CI/CD pipeline works step by step
- What problems CI/CD solves
- Key concepts: pipelines, stages, jobs, artifacts
- How CI/CD relates to QA work

---

## 1. The Problem CI/CD Solves

### Without CI/CD (the old way):

```
Developer writes code
        ↓
Waits weeks to merge with other developers' code
        ↓
"Integration day" — everyone merges → chaos, conflicts, broken code
        ↓
Manual testing for days/weeks
        ↓
Manual deployment (SSH to server, copy files, pray it works)
        ↓
Something breaks in production at 3 AM
        ↓
😱 Panic
```

### With CI/CD:

```
Developer pushes code
        ↓
Automatically builds, tests, and validates (in minutes)
        ↓
Automatically deploys to test environment
        ↓
QA tests on the environment
        ↓
Approved → automatically deploys to production
        ↓
😎 Confidence
```

---

## 2. What is CI/CD?

CI/CD is actually three related practices:

### CI — Continuous Integration

**"Merge code frequently. Build and test automatically."**

```
Developer A pushes code ──→ ┌──────────────┐
                            │   CI Server   │ → Build → Test → ✅ or ❌
Developer B pushes code ──→ │  (automated)  │
                            └──────────────┘
```

**What happens:**
1. Developers push code to a shared repository (Git) multiple times per day
2. Each push triggers an **automatic build**
3. **Automated tests** run against the build
4. If anything fails → team is notified immediately
5. The goal: code is always in a **buildable, testable state**

**Benefits:**
- 🐛 Bugs are found quickly (within minutes of being introduced)
- 🔀 Merge conflicts are small and easy to resolve
- ✅ Every commit is verified by automated tests
- 👥 Team has confidence that the code works

### CD — Continuous Delivery

**"The code is always ready to deploy. Deployment is a one-click decision."**

```
CI passes ──→ Deploy to Staging ──→ QA Testing ──→ [Manual Approval] ──→ Deploy to Production
                                                    ↑
                                              Human decides
```

**What happens:**
1. After CI passes, the code is **automatically deployed to a staging environment**
2. QA and stakeholders test on staging
3. When approved, a **single button click** deploys to production
4. The code is always in a **deployable state**

### CD — Continuous Deployment

**"Every change that passes all tests goes to production automatically."**

```
CI passes ──→ Deploy to Staging ──→ Automated Tests Pass ──→ Deploy to Production (automatic!)
```

**The difference from Continuous Delivery:**
- **Delivery** = automatic deployment to staging, **manual** approval for production
- **Deployment** = automatic deployment to **everything**, including production

Most companies use **Continuous Delivery** (with manual approval for production).

---

## 3. The CI/CD Pipeline

A **pipeline** is the automated workflow that code goes through from commit to deployment.

### Typical Pipeline Stages:

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  SOURCE   │───→│  BUILD   │───→│   TEST   │───→│  DEPLOY  │───→│ DEPLOY   │
│           │    │          │    │          │    │ (staging) │    │ (prod)   │
│ Git push  │    │ Compile  │    │ Run tests│    │          │    │          │
│ PR merge  │    │ Package  │    │ Analyze  │    │ Auto     │    │ Manual   │
│           │    │          │    │          │    │          │    │ approval │
└──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
     ✅              ✅              ✅              ✅              ✅
  Trigger         Compile        Unit tests     Deploy to       Deploy to
  the             the code       Integration    staging env     production
  pipeline                       tests
                                 Code quality
```

### Stage Details:

#### Stage 1: Source (Trigger)
- Pipeline starts when code is pushed or a Pull Request is created
- Can also be triggered on a schedule (e.g., nightly builds)

#### Stage 2: Build
- **Compile** the source code
- **Download dependencies** (libraries the code needs)
- **Package** the application (create a JAR file, Docker image, etc.)
- If the code doesn't compile → pipeline fails ❌

#### Stage 3: Test
- **Unit tests** — test individual functions/methods
- **Integration tests** — test components working together
- **Static analysis** — check code quality without running it (SonarQube, linters)
- **Security scans** — check for known vulnerabilities
- If any test fails → pipeline fails ❌

#### Stage 4: Deploy to Staging
- Deploy the built application to a **staging environment**
- Staging is a copy of production (same configuration, similar data)
- QA and stakeholders can test here

#### Stage 5: Deploy to Production
- After approval, deploy to the **production environment**
- This is what real users see
- Usually requires **manual approval** or passes through additional checks

---

## 4. Key Concepts

### Pipeline
The entire automated workflow from start to finish.

### Stage
A group of related steps (e.g., "Build", "Test", "Deploy"). Stages usually run sequentially.

### Job
A specific task within a stage. Jobs within the same stage can run **in parallel**.

```
Pipeline
├── Stage: Build
│   └── Job: Compile and Package
├── Stage: Test
│   ├── Job: Unit Tests          (parallel)
│   ├── Job: Integration Tests   (parallel)
│   └── Job: Code Quality Check  (parallel)
├── Stage: Deploy Staging
│   └── Job: Deploy to staging server
└── Stage: Deploy Production
    └── Job: Deploy to production server
```

### Artifact
A file produced by the pipeline (e.g., JAR file, Docker image, test report). Artifacts are passed between stages.

### Environment
Where the application runs:

| Environment | Purpose | Who uses it |
|-------------|---------|-------------|
| **Development (Dev)** | Developers test their changes | Developers |
| **Testing / QA** | QA team tests features | QA team |
| **Staging** | Pre-production testing | QA, stakeholders |
| **Production (Prod)** | Live system for real users | End users |

### Trigger
What starts the pipeline:

| Trigger | When |
|---------|------|
| **Push** | Code is pushed to a branch |
| **Pull Request** | A PR is created or updated |
| **Merge** | Code is merged to main/master |
| **Schedule** | Runs at specific times (cron) |
| **Manual** | Someone clicks "Run Pipeline" |

---

## 5. Version Control & Branching (Git Basics for CI/CD)

CI/CD relies heavily on Git. Here's how teams typically use branches:

### Common Branching Strategy:

```
main (production)  ────●────────────────●──────────────────●────→
                       │                ↑                  ↑
                       │     merge PR   │       merge PR   │
                       │                │                  │
develop            ────●──●──●──●──●────●──●──●──●──●──●──●────→
                          │        ↑       │           ↑
                          │  merge │       │     merge │
                          │        │       │           │
feature/login         ────●──●──●──┘       │           │
                                           │           │
feature/user-api                       ────●──●──●──●──┘
```

| Branch | Purpose | CI/CD Behavior |
|--------|---------|----------------|
| `main` / `master` | Production-ready code | Deploys to production |
| `develop` | Latest development code | Deploys to staging |
| `feature/*` | New feature development | Runs tests only |
| `release/*` | Release preparation | Deploys to staging |
| `hotfix/*` | Emergency production fixes | Fast-track to production |

### Pull Requests (PRs):

A Pull Request is a request to merge code from one branch to another:

```
feature/login ──→ [Pull Request] ──→ develop
                        │
                        ├── CI pipeline runs automatically
                        ├── Code review by team members
                        ├── All tests must pass ✅
                        └── After approval → merge
```

**CI/CD checks on PRs:**
- ✅ Code compiles
- ✅ All tests pass
- ✅ Code quality meets standards
- ✅ No security vulnerabilities
- ✅ Code review approved

---

## 6. CI/CD Tools

| Tool | Type | Used by |
|------|------|---------|
| **Jenkins** | Self-hosted CI/CD server | Large enterprises |
| **GitHub Actions** | Cloud-based, integrated with GitHub | GitHub projects |
| **GitLab CI/CD** | Integrated with GitLab | GitLab projects |
| **Azure DevOps** | Microsoft cloud CI/CD | Enterprise, Azure users |
| **CircleCI** | Cloud-based CI/CD | Startups, open source |
| **Travis CI** | Cloud-based CI/CD | Open source projects |
| **TeamCity** | Self-hosted (JetBrains) | JetBrains ecosystem |

### Example: GitHub Actions Pipeline

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Build with Maven
        run: mvn clean compile

  test:
    needs: build                    # Runs after build succeeds
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Run unit tests
        run: mvn test

      - name: Run integration tests
        run: mvn verify -P integration-tests

  deploy-staging:
    needs: test                     # Runs after test succeeds
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to staging
        run: echo "Deploying to staging..."

  deploy-production:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production          # Requires manual approval
    steps:
      - name: Deploy to production
        run: echo "Deploying to production..."
```

---

## 7. How CI/CD Helps QA

### Before CI/CD:
- "Works on my machine" 🤷 — no standard build process
- "I broke something?" — no immediate feedback
- "When will the build be ready?" — manual builds take time
- "Is staging up to date?" — manual deployments

### After CI/CD:

| QA Benefit | How |
|------------|-----|
| **Faster feedback** | Know within minutes if a change broke something |
| **Consistent environments** | Same build process every time |
| **Always-ready staging** | New code automatically deployed to staging |
| **Test reports** | Automated test results available in the pipeline |
| **Confidence in releases** | Multiple verification stages before production |
| **Rollback capability** | Can quickly deploy previous version if needed |

### What QA should know about the pipeline:

1. **Where to find build status** — Is the latest build passing? ✅ or ❌
2. **Where to find test reports** — Which tests passed/failed?
3. **How to trigger a build** — Can you re-run the pipeline?
4. **Which environment is deployed** — What version is on staging? On production?
5. **How to read pipeline logs** — If something fails, where to look?

---

## 📝 Exercises

### Exercise 1: Map the Pipeline

Draw a CI/CD pipeline for this scenario:

> A Java Spring Boot application with a PostgreSQL database. The team uses GitHub for version control. They want:
> - Automatic builds on every push
> - Unit and integration tests
> - Code quality analysis
> - Automatic deployment to staging from `develop` branch
> - Manual approval before deploying to production from `main` branch

Identify:
1. What stages does the pipeline need?
2. What jobs go in each stage?
3. What triggers each stage?
4. Where does human intervention happen?

<details>
<summary>Click to see answer</summary>

```
Trigger: Push to any branch

Stage 1: Build
├── Job: Compile Java code (mvn compile)
├── Job: Download dependencies
└── Artifact: JAR file

Stage 2: Test
├── Job: Unit tests (mvn test)             [parallel]
├── Job: Integration tests (mvn verify)    [parallel]
└── Job: Code quality (SonarQube)          [parallel]

Stage 3: Deploy to Staging
├── Trigger: Only on 'develop' branch
├── Job: Deploy JAR to staging server
└── Job: Run smoke tests on staging

Stage 4: Deploy to Production
├── Trigger: Only on 'main' branch
├── Gate: Manual approval required ← Human intervention
├── Job: Deploy JAR to production server
└── Job: Run smoke tests on production
```

</details>

---

### Exercise 2: Identify the Failure

For each scenario, identify where in the pipeline the failure would occur and what the team should do:

1. A developer pushed code that has a syntax error
2. A developer's code compiles but breaks an existing unit test
3. The code passes all tests but has a security vulnerability
4. The staging deployment fails because the server ran out of disk space
5. Everything passes, but QA finds a bug during manual testing on staging

<details>
<summary>Click to see answers</summary>

1. **Build stage** — code won't compile. Developer needs to fix the syntax error.
2. **Test stage** — unit tests fail. Developer needs to fix the code or update the test if the change was intentional.
3. **Test stage** (security scan) — pipeline should fail. Developer needs to fix the vulnerability or update the dependency.
4. **Deploy staging stage** — infrastructure issue. DevOps needs to clean disk space or increase storage.
5. **Not caught by pipeline** — QA rejects the deployment. Developer fixes the bug, pushes new code, pipeline runs again.

</details>

---

### Exercise 3: Read a Pipeline Configuration

Look at the GitHub Actions example in Section 6 and answer:

1. What triggers this pipeline?
2. How many jobs does it have?
3. Which job runs first?
4. What happens if the `build` job fails?
5. When does `deploy-staging` run?
6. When does `deploy-production` run?
7. What's special about the production deployment?

<details>
<summary>Click to see answers</summary>

1. Push to `main` or `develop`, and Pull Requests to `main` or `develop`
2. 4 jobs: `build`, `test`, `deploy-staging`, `deploy-production`
3. `build` runs first (others have `needs:` dependencies)
4. `test` won't run (it `needs: build`), and neither will deployments
5. After `test` passes AND only when pushing to `develop` branch
6. After `test` passes AND only when pushing to `main` branch
7. It uses `environment: production` which requires **manual approval** in GitHub

</details>

---

### Exercise 4: CI/CD for QA Workflow

Describe how you would use CI/CD in your daily QA work:

1. A developer finishes a feature and creates a Pull Request. What do you check first?
2. The PR pipeline shows ✅ all green. What does this mean? Can you trust it?
3. The code is merged to `develop` and deployed to staging. What do you do?
4. You find a bug on staging. What's the process?
5. Staging is approved. What happens next?

---

### Exercise 5: Compare CI/CD Tools

Research and fill in this comparison table:

| Feature | Jenkins | GitHub Actions | GitLab CI/CD |
|---------|---------|---------------|--------------|
| Hosting | ? | ? | ? |
| Configuration file | ? | ? | ? |
| Free tier | ? | ? | ? |
| Best for | ? | ? | ? |
| Pipeline syntax | ? | ? | ? |

---

## 📚 Additional Resources

- [What is CI/CD? (Red Hat)](https://www.redhat.com/en/topics/devops/what-is-ci-cd) — Clear explanation of CI/CD
- [GitHub Actions Documentation](https://docs.github.com/en/actions) — Learn GitHub Actions
- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/) — Learn GitLab CI/CD
- [CI/CD Pipeline: A Gentle Introduction (Semaphore)](https://semaphoreci.com/blog/cicd-pipeline) — Detailed pipeline guide

---

## ✅ Self-Check

After completing this course, you should be able to:

- [ ] Explain what CI, CD (Delivery), and CD (Deployment) mean
- [ ] Describe the stages of a typical CI/CD pipeline
- [ ] Explain the difference between stages, jobs, and artifacts
- [ ] Understand how Git branching relates to CI/CD
- [ ] Read a basic pipeline configuration file (YAML)
- [ ] Know what happens when a pipeline stage fails
- [ ] Explain how CI/CD benefits QA work
- [ ] Identify common CI/CD tools and their differences
- [ ] Describe the flow from code commit to production deployment