# Course 1: Test Strategy & Processes

## 📖 What You Will Learn

- How to create and own a Test Strategy and Test Plan
- Fundamental test process activities (ISTQB-aligned)
- Traceability Matrix for requirements coverage
- Agile approaches and adaptive testing

---

## 1. Test Strategy & Test Plan

### Test Strategy

A **Test Strategy** is a high-level document that defines the overall approach to testing for a project or organization.

```
Test Strategy Contents:
┌─────────────────────────────────────────────────┐
│  1. Scope & Objectives                          │
│     - What will be tested / not tested          │
│     - Quality goals                             │
│                                                 │
│  2. Test Levels                                 │
│     - Unit → Integration → System → Acceptance  │
│                                                 │
│  3. Test Types                                  │
│     - Functional, Non-functional, Regression    │
│                                                 │
│  4. Test Approach                               │
│     - Manual vs. Automated                      │
│     - Risk-based prioritization                 │
│                                                 │
│  5. Tools & Infrastructure                      │
│     - Jira, TestRail, Selenium, Postman, etc.   │
│                                                 │
│  6. Environment Requirements                    │
│     - DEV, QA, Staging, Production              │
│                                                 │
│  7. Entry & Exit Criteria                       │
│     - When to start / stop testing              │
│                                                 │
│  8. Roles & Responsibilities                    │
│     - Who does what                             │
│                                                 │
│  9. Risk Management                             │
│     - Identified risks and mitigation           │
│                                                 │
│  10. Deliverables                               │
│      - Test reports, metrics, sign-off          │
└─────────────────────────────────────────────────┘
```

### Test Plan

A **Test Plan** is a more detailed, project-specific document derived from the Test Strategy.

```
Test Plan vs Test Strategy:

  Test Strategy                    Test Plan
  ─────────────                    ─────────
  Organization-wide                Project-specific
  Long-term                        Time-bound
  "How we test in general"         "How we test THIS project"
  Rarely changes                   Updated per release
  Defined by QA Lead               Defined by QA team
```

**Test Plan Key Sections (IEEE 829):**

| Section | Description | Example |
|---------|-------------|---------|
| Identifier | Unique ID | TP-2024-LOGIN-001 |
| Introduction | Purpose and scope | "Test plan for Login module v2.0" |
| Test Items | Features to test | Login, SSO, Password Reset |
| Features Not Tested | Excluded scope | Admin panel, Reporting |
| Approach | Testing techniques | BVA, EP, exploratory |
| Pass/Fail Criteria | How to judge | All P1 tests pass, 0 critical bugs |
| Suspension Criteria | When to pause | Build fails, env unavailable |
| Deliverables | Output artifacts | Test report, bug list, metrics |
| Schedule | Timeline | Sprint 1-3: Functional, Sprint 4: Regression |
| Staffing | Resources needed | 2 manual QA, 1 automation QA |

---

## 2. Fundamental Test Process Activities

The ISTQB defines seven key test process activities:

```
┌──────────────────────────────────────────────────────────────────┐
│                    TEST PROCESS LIFECYCLE                         │
│                                                                  │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │ 1. Test     │───→│ 2. Test     │───→│ 3. Test     │         │
│  │ Planning    │    │ Monitoring & │    │ Analysis    │         │
│  │             │    │ Control     │    │             │         │
│  └─────────────┘    └─────────────┘    └──────┬──────┘         │
│                                               │                 │
│  ┌─────────────┐    ┌─────────────┐    ┌──────▼──────┐         │
│  │ 7. Test     │←───│ 6. Test     │←───│ 4. Test     │         │
│  │ Completion  │    │ Execution   │    │ Design      │         │
│  │             │    │             │    │             │         │
│  └─────────────┘    └──────┬──────┘    └──────┬──────┘         │
│                            │                   │                 │
│                     ┌──────▼──────┐    ┌──────▼──────┐         │
│                     │             │←───│ 5. Test     │         │
│                     │             │    │ Implement.  │         │
│                     └─────────────┘    └─────────────┘         │
└──────────────────────────────────────────────────────────────────┘
```

### 2.1 Test Planning

```
Activities:
  - Define scope, objectives, approach
  - Identify resources, schedule, risks
  - Determine entry/exit criteria
  - Select tools and techniques
  - Create test plan document

Outputs:
  - Test Plan
  - Resource allocation
  - Schedule/timeline
```

### 2.2 Test Monitoring and Control

```
Monitoring (tracking progress):
  - Tests executed vs. planned
  - Pass/fail rates
  - Defect discovery rate
  - Coverage metrics
  - Schedule adherence

Control (taking corrective action):
  - Re-prioritize tests if behind schedule
  - Add resources if needed
  - Adjust scope based on risks
  - Escalate blockers

Tools: Jira dashboards, TestRail reports, Allure trends
```

### 2.3 Test Analysis ("What to test?")

```
Activities:
  - Analyze requirements, user stories, acceptance criteria
  - Identify test conditions
  - Evaluate testability of requirements
  - Identify gaps or ambiguities in requirements

Example:
  Requirement: "Users must be able to log in with email and password"
  
  Test Conditions:
    ✓ Valid email + valid password → success
    ✓ Valid email + invalid password → error
    ✓ Invalid email format → validation error
    ✓ Empty fields → validation error
    ✓ SQL injection attempt → blocked
    ✓ Locked account → specific error message
```

### 2.4 Test Design ("How to test?")

```
Activities:
  - Create test cases from test conditions
  - Apply test design techniques (BVA, EP, decision tables)
  - Design test data
  - Design expected results
  - Identify test environment needs

Example Test Case:
  ID: TC-LOGIN-001
  Title: Valid login with correct credentials
  Precondition: User account exists and is active
  Steps:
    1. Navigate to login page
    2. Enter valid email
    3. Enter valid password
    4. Click "Login"
  Expected: User redirected to dashboard
  Priority: P1
```

### 2.5 Test Implementation ("Is everything ready?")

```
Activities:
  - Create test procedures (execution order)
  - Prepare test data
  - Set up test environments
  - Prepare automated test scripts
  - Create test suites
  - Verify test environment readiness

Checklist:
  □ Test data created/loaded
  □ Test environment configured
  □ Test accounts provisioned
  □ Automation scripts ready
  □ Third-party integrations mocked/available
  □ Test suite organized by priority
```

### 2.6 Test Execution

```
Activities:
  - Execute tests (manual or automated)
  - Compare actual vs. expected results
  - Log defects for failures
  - Record test results
  - Re-test fixed defects
  - Regression testing

Defect Report Template:
  ┌──────────────────────────────────────┐
  │ Title: Login fails with valid creds  │
  │ Severity: Critical                   │
  │ Priority: P1                         │
  │ Environment: QA, Chrome 120          │
  │ Steps to Reproduce:                  │
  │   1. Go to /login                    │
  │   2. Enter valid email/password      │
  │   3. Click Login                     │
  │ Expected: Redirect to dashboard      │
  │ Actual: 500 Internal Server Error    │
  │ Attachments: screenshot, logs        │
  └──────────────────────────────────────┘
```

### 2.7 Test Completion

```
Activities:
  - Collect and archive test artifacts
  - Evaluate exit criteria fulfillment
  - Create test summary report
  - Identify lessons learned
  - Hand over test assets for maintenance

Test Summary Report:
  - Total tests: 250
  - Passed: 235 (94%)
  - Failed: 10 (4%)
  - Blocked: 5 (2%)
  - Critical bugs open: 0
  - Coverage: 97% of requirements
  - Recommendation: Ready for release ✓
```

---

## 3. Traceability Matrix

A **Traceability Matrix** maps requirements to test cases, ensuring full coverage.

```
┌──────────────┬─────────────────┬──────────────┬────────┬─────────┐
│ Requirement  │ Description     │ Test Cases   │ Status │ Coverage│
├──────────────┼─────────────────┼──────────────┼────────┼─────────┤
│ REQ-001      │ User login      │ TC-001..005  │ Passed │ 100%    │
│ REQ-002      │ Password reset  │ TC-006..009  │ Passed │ 100%    │
│ REQ-003      │ User profile    │ TC-010..015  │ Failed │ 83%     │
│ REQ-004      │ Admin panel     │ TC-016..020  │ Blocked│ 60%     │
│ REQ-005      │ Search          │ —            │ No TC  │ 0%  ⚠  │
└──────────────┴─────────────────┴──────────────┴────────┴─────────┘

Key Benefits:
  ✓ Identifies untested requirements (REQ-005 above!)
  ✓ Shows test coverage per requirement
  ✓ Tracks test execution status
  ✓ Supports impact analysis when requirements change
  ✓ Required for audit and compliance
```

### Types of Traceability

```
Forward Traceability:
  Requirements → Test Cases
  "Is every requirement tested?"

Backward Traceability:
  Test Cases → Requirements
  "Does every test case trace to a requirement?"

Bi-directional Traceability:
  Both directions — the gold standard
  Ensures no orphan tests and no untested requirements
```

---

## 4. Agile Approaches and Adaptive Testing

### Testing in Agile

```
Agile Testing Quadrants (Brian Marick):
                    
         Business-Facing
              │
    Q2        │        Q3
  Functional  │  Exploratory
  Tests,      │  Testing,
  Story Tests,│  Usability,
  Prototypes  │  UAT, Alpha/Beta
              │
  ────────────┼────────────────
              │
    Q1        │        Q4
  Unit Tests, │  Performance,
  Component   │  Load, Security
  Tests       │  "-ility" Tests
              │
         Technology-Facing

  Q1 & Q2: Guide development (written before/during code)
  Q3 & Q4: Critique the product (written after code)
```

### Adaptive Testing for Changing Requirements

```
Challenge: Requirements change mid-sprint

Strategies:
  1. Lightweight test cases — easy to update
  2. Exploratory testing — adapt in real-time
  3. Risk-based approach — test high-risk areas first
  4. Automation — quick regression when things change
  5. Continuous communication — daily standups, refinement

Example Sprint Adaptation:
  Day 1-2: Story refined, write test conditions
  Day 3: Requirement changed → update test conditions
  Day 4-5: Execute tests, exploratory session
  Day 6: Regression check, bug fixes verified
  Day 7-8: Test completion, demo preparation
```

### Whole-Team Approach to Quality

```
In Agile, quality is everyone's responsibility:

  Developer:  Writes unit tests, reviews test automation
  QA:         Designs tests, exploratory testing, automation
  PO:         Defines acceptance criteria, UAT
  Scrum Master: Removes testing blockers
  
  Everyone:   Participates in 3 Amigos sessions
              (PO + Dev + QA discuss stories together)
```

---

## ✅ Self-Check Questions

1. What is the difference between a Test Strategy and a Test Plan?

<details>
<summary>Answer</summary>

A **Test Strategy** is a high-level, organization-wide document defining the general testing approach, tools, and standards. It rarely changes. A **Test Plan** is project-specific, time-bound, and details how testing will be done for a particular project or release. The Test Plan is derived from the Test Strategy and is updated per release cycle.

</details>

2. Name all seven fundamental test process activities in order.

<details>
<summary>Answer</summary>

1. **Test Planning** — define scope, objectives, approach, resources
2. **Test Monitoring & Control** — track progress, take corrective actions
3. **Test Analysis** — identify test conditions from requirements ("what to test")
4. **Test Design** — create test cases, apply techniques ("how to test")
5. **Test Implementation** — prepare data, environments, suites ("is everything ready")
6. **Test Execution** — run tests, log defects, compare results
7. **Test Completion** — archive artifacts, create summary report, lessons learned

</details>

3. What is the purpose of a Traceability Matrix?

<details>
<summary>Answer</summary>

A Traceability Matrix maps requirements to test cases to ensure: (1) every requirement has at least one test case (forward traceability), (2) every test case links to a requirement (backward traceability), (3) coverage gaps are visible, (4) impact of requirement changes can be assessed quickly. It's essential for audit, compliance, and ensuring no requirements are missed during testing.

</details>

4. How does testing differ in Agile vs. Waterfall?

<details>
<summary>Answer</summary>

In **Waterfall**, testing is a distinct phase after development, with comprehensive upfront documentation. In **Agile**, testing is continuous throughout each sprint — QA participates from story refinement to demo. Key Agile differences: lightweight test cases, exploratory testing, whole-team quality ownership, 3 Amigos sessions, and adaptive testing that responds to changing requirements. Automation is critical in Agile for fast regression feedback.

</details>

5. What entry and exit criteria would you define for system testing?

<details>
<summary>Answer</summary>

**Entry Criteria:** (1) Test plan approved, (2) test environment ready and stable, (3) test data prepared, (4) all critical/high unit and integration tests passing, (5) build deployed to QA environment. **Exit Criteria:** (1) All P1/P2 test cases executed, (2) 95%+ pass rate, (3) zero critical/blocker defects open, (4) all major defects resolved or deferred with stakeholder approval, (5) test summary report completed, (6) stakeholder sign-off obtained.

</details>