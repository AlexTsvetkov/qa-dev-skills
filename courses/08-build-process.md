# Course 08: Build Process

## 📖 What You Will Learn

- What a build is and why it matters
- How Java applications are built (Maven & Gradle)
- What happens during each build phase
- Understanding build artifacts (JAR, WAR, Docker images)
- How to read build logs and troubleshoot failures
- What QA should know about the build process

---

## 1. What is a Build?

A **build** is the process of transforming source code into a runnable application.

```
Source Code    →    BUILD PROCESS    →    Runnable Application
(.java files)      (compile, test,       (.jar file, Docker image)
                    package)
```

### Why QA cares about builds:
- 🔍 **Build failures block testing** — if the build fails, there's nothing to test
- 📦 **Artifacts are what you test** — you test the built application, not source code
- 🏷️ **Build versions** — you need to know which build you're testing
- 🐛 **Build logs reveal issues** — compilation errors, dependency problems

---

## 2. Maven Build Tool

**Maven** is the most common build tool for Java projects. It uses a `pom.xml` file for configuration.

### Maven Build Lifecycle:

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│ validate  │──→│ compile  │──→│   test   │──→│ package  │──→│ verify   │──→│ install  │
│           │   │          │   │          │   │          │   │          │   │          │
│ Check     │   │ Compile  │   │ Run unit │   │ Create   │   │ Run      │   │ Install  │
│ project   │   │ Java     │   │ tests    │   │ JAR/WAR  │   │ integr.  │   │ to local │
│ is correct│   │ code     │   │          │   │ file     │   │ tests    │   │ repo     │
└──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘
```

### Common Maven Commands:

| Command | What it does |
|---------|-------------|
| `mvn compile` | Compile source code |
| `mvn test` | Compile + run unit tests |
| `mvn package` | Compile + test + create JAR/WAR |
| `mvn verify` | Compile + test + package + run integration tests |
| `mvn clean` | Delete previous build output (`target/` folder) |
| `mvn clean package` | Clean + full build with packaging |
| `mvn clean package -DskipTests` | Build without running tests |

### pom.xml (Project Object Model):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project>
    <groupId>com.example</groupId>           <!-- Organization -->
    <artifactId>my-application</artifactId>  <!-- Project name -->
    <version>1.2.3</version>                 <!-- Version -->
    <packaging>jar</packaging>               <!-- Output type -->

    <dependencies>
        <!-- Libraries the project needs -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>3.2.0</version>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.7.0</version>
        </dependency>
        <!-- Test dependencies -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>          <!-- Only used for testing -->
        </dependency>
    </dependencies>
</project>
```

### Build Output:

```
my-application/
├── src/                          ← Source code (input)
├── target/                       ← Build output (generated)
│   ├── classes/                  ← Compiled Java classes
│   ├── test-classes/             ← Compiled test classes
│   ├── surefire-reports/         ← Test reports (XML)
│   ├── my-application-1.2.3.jar ← The built application ⭐
│   └── maven-status/            ← Build metadata
└── pom.xml                       ← Build configuration
```

---

## 3. Gradle Build Tool

**Gradle** is a modern alternative to Maven, using `build.gradle` (Groovy) or `build.gradle.kts` (Kotlin).

### build.gradle example:

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.0'
}

group = 'com.example'
version = '1.2.3'

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.postgresql:postgresql:42.7.0'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

### Common Gradle Commands:

| Command | What it does |
|---------|-------------|
| `gradle build` | Compile + test + package |
| `gradle test` | Run tests |
| `gradle clean build` | Clean + full build |
| `gradle bootJar` | Create Spring Boot JAR |

---

## 4. Build Artifacts

An **artifact** is the output of the build process — the thing you deploy and test.

### Common Artifact Types:

| Type | Extension | Description |
|------|-----------|-------------|
| **JAR** | `.jar` | Java Archive — standalone Java application |
| **WAR** | `.war` | Web Archive — web app deployed to app server |
| **Docker Image** | — | Containerized application |
| **NPM Package** | `.tgz` | JavaScript package |

### Version Naming:

```
my-application-1.2.3.jar
                │ │ │
                │ │ └── Patch (bug fixes)
                │ └──── Minor (new features, backward compatible)
                └────── Major (breaking changes)
```

Special versions:
- `1.2.3-SNAPSHOT` — development version (not stable)
- `1.2.3-RC1` — Release Candidate (almost ready)
- `1.2.3-RELEASE` — Production-ready

---

## 5. Docker Builds

Many modern applications are packaged as **Docker images**.

### Dockerfile:

```dockerfile
FROM eclipse-temurin:17-jre-alpine       # Base image with Java 17
WORKDIR /app                              # Working directory
COPY target/my-application-1.2.3.jar app.jar  # Copy built JAR
EXPOSE 8080                               # Port the app listens on
ENTRYPOINT ["java", "-jar", "app.jar"]    # Command to start the app
```

### Docker Build Commands:

```bash
# Build Docker image
docker build -t my-application:1.2.3 .

# Run the container
docker run -p 8080:8080 my-application:1.2.3

# List images
docker images

# Push to registry (share with team)
docker push registry.example.com/my-application:1.2.3
```

---

## 6. Reading Build Logs

### Successful Maven Build:

```
[INFO] --- maven-compiler-plugin:3.11.0:compile (default-compile) ---
[INFO] Compiling 42 source files to /target/classes
[INFO]
[INFO] --- maven-surefire-plugin:3.2.0:test (default-test) ---
[INFO] Running com.example.UserServiceTest
[INFO] Tests run: 15, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 2.3 s
[INFO]
[INFO] --- maven-jar-plugin:3.3.0:jar (default-jar) ---
[INFO] Building jar: /target/my-application-1.2.3.jar
[INFO] BUILD SUCCESS
[INFO] Total time: 25 s
```

### Common Build Failures:

**1. Compilation Error:**
```
[ERROR] COMPILATION ERROR :
[ERROR] /src/main/java/com/example/UserService.java:[42,15]
    cannot find symbol: variable userRepositry
[ERROR] BUILD FAILURE
```
→ Typo in code (`userRepositry` instead of `userRepository`)

**2. Test Failure:**
```
[ERROR] Tests run: 15, Failures: 2, Errors: 0
[ERROR] UserServiceTest.testCreate:45 expected:<201> but was:<500>
[ERROR] BUILD FAILURE
```
→ A test assertion failed

**3. Dependency Not Found:**
```
[ERROR] Failed to execute goal: Could not resolve dependencies
[ERROR] Could not find artifact com.example:my-library:jar:2.0.0
[ERROR] BUILD FAILURE
```
→ A required library can't be downloaded

---

## 📝 Exercises

### Exercise 1: Read pom.xml

Given a `pom.xml`, identify:
1. What is the project name and version?
2. What dependencies does it use?
3. What type of artifact will be produced (JAR or WAR)?

### Exercise 2: Interpret Build Logs

For each build log snippet, identify:
- Which phase failed?
- What was the error?
- Who should fix it (developer, DevOps, QA)?

### Exercise 3: Version Numbers

Put these versions in order from oldest to newest:
`1.0.0`, `1.0.1`, `2.0.0-SNAPSHOT`, `1.1.0`, `2.0.0`, `1.1.0-RC1`

---

## 📚 Additional Resources

- [Maven in 5 Minutes](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html) — Quick Maven intro
- [Gradle Getting Started](https://docs.gradle.org/current/userguide/getting_started.html) — Gradle basics
- [Docker Getting Started](https://docs.docker.com/get-started/) — Docker fundamentals
- [Semantic Versioning](https://semver.org/) — Version numbering standard

---

## ✅ Self-Check

- [ ] Explain what a build process does
- [ ] Understand the Maven build lifecycle phases
- [ ] Read a `pom.xml` and identify key information
- [ ] Know common Maven/Gradle commands
- [ ] Explain what JAR, WAR, and Docker images are
- [ ] Read build logs and identify errors
- [ ] Understand semantic versioning (major.minor.patch)
- [ ] Explain the difference between SNAPSHOT and RELEASE versions