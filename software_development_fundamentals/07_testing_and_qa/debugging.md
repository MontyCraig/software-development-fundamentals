# Debugging

## Overview

Debugging is the systematic process of finding and resolving defects in
software. Effective debugging is a skill that combines logical reasoning,
tool knowledge, and pattern recognition.

```
The Debugging Cycle:

  [Observe Symptom] --> [Form Hypothesis] --> [Test Hypothesis]
         ^                                          |
         |                                          v
         +------ [Hypothesis Wrong] <------ [Analyze Results]
                                                    |
                                          [Hypothesis Correct]
                                                    |
                                                    v
                                              [Fix the Bug]
                                                    |
                                                    v
                                            [Verify the Fix]
```

---

## The Scientific Debugging Method

Apply the scientific method to debugging: observe, hypothesize, experiment,
analyze, repeat.

```
PROCEDURE scientificDebug(symptom: BugReport):

    // Step 1: Reproduce the bug consistently
    SET reproduction = CALL reproduceReliably(symptom)
    IF reproduction IS NOT RELIABLE THEN
        CALL gatherMoreInformation(symptom)
        RETURN TO Step 1
    END IF

    // Step 2: Observe and gather data
    SET observations = CALL collectEvidence(reproduction)
    // Logs, error messages, stack traces, system state

    // Step 3: Form a hypothesis
    SET hypothesis = CALL formHypothesis(observations)
    // "The bug occurs because X happens when condition Y is true"

    // Step 4: Design an experiment to test the hypothesis
    SET experiment = CALL designExperiment(hypothesis)
    // Change one variable to confirm or deny

    // Step 5: Run the experiment
    SET result = CALL runExperiment(experiment)

    // Step 6: Analyze
    IF result CONFIRMS hypothesis THEN
        CALL implementFix(hypothesis.rootCause)
        CALL verifyFix(reproduction)
    ELSE
        CALL refineHypothesis(observations, result)
        RETURN TO Step 3
    END IF

END PROCEDURE
```

### Key Principles

```
+---+-----------------------------------------------------------+
| 1 | Reproduce first. If you cannot reproduce it, you cannot   |
|   | verify you fixed it.                                      |
+---+-----------------------------------------------------------+
| 2 | Change one thing at a time. Multiple simultaneous changes |
|   | make it impossible to identify which one mattered.        |
+---+-----------------------------------------------------------+
| 3 | Question your assumptions. The bug exists precisely        |
|   | because something is not behaving as you assumed.         |
+---+-----------------------------------------------------------+
| 4 | Read the error message. Carefully. Completely.            |
+---+-----------------------------------------------------------+
| 5 | Check the most recent change. Most bugs are introduced    |
|   | by the most recent modification.                         |
+---+-----------------------------------------------------------+
```

---

## Binary Search Debugging

When the bug exists somewhere in a large codebase or a long sequence of
operations, use binary search to narrow it down.

```
Binary Search Debugging Process:

  Bug appears after 100 operations (or in 1000 lines of code).

  Step 1: Check the midpoint (operation 50)
          Is the bug present? YES --> Bug is in operations 1-50
                              NO  --> Bug is in operations 51-100

  Step 2: Check the new midpoint (operation 25 or 75)
          Continue halving until the exact operation is found.

  Iterations needed: log2(N)
    100 operations: ~7 checks
    1000 lines: ~10 checks
```

### Applied to Version History

```
  Working version: commit A (known good)
  Broken version:  commit Z (known bad)

  Commits: A -- B -- C -- D -- E -- ... -- X -- Y -- Z

  Step 1: Test commit M (midpoint)
  Step 2: If M is good, test midpoint between M and Z
          If M is bad, test midpoint between A and M
  Step 3: Repeat until the exact breaking commit is found

  This is automated in many version control systems as "bisect."
```

---

## Rubber Duck Debugging

Explain the code line by line to an inanimate object (or a colleague who
just listens). The act of articulation forces you to examine assumptions.

```
Process:

  1. Get a rubber duck (or equivalent)
  2. Explain what the code is SUPPOSED to do
  3. Walk through the code line by line
  4. Explain what EACH line actually does
  5. When your explanation diverges from the code's
     actual behavior, you have found the bug

  Why it works:
  - Forces you to slow down and read every line
  - Converts passive reading to active reasoning
  - Exposes assumptions you did not know you had
  - The bug is often found before you finish explaining
```

---

## Common Bug Categories

```
+---+---------------------------+--------------------------------------+
| # | Category                  | Examples                             |
+---+---------------------------+--------------------------------------+
| 1 | Off-by-one errors         | Loop runs one too many or too few    |
|   |                           | times; index out of bounds           |
+---+---------------------------+--------------------------------------+
| 2 | Null/undefined references | Accessing a field on a null object   |
+---+---------------------------+--------------------------------------+
| 3 | Type errors               | Wrong data type passed to function;  |
|   |                           | implicit type conversion surprises  |
+---+---------------------------+--------------------------------------+
| 4 | Logic errors              | Wrong operator (AND vs OR), wrong    |
|   |                           | comparison (< vs <=), inverted       |
|   |                           | condition                            |
+---+---------------------------+--------------------------------------+
| 5 | State management bugs     | Shared mutable state, stale cache,   |
|   |                           | uninitialized variables              |
+---+---------------------------+--------------------------------------+
| 6 | Concurrency bugs          | Race conditions, deadlocks, data     |
|   |                           | corruption from parallel access     |
+---+---------------------------+--------------------------------------+
| 7 | Resource leaks            | Unclosed connections, file handles,  |
|   |                           | memory not freed                     |
+---+---------------------------+--------------------------------------+
| 8 | Integration errors        | API contract changed, incompatible   |
|   |                           | data formats, encoding mismatches   |
+---+---------------------------+--------------------------------------+
| 9 | Environment bugs          | Works on dev, fails in production    |
|   |                           | due to configuration differences    |
+---+---------------------------+--------------------------------------+
|10 | Boundary/edge cases       | Empty input, maximum values, special |
|   |                           | characters, Unicode issues          |
+---+---------------------------+--------------------------------------+
```

---

## Debugging Strategies

### Strategy 1: Print/Log Debugging

Add output statements to trace execution flow and inspect variable values.

```
FUNCTION processOrder(order: Order) -> Result:
    LOG "processOrder called with order.id=" + order.id
    LOG "  items count=" + LENGTH(order.items)

    SET total = CALL calculateTotal(order)
    LOG "  calculated total=" + total

    IF total <= 0 THEN
        LOG "  ERROR: total is non-positive, returning error"
        RETURN Error("Invalid total")
    END IF

    SET result = CALL submitToPayment(total)
    LOG "  payment result=" + result.status

    RETURN result
END FUNCTION
```

```
Log Level Strategy:

  +-------+----------------------------------------------+
  | ERROR | Something failed. Needs immediate attention. |
  | WARN  | Something unexpected but recoverable.        |
  | INFO  | High-level flow (request received, completed)|
  | DEBUG | Detailed internal state (variable values)    |
  | TRACE | Line-by-line execution (very verbose)        |
  +-------+----------------------------------------------+
```

### Strategy 2: Breakpoint Debugging

Pause execution at specific points and inspect the program state.

```
Breakpoint Debugging Workflow:

  1. SET breakpoint at the suspicious line
  2. RUN the program until it hits the breakpoint
  3. INSPECT local variables, call stack, and object state
  4. STEP through code one line at a time
     - Step Over: Execute current line, move to next
     - Step Into: Enter the function being called
     - Step Out: Finish current function, return to caller
  5. WATCH expressions: Monitor specific values as you step
  6. CONTINUE to next breakpoint or end of execution
```

```
Breakpoint Types:

  +---------------------+------------------------------------------+
  | Line breakpoint     | Pause at a specific line number          |
  | Conditional         | Pause only when a condition is true      |
  |                     | (e.g., break when count > 100)           |
  | Data breakpoint     | Pause when a variable's value changes    |
  | Exception breakpoint| Pause when a specific error is raised    |
  +---------------------+------------------------------------------+
```

### Strategy 3: Divide and Conquer

Systematically isolate the bug by eliminating sections of code.

```
PROCEDURE divideAndConquer(buggySystem: System):
    // Comment out or bypass half the code
    // Does the bug still occur?
    //   YES --> Bug is in the remaining half
    //   NO  --> Bug is in the removed half
    // Repeat on the guilty half
END PROCEDURE
```

---

## Memory Debugging

### Common Memory Issues

```
+---+---------------------------+--------------------------------------+
| # | Issue                     | Symptom                              |
+---+---------------------------+--------------------------------------+
| 1 | Memory leak               | Memory usage grows continuously     |
| 2 | Use after free            | Crash or corrupted data             |
| 3 | Buffer overflow           | Overwriting adjacent memory          |
| 4 | Double free               | Crash on second deallocation         |
| 5 | Stack overflow            | Deep recursion exhausts call stack  |
| 6 | Fragmentation             | Allocation fails despite free memory|
+---+---------------------------+--------------------------------------+
```

### Memory Debugging Strategy

```
PROCEDURE debugMemoryLeak():
    1. MEASURE baseline memory usage
    2. PERFORM a known operation N times
    3. MEASURE memory usage after operations
    4. IF memory grew proportionally to N:
         Leak confirmed in that operation
    5. NARROW DOWN to the specific allocation
       that is not being freed
    6. ADD proper resource cleanup
    7. VERIFY memory stabilizes after operations
END PROCEDURE
```

---

## Race Condition Debugging

Race conditions occur when the system's behavior depends on the timing of
concurrent operations.

```
Race Condition Example:

  Thread A:                    Thread B:
  READ balance = 100           READ balance = 100
  SET balance = 100 - 30       SET balance = 100 - 50
  WRITE balance = 70           WRITE balance = 50

  Expected: 100 - 30 - 50 = 20
  Actual:   50 (Thread B's write overwrites Thread A's)
```

### Race Condition Debugging Strategy

```
+---+-----------------------------------------------------------+
| 1 | Add logging with timestamps to concurrent operations      |
| 2 | Insert artificial delays to amplify the race window       |
| 3 | Use tools that detect data races (thread sanitizers)      |
| 4 | Look for shared mutable state without synchronization     |
| 5 | Check for missing locks, atomic operations, or barriers   |
| 6 | Test under high contention (many concurrent threads)      |
+---+-----------------------------------------------------------+
```

---

## Post-Mortem Analysis

After fixing a significant bug, conduct a blameless post-mortem.

```
Post-Mortem Template:

  +---------------------------------------------------+
  | INCIDENT POST-MORTEM                              |
  +---------------------------------------------------+
  | Date:        When the bug was discovered          |
  | Duration:    How long it was present / active     |
  | Impact:      Who and what was affected            |
  | Root Cause:  The fundamental reason the bug       |
  |              existed                              |
  | Timeline:    Sequence of events from discovery    |
  |              to resolution                        |
  | Fix Applied: What was changed to resolve it       |
  | Prevention:  How to prevent similar bugs:         |
  |   - New tests added?                              |
  |   - Code review checklist updated?                |
  |   - Monitoring/alerting added?                    |
  |   - Process changes?                              |
  | Lessons:     What did we learn?                   |
  +---------------------------------------------------+
```

### Five Whys Technique

Keep asking "why" until you reach the root cause.

```
  Problem: Users see incorrect account balances.
  Why? --> The balance calculation has a rounding error.
  Why? --> Floating-point arithmetic is used for currency.
  Why? --> The original developer used default number types.
  Why? --> There was no standard for currency handling.
  Why? --> We lacked coding guidelines for financial data.

  Root cause: Missing coding standards for financial calculations.
  Fix: Establish standards + use fixed-precision arithmetic.
```

---

## Debugging Checklist

```
+----------------------------------------------------------+
| Debugging Checklist                                      |
+----------------------------------------------------------+
| [ ] Bug is reproducible with clear steps                 |
| [ ] Error messages and logs have been read completely    |
| [ ] Recent changes have been reviewed                    |
| [ ] Assumptions about inputs and state are verified      |
| [ ] One variable is changed at a time                    |
| [ ] Root cause is identified (not just symptom patched)  |
| [ ] Fix is verified against the original reproduction    |
| [ ] Regression test is added to prevent recurrence       |
| [ ] Post-mortem is conducted for significant bugs        |
+----------------------------------------------------------+
```

Debugging is not random guessing. It is a disciplined, systematic process.
The best debuggers are methodical, patient, and relentlessly curious about
why things behave the way they do.
