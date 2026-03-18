# Course 6: Test Design Techniques

## 📖 What You Will Learn

- Core test design techniques (BVA, EP, Decision Tables, State Transitions)
- Exploratory and session-based testing
- Test pyramid understanding
- Choosing the right technique for the right situation

---

## 1. Equivalence Partitioning (EP)

```
Divide input data into groups (partitions) where all values
in a partition are expected to behave the same way.

  Rule: Age must be 18–65

  Partitions:
    ┌────────────┬────────────┬────────────┐
    │  Invalid   │   Valid    │  Invalid   │
    │  < 18      │  18 – 65   │  > 65      │
    └────────────┴────────────┴────────────┘

  Test cases (one value per partition):
    - age = 10   → Invalid (below minimum)
    - age = 30   → Valid
    - age = 80   → Invalid (above maximum)

  Why it works:
    If age=30 works, age=25 likely works too (same partition)
    Reduces test cases from hundreds to a few representatives

  Apply to:
    ✓ Numeric ranges
    ✓ String lengths
    ✓ Dropdown selections
    ✓ File sizes
    ✓ Date ranges
```

---

## 2. Boundary Value Analysis (BVA)

```
Test at the EDGES of equivalence partitions,
where most bugs occur.

  Rule: Age must be 18–65

  Boundaries and test values:
    ┌──────┬──────┬──────┬──────┬──────┬──────┐
    │  17  │  18  │  19  │  64  │  65  │  66  │
    │ INV  │ VAL  │ VAL  │ VAL  │ VAL  │ INV  │
    └──────┴──────┴──────┴──────┴──────┴──────┘
      ↑      ↑      ↑      ↑      ↑      ↑
    below  lower  above  below  upper  above
    min    bound  min    max    bound  max

  Two-value BVA (minimum):
    17, 18, 65, 66

  Three-value BVA (thorough):
    17, 18, 19, 64, 65, 66

  Common boundary bugs:
    - Off-by-one errors (< vs <=)
    - age >= 18 vs age > 18
    - Empty string vs one character
    - Array index 0 vs -1
    - Max integer, min integer
```

---

## 3. Decision Tables

```
For features with MULTIPLE CONDITIONS that interact.

  Example: Discount rules
    - Premium member: yes/no
    - Order > $100: yes/no
    - Has coupon: yes/no

  Decision Table:
  ┌─────────────────┬────┬────┬────┬────┬────┬────┬────┬────┐
  │ Conditions       │ R1 │ R2 │ R3 │ R4 │ R5 │ R6 │ R7 │ R8 │
  ├─────────────────┼────┼────┼────┼────┼────┼────┼────┼────┤
  │ Premium member   │ Y  │ Y  │ Y  │ Y  │ N  │ N  │ N  │ N  │
  │ Order > $100     │ Y  │ Y  │ N  │ N  │ Y  │ Y  │ N  │ N  │
  │ Has coupon       │ Y  │ N  │ Y  │ N  │ Y  │ N  │ Y  │ N  │
  ├─────────────────┼────┼────┼────┼────┼────┼────┼────┼────┤
  │ Actions:         │    │    │    │    │    │    │    │    │
  │ 20% discount     │ X  │    │    │    │    │    │    │    │
  │ 15% discount     │    │ X  │ X  │    │    │    │    │    │
  │ 10% discount     │    │    │    │ X  │ X  │    │    │    │
  │ 5% discount      │    │    │    │    │    │ X  │ X  │    │
  │ No discount      │    │    │    │    │    │    │    │ X  │
  └─────────────────┴────┴────┴────┴────┴────┴────┴────┴────┘

  Rules: 2^N combinations (N = number of conditions)
  3 conditions → 2³ = 8 rules → 8 test cases

  Best for: Business rules, permissions, pricing, eligibility
```

---

## 4. State Transition Testing

```
For systems that behave differently based on current STATE.

  Example: User Account States

  ┌──────────┐   Register   ┌──────────┐   Verify    ┌──────────┐
  │   New    │─────────────→│ Pending  │────────────→│  Active  │
  └──────────┘              └──────────┘             └────┬─────┘
                                 │                        │
                            Timeout (48h)            3 wrong passwords
                                 │                        │
                                 ▼                        ▼
                            ┌──────────┐            ┌──────────┐
                            │ Expired  │            │  Locked  │
                            └──────────┘            └────┬─────┘
                                                         │
                                                    Admin unlock
                                                         │
                                                         ▼
                                                    ┌──────────┐
                                                    │  Active  │
                                                    └──────────┘

  State Transition Table:
  ┌───────────┬───────────────────┬───────────┐
  │ Current   │ Event             │ Next State│
  ├───────────┼───────────────────┼───────────┤
  │ New       │ Register          │ Pending   │
  │ Pending   │ Verify email      │ Active    │
  │ Pending   │ Timeout (48h)     │ Expired   │
  │ Active    │ 3 wrong passwords │ Locked    │
  │ Locked    │ Admin unlock      │ Active    │
  │ Active    │ Deactivate        │ Inactive  │
  └───────────┴───────────────────┴───────────┘

  Test cases:
    1. Happy path: New → Pending → Active
    2. Timeout: New → Pending → Expired
    3. Lock/unlock: Active → Locked → Active
    4. Invalid transition: New → Active (should fail!)
```

---

## 5. Exploratory Testing

```
Simultaneous learning, test design, and execution.
The tester uses their experience and intuition.

  Session-Based Test Management (SBTM):
  ┌──────────────────────────────────────────────┐
  │ Session Charter:                             │
  │   "Explore the checkout flow with focus on   │
  │    payment edge cases for 60 minutes"        │
  │                                              │
  │ Time Box: 60 minutes                         │
  │ Tester: Nastya                               │
  │ Area: Checkout / Payment                     │
  │                                              │
  │ Notes:                                       │
  │ - Tried empty cart → proper error ✓          │
  │ - Expired card → unclear error message ✗ BUG │
  │ - Switching payment methods mid-flow → crash │
  │ - International address → ZIP validation off │
  │                                              │
  │ Bugs found: 2                                │
  │ Questions: 1 (ZIP format requirements?)      │
  │ Coverage: Payment methods, error handling     │
  └──────────────────────────────────────────────┘

  Exploratory Testing Heuristics:
    SFDPOT:
      S - Structure    (what is it made of?)
      F - Function     (what does it do?)
      D - Data         (what data does it process?)
      P - Platform     (what does it run on?)
      O - Operations   (how is it used in practice?)
      T - Time         (what happens over time?)
```

---

## 6. Test Pyramid

```
                    /\
                   /  \
                  / E2E \          Few, slow, expensive
                 / (UI)  \        - Full user flows
                /──────────\      - Browser automation
               /            \
              / Integration  \    Medium number, medium speed
             /   (API/Service)\   - API contract tests
            /──────────────────\  - Component integration
           /                    \
          /      Unit Tests      \  Many, fast, cheap
         /     (Component/Logic)  \ - Individual functions
        /──────────────────────────\ - Isolated, no dependencies
       ──────────────────────────────

  Key Principles:
    1. MORE tests at the bottom (unit) = fast, cheap, stable
    2. FEWER tests at the top (E2E) = slow, expensive, flaky
    3. Each layer tests different things — no duplication
    4. If you can test at a lower level, do it there

  Anti-pattern — Ice Cream Cone:
       ────────────────────────────
      \   Manual Testing (lots!)  /
       \                         /
        \   E2E Tests (many)    /
         \                     /
          \  Integration (few)/
           \                 /
            \ Unit (none!)  /
             ──────────────
    ✗ Slow feedback, expensive, fragile
```

---

## 7. Choosing the Right Technique

```
┌──────────────────────┬───────────────────────────────────┐
│ Situation            │ Best Technique                    │
├──────────────────────┼───────────────────────────────────┤
│ Numeric input field  │ BVA + Equivalence Partitioning    │
│ Complex business     │ Decision Tables                   │
│   rules              │                                   │
│ Workflow with states │ State Transition Testing          │
│ New/unknown feature  │ Exploratory Testing               │
│ Login/form fields    │ EP + BVA + Negative testing       │
│ API with many params │ Pairwise + EP                     │
│ Regression suite     │ Risk-based selection              │
│ User journeys        │ E2E scenario testing              │
│ Data processing      │ EP for data + SQL validation      │
└──────────────────────┴───────────────────────────────────┘
```

---

## ✅ Self-Check Questions

1. Explain Equivalence Partitioning with an example.

<details>
<summary>Answer</summary>

**Equivalence Partitioning** divides input data into groups where all values are expected to behave the same way. Example: Password length must be 8–20 characters. Partitions: (1) Invalid: 0–7 chars, (2) Valid: 8–20 chars, (3) Invalid: 21+ chars. Test one value per partition: 5 chars (invalid), 12 chars (valid), 25 chars (invalid). This reduces test cases — if 12 works, 15 likely works too.

</details>

2. What is Boundary Value Analysis and why is it important?

<details>
<summary>Answer</summary>

**BVA** tests values at the edges of equivalence partitions, where most bugs occur (off-by-one errors, `<` vs `<=`). For a range 18–65: test 17 (just below), 18 (lower boundary), 19 (just above lower), 64 (just below upper), 65 (upper boundary), 66 (just above). BVA is important because developers often make mistakes at boundaries — using `>` instead of `>=`, or `< 65` instead of `<= 65`.

</details>

3. Create a decision table for a login feature with conditions: valid username, valid password, account active.

<details>
<summary>Answer</summary>

| Rule | Valid Username | Valid Password | Account Active | Result |
|------|:---:|:---:|:---:|---|
| R1 | Y | Y | Y | Login success |
| R2 | Y | Y | N | Account inactive error |
| R3 | Y | N | Y | Invalid credentials |
| R4 | Y | N | N | Invalid credentials |
| R5 | N | Y | Y | Invalid credentials |
| R6 | N | Y | N | Invalid credentials |
| R7 | N | N | Y | Invalid credentials |
| R8 | N | N | N | Invalid credentials |

3 conditions → 2³ = 8 rules. Only R1 succeeds. R2 shows account status check after credential validation.

</details>

4. Describe the Test Pyramid and why it matters.

<details>
<summary>Answer</summary>

The **Test Pyramid** has three layers: (1) **Unit tests** (base) — many, fast, cheap, test individual functions in isolation; (2) **Integration/API tests** (middle) — medium count, test component interactions and API contracts; (3) **E2E/UI tests** (top) — few, slow, expensive, test full user flows. It matters because: more tests at the bottom = faster feedback, lower cost, more stability. The anti-pattern (Ice Cream Cone) has many E2E/manual tests and few unit tests = slow, expensive, fragile.

</details>

5. What is exploratory testing and when should you use it?

<details>
<summary>Answer</summary>

**Exploratory testing** is simultaneous learning, test design, and execution — the tester uses experience and intuition rather than predefined scripts. Use it when: (1) a new feature with unclear requirements arrives, (2) you need to quickly assess quality, (3) formal test cases might miss edge cases, (4) after scripted testing to find additional bugs. Session-Based Test Management (SBTM) structures it with charters, time boxes, and notes. Heuristics like SFDPOT guide exploration.

</details>