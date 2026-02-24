# User Acceptance Testing (UAT)

## Definition

User Acceptance Testing is the final phase of testing where real users or
stakeholders verify that the system meets their business requirements and is
ready for production use. UAT answers the question: "Does this system do what
we asked for?"

```
Testing Progression:

  [Unit Tests] --> [Integration Tests] --> [System Tests] --> [UAT]
   Developers      Developers/QA          QA Team           End Users
   "Does it work   "Do modules work      "Does the whole   "Does it meet
    correctly?"     together?"            system work?"     our needs?"
```

UAT is not about finding bugs (those should be caught earlier). It is about
validating that the system satisfies business requirements and is acceptable
for release.

---

## UAT in the SDLC

```
Software Development Lifecycle with UAT:

  Requirements --> Design --> Build --> Test --> [UAT] --> Deploy
                                                 ^
                                                 |
                                     Business stakeholders
                                     validate requirements
                                     are met

  UAT is the GATE between "done building" and "released to production."
  The system does not go live until UAT is passed and signed off.
```

---

## Types of UAT

### 1. Alpha Testing

Performed by internal users at the development site before external release.

```
+---------------------------------------------------+
| Alpha Testing                                     |
+---------------------------------------------------+
| Who:    Internal employees, development team      |
| Where:  Development or staging environment        |
| When:   Before beta testing                       |
| Goal:   Catch major usability and function issues |
+---------------------------------------------------+
```

### 2. Beta Testing

Performed by a selected group of external users in their real environment.

```
+---------------------------------------------------+
| Beta Testing                                      |
+---------------------------------------------------+
| Who:    Selected external users / early adopters  |
| Where:  Users' own environments                   |
| When:   After alpha testing, before GA release    |
| Goal:   Real-world validation with diverse setups |
+---------------------------------------------------+
```

### 3. Contract Acceptance Testing

Verifies the system meets the criteria defined in a formal contract between
the customer and the development organization.

```
+---------------------------------------------------+
| Contract Acceptance Testing                       |
+---------------------------------------------------+
| Who:    Customer's designated test team            |
| Where:  Customer-specified environment             |
| When:   Before contract payment milestone          |
| Goal:   Verify contractual obligations are met    |
+---------------------------------------------------+
```

### 4. Regulation (Compliance) Acceptance Testing

Verifies the system meets regulatory, legal, or industry standards.

```
+---------------------------------------------------+
| Regulation Acceptance Testing                     |
+---------------------------------------------------+
| Who:    Compliance officers, auditors              |
| Where:  Controlled, auditable environment          |
| When:   Before deployment to regulated markets     |
| Goal:   Verify compliance with regulations         |
| Examples: Healthcare, finance, government, safety  |
+---------------------------------------------------+
```

---

## UAT Process

### Phase 1: Plan

```
UAT Plan Contents:

  +---+--------------------------------------------+
  | 1 | Scope: Which features are being tested     |
  | 2 | Participants: Who will test                 |
  | 3 | Schedule: Start date, duration, end date    |
  | 4 | Environment: Where testing will occur       |
  | 5 | Test data: What data is needed              |
  | 6 | Entry criteria: What must be true to start  |
  | 7 | Exit criteria: What defines completion      |
  | 8 | Roles and responsibilities                  |
  | 9 | Defect reporting process                    |
  |10 | Sign-off process and authority              |
  +---+--------------------------------------------+
```

### Phase 2: Design Scenarios

UAT scenarios are written from the business perspective, not technical.

```
SCENARIO: Employee submits expense report

  GIVEN an employee logged into the expense system
    AND the employee has receipts for a business trip

  WHEN the employee creates a new expense report
    AND attaches receipt images
    AND enters amounts for meals, transport, and lodging
    AND submits the report for approval

  THEN the report appears in the manager's approval queue
    AND the employee receives a confirmation notification
    AND the report status shows "Pending Approval"
```

### Phase 3: Execute

```
Execution Process:

  1. Testers log in with their own accounts
  2. Follow test scenarios step by step
  3. Record actual results vs expected results
  4. Log defects with severity and screenshots
  5. Retest after defect fixes
  6. Track progress against total scenarios
```

```
UAT Execution Tracker:

  +----------+-------------------+--------+----------+---------+
  | Scenario | Description       | Tester | Status   | Defects |
  +----------+-------------------+--------+----------+---------+
  | UAT-001  | Submit expense    | Sarah  | PASSED   | 0       |
  | UAT-002  | Approve expense   | Mike   | FAILED   | 1       |
  | UAT-003  | Reject expense    | Sarah  | PASSED   | 0       |
  | UAT-004  | Export PDF report | Mike   | BLOCKED  | 1       |
  | UAT-005  | Search expenses   | Lisa   | PASSED   | 0       |
  | UAT-006  | Edit draft report | Lisa   | PENDING  | -       |
  +----------+-------------------+--------+----------+---------+
  | Total: 6 | Passed: 3 | Failed: 1 | Blocked: 1 | Pending: 1 |
  +----------+-------------------+--------+----------+---------+
```

### Phase 4: Report

```
UAT Summary Report:

  +---------------------------------------------------+
  | UAT Summary                                       |
  +---------------------------------------------------+
  | Project:     Expense Management System v2.0       |
  | Period:      Jan 10-17, 2026                      |
  | Testers:     3 business users                     |
  +---------------------------------------------------+
  | Total scenarios:    42                            |
  | Passed:             38 (90%)                      |
  | Failed:              3 (7%)                       |
  | Blocked:             1 (3%)                       |
  +---------------------------------------------------+
  | Critical defects:    0                            |
  | High defects:        1 (workaround available)     |
  | Medium defects:      2                            |
  | Low defects:         4                            |
  +---------------------------------------------------+
  | Recommendation: CONDITIONAL PASS                  |
  | Condition: Fix high defect before go-live         |
  +---------------------------------------------------+
```

---

## Acceptance Criteria Format

### Given-When-Then (Gherkin Syntax)

```
FEATURE: Password Reset

  SCENARIO: Successful password reset
    GIVEN a registered user with email "user@example.test"
    WHEN they request a password reset
    THEN they receive a reset link via email within 5 minutes
    AND the link expires after 24 hours

  SCENARIO: Reset with invalid email
    GIVEN an email "unknown@example.test" is not registered
    WHEN they request a password reset
    THEN the system shows "If this email is registered, you will receive a link"
    AND no email is sent
    AND the response time is the same as for valid emails (prevent enumeration)

  SCENARIO: Reset link already used
    GIVEN a user has already used their reset link
    WHEN they click the same link again
    THEN the system shows "This link has already been used"
    AND offers a new reset request option
```

### Acceptance Criteria Best Practices

```
+----------------------------------------------------------+
| Good Acceptance Criteria                                 |
+----------------------------------------------------------+
| [ ] Written in business language, not technical jargon   |
| [ ] Each criterion is independently verifiable           |
| [ ] Includes both positive and negative scenarios        |
| [ ] Specifies measurable outcomes (not vague qualities)  |
| [ ] Agreed upon by both business and development teams   |
| [ ] Does not prescribe implementation details            |
+----------------------------------------------------------+
```

---

## UAT vs System Testing

```
+---------------------+---------------------------+---------------------+
| Aspect              | System Testing            | UAT                 |
+---------------------+---------------------------+---------------------+
| Performed by        | QA team / testers         | Business users      |
| Perspective         | Technical requirements    | Business needs      |
| Test design         | From specifications       | From user stories   |
| Environment         | QA environment            | UAT / staging env   |
| Goal                | Find defects              | Validate value      |
| Pass criteria       | All test cases pass       | Business sign-off   |
| Defect focus        | Any defect                | Business-impacting  |
| Scripted            | Detailed test scripts     | Scenario-based      |
+---------------------+---------------------------+---------------------+
```

---

## Stakeholder Management

### Identifying UAT Participants

```
+---+---------------------------+--------------------------------------+
| # | Role                      | Responsibility in UAT                |
+---+---------------------------+--------------------------------------+
| 1 | Business Sponsor          | Approves UAT plan, final sign-off   |
+---+---------------------------+--------------------------------------+
| 2 | Product Owner             | Defines acceptance criteria,         |
|   |                           | prioritizes defects                  |
+---+---------------------------+--------------------------------------+
| 3 | Subject Matter Experts    | Design and execute test scenarios    |
+---+---------------------------+--------------------------------------+
| 4 | End Users (testers)       | Execute tests, report issues         |
+---+---------------------------+--------------------------------------+
| 5 | UAT Coordinator           | Manages schedule, tracks progress,   |
|   |                           | facilitates communication            |
+---+---------------------------+--------------------------------------+
| 6 | Development Support       | Available for defect analysis and    |
|   |                           | quick fixes during UAT period       |
+---+---------------------------+--------------------------------------+
```

### Communication Plan

```
+---+------------------------------+---------------------------+
| # | Communication                | Frequency                 |
+---+------------------------------+---------------------------+
| 1 | UAT kickoff meeting          | Once (start of UAT)       |
| 2 | Daily progress update        | Daily (email or standup)  |
| 3 | Defect triage meeting        | As needed (on demand)     |
| 4 | UAT status dashboard         | Continuously updated      |
| 5 | UAT completion report        | Once (end of UAT)         |
| 6 | Go/No-Go decision meeting    | Once (after UAT report)   |
+---+------------------------------+---------------------------+
```

---

## Sign-Off Process

```
Sign-Off Decision Tree:

  Are all critical/high defects resolved?
       |
    YES --> Are all acceptance criteria met?
    NO  --> Fix critical defects. Retest. Return to start.
                |
             YES --> Business sponsor reviews UAT report
             NO  --> Identify gaps. Negotiate scope or fix.
                        |
                   Sponsor approves?
                        |
                     YES --> GO LIVE
                     NO  --> Address concerns. Return to review.
```

```
UAT Sign-Off Document:

  +---------------------------------------------------+
  | UAT SIGN-OFF                                      |
  +---------------------------------------------------+
  | Project:  Expense Management System v2.0          |
  | Date:     2026-01-17                              |
  |                                                   |
  | I confirm that User Acceptance Testing has been   |
  | completed and the system meets the agreed-upon    |
  | acceptance criteria for production release.       |
  |                                                   |
  | Outstanding items:                                |
  |   - DEF-042: Low priority, scheduled for v2.1    |
  |   - DEF-055: Low priority, cosmetic only         |
  |                                                   |
  | Decision: APPROVED FOR RELEASE                    |
  |                                                   |
  | Signed: _________________________                |
  |         Business Sponsor                          |
  +---------------------------------------------------+
```

---

## Entry and Exit Criteria

```
+------------------------------+--------------------------------------+
| Entry Criteria               | Exit Criteria                        |
| (must be true to START UAT)  | (must be true to END UAT)            |
+------------------------------+--------------------------------------+
| System testing completed     | All UAT scenarios executed            |
| All critical defects fixed   | No open critical/high defects        |
| UAT environment available    | Acceptance criteria verified          |
| Test data prepared           | UAT report completed                 |
| UAT plan approved            | Sign-off obtained                    |
| Users trained on system      | Outstanding items documented         |
+------------------------------+--------------------------------------+
```

---

## Summary

UAT is the bridge between building software and releasing it. It ensures that
what was built matches what was requested, validated by the people who will
actually use the system. Without proper UAT, teams risk deploying software that
is technically correct but fails to meet real business needs.
