# Course 06: Non-Functional & Platform Testing

## 📖 What You Will Learn

- Performance and load testing fundamentals
- Security validation basics (OWASP Top 10)
- Cross-platform and cross-browser testing
- Observability and test health monitoring
- Basic mobile testing awareness

---

## 1. Performance and Load Testing Fundamentals

### What is Performance Testing?

Performance testing verifies that the system meets **speed, scalability, and stability** requirements under expected and peak load conditions.

### Types of Performance Tests:

| Type | Purpose | Example |
|------|---------|---------|
| **Load Testing** | Verify behavior under expected load | 500 concurrent users browsing products |
| **Stress Testing** | Find breaking point beyond normal load | Increase from 500 to 5,000 users until failure |
| **Spike Testing** | Handle sudden bursts of traffic | Flash sale: 0 to 10,000 users in 10 seconds |
| **Soak/Endurance** | Stability over extended time | 200 users for 24 hours (find memory leaks) |
| **Scalability** | How well system scales with resources | Does doubling servers double capacity? |

### Key Performance Metrics:

| Metric | What It Measures | Target Example |
|--------|-----------------|----------------|
| **Response Time** | Time from request to response | < 200ms (API), < 3s (page load) |
| **Throughput** | Requests handled per second | > 1000 req/sec |
| **Error Rate** | % of failed requests | < 1% under load |
| **Concurrent Users** | Simultaneous active users | 500+ without degradation |
| **P95/P99 Latency** | 95th/99th percentile response time | P95 < 500ms, P99 < 1s |
| **CPU/Memory** | Server resource usage | < 80% under peak load |

### Performance Testing with k6:

```javascript
// k6 load test script
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '2m', target: 100 },  // Ramp up to 100 users
    { duration: '5m', target: 100 },  // Stay at 100 for 5 min
    { duration: '2m', target: 0 },    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% under 500ms
    http_req_failed: ['rate<0.01'],    // <1% errors
  },
};

export default function () {
  const res = http.get('https://api.example.com/products');
  
  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
  
  sleep(1); // Think time between requests
}
```

### Performance Test Checklist for QA:

- [ ] Define performance SLAs with stakeholders
- [ ] Identify critical user journeys to test
- [ ] Set up realistic test data volume
- [ ] Run baseline test with single user
- [ ] Run load test at expected peak
- [ ] Run stress test to find breaking point
- [ ] Monitor server resources during tests
- [ ] Analyze results and identify bottlenecks
- [ ] Report findings with recommendations

---

## 2. Security Validation Basics (OWASP Top 10)

### OWASP Top 10 — What QA Should Know

The **OWASP Top 10** lists the most critical web application security risks.

| # | Risk | QA Verification |
|---|------|----------------|
| 1 | **Broken Access Control** | Can user A access user B's data? Can regular user access admin endpoints? |
| 2 | **Cryptographic Failures** | Are passwords hashed? Is data transmitted over HTTPS? Are API keys exposed? |
| 3 | **Injection** | Does SQL injection work? XSS in input fields? Command injection? |
| 4 | **Insecure Design** | Are there rate limits? Business logic flaws? Missing validation? |
| 5 | **Security Misconfiguration** | Default credentials? Verbose error messages? Unnecessary features? |
| 6 | **Vulnerable Components** | Are dependencies up to date? Known CVEs in libraries? |
| 7 | **Authentication Failures** | Weak passwords allowed? Brute force possible? Session management? |
| 8 | **Data Integrity Failures** | Can users tamper with data? Are updates verified? |
| 9 | **Logging Failures** | Are security events logged? Are logs tamper-proof? |
| 10 | **SSRF** | Can the server be tricked into making unintended requests? |

### Security Tests QA Can Perform:

```
# Access Control Tests
1. Access another user's profile: GET /api/users/OTHER_USER_ID → should be 403
2. Access admin panel as regular user → should be 403
3. Access API without token → should be 401
4. Use expired token → should be 401
5. Use token from different environment → should be 401

# Input Validation / Injection Tests
6. SQL injection: name = "'; DROP TABLE users; --" → should be rejected
7. XSS: name = "<script>alert('xss')</script>" → should be escaped
8. Large payload: send 10MB body → should be rejected (413)
9. Special characters in all fields → should be handled safely

# Authentication Tests
10. Login with wrong password 10 times → should lock account/rate limit
11. Password "123" → should be rejected (too weak)
12. Session timeout after inactivity → should require re-login

# Data Exposure Tests
13. Error responses don't expose stack traces
14. API doesn't return password hashes
15. Sensitive data in logs is masked
```

### Security Checklist for QA:

| Area | Check |
|------|-------|
| **HTTPS** | All endpoints use HTTPS, HTTP redirects to HTTPS |
| **Headers** | Security headers present (CSP, X-Frame-Options, etc.) |
| **Tokens** | JWT tokens expire, refresh tokens rotate |
| **Passwords** | Minimum complexity enforced, stored hashed |
| **Errors** | No stack traces or internal details in responses |
| **CORS** | Only allowed origins can make requests |
| **Rate Limiting** | Brute force protection on login and sensitive endpoints |

---

## 3. Cross-Platform and Cross-Browser Testing

### Browser Testing Matrix:

| Browser | Engine | Market Share | Priority |
|---------|--------|-------------|----------|
| Chrome | Chromium/Blink | ~65% | 🔴 Critical |
| Safari | WebKit | ~18% | 🔴 Critical (especially mobile) |
| Firefox | Gecko | ~3% | 🟡 Medium |
| Edge | Chromium/Blink | ~5% | 🟡 Medium |
| Samsung Internet | Chromium | ~2% | 🟢 Low (mobile) |

### Cross-Browser Testing Strategy:

```typescript
// Playwright cross-browser config
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  projects: [
    // Desktop browsers
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
    
    // Mobile viewports
    { name: 'mobile-chrome', use: { ...devices['Pixel 5'] } },
    { name: 'mobile-safari', use: { ...devices['iPhone 13'] } },
    
    // Tablet
    { name: 'tablet', use: { ...devices['iPad Pro 11'] } },
  ],
});
```

### What to Check Cross-Browser:

| Category | What to Verify |
|----------|---------------|
| **Layout** | Page renders correctly, no overlapping elements |
| **Fonts** | Correct fonts load, fallbacks work |
| **Forms** | Input types work (date picker, dropdowns, validation) |
| **JavaScript** | No console errors, features work |
| **CSS** | Flexbox/Grid layouts, animations, media queries |
| **Images** | All images load, responsive images work |
| **Navigation** | Links, routing, back/forward buttons |
| **Performance** | Acceptable load times across browsers |

### Responsive Design Testing:

| Breakpoint | Width | Devices |
|-----------|-------|---------|
| Mobile S | 320px | iPhone SE, older phones |
| Mobile M | 375px | iPhone X/12/13 |
| Mobile L | 425px | Large phones |
| Tablet | 768px | iPad, Android tablets |
| Laptop | 1024px | Small laptops |
| Desktop | 1440px | Standard desktop |
| 4K | 2560px | Large monitors |

---

## 4. Observability and Test Health Monitoring

### What is Observability?

Observability is the ability to understand the **internal state** of a system from its **external outputs**: logs, metrics, and traces.

### Three Pillars of Observability:

| Pillar | What It Provides | Tools |
|--------|-----------------|-------|
| **Logs** | Detailed event records | ELK Stack, Splunk, CloudWatch |
| **Metrics** | Numerical measurements over time | Prometheus, Grafana, Datadog |
| **Traces** | Request flow across services | Jaeger, Zipkin, OpenTelemetry |

### QA's Role in Observability:

```
1. VERIFY LOGGING
   - Are errors logged with enough detail?
   - Are log levels correct (INFO, WARN, ERROR)?
   - Are sensitive data masked in logs?
   - Can you trace a user action through the logs?

2. MONITOR DASHBOARDS
   - Error rate dashboard after deployment
   - Response time trends
   - Alert thresholds are set correctly

3. TEST HEALTH MONITORING
   - Health check endpoints return correct status
   - Readiness/liveness probes work
   - Alerts fire when thresholds are breached
```

### Test Health Dashboard Metrics:

| Metric | Description | Target |
|--------|-------------|--------|
| Test pass rate | % of tests passing | > 98% |
| Flaky test rate | % of inconsistent tests | < 2% |
| Test execution time | Total suite duration | < 30 min for full regression |
| Test coverage | % of requirements covered | > 90% |
| Defect escape rate | Bugs found in production | < 5% of total bugs |

---

## 5. Basic Mobile Testing Awareness

### Mobile Testing Considerations:

| Factor | What to Test |
|--------|-------------|
| **Screen sizes** | Various resolutions (320px to 428px width) |
| **Orientation** | Portrait and landscape modes |
| **Touch gestures** | Tap, swipe, pinch-to-zoom, long press |
| **Network conditions** | WiFi, 4G, 3G, offline mode |
| **Interruptions** | Incoming calls, notifications, app switching |
| **Battery** | Performance under low battery mode |
| **Permissions** | Camera, location, notifications, storage |
| **App lifecycle** | Background, foreground, kill and restart |

### Mobile Testing Tools:

| Tool | Purpose |
|------|---------|
| **Appium** | Cross-platform mobile automation (iOS + Android) |
| **XCTest** | Native iOS testing |
| **Espresso** | Native Android testing |
| **BrowserStack/Sauce Labs** | Cloud device farms |
| **Chrome DevTools** | Mobile emulation in desktop browser |
| **Playwright** | Mobile web emulation |

### Mobile Web vs. Native App vs. Hybrid:

| Aspect | Mobile Web | Native App | Hybrid |
|--------|-----------|-----------|--------|
| **Technology** | HTML/CSS/JS in browser | Swift/Kotlin | React Native, Flutter |
| **Testing Tools** | Playwright, Cypress | XCTest, Espresso | Appium, Detox |
| **Distribution** | URL | App Store | App Store |
| **Performance** | Good | Best | Good |
| **QA Focus** | Browser compatibility | OS version compatibility | Both |

---

## 📝 Exercises

### Exercise 1: Performance Test Plan

Design a performance test plan for an e-commerce checkout flow. Define:
1. What to test
2. Load profiles
3. Acceptance criteria
4. Monitoring metrics

<details>
<summary>Click to see answer</summary>

**Scenarios to test:**
- Browse products (GET /products) — highest traffic
- Add to cart (POST /cart) — medium traffic
- Checkout (POST /orders) — lowest traffic but most critical
- Payment processing (POST /payments) — slowest operation

**Load profiles:**
- Normal: 200 concurrent users, 60% browsing, 30% cart, 10% checkout
- Peak (Black Friday): 2000 concurrent users, same ratios
- Spike: 0 to 5000 users in 30 seconds

**Acceptance criteria:**
- Response time: P95 < 500ms for browse, P95 < 2s for checkout
- Throughput: > 500 requests/sec at normal load
- Error rate: < 0.5% at normal, < 2% at peak
- Zero data loss for payment transactions

**Monitoring:**
- Application: response times, error rates, throughput
- Infrastructure: CPU, memory, disk I/O, network
- Database: query times, connection pool, deadlocks
- External: payment gateway latency, CDN cache hit rate

</details>

### Exercise 2: Security Test Cases

Write 10 security test cases for a user registration endpoint `POST /api/register`.

<details>
<summary>Click to see answer</summary>

1. **SQL injection**: email = `"' OR 1=1; --"` → should return 400, not expose DB
2. **XSS**: name = `"<script>alert('xss')</script>"` → should escape/reject
3. **Weak password**: password = `"123"` → should return 400 with password policy
4. **Password in response**: Response should NOT contain the password field
5. **Duplicate email**: Register with existing email → should return 409
6. **Rate limiting**: 50 registrations in 1 minute from same IP → should return 429
7. **Large payload**: 10MB request body → should return 413
8. **Missing HTTPS**: HTTP request → should redirect to HTTPS or refuse
9. **CORS**: Request from unauthorized origin → should be blocked
10. **Password hashing**: Check DB — password should be bcrypt/argon2 hash, not plaintext

</details>

---

## ✅ Self-Check Questions

- [ ] What are the types of performance tests and when to use each?

<details>
<summary>Answer</summary>

**Load testing**: verify normal/expected traffic — run regularly. **Stress testing**: find breaking point — run before major releases. **Spike testing**: sudden traffic bursts — for flash sales, marketing events. **Soak testing**: extended duration — find memory leaks, connection exhaustion. **Scalability testing**: verify horizontal/vertical scaling — for capacity planning. Key metrics: response time (P95/P99), throughput (req/sec), error rate, resource utilization.

</details>

- [ ] What OWASP Top 10 risks should QA test for?

<details>
<summary>Answer</summary>

Priority for QA: (1) **Broken Access Control** — can users access unauthorized resources? (2) **Injection** — SQL injection, XSS in input fields. (3) **Authentication Failures** — weak passwords, missing rate limiting, session management. (4) **Security Misconfiguration** — verbose errors, default credentials. (5) **Cryptographic Failures** — HTTPS enforcement, password hashing. QA should verify these during regular testing, not just in security-specific testing phases.

</details>

- [ ] How do you approach cross-browser testing efficiently?

<details>
<summary>Answer</summary>

(1) Prioritize browsers by user analytics (Chrome + Safari cover ~83%). (2) Run full test suite on primary browser (Chrome), critical paths on others. (3) Use tools like Playwright that support multiple browser engines. (4) Test responsive breakpoints: mobile (375px), tablet (768px), desktop (1440px). (5) Focus cross-browser checks on layout, forms, CSS features, and JavaScript compatibility. (6) Use cloud device farms (BrowserStack) for devices you don't have locally.

</details>

- [ ] What is observability and how does QA use it?

<details>
<summary>Answer</summary>

Observability = understanding system state from external outputs: **Logs** (event records), **Metrics** (numerical data over time), **Traces** (request flow across services). QA uses it to: verify errors are logged correctly, monitor dashboards after deployments, validate alerting thresholds, trace bugs through distributed systems, and maintain test health dashboards tracking pass rates, flaky tests, and execution times.

</details>