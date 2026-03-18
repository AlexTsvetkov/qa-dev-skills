# Course 09: Test Design & Methodologies (Senior Core)

## 📖 What You Will Learn

- Test design techniques: boundary value analysis, equivalence partitioning, decision tables, state transitions
- Exploratory and session-based testing
- Test pyramid understanding
- QA process improvement and continuous learning

---

## 1. Test Design Techniques

### Why Formal Techniques Matter

You can't test everything. Design techniques help you select the **smallest set of tests** that finds the **most bugs**.

### Equivalence Partitioning (EP)

Divide inputs into groups (partitions) where the system should behave the same way. Test one value from each partition.

```
Input: Age field (valid range: 18-65)

Partitions:
  Invalid low:   age < 18     → Test with 15
  Valid:          18 ≤ age ≤ 65 → Test with 30
  Invalid high:  age > 65     → Test with 70

3 tests instead of testing every number from 0 to 200.
```

### Boundary Value Analysis (BVA)

Bugs cluster at boundaries. Test values at, just below, and just above each boundary.

```
Input: Age field (valid range: 18-65)

Boundaries:        17 | 18 | 19 ... 64 | 65 | 66
                    ↑    ↑    ↑      ↑    ↑    ↑
               invalid valid valid  valid valid invalid

Test values: 17, 18, 19, 64, 65, 66  (6 tests)
```

### Decision Tables

For features with multiple conditions and combinations.

```
Feature: Free shipping rules
- Order > $100: free shipping
- Premium member: free shipping
- Promo code FREESHIP: free shipping

| # | Order > $100 | Premium | Promo Code | Free Shipping? |
|---|-------------|---------|------------|---------------|
| 1 | No          | No      | No         | No            |
| 2 | Yes         | No      | No         | Yes           |
| 3 | No          | Yes     | No         | Yes           |
| 4 | No          | No      | Yes        | Yes           |
| 5 | Yes         | Yes     | No         | Yes           |
| 6 | Yes         | No      | Yes        | Yes           |
| 7 | No          | Yes     | Yes        | Yes           |
| 8 | Yes         | Yes     | Yes        | Yes           |
```

### State Transition Testing

For features where behavior depends on the current state.

```
Order States:
  [Created] → Pay → [Paid] → Ship → [Shipped] → Deliver → [Delivered]
                                         ↓
       [Created] → Cancel → [Cancelled]  Return → [Returned]

Test valid transitions:
  Created → Paid ✅
  Paid → Shipped ✅
  Shipped → Delivered ✅

Test invalid transitions:
  Created → Shipped ❌ (can't skip payment)
  Delivered → Paid ❌ (can't go backward)
  Cancelled → Shipped ❌ (can't ship cancelled order)
```

### Pairwise/Combinatorial Testing

When there are too many combinations to test all, test all **pairs** of values.

```
Browser: Chrome, Firefox, Safari
OS: Windows, macOS, Linux
Language: English, Spanish, French

All combinations: 3 × 3 × 3 = 27 tests
Pairwise (all pairs covered): ~9 tests

| # | Browser | OS      | Language |
|---|---------|---------|----------|
| 1 | Chrome  | Windows | English  |
| 2 | Chrome  | macOS   | Spanish  |
| 3 | Chrome  | Linux   | French   |
| 4 | Firefox | Windows | French   |
| 5 | Firefox | macOS   | English  |
| 6 | Firefox | Linux   | Spanish  |
| 7 | Safari  | Windows | Spanish  |
| 8 | Safari  | macOS   | French   |
| 9 | Safari  | Linux   | English  |
```

---

## 2. Exploratory and Session-Based Testing

### Exploratory Testing

Simultaneous test design, execution, and learning — guided by skill, experience, and intuition.

```
Exploratory Testing Charter:

EXPLORE: [area/feature]
WITH: [resources/techniques/data]
TO DISCOVER: [what you're looking for]

Example:
EXPLORE: Checkout flow
WITH: Various payment methods, edge case amounts ($0.01, $99999)
TO DISCOVER: Payment processing errors and UI issues
```

### Session-Based Test Management (SBTM)

Structure exploratory testing into timed sessions with clear charters.

```
Session Sheet:
- Charter: Test checkout with international addresses
- Tester: Nastya
- Duration: 60 minutes
- Start: 14:00

Session Notes:
14:00 - Starting with UK addresses, postcode format validation
14:12 - BUG: UK postcodes with spaces rejected (e.g., "SW1A 1AA")
14:20 - Testing Japanese addresses — character encoding works
14:35 - German addresses with umlauts (ä, ö, ü) — works
14:42 - BUG: Address line 2 truncated at 50 chars (no UI warning)
14:55 - Testing very long city names — handled correctly

Summary:
- Bugs found: 2
- Areas covered: UK, Japan, Germany, edge cases
- Areas not covered: Middle East RTL, Latin America
- Follow-up: Need RTL testing session
```

### When to Use Each Approach:

| Approach | Best For |
|----------|----------|
| Scripted test cases | Regression, compliance, audit |
| Exploratory testing | New features, complex workflows, finding unexpected bugs |
| Session-based | Structured exploration with tracking |
| Ad-hoc testing | Quick sanity checks, smoke testing |

---

## 3. Test Pyramid

### The Classic Test Pyramid:

```
        /  E2E  \          Few, slow, expensive
       /  Tests  \         (Browser/UI tests)
      /───────────\
     / Integration \       Medium amount
    /    Tests      \      (API, service tests)
   /─────────────────\
  /    Unit Tests      \   Many, fast, cheap
 /______________________\  (Function/class level)
```

### Practical Distribution:

| Level | % of Tests | Speed | Maintained By | Example |
|-------|-----------|-------|---------------|---------|
| Unit | 60-70% | Milliseconds | Developers | Function returns correct value |
| Integration | 20-30% | Seconds | Dev + QA | API endpoint returns correct response |
| E2E | 5-10% | Minutes | QA | User can complete checkout flow |

### Anti-Patterns:

```
Ice Cream Cone (Bad):        Pyramid (Good):
    ___________                    /\
   |  Manual   |                  /E2E\
   |  Testing  |                 /─────\
   |___________|                / Integ \
   | E2E Tests |               /─────────\
   |___________|              /   Unit     \
   | Few Unit  |             /_______________\
   |___________|

Problem: Slow, expensive,     Fast, reliable,
fragile, hard to maintain     easy to maintain
```

### Choosing the Right Test Level:

| What to Test | Best Level | Why |
|-------------|-----------|-----|
| Business logic calculation | Unit | Fast, isolated |
| API request/response | Integration | Tests real HTTP |
| Database queries | Integration | Tests real DB |
| Full user journey | E2E | Tests entire stack |
| Visual layout | E2E / Visual | Needs real browser |
| Third-party integration | Integration + Mock | Isolate external dependency |

---

## 4. QA Process Improvement

### Continuous Improvement Cycle:

```
  MEASURE → ANALYZE → IMPROVE → MEASURE ...

1. MEASURE: Collect data on current process
   - Bug escape rate, test execution time, flaky test %

2. ANALYZE: Identify problems and root causes
   - Why are bugs escaping? Where are bottlenecks?

3. IMPROVE: Implement changes
   - Add API tests, automate regression, fix flaky tests

4. MEASURE: Verify improvement
   - Did bug escape rate decrease? Is feedback faster?
```

### Common QA Process Improvements:

| Problem | Improvement |
|---------|-------------|
| Slow feedback | Shift testing left, add unit/API tests |
| Bugs escape to production | Add missing test coverage, improve test design |
| Flaky tests ignored | Quarantine policy, dedicated fix sprints |
| Manual regression takes days | Automate critical paths first |
| QA is bottleneck | Enable developer testing, self-service test environments |
| No visibility into quality | Create quality dashboards |

### QA Maturity Model:

| Level | Characteristics |
|-------|----------------|
| **1 — Ad Hoc** | No process, reactive testing only |
| **2 — Managed** | Test cases exist, basic bug tracking |
| **3 — Defined** | Test strategy, automation framework, metrics |
| **4 — Measured** | Data-driven decisions, continuous improvement |
| **5 — Optimized** | Predictive quality, shift-left, quality culture |

---

## 📝 Exercises

### Exercise 1: Apply Test Design Techniques

Given a password validation feature (8-20 chars, at least 1 uppercase, 1 lowercase, 1 digit, 1 special char), design test cases using BVA and EP.

<details>
<summary>Click to see answer</summary>

**Equivalence Partitions:**
- Length: too short (<8), valid (8-20), too long (>20)
- Uppercase: present, missing
- Lowercase: present, missing
- Digit: present, missing
- Special char: present, missing

**Boundary Values for length:** 7, 8, 20, 21

**Test cases:**

| # | Password | Expected | Technique |
|---|----------|----------|-----------|
| 1 | `Ab1@xyz` (7 chars) | Invalid — too short | BVA |
| 2 | `Ab1@xyzw` (8 chars) | Valid — min length | BVA |
| 3 | `Ab1@` + 16 chars (20) | Valid — max length | BVA |
| 4 | `Ab1@` + 17 chars (21) | Invalid — too long | BVA |
| 5 | `ab1@xyzw` | Invalid — no uppercase | EP |
| 6 | `AB1@XYZW` | Invalid — no lowercase | EP |
| 7 | `Abx@xyzw` | Invalid — no digit | EP |
| 8 | `Ab1xxyzw` | Invalid — no special | EP |
| 9 | `Ab1@xyzw` | Valid — all criteria | EP |

</details>

### Exercise 2: Write an Exploratory Testing Charter

Write 3 exploratory testing charters for testing a new "dark mode" feature on a web application.

<details>
<summary>Click to see answer</summary>

**Charter 1:**
EXPLORE: All page types in dark mode (home, product listing, checkout, account)
WITH: Chrome and Safari on desktop
TO DISCOVER: Pages where dark mode is not applied or has visual defects

**Charter 2:**
EXPLORE: Dark mode toggle behavior and persistence
WITH: Multiple sessions, different browsers, incognito mode
TO DISCOVER: Whether preference persists correctly across sessions, browsers, and page reloads

**Charter 3:**
EXPLORE: Dark mode accessibility and readability
WITH: Color contrast analyzers, screen readers, various content types (text, images, charts)
TO DISCOVER: Contrast ratio violations, unreadable text, invisible UI elements, accessibility compliance issues

</details>

---

## ✅ Self-Check Questions

- [ ] When would you use each test design technique?

<details>
<summary>Answer</summary>

**Equivalence Partitioning**: When inputs have clear valid/invalid ranges — reduces tests by testing one value per partition. **Boundary Value Analysis**: Always combine with EP — test at the edges where bugs cluster. **Decision Tables**: When multiple conditions combine to produce different outcomes (business rules). **State Transition**: When the system has distinct states and behavior depends on current state (orders, workflows). **Pairwise**: When there are many input combinations and full coverage is impractical.

</details>

- [ ] What is the test pyramid and why does it matter?

<details>
<summary>Answer</summary>

The test pyramid recommends: many fast **unit tests** (60-70%) at the base, medium **integration tests** (20-30%) in the middle, and few slow **E2E tests** (5-10%) at the top. It matters because: (1) Fast feedback — unit tests run in milliseconds. (2) Cost-effective — lower tests are cheaper to write and maintain. (3) Reliable — fewer flaky tests at lower levels. (4) The anti-pattern "ice cream cone" (many E2E, few unit tests) leads to slow, fragile, expensive test suites.

</details>

- [ ] How does exploratory testing differ from scripted testing?

<details>
<summary>Answer</summary>

**Scripted testing**: predefined steps, expected results, repeatable, good for regression. **Exploratory testing**: simultaneous design and execution, guided by tester's skill and intuition, discovers unexpected bugs. Exploratory is structured with **charters** (what to explore, how, what to find) and **sessions** (timed, documented). It's not random — it's skilled investigation. Best used for new features, complex workflows, and finding bugs that scripted tests miss.

</details>

- [ ] How do you drive QA process improvement?

<details>
<summary>Answer</summary>

Follow the cycle: **Measure** current state (bug escape rate, test times, flaky rate) → **Analyze** root causes of problems → **Improve** with targeted changes (automate regression, add API tests, fix flaky tests) → **Measure** again to verify improvement. Prioritize improvements by impact. Use a maturity model to understand current level and set goals. Key improvements: shift-left testing, automation of critical paths, quality dashboards for visibility, and enabling developer testing.

</details>