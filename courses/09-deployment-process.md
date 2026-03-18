# Course 09: Deployment Process

## 📖 What You Will Learn

- What deployment is and how it works
- Deployment environments (dev, staging, production)
- Common deployment strategies (rolling, blue-green, canary)
- Containers and Kubernetes basics
- Rollbacks — what to do when things go wrong
- QA's role in the deployment process

---

## 1. What is Deployment?

**Deployment** is the process of making a built application available for use on a server or environment.

```
Built Artifact          DEPLOYMENT           Running Application
(JAR, Docker image) ──────────────────→  (accessible via URL)
                     Copy, configure,
                     start, verify
```

### Deployment vs Build:
| Concept | What it does |
|---------|-------------|
| **Build** | Transforms source code → artifact (JAR, Docker image) |
| **Deploy** | Takes artifact → installs and runs it on a server |

---

## 2. Deployment Environments

```
┌────────────┐    ┌────────────┐    ┌────────────┐    ┌────────────┐
│    DEV      │───→│   QA/TEST  │───→│  STAGING   │───→│ PRODUCTION │
│             │    │            │    │            │    │            │
│ Developers  │    │ QA team    │    │ Final check│    │ Real users │
│ test code   │    │ tests      │    │ before prod│    │            │
│             │    │ features   │    │            │    │            │
│ Unstable    │    │ May break  │    │ Prod-like  │    │ Must be    │
│             │    │            │    │            │    │ stable     │
└────────────┘    └────────────┘    └────────────┘    └────────────┘
```

| Environment | Purpose | Stability | Data |
|-------------|---------|-----------|------|
| **Dev** | Developer testing | Low — breaks often | Fake/test data |
| **QA / Test** | QA testing features | Medium | Test data |
| **Staging** | Pre-production validation | High — mirrors prod | Anonymized prod data |
| **Production** | Real users | Must be stable | Real data |

### Environment Configuration:

Each environment has its own settings:

```yaml
# Dev
database.url: jdbc:postgresql://dev-db:5432/myapp
api.url: https://dev.example.com/api
logging.level: DEBUG

# Staging
database.url: jdbc:postgresql://staging-db:5432/myapp
api.url: https://staging.example.com/api
logging.level: INFO

# Production
database.url: jdbc:postgresql://prod-db:5432/myapp
api.url: https://api.example.com
logging.level: WARN
```

---

## 3. Deployment Strategies

### Strategy 1: Rolling Deployment

Replace instances one at a time:

```
Time 0:  [v1] [v1] [v1] [v1]     ← All running v1
Time 1:  [v2] [v1] [v1] [v1]     ← First instance updated
Time 2:  [v2] [v2] [v1] [v1]     ← Second instance updated
Time 3:  [v2] [v2] [v2] [v1]     ← Third instance updated
Time 4:  [v2] [v2] [v2] [v2]     ← All running v2 ✅
```

✅ Zero downtime | ⚠️ Mixed versions during rollout

### Strategy 2: Blue-Green Deployment

Run two identical environments, switch traffic:

```
Before:   Traffic ──→ [BLUE: v1]     [GREEN: idle]
Deploy:   Traffic ──→ [BLUE: v1]     [GREEN: v2] ← Deploy v2 to green
Switch:   Traffic ──→ [GREEN: v2]    [BLUE: v1]  ← Switch traffic
Verify:   Traffic ──→ [GREEN: v2]    [BLUE: v1]  ← Blue is rollback option
```

✅ Instant rollback | ⚠️ Requires double infrastructure

### Strategy 3: Canary Deployment

Send a small % of traffic to the new version first:

```
Step 1:   [v1 ← 95% traffic]   [v2 ← 5% traffic]    ← Test with few users
Step 2:   [v1 ← 75% traffic]   [v2 ← 25% traffic]   ← Increase if OK
Step 3:   [v1 ← 50% traffic]   [v2 ← 50% traffic]   ← Half and half
Step 4:   [v2 ← 100% traffic]                         ← Full rollout ✅
```

✅ Low risk — catches issues early | ⚠️ Complex to set up

---

## 4. Containers and Kubernetes

### Docker Containers:

A **container** packages the application with everything it needs to run:

```
┌─────────────────────┐
│     Container        │
│  ┌───────────────┐  │
│  │  Application   │  │
│  │  (my-app.jar)  │  │
│  ├───────────────┤  │
│  │  Java Runtime  │  │
│  ├───────────────┤  │
│  │  Linux (Alpine)│  │
│  └───────────────┘  │
└─────────────────────┘
```

**Why containers?**
- 📦 **Consistent** — same container runs everywhere (dev, staging, prod)
- 🚀 **Fast** — start in seconds
- 🔒 **Isolated** — doesn't interfere with other applications
- 📏 **Scalable** — run multiple copies easily

### Kubernetes (K8s):

**Kubernetes** manages containers in production — scaling, healing, and routing.

```
┌─────────────────────────────────────────┐
│           KUBERNETES CLUSTER             │
│                                          │
│  ┌──────────┐  ┌──────────┐  ┌────────┐ │
│  │  Pod      │  │  Pod      │  │ Pod    │ │
│  │ [my-app]  │  │ [my-app]  │  │[my-app]│ │
│  └──────────┘  └──────────┘  └────────┘ │
│        ↑              ↑            ↑     │
│        └──────────────┼────────────┘     │
│                  Load Balancer            │
│                       ↑                  │
└───────────────────────┼──────────────────┘
                        │
                   User Traffic
```

Key concepts:
| Concept | What it is |
|---------|-----------|
| **Pod** | Smallest unit — one or more containers |
| **Deployment** | Manages pods (scaling, updates) |
| **Service** | Network endpoint to access pods |
| **Namespace** | Logical isolation (dev, staging, prod) |

### Useful kubectl commands for QA:

```bash
# See what's running
kubectl get pods                        # List all pods
kubectl get pods -n staging             # List pods in staging namespace

# Check pod health
kubectl describe pod my-app-abc123      # Detailed pod info
kubectl logs my-app-abc123              # View application logs
kubectl logs my-app-abc123 --tail=100   # Last 100 lines

# Check deployments
kubectl get deployments                 # List deployments
kubectl rollout status deployment/my-app # Check deployment progress
```

---

## 5. Rollbacks

When a deployment causes issues, you **rollback** to the previous version.

### When to rollback:
- ❌ Application crashes after deployment
- ❌ Error rate spikes dramatically
- ❌ Critical functionality is broken
- ❌ Performance degrades significantly

### How to rollback:

```bash
# Kubernetes rollback
kubectl rollout undo deployment/my-app

# Jenkins — re-run the previous successful build

# Docker — run the previous image tag
docker run my-application:1.2.2    # Previous version
```

### Rollback checklist for QA:

1. ✅ Verify the rollback completed (previous version is running)
2. ✅ Run smoke tests on the rolled-back version
3. ✅ Verify the original issue is gone
4. ✅ Report what happened and what was observed
5. ✅ Keep evidence (screenshots, logs, error messages)

---

## 6. Health Checks and Monitoring

After deployment, the system needs to verify the application is healthy.

### Health Check Endpoints:

```
GET /actuator/health        → { "status": "UP" }
GET /actuator/health/db     → { "status": "UP", "database": "PostgreSQL" }
GET /actuator/health/ready  → { "status": "UP" }  (ready to receive traffic)
GET /actuator/health/live   → { "status": "UP" }  (application is alive)
```

### What to monitor after deployment:

| Metric | What to watch |
|--------|--------------|
| **Error rate** | Should not increase after deployment |
| **Response time** | Should remain similar or improve |
| **CPU/Memory** | Should not spike unexpectedly |
| **Health checks** | Should return "UP" |
| **User complaints** | Monitor support channels |

---

## 📝 Exercises

### Exercise 1: Choose a Strategy

For each scenario, which deployment strategy would you recommend?

1. A small update to fix a typo in an error message
2. A major database migration that changes the schema
3. A new feature that you want to test with 5% of users first
4. A critical security patch that needs to go live immediately

### Exercise 2: Read Deployment Status

Given this Kubernetes output:
```
NAME        READY   STATUS    RESTARTS   AGE
my-app-a1   1/1     Running   0          5m
my-app-b2   1/1     Running   0          5m
my-app-c3   0/1     CrashLoopBackOff   3   2m
```

1. How many pods are running?
2. What's wrong with `my-app-c3`?
3. What would you do next?

### Exercise 3: Deployment Checklist

Create a deployment verification checklist that QA should follow after every deployment. Include at least 8 items.

---

## 📚 Additional Resources

- [Kubernetes Basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/) — Interactive K8s tutorial
- [Docker Getting Started](https://docs.docker.com/get-started/) — Docker fundamentals
- [Deployment Strategies (Kubernetes)](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy) — Official docs
- [Blue-Green Deployments (Martin Fowler)](https://martinfowler.com/bliki/BlueGreenDeployment.html) — Explanation

---

## ✅ Self-Check

- [ ] Explain the purpose of each environment (dev, QA, staging, prod)

<details>
<summary>Answer</summary>

**Development (Dev)** — where developers run and test their code locally or on a shared dev server; unstable, frequently changing. **QA / Testing** — dedicated environment for QA to test features, run automated tests, and verify bug fixes; more stable than dev. **Staging** — a near-exact replica of production; used for final verification, performance testing, and stakeholder demos before release. **Production (Prod)** — the live environment serving real users; highest stability requirements, changes go through all previous environments first. Each environment may have different configurations (database, API keys, feature flags).

</details>

- [ ] Describe rolling, blue-green, and canary deployment strategies

<details>
<summary>Answer</summary>

**Rolling deployment** — gradually replaces old instances with new ones, one at a time; no downtime, but both versions run simultaneously during the rollout; risk: if the new version has a bug, some users are affected before it's caught. **Blue-green deployment** — maintains two identical environments (blue = current, green = new); deploy to green, test it, then switch traffic from blue to green; instant rollback by switching back to blue. **Canary deployment** — routes a small percentage of traffic (e.g., 5%) to the new version first; monitor for errors; gradually increase traffic if healthy; safest but most complex.

</details>

- [ ] Explain what Docker containers are and why they're useful

<details>
<summary>Answer</summary>

A **Docker container** is a lightweight, isolated runtime environment that packages an application with everything it needs to run (code, runtime, libraries, OS dependencies). Benefits: **consistency** — "works on my machine" problem is eliminated; the container runs identically everywhere. **Isolation** — containers don't interfere with each other. **Portability** — runs on any machine with Docker installed. **Scalability** — easy to spin up multiple instances. **Reproducibility** — same image produces the same behavior every time. Containers are created from Docker **images** (immutable templates).

</details>

- [ ] Know basic kubectl commands to check pod status and logs

<details>
<summary>Answer</summary>

- `kubectl get pods` — list all pods and their status (Running, Pending, CrashLoopBackOff, Error)
- `kubectl get pods -n <namespace>` — list pods in a specific namespace
- `kubectl describe pod <name>` — detailed pod info, events, error reasons
- `kubectl logs <pod-name>` — view application logs from a pod
- `kubectl logs <pod-name> --previous` — view logs from a crashed/restarted pod
- `kubectl get deployments` — check deployment status and replica counts
- `kubectl rollout status deployment/<name>` — watch deployment progress

</details>

- [ ] Understand when and how to rollback a deployment

<details>
<summary>Answer</summary>

**When to rollback**: increased error rates after deployment, health checks failing, critical bugs found in production, performance degradation, or user-reported issues. **How to rollback**: in Kubernetes: `kubectl rollout undo deployment/<name>` reverts to the previous version. In blue-green: switch traffic back to the blue (old) environment. In CI/CD: re-deploy the previous known-good artifact version. Always have a rollback plan **before** deploying. Rollbacks should be fast (minutes, not hours) and tested as part of the deployment process.

</details>

- [ ] Know what health checks are and what to monitor after deployment

<details>
<summary>Answer</summary>

**Health checks** are automated probes that verify an application is running correctly. **Liveness check** — is the app alive? (restart if not). **Readiness check** — is the app ready to receive traffic? (remove from load balancer if not). After deployment, monitor: **error rates** (4xx/5xx responses), **response times** (latency), **CPU/memory usage**, **application logs** for exceptions, **database connections**, and **key business metrics** (e.g., orders per minute). Compare post-deployment metrics against pre-deployment baselines.

</details>

- [ ] Create a deployment verification checklist

<details>
<summary>Answer</summary>

Post-deployment checklist: ✅ Application is running (health endpoint returns 200). ✅ Smoke tests pass (critical paths work — login, main features). ✅ Error rate is normal (no spike in 5xx errors). ✅ Response times are normal (no performance degradation). ✅ Logs show no unexpected errors or exceptions. ✅ Database migrations completed successfully. ✅ External integrations are working (APIs, third-party services). ✅ Monitoring dashboards look healthy. ✅ Key business metrics are stable. ✅ Rollback plan is confirmed and ready if needed.

</details>
