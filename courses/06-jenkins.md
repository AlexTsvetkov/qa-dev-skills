# Course 06: Jenkins

## 📖 What You Will Learn

- What Jenkins is and how it works
- Jenkins architecture: master, agents, jobs, pipelines
- How to navigate the Jenkins UI
- How to read Jenkinsfile pipeline definitions
- How to read build logs and find errors
- Jenkins from a QA perspective

---

## 1. What is Jenkins?

**Jenkins** is an open-source automation server used for CI/CD. It's the most widely used CI/CD tool in enterprise environments.

### Key characteristics:
- **Self-hosted** — runs on your company's servers
- **Plugin-based** — thousands of plugins extend functionality
- **Pipeline as Code** — pipelines defined in `Jenkinsfile`
- **Distributed** — can run builds on multiple machines (agents)

---

## 2. Jenkins Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     JENKINS MASTER                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────────┐ │
│  │ Web UI   │  │ Job      │  │ Build    │  │ Plugin      │ │
│  │          │  │ Scheduler│  │ Queue    │  │ Manager     │ │
│  └──────────┘  └──────────┘  └──────────┘  └─────────────┘ │
└───────────┬──────────────────────┬──────────────────────────┘
            │                      │
     ┌──────┴──────┐        ┌─────┴───────┐
     │  AGENT 1    │        │  AGENT 2     │
     │  (Linux)    │        │  (Windows)   │
     │  Runs builds│        │  Runs builds │
     └─────────────┘        └──────────────┘
```

| Component | Role |
|-----------|------|
| **Master** | Manages everything: scheduling, UI, configuration |
| **Agent (Node)** | Machine that executes builds |
| **Executor** | A slot on an agent that can run one build |
| **Job/Project** | A defined task (build, test, deploy) |
| **Build** | One execution of a job |
| **Pipeline** | A series of stages defined in code |

---

## 3. Jenkins UI Navigation

### Dashboard (Home Page):

```
┌─────────────────────────────────────────────────────────────┐
│ Jenkins                                    [user] [log out]  │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  📊 Build Queue        │  Job Name          Last Build  Status│
│  (empty)               │  ─────────────────────────────────  │
│                        │  my-app-pipeline   #142    ✅ Success│
│  🖥️ Build Executor     │  api-tests         #89     ❌ Failed │
│  Agent-1: idle         │  nightly-build     #56     ✅ Success│
│  Agent-2: building     │  deploy-staging    #33     ⚡ Running│
│                        │                                      │
└─────────────────────────────────────────────────────────────┘
```

### Build Status Icons:

| Icon | Meaning |
|------|---------|
| ☀️ Blue/Green circle | Last build succeeded |
| ⛈️ Red circle | Last build failed |
| 🌤️ Yellow circle | Last build unstable (tests failed) |
| ⚫ Grey circle | Never built / disabled |
| ⚡ Flashing | Currently building |

### Key Pages:

| Page | How to access | What it shows |
|------|--------------|---------------|
| **Dashboard** | Click "Jenkins" logo | All jobs overview |
| **Job page** | Click job name | Build history, configuration |
| **Build page** | Click build number (#142) | Build details, logs |
| **Console Output** | Click "Console Output" on build page | Full build log |
| **Test Results** | Click "Test Result" on build page | Test pass/fail details |
| **Pipeline view** | Click "Pipeline" on job page | Visual stage overview |

---

## 4. Jenkinsfile (Pipeline as Code)

Modern Jenkins uses **Declarative Pipelines** defined in a `Jenkinsfile` stored in the project's Git repository.

### Basic Jenkinsfile:

```groovy
pipeline {
    agent any                          // Run on any available agent

    environment {
        APP_NAME = 'my-application'
        JAVA_VERSION = '17'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building the application...'
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            parallel {                 // Run tests in parallel
                stage('Unit Tests') {
                    steps {
                        sh 'mvn test'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'mvn verify -P integration-tests'
                    }
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar'
            }
        }

        stage('Deploy to Staging') {
            when {
                branch 'develop'       // Only on develop branch
            }
            steps {
                echo 'Deploying to staging...'
                sh './deploy.sh staging'
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            input {                    // Manual approval
                message 'Deploy to production?'
                ok 'Yes, deploy!'
            }
            steps {
                echo 'Deploying to production...'
                sh './deploy.sh production'
            }
        }
    }

    post {
        always {
            junit 'target/surefire-reports/*.xml'    // Publish test results
            cleanWs()                                 // Clean workspace
        }
        success {
            echo '✅ Pipeline succeeded!'
        }
        failure {
            echo '❌ Pipeline failed!'
            // Send notification (email, Slack, etc.)
        }
    }
}
```

### Key Jenkinsfile Sections:

| Section | Purpose |
|---------|---------|
| `pipeline` | Root element — wraps everything |
| `agent` | Where to run (which machine) |
| `environment` | Define environment variables |
| `stages` | List of pipeline stages |
| `stage` | A named group of steps |
| `steps` | Actual commands to execute |
| `when` | Conditions for running a stage |
| `input` | Wait for manual approval |
| `parallel` | Run stages in parallel |
| `post` | Actions after pipeline completes |

### Common `when` conditions:

```groovy
when { branch 'main' }                    // Only on main branch
when { branch 'feature/*' }               // Any feature branch
when { expression { env.DEPLOY == 'true' } } // Custom condition
when { changeset '**/*.java' }             // Only if Java files changed
when { tag 'v*' }                          // Only on version tags
```

---

## 5. Reading Jenkins Build Logs

The **Console Output** is where you find what happened during a build.

### Example: Successful build log

```
Started by user admin
Running on agent-1 in /var/jenkins/workspace/my-app
[Pipeline] Start of Pipeline
[Pipeline] stage (Build)
[Pipeline] sh
+ mvn clean compile
[INFO] Scanning for projects...
[INFO] Building my-application 1.0.0
[INFO] --- maven-compiler-plugin:3.11.0:compile ---
[INFO] Compiling 42 source files to /target/classes
[INFO] BUILD SUCCESS
[Pipeline] stage (Test)
[Pipeline] sh
+ mvn test
[INFO] Running com.example.UserServiceTest
[INFO] Tests run: 15, Failures: 0, Errors: 0, Skipped: 0
[INFO] BUILD SUCCESS
[Pipeline] stage (Package)
[Pipeline] sh
+ mvn package -DskipTests
[INFO] Building jar: /target/my-application-1.0.0.jar
[INFO] BUILD SUCCESS
[Pipeline] archiveArtifacts
Archiving artifacts
[Pipeline] End of Pipeline
Finished: SUCCESS
```

### Example: Failed build log

```
[Pipeline] stage (Test)
[Pipeline] sh
+ mvn test
[INFO] Running com.example.UserServiceTest
[ERROR] Tests run: 15, Failures: 2, Errors: 0, Skipped: 0
[ERROR] UserServiceTest.testCreateUser:45 Expected: 201, Got: 500
[ERROR] UserServiceTest.testDeleteUser:78 NullPointerException
[INFO] BUILD FAILURE
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage (Declarative: Post Actions)
[Pipeline] junit
Recording test results
Test Summary: 13 passed, 2 failed
[Pipeline] End of Pipeline
Finished: FAILURE
```

### How to find errors in logs:

1. **Scroll to the bottom** — the final status is at the end
2. **Search for `ERROR`** — find error messages
3. **Search for `FAILURE`** — find the failing stage
4. **Look for test names** — identify which tests failed
5. **Check the stage** — which stage failed (Build? Test? Deploy?)

---

## 6. Jenkins for QA

### What QA should do with Jenkins:

| Task | How |
|------|-----|
| **Check build status** | Look at the dashboard — is the build green? |
| **Read test reports** | Click on build → Test Results |
| **Trigger a build** | Click "Build Now" or "Build with Parameters" |
| **Check deployment status** | Look at deploy stage — did it succeed? |
| **Find failed test details** | Console Output → search for FAILURE |
| **Get build artifacts** | Click on build → find JAR/reports |

### Build Parameters (Parameterized Builds):

Some Jenkins jobs accept parameters that you can set before running:

```groovy
parameters {
    string(name: 'ENVIRONMENT', defaultValue: 'staging', description: 'Target environment')
    choice(name: 'BROWSER', choices: ['chrome', 'firefox'], description: 'Browser for tests')
    booleanParam(name: 'RUN_SMOKE_TESTS', defaultValue: true, description: 'Run smoke tests?')
}
```

When you click **"Build with Parameters"**, you can set these values before the build starts.

---

## 📝 Exercises

### Exercise 1: Read the Jenkinsfile

Look at the Jenkinsfile in Section 4 and answer:
1. How many stages does this pipeline have?
2. Which stages run in parallel?
3. What happens on the `develop` branch vs `main` branch?
4. Where is manual approval required?
5. What happens after every build (success or failure)?

### Exercise 2: Diagnose Build Failures

Given the failed build log in Section 5:
1. Which stage failed?
2. How many tests failed?
3. What were the test names that failed?
4. What was the error in `testCreateUser`?
5. What was the error in `testDeleteUser`?

### Exercise 3: Jenkins in Your Workflow

Describe how you would use Jenkins when:
1. A developer says "my code is ready for testing"
2. You need to deploy a specific version to staging
3. Your automated tests failed in the nightly build
4. You need to test a hotfix urgently

---

## 📚 Additional Resources

- [Jenkins User Documentation](https://www.jenkins.io/doc/) — Official docs
- [Jenkins Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/) — Jenkinsfile reference
- [Blue Ocean Plugin](https://www.jenkins.io/projects/blueocean/) — Modern Jenkins UI

---

## ✅ Self-Check

- [ ] Navigate the Jenkins UI and find jobs, builds, and logs
- [ ] Read a Jenkinsfile and understand its stages
- [ ] Find and interpret build errors in Console Output
- [ ] Trigger a build manually (with or without parameters)
- [ ] Find test results and identify which tests failed
- [ ] Explain the difference between Jenkins master and agents
- [ ] Understand `when` conditions for conditional stage execution