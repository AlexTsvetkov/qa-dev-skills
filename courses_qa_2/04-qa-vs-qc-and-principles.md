# Course 4: QA vs QC and Principles

## 📖 What You Will Learn

- Difference between Quality Assurance and Quality Control
- Seven principles of software testing (ISTQB)

---

## 1. Quality Assurance vs. Quality Control

```
┌───────────────────────────┬──────────────────────────────────┐
│   Quality Assurance (QA)  │   Quality Control (QC)           │
├───────────────────────────┼──────────────────────────────────┤
│ Process-oriented          │ Product-oriented                 │
│ Preventive                │ Detective (reactive)             │
│ Focuses on PROCESS        │ Focuses on PRODUCT               │
│ "Are we doing it right?"  │ "Did we build it right?"         │
│ Proactive                 │ Reactive                         │
│ Applied throughout SDLC   │ Applied during/after testing     │
│ Everyone's responsibility │ Testing team's responsibility    │
│ Aims to prevent defects   │ Aims to find defects             │
└───────────────────────────┴──────────────────────────────────┘
```

### QA Activities (Prevention)

```
- Defining test processes and standards
- Creating test strategies and plans
- Establishing coding standards
- Conducting reviews and walkthroughs
- Defining quality metrics and KPIs
- Process audits and improvements
- Training and mentoring
- Selecting tools and frameworks
- Defining Definition of Done
- Risk analysis and mitigation planning
```

### QC Activities (Detection)

```
- Executing test cases
- Bug reporting and tracking
- Functional testing
- Regression testing
- Performance testing
- Exploratory testing
- User acceptance testing
- API testing and validation
- Code inspections
- Analyzing test results
```

### How They Work Together

```
                 QA (Process)
                 ┌──────────┐
                 │ Define    │
                 │ standards │
                 │ & process │
                 └─────┬────┘
                       │
    ┌──────────────────▼──────────────────┐
    │         Development Process          │
    │  Requirements → Design → Code        │
    └──────────────────┬──────────────────┘
                       │
                 ┌─────▼────┐
                 │ QC       │
                 │ Execute  │
                 │ tests &  │
                 │ find bugs│
                 └─────┬────┘
                       │
                 ┌─────▼────┐
                 │ Feedback │ → Improve QA processes
                 └──────────┘

  Example:
    QA: "All API endpoints must have automated contract tests"
    QC: Execute those contract tests, report failures
    QA: Analyze failure patterns, improve the process
```

---

## 2. Seven Principles of Software Testing (ISTQB)

### Principle 1: Testing Shows the Presence of Defects, Not Their Absence

```
Testing can show that defects are present,
but CANNOT prove there are no defects.

  ✓ "We found 0 bugs" ≠ "There are 0 bugs"
  ✓ Testing reduces the probability of undiscovered defects
  ✓ Even exhaustive testing cannot guarantee bug-free software

  Example:
    You test login with 50 combinations → all pass
    But the 51st untested combination might fail
    Testing reduces risk, it doesn't eliminate it
```

### Principle 2: Exhaustive Testing Is Impossible

```
Testing everything (all inputs, combinations, paths) is not feasible.

  Example: A form with 5 fields, each with 10 possible values
  Total combinations = 10^5 = 100,000 test cases!

  Solution: Use risk analysis and prioritization
    - Risk-based testing (focus on high-risk areas)
    - Equivalence partitioning (group similar inputs)
    - Boundary value analysis (test edges)
    - Pairwise testing (cover parameter combinations)
```

### Principle 3: Early Testing Saves Time and Money

```
Cost of fixing defects by phase:

  Requirements:  $1        ←── Cheapest
  Design:        $5
  Coding:        $10
  Testing:       $50
  Production:    $500+     ←── Most expensive

  Shift-Left Approach:
    ✓ Review requirements early
    ✓ Participate in design discussions
    ✓ Write test cases during requirement analysis
    ✓ Automate early (unit tests, API tests)
    ✓ 3 Amigos sessions before development starts
```

### Principle 4: Defects Cluster Together

```
A small number of modules contain most defects.

  Pareto Principle (80/20 rule):
    80% of defects are found in 20% of modules

  ┌───────────────────────────────────────────┐
  │ Module A: ████████████████████  (60%)     │
  │ Module B: ██████████  (25%)               │
  │ Module C: ███  (10%)                      │
  │ Module D: █  (5%)                         │
  └───────────────────────────────────────────┘

  Implication:
    - Track defect density per module
    - Focus testing on historically buggy areas
    - Newly developed or recently changed code is riskier
```

### Principle 5: Beware of the Pesticide Paradox

```
Repeating the same tests will eventually stop finding new bugs.

  Like pesticide — bugs develop resistance:
    Sprint 1: Run 100 tests → find 15 bugs
    Sprint 2: Run same 100 tests → find 3 bugs
    Sprint 3: Run same 100 tests → find 0 bugs  ← NOT bug-free!

  Solution:
    ✓ Regularly review and update test cases
    ✓ Add new tests for new features and changes
    ✓ Use exploratory testing for fresh perspectives
    ✓ Rotate testers across modules
    ✓ Apply different test techniques
```

### Principle 6: Testing Is Context-Dependent

```
Testing approach depends on the context:

  Medical software:    Safety-critical, extensive V&V, regulatory
  E-commerce:          Performance, security, payment testing
  Mobile game:         Usability, performance, compatibility
  Banking API:         Security, data integrity, compliance
  Internal tool:       Functional, basic usability

  There is no one-size-fits-all testing approach.
  The strategy must match the project's risk profile.
```

### Principle 7: Absence-of-Errors Is a Fallacy

```
Finding and fixing all bugs does not guarantee a successful product.

  A system with 0 bugs but:
    ✗ Doesn't meet user needs → FAILURE
    ✗ Has terrible UX → FAILURE
    ✗ Solves the wrong problem → FAILURE
    ✗ Is too slow for users → FAILURE

  Testing must verify:
    ✓ Requirements are correct (validation)
    ✓ Product solves the real problem
    ✓ Users can actually use it
    ✓ Business objectives are met
```

---

## ✅ Self-Check Questions

1. What is the fundamental difference between QA and QC?

<details>
<summary>Answer</summary>

**QA (Quality Assurance)** is process-oriented and preventive — it focuses on establishing processes, standards, and practices to prevent defects from occurring. **QC (Quality Control)** is product-oriented and detective — it focuses on executing tests and finding defects in the actual product. QA asks "Are we doing it right?" while QC asks "Did we build it right?" QA is proactive; QC is reactive.

</details>

2. Explain the Pesticide Paradox and how to counter it.

<details>
<summary>Answer</summary>

The **Pesticide Paradox** states that repeating the same tests will eventually stop finding new bugs — similar to how pests become resistant to pesticide. To counter it: (1) regularly review and update test cases, (2) add new tests for new features and code changes, (3) use exploratory testing for fresh perspectives, (4) rotate testers across modules, (5) apply different test design techniques to the same features.

</details>

3. Why is exhaustive testing impossible? What do we do instead?

<details>
<summary>Answer</summary>

Exhaustive testing (testing all possible inputs, combinations, and paths) is impossible due to the astronomical number of combinations. Instead, we use: (1) **risk-based testing** — focus on high-risk areas, (2) **equivalence partitioning** — group similar inputs, (3) **boundary value analysis** — test edges, (4) **pairwise testing** — cover parameter combinations efficiently, (5) **prioritization** — test the most important scenarios first.

</details>

4. What does "Absence-of-Errors Fallacy" mean in practice?

<details>
<summary>Answer</summary>

It means that finding and fixing all bugs does not guarantee a successful product. A bug-free system can still fail if it doesn't meet user needs, has terrible UX, solves the wrong problem, or doesn't achieve business objectives. Testing must include **validation** (building the right product) in addition to **verification** (building the product right). User acceptance testing and business alignment are just as important as functional correctness.

</details>

5. Give three examples of QA activities and three examples of QC activities.

<details>
<summary>Answer</summary>

**QA activities (preventive):** (1) Defining test strategy and processes, (2) Conducting requirement reviews and walkthroughs, (3) Establishing coding standards and Definition of Done. **QC activities (detective):** (1) Executing test cases and logging bugs, (2) Performing regression testing, (3) Conducting exploratory testing sessions. QA prevents defects by improving processes; QC detects defects by testing the product.

</details>