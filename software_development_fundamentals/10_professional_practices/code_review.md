# Code Review

## Introduction

Code review is the systematic examination of source code by someone other than the
original author. It is one of the most effective quality practices available to a
development team -- not because it catches every bug, but because it creates a
culture of shared ownership, collective learning, and explicit quality standards.

```
WHY CODE REVIEW MATTERS
+-------------------------------------------------------------+
|                                                             |
|  Defect         Knowledge       Code            Team        |
|  Detection      Sharing         Consistency     Ownership   |
|  +---------+    +---------+    +---------+    +---------+   |
|  | Catch   |    | Spread  |    | Enforce |    | Nobody  |   |
|  | bugs    |    | domain  |    | style & |    | owns    |   |
|  | before  |    | knowl-  |    | archi-  |    | code    |   |
|  | they    |    | edge    |    | tecture |    | alone   |   |
|  | ship    |    | broadly |    | norms   |    |         |   |
|  +---------+    +---------+    +---------+    +---------+   |
|                                                             |
+-------------------------------------------------------------+
```

## The Review Workflow

A typical code review follows a well-defined lifecycle from the moment a change
is ready for inspection to the moment it is merged.

```
REVIEW WORKFLOW
                                                          
  Author                  Reviewer                 Repository
  ------                  --------                 ----------
    |                        |                         |
    |  1. Submit change      |                         |
    |----------------------->|                         |
    |                        |                         |
    |  2. Review & comment   |                         |
    |<-----------------------|                         |
    |                        |                         |
    |  3. Address feedback   |                         |
    |----------------------->|                         |
    |                        |                         |
    |  4. Re-review          |                         |
    |<-----------------------|                         |
    |                        |                         |
    |  5. Approve            |                         |
    |<-----------------------|                         |
    |                        |                         |
    |  6. Merge              |                         |
    |------------------------------------------------->|
    |                        |                         |
```

### Step-by-Step

1. **Author submits**: The author creates a change request with a clear title,
   description, and context for the change.
2. **Reviewer examines**: One or more reviewers read the code, run it mentally
   or locally, and leave comments.
3. **Author responds**: The author addresses each comment -- by changing the code,
   explaining the reasoning, or discussing alternatives.
4. **Iteration**: Steps 2-3 repeat until both parties are satisfied.
5. **Approval**: The reviewer formally approves the change.
6. **Merge**: The change is integrated into the main codebase.

## The Review Checklist

Use a structured checklist to ensure consistency across reviews. Not every item
applies to every change, but the checklist keeps the reviewer from overlooking
important categories.

```
REVIEW CHECKLIST
+---+---------------------------+-----------------------------------+
| # | Category                  | Key Questions                     |
+---+---------------------------+-----------------------------------+
| 1 | Correctness               | Does the code do what it claims?  |
|   |                           | Are edge cases handled?           |
+---+---------------------------+-----------------------------------+
| 2 | Readability               | Can a new team member understand  |
|   |                           | this in 5 minutes?                |
+---+---------------------------+-----------------------------------+
| 3 | Performance               | Are there unnecessary loops,      |
|   |                           | allocations, or blocking calls?   |
+---+---------------------------+-----------------------------------+
| 4 | Security                  | Is input validated? Are secrets   |
|   |                           | kept out of the code?             |
+---+---------------------------+-----------------------------------+
| 5 | Tests                     | Are new behaviors covered? Do     |
|   |                           | existing tests still pass?        |
+---+---------------------------+-----------------------------------+
| 6 | Design                    | Does this fit the existing        |
|   |                           | architecture? Is it over-built?   |
+---+---------------------------+-----------------------------------+
| 7 | Documentation             | Are public interfaces documented? |
|   |                           | Is the commit message clear?      |
+---+---------------------------+-----------------------------------+
```

### Correctness

- Does the logic match the stated requirements?
- Are boundary conditions handled (empty inputs, maximum values, null cases)?
- Are error paths exercised, not just the happy path?
- Is state mutated safely in concurrent contexts?

### Readability

- Are names descriptive and consistent with the codebase conventions?
- Is the control flow straightforward, or does it require mental gymnastics?
- Are comments explaining *why*, not restating *what* the code does?
- Could long functions be broken into smaller, named steps?

### Performance

- Are there accidental O(n^2) patterns hidden in nested loops?
- Is work being repeated that could be cached or memoized?
- Are resources (connections, file handles) properly released?

### Security

- Is user input validated and sanitized before use?
- Are credentials, tokens, or keys absent from the change?
- Does the code follow the principle of least privilege?
- Are cryptographic operations using well-established algorithms?

### Tests

- Does the change include tests for new behavior?
- Do tests cover both positive and negative cases?
- Are tests deterministic (no flakiness from timing or ordering)?

## Giving Constructive Feedback

The way feedback is delivered matters as much as its content. Poorly worded
comments shut down communication; well-worded ones invite collaboration.

```
FEEDBACK SPECTRUM

  Destructive <---------------------------------> Constructive

  "This is wrong"    "Why did you    "Could we       "Nice approach!
                      do it this     consider X      One thing to
                      way?"          here? It        consider: X
                                     handles the     would handle
                                     edge case       the edge case
                                     where..."       where..."
```

### Examples of Good vs Bad Comments

**Bad**: "This is terrible. Rewrite it."
**Good**: "This function is handling three responsibilities. Could we extract
the validation into its own function? That would make each piece easier to test."

**Bad**: "Wrong."
**Good**: "This will fail when the input list is empty because the loop body
assumes at least one element. We could add a guard clause at the top."

**Bad**: "I would never do it this way."
**Good**: "An alternative approach might be to use a lookup table here instead
of the chain of conditionals. It would reduce the cyclomatic complexity and
make it easier to add new cases. What do you think?"

### Principles for Good Feedback

1. **Comment on the code, not the person.** Say "this function" not "you."
2. **Ask questions rather than make demands.** "Could we..." invites dialogue.
3. **Explain the *why*.** Give the reason behind the suggestion.
4. **Distinguish blockers from suggestions.** Label comments as "blocking",
   "suggestion", or "nit" so the author knows what must change.
5. **Acknowledge good work.** Positive feedback reinforces good practices.

## Receiving Feedback Gracefully

Receiving critique on your work is a skill that improves with practice.

1. **Assume good intent.** The reviewer is trying to improve the code, not
   attack you personally.
2. **Separate ego from code.** The code is a shared artifact; improving it
   benefits everyone.
3. **Ask for clarification.** If a comment is unclear, ask for an example or
   a more specific suggestion.
4. **Disagree respectfully.** If you believe the current approach is correct,
   explain your reasoning with evidence.
5. **Thank your reviewers.** They invested time to make the code better.

## Automated vs Manual Review

Both automated and manual review have a role. They complement each other rather
than compete.

```
AUTOMATED vs MANUAL REVIEW

  AUTOMATED                              MANUAL
  +-----------------------------+       +-----------------------------+
  | - Style and formatting      |       | - Design and architecture   |
  | - Known bug patterns        |       | - Business logic correctness|
  | - Security vulnerability    |       | - Naming and readability    |
  |   scanning                  |       | - Algorithmic efficiency    |
  | - Test coverage reporting   |       | - Missing test scenarios    |
  | - Dependency vulnerability  |       | - Domain-specific knowledge |
  |   checks                    |       | - Maintainability judgment  |
  +-----------------------------+       +-----------------------------+
         |                                       |
         v                                       v
  Fast, consistent, tireless         Nuanced, contextual, creative
```

**Best practice**: Let automated tools handle the mechanical checks so that
human reviewers can focus on design, logic, and maintainability.

## Common Anti-Patterns

### Rubber-Stamping

Approving changes without reading them. This provides false confidence and
defeats the purpose of review.

**Signs**: Approvals in under a minute, no comments ever, "LGTM" on every change.

**Fix**: Require at least one substantive comment per review. Track review
thoroughness metrics.

### Nitpicking

Focusing exclusively on trivial style issues while ignoring substantive problems.

**Signs**: Dozens of comments about spacing and naming, zero comments about logic.

**Fix**: Automate style enforcement. Reserve human review for substance.

### Gatekeeping

Using review authority to block changes for personal preference rather than
objective quality concerns.

**Signs**: One reviewer repeatedly blocks changes, subjective objections without
evidence, "I just don't like it."

**Fix**: Require reviewers to cite a specific principle or risk. Rotate reviewers.

### Ping-Pong Reviews

Endless back-and-forth where small issues are raised one at a time across
many review rounds.

**Signs**: Five or more review cycles for a simple change.

**Fix**: Batch feedback -- raise all concerns in a single pass.

## Review Metrics

Measuring review effectiveness helps teams improve over time.

```
KEY REVIEW METRICS

  +--------------------+------------------------------------------+
  | Metric             | What It Measures                         |
  +--------------------+------------------------------------------+
  | Time to First      | How quickly reviewers engage with a      |
  | Review             | submitted change (hours)                 |
  +--------------------+------------------------------------------+
  | Review Cycle Time  | Total time from submission to merge      |
  |                    | (hours or days)                          |
  +--------------------+------------------------------------------+
  | Comments per       | Volume of feedback per change            |
  | Review             | (aim for 2-8 substantive)                |
  +--------------------+------------------------------------------+
  | Defect Escape Rate | Bugs found in production that were in    |
  |                    | reviewed code (lower is better)          |
  +--------------------+------------------------------------------+
  | Review Coverage    | Percentage of changes that go through    |
  |                    | review (aim for 100%)                    |
  +--------------------+------------------------------------------+
```

### Tracking Defect Density

```
FUNCTION CalculateDefectDensity(changes)
    SET total_defects TO 0
    SET total_lines_reviewed TO 0

    FOR EACH change IN changes DO
        SET total_defects TO total_defects + change.defects_found
        SET total_lines_reviewed TO total_lines_reviewed + change.lines_changed
    END FOR

    IF total_lines_reviewed = 0 THEN
        RETURN 0
    END IF

    RETURN total_defects / total_lines_reviewed * 1000
END FUNCTION

// Result: defects per 1000 lines reviewed
```

### Review Effectiveness Score

```
FUNCTION CalculateReviewEffectiveness(period)
    SET bugs_caught_in_review TO CountBugsFoundInReview(period)
    SET bugs_escaped_to_production TO CountProductionBugs(period)
    SET total_bugs TO bugs_caught_in_review + bugs_escaped_to_production

    IF total_bugs = 0 THEN
        RETURN 1.0
    END IF

    RETURN bugs_caught_in_review / total_bugs
END FUNCTION

// Effectiveness of 0.85 means 85% of bugs caught before production
```

## Summary

Effective code review requires both technical skill and interpersonal care.
The best review processes are fast, constructive, and focused on what matters
most: shipping correct, maintainable, secure software.

```
CODE REVIEW PRINCIPLES
+-----------------------------------------------------------+
|                                                           |
|  1. Review the code, not the person                       |
|  2. Automate the mechanical, humanize the judgment        |
|  3. Keep changes small and reviewable                     |
|  4. Respond to reviews within one business day            |
|  5. Distinguish blocking issues from suggestions          |
|  6. Every review is a teaching and learning opportunity   |
|                                                           |
+-----------------------------------------------------------+
```

---

## Practice Problems

1. **Feedback rewrite**: Rewrite the following review comment to be constructive:
   "This code is a mess. I can't even follow what it's doing. Fix it."
   Write at least two alternative comments that are specific and actionable.

2. **Checklist application**: Given the following pseudocode, list at least four
   review concerns across different checklist categories:
   ```
   FUNCTION ProcessOrder(order)
       SET total TO 0
       FOR EACH item IN order.items DO
           SET total TO total + item.price * item.quantity
       END FOR
       ChargePayment(order.customer_id, total)
       SendConfirmation(order.customer_email, total)
       RETURN "success"
   END FUNCTION
   ```

3. **Metric interpretation**: A team has the following review metrics:
   - Average time to first review: 4 hours
   - Average comments per review: 0.3
   - Defect escape rate: 15%
   Identify what these numbers suggest and propose two specific improvements.

4. **Anti-pattern identification**: A team lead reviews all pull requests and
   frequently requests changes based on personal style preferences not documented
   in the team's coding standards. Other team members rarely review code because
   the lead always gets to it first. Name the anti-patterns present and propose
   a plan to address them.

5. **Process design**: Design a code review process for a team of six developers.
   Specify: how reviewers are assigned, maximum review turnaround time, what
   automated checks run before human review, and how disagreements are resolved.
