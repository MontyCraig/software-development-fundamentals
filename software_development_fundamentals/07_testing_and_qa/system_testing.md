# System Testing

## Overview

System testing validates the complete, integrated system against its specified
requirements. It treats the entire application as a black box, testing it from
the end-user's perspective through its external interfaces.

```
Scope Comparison:

  Unit Test:        [Function]
  Integration Test: [Module A] <-> [Module B]
  System Test:      [  Entire Application as a Whole  ]
                    (database, APIs, UI, services, etc.)
```

---

## End-to-End Testing (E2E)

End-to-end tests simulate real user workflows through the complete system,
from input to final output.

```
E2E Test Flow:

  [User Action]     [System Processing]          [Verification]
       |                   |                          |
  Fill login form --> Authenticate user -------> Dashboard loads
       |                   |                          |
  Click "New Order" -> Create order record ---> Order appears in list
       |                   |                          |
  Submit payment --> Process via gateway ------> Confirmation shown
```

### E2E Test Example

```
FUNCTION test_complete_purchase_workflow():
    // Step 1: User browses catalog
    CALL browser.navigateTo("/catalog")
    ASSERT page.title = "Product Catalog"

    // Step 2: User adds item to cart
    CALL browser.click("Add to Cart", product: "Widget A")
    ASSERT cart.itemCount = 1

    // Step 3: User proceeds to checkout
    CALL browser.click("Checkout")
    CALL browser.fillForm({
        address: "123 Test Street",
        payment: "test-card-4242"
    })

    // Step 4: User submits order
    CALL browser.click("Place Order")

    // Step 5: Verify confirmation
    ASSERT page.contains("Order Confirmed")
    ASSERT page.contains("Order #")

    // Step 6: Verify email was sent
    SET emails = CALL emailInbox.getRecent()
    ASSERT emails[0].subject CONTAINS "Order Confirmation"
END FUNCTION
```

### E2E Test Considerations

```
+-------------------------------+--------------------------------------+
| Challenge                     | Mitigation                           |
+-------------------------------+--------------------------------------+
| Slow execution                | Run only critical paths in E2E;     |
|                               | rely on lower-level tests for rest  |
+-------------------------------+--------------------------------------+
| Flaky results                 | Use explicit waits, not timers;     |
|                               | retry on transient failures         |
+-------------------------------+--------------------------------------+
| Difficult debugging           | Capture screenshots, logs, and      |
|                               | network traces on failure           |
+-------------------------------+--------------------------------------+
| Environment dependency        | Use dedicated, stable test env      |
+-------------------------------+--------------------------------------+
| High maintenance cost         | Limit E2E count; use page objects   |
|                               | or similar abstraction patterns     |
+-------------------------------+--------------------------------------+
```

---

## Performance Testing

### Load Testing

Verify system behavior under expected concurrent user loads.

```
Load Test Profile:

  Users
  200 |            ___________
      |           /           \
  150 |          /             \
      |         /               \
  100 |        /                 \
      |       /                   \
   50 |      /                     \
      |     /                       \
    0 +----+---+---+---+---+---+----+-->
       0   5  10  15  20  25  30  35
                Minutes

  Ramp up: 0-10 min (gradual increase to 200 users)
  Steady:  10-25 min (sustained 200 concurrent users)
  Ramp down: 25-35 min (gradual decrease)

  Pass Criteria:
    - Response time p95 < 2 seconds
    - Error rate < 1%
    - Throughput > 100 requests/second
```

### Stress Testing

Push the system beyond normal capacity to find its breaking point.

```
Stress Test Profile:

  Users
  500 |                          *  (system degrades)
      |                       *
  400 |                    *
      |                 *
  300 |              *
      |           *
  200 |        *        <-- Normal capacity
      |     *
  100 |  *
      +--+--+--+--+--+--+--+--+-->
       0  5 10 15 20 25 30 35 40
                Minutes

  Goals:
    - Find maximum capacity
    - Verify graceful degradation (not crashes)
    - Confirm recovery after load reduces
```

### Spike Testing

Test sudden, dramatic increases in load.

```
Spike Test Profile:

  Users
  500 |      *****
      |     *     *
  400 |    *       *
      |   *         *
   50 |***           *****
      +--+--+--+--+--+--+-->
       0  5 10 15 20 25 30
                Minutes

  Goals:
    - System survives sudden traffic spikes
    - Auto-scaling triggers correctly (if applicable)
    - No data loss during spike
```

### Soak Testing (Endurance)

Run the system under normal load for an extended period.

```
Soak Test Profile:

  Users
  200 |  ________________________________
      | /                                \
  100 |/                                  \
      +--+---+---+---+---+---+---+---+----+-->
       0  2   4   6   8  10  12  14  16  18
                    Hours

  Goals:
    - No memory leaks (memory usage stays stable)
    - No resource exhaustion (file handles, connections)
    - Performance does not degrade over time
    - Logs do not fill up disk space
```

---

## Security Testing Basics

```
+---+---------------------------+--------------------------------------+
| # | Test Category             | What to Verify                       |
+---+---------------------------+--------------------------------------+
| 1 | Authentication            | Login works, lockout after failures, |
|   |                           | session expiration, token handling   |
+---+---------------------------+--------------------------------------+
| 2 | Authorization             | Users can only access permitted      |
|   |                           | resources; privilege escalation      |
|   |                           | is prevented                        |
+---+---------------------------+--------------------------------------+
| 3 | Input Validation          | Injection attacks (command, query),  |
|   |                           | cross-site scripting (XSS),         |
|   |                           | buffer overflows                     |
+---+---------------------------+--------------------------------------+
| 4 | Data Protection           | Encryption at rest and in transit,   |
|   |                           | sensitive data not exposed in logs  |
+---+---------------------------+--------------------------------------+
| 5 | Error Handling            | Error messages do not reveal system |
|   |                           | internals or stack traces           |
+---+---------------------------+--------------------------------------+
| 6 | Session Management        | Sessions expire, tokens rotate,     |
|   |                           | concurrent session limits           |
+---+---------------------------+--------------------------------------+
```

---

## Accessibility Testing

Verify that the system is usable by people with diverse abilities.

```
+---+---------------------------+--------------------------------------+
| # | Test Area                 | What to Verify                       |
+---+---------------------------+--------------------------------------+
| 1 | Screen Reader Compat.     | All content is accessible via        |
|   |                           | assistive technology                 |
+---+---------------------------+--------------------------------------+
| 2 | Keyboard Navigation       | All functions reachable by keyboard  |
+---+---------------------------+--------------------------------------+
| 3 | Color Contrast            | Text meets minimum contrast ratios  |
+---+---------------------------+--------------------------------------+
| 4 | Text Alternatives         | Images have descriptive alt text     |
+---+---------------------------+--------------------------------------+
| 5 | Form Labels               | All form inputs have associated      |
|   |                           | labels                              |
+---+---------------------------+--------------------------------------+
| 6 | Focus Management          | Focus order is logical; focus is     |
|   |                           | visible                             |
+---+---------------------------+--------------------------------------+
```

---

## Compatibility Testing

```
+---+---------------------------+--------------------------------------+
| # | Dimension                 | What to Verify                       |
+---+---------------------------+--------------------------------------+
| 1 | Browser Compatibility     | Works across major browsers          |
+---+---------------------------+--------------------------------------+
| 2 | OS Compatibility          | Works across operating systems       |
+---+---------------------------+--------------------------------------+
| 3 | Device Compatibility      | Works on desktop, tablet, mobile     |
+---+---------------------------+--------------------------------------+
| 4 | Resolution Compatibility  | Renders correctly at various screen  |
|   |                           | resolutions                         |
+---+---------------------------+--------------------------------------+
| 5 | Backward Compatibility    | New version works with old data      |
|   |                           | formats and configurations          |
+---+---------------------------+--------------------------------------+
```

---

## Smoke Testing

Smoke tests verify that the most critical functions of the system work after
a build or deployment. They are shallow, fast, and answer one question:
"Is this build stable enough to warrant further testing?"

```
Smoke Test Example (E-Commerce):

  1. Application starts without errors          [PASS/FAIL]
  2. Login page loads                           [PASS/FAIL]
  3. User can log in with valid credentials     [PASS/FAIL]
  4. Product catalog displays products          [PASS/FAIL]
  5. User can add item to cart                  [PASS/FAIL]
  6. Checkout page loads                        [PASS/FAIL]

  If ANY smoke test fails: STOP. Do not proceed with
  further testing. Fix the build first.
```

---

## Regression Testing

Regression testing verifies that previously working functionality still works
after changes (bug fixes, new features, refactoring).

```
Regression Strategy:

  [Change Made] --> [Run Affected Tests] --> [Run Full Suite]
                                                    |
                                           All pass? --> Ship
                                           Any fail? --> Fix regression
```

```
+----------------------------------------------+
| Regression Test Selection Strategies         |
+----------------------------------------------+
| Retest all: Run entire test suite            |
|   + Thorough; - Slow                         |
| Selective: Run tests for changed modules     |
|   + Fast; - May miss cross-module effects    |
| Risk-based: Prioritize tests for high-risk   |
|   areas and frequently failing modules       |
|   + Balanced; - Requires risk assessment     |
+----------------------------------------------+
```

---

## Test Environment Management

```
Environment Pipeline:

  [Source Code] --> [Build] --> [Dev Env] --> [Test Env] --> [Staging] --> [Prod]
                                  |              |             |
                              Unit tests   Integration   System tests
                                           tests         E2E tests
                                                         Performance
                                                         Security
```

### Environment Parity

```
+----------------------------------------------------------+
| Environment Parity Checklist                             |
+----------------------------------------------------------+
| [ ] Same operating system version as production          |
| [ ] Same database version and configuration              |
| [ ] Same network topology (or close approximation)       |
| [ ] Same third-party service versions                    |
| [ ] Realistic data volume (anonymized production data)   |
| [ ] Same security configuration                         |
| [ ] Same monitoring and logging setup                    |
+----------------------------------------------------------+
```

The closer the test environment is to production, the more reliable the
system test results will be.

---

## Summary

```
+---------------------------+-------------------------------------------+
| Test Type                 | Key Question                              |
+---------------------------+-------------------------------------------+
| E2E                       | Do complete user workflows work?          |
| Load                      | Can we handle expected traffic?           |
| Stress                    | What happens beyond capacity?             |
| Spike                     | Can we survive sudden traffic bursts?     |
| Soak                      | Are there memory leaks or slow degrades?  |
| Security                  | Are we protected against common attacks?  |
| Accessibility             | Can all users use the system?             |
| Compatibility             | Does it work on all target platforms?     |
| Smoke                     | Is this build worth testing further?      |
| Regression                | Did changes break existing functionality? |
+---------------------------+-------------------------------------------+
```

System testing is the last line of defense before the software reaches users.
It validates that the system works as a whole, not just as a collection of
working parts.
