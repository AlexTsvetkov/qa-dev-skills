# Course 3: Quality Management

## 📖 What You Will Learn

- Project risks vs. product risks
- Risk-based testing techniques
- Metrics for measuring quality
- Verification & Validation concepts
- Validation in API responses

---

## 1. Project Risks vs. Product Risks

### Project Risks (Affect the Schedule/Resources)

```
Project risks threaten the ability to deliver the project:

  ┌─────────────────────────────────────────────────────┐
  │  Organizational        │ Technical                   │
  │  - Staff leaving       │ - Environment unavailable   │
  │  - Budget cuts         │ - Tool limitations          │
  │  - Unclear scope       │ - Late deliveries from dev  │
  │  - Skill gaps in team  │ - Integration problems      │
  │  - Supplier delays     │ - Complex architecture      │
  └─────────────────────────────────────────────────────┘

  Mitigation: Project management, contingency planning, risk registers
```

### Product Risks (Affect the Software Quality)

```
Product risks threaten the quality of the delivered product:

  ┌───────────────────────────────────────────────────────┐
  │  Functional             │ Non-Functional              │
  │  - Incorrect logic      │ - Poor performance          │
  │  - Missing features     │ - Security vulnerabilities  │
  │  - Data corruption      │ - Usability issues          │
  │  - Integration failures │ - Reliability problems      │
  │  - Calculation errors   │ - Scalability limits        │
  └───────────────────────────────────────────────────────┘

  Mitigation: Testing! — risk-based test prioritization
```

---

## 2. Risk-Based Testing

```
Risk Level = Likelihood × Impact

  ┌─────────┬──────────┬──────────┬──────────┐
  │         │ Low      │ Medium   │ High     │
  │         │ Impact   │ Impact   │ Impact   │
  ├─────────┼──────────┼──────────┼──────────┤
  │ High    │ Medium   │ HIGH     │ CRITICAL │
  │ Likely  │          │          │          │
  ├─────────┼──────────┼──────────┼──────────┤
  │ Medium  │ Low      │ Medium   │ HIGH     │
  │ Likely  │          │          │          │
  ├─────────┼──────────┼──────────┼──────────┤
  │ Low     │ Low      │ Low      │ Medium   │
  │ Likely  │          │          │          │
  └─────────┴──────────┴──────────┴──────────┘

Actions by Risk Level:
  CRITICAL → Extensive testing, multiple techniques, automation
  HIGH     → Thorough testing, prioritize early
  MEDIUM   → Standard testing effort
  LOW      → Minimal testing, may defer

Example:
  Feature: Payment Processing
  Likelihood: Medium (new feature, complex logic)
  Impact: Critical (financial loss, legal issues)
  Risk Level: HIGH → Test extensively with BVA, negative tests, security
```

---

## 3. Quality Metrics

```
┌────────────────────────┬───────────────────────────────────────┐
│ Metric                 │ Formula / Description                 │
├────────────────────────┼───────────────────────────────────────┤
│ Test Coverage          │ (Tests executed / Total planned) ×100 │
│ Pass Rate              │ (Passed / Executed) × 100             │
│ Defect Density         │ Defects / Size (KLOC or module)       │
│ Defect Detection Rate  │ Defects found by QA / Total defects   │
│ Defect Leakage         │ Prod defects / Total defects × 100    │
│ MTTR                   │ Mean Time To Resolve a defect         │
│ Test Automation Rate   │ Automated tests / Total tests × 100   │
│ Requirements Coverage  │ Requirements with tests / Total reqs  │
└────────────────────────┴───────────────────────────────────────┘

Dashboard Example:
  ┌──────────────────────────────────────┐
  │ Release 2.5 Quality Dashboard       │
  │                                      │
  │ Test Coverage:       97%  ✅         │
  │ Pass Rate:           94%  ⚠️         │
  │ Defect Leakage:      2%  ✅         │
  │ Critical Bugs Open:  0   ✅         │
  │ Automation Rate:     65%  ⚠️         │
  │ Avg. MTTR:           1.5 days ✅    │
  └──────────────────────────────────────┘
```

---

## 4. Verification & Validation

```
Verification: "Are we building the product RIGHT?"
  - Reviews, walkthroughs, inspections
  - Static analysis
  - Check against specifications
  - Done WITHOUT executing code

Validation: "Are we building the RIGHT product?"
  - Testing the actual software
  - Dynamic testing (execute code)
  - User acceptance testing
  - Check against user needs

  Specification ──→ [Verification] ──→ Does code match spec?
  User Needs   ──→ [Validation]   ──→ Does product meet needs?
```

### Verification Techniques

```
  - Peer reviews and code reviews
  - Walkthroughs (author explains)
  - Inspections (formal, checklist-based)
  - Static analysis tools (SonarQube, ESLint)
  - Requirements review
```

### Validation Techniques

```
  - Functional testing
  - Integration testing
  - System testing
  - User Acceptance Testing (UAT)
  - Alpha / Beta testing
```

---

## 5. Validation in API Responses

```
API Response Validation Checklist:
  ┌──────────────────────────────────────────────────────┐
  │ 1. Status Code        │ 200, 201, 400, 404, 500     │
  │ 2. Response Body      │ JSON structure, field types  │
  │ 3. Schema Validation  │ Required fields present      │
  │ 4. Data Types         │ string, int, boolean, array  │
  │ 5. Business Rules     │ Correct calculations/logic   │
  │ 6. Error Messages     │ Clear, consistent format     │
  │ 7. Headers            │ Content-Type, Auth tokens    │
  │ 8. Response Time      │ Within SLA (e.g., <500ms)    │
  │ 9. Pagination         │ Correct page/size/total      │
  │ 10. Edge Cases        │ Empty arrays, null values    │
  └──────────────────────────────────────────────────────┘

Example — Validate GET /api/users/123:
  Status:   200 OK
  Headers:  Content-Type: application/json
  Body:
    {
      "id": 123,           ← integer, matches request
      "name": "Nastya",    ← non-empty string
      "email": "n@ex.com", ← valid email format
      "role": "QA",        ← from allowed enum
      "active": true       ← boolean
    }
  Time:     180ms (< 500ms SLA ✅)
```

---

## ✅ Self-Check Questions

1. What is the difference between project risks and product risks?

<details>
<summary>Answer</summary>

**Project risks** affect the project's ability to deliver (schedule, budget, resources) — e.g., staff leaving, environment unavailable. **Product risks** affect the quality of the delivered software — e.g., incorrect logic, security vulnerabilities, poor performance. QA primarily mitigates product risks through testing, while project risks are managed through project management.

</details>

2. How do you calculate risk level and prioritize testing accordingly?

<details>
<summary>Answer</summary>

Risk Level = Likelihood × Impact. A risk matrix categorizes features as Critical, High, Medium, or Low. Critical-risk areas (high likelihood + high impact) get extensive testing with multiple techniques. High-risk areas are tested thoroughly and early. Medium gets standard effort. Low-risk areas may get minimal or deferred testing. This ensures testing effort is focused where failures would cause the most damage.

</details>

3. Name five key quality metrics and explain what each measures.

<details>
<summary>Answer</summary>

(1) **Test Coverage** — percentage of planned tests executed; (2) **Pass Rate** — percentage of executed tests that passed; (3) **Defect Density** — number of defects per unit of code size (KLOC); (4) **Defect Leakage** — percentage of defects found in production vs. total defects; (5) **MTTR** — Mean Time To Resolve a defect. Together they show testing completeness, product stability, code quality, testing effectiveness, and team responsiveness.

</details>

4. Explain the difference between Verification and Validation with examples.

<details>
<summary>Answer</summary>

**Verification** = "Are we building the product right?" — checking that implementation matches specifications without executing code (code reviews, inspections, static analysis). **Validation** = "Are we building the right product?" — executing the software to confirm it meets user needs (functional testing, UAT, beta testing). Example: Verification = reviewing login code matches the spec; Validation = actually logging in and confirming the user experience works.

</details>

5. What aspects should you validate in an API response?

<details>
<summary>Answer</summary>

(1) **Status code** — correct HTTP code for the scenario, (2) **Response body** — correct JSON structure and field values, (3) **Schema** — all required fields present with correct types, (4) **Business rules** — calculations and logic are correct, (5) **Error messages** — clear and consistent format, (6) **Headers** — Content-Type, auth tokens, (7) **Response time** — within SLA, (8) **Edge cases** — empty arrays, null values, missing optional fields, (9) **Pagination** — correct page, size, total count.

</details>