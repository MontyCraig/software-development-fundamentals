# Estimation and Planning

## Introduction

Estimation is one of the hardest problems in software development. Unlike
manufacturing, where tasks are repetitive and predictable, software work is
primarily creative -- every feature involves solving a problem that has not been
solved in exactly that way before. Despite this difficulty, teams must make
forecasts to coordinate work, allocate budgets, and set expectations.

This chapter covers techniques that acknowledge uncertainty honestly rather than
pretending it does not exist.

## Why Estimation Is Hard

### The Cone of Uncertainty

At the start of a project, the range of possible outcomes is enormous. As
decisions are made and work is completed, uncertainty narrows. Estimates made
early are inherently less reliable than estimates made later.

```
THE CONE OF UNCERTAINTY

  Estimate
  Range
  (multiplier)
    |
 4x |  *
    |   *
 3x |    *
    |     *
 2x |      * *
    |          * *
 1x |              * * * * * * * *     <-- Actual effort
    |          * *
0.5x|      * *
    |     *
0.25x|   *
    |  *
    +----+----+----+----+----+----+---> Project Phase
     Concept  Reqts  Design  Code  Test  Ship

  Early estimates can be off by 4x in either direction.
  They only converge as work progresses.
```

### Hofstadter's Law

> "It always takes longer than you expect, even when you take into account
> Hofstadter's Law." -- Douglas Hofstadter

This recursive observation captures a deep truth: humans are systematically
optimistic about effort estimates, and knowing this does not fully correct for it.

## Story Points vs Time-Based Estimation

There are two broad approaches to estimation. Each has trade-offs.

```
ESTIMATION APPROACHES

  STORY POINTS                          TIME-BASED
  +-------------------------------+    +-------------------------------+
  | - Measure relative complexity |    | - Measure absolute duration   |
  | - "This is twice as hard      |    | - "This will take 3 days"    |
  |    as that"                   |    |                               |
  | - Team velocity calibrates    |    | - Directly maps to calendar  |
  |   over time                   |    | - Stakeholders understand it |
  | - Avoids false precision      |    | - Prone to anchoring bias    |
  | - Harder for stakeholders     |    | - Punishes slower workers    |
  |   to interpret                |    | - Creates implicit commitment|
  +-------------------------------+    +-------------------------------+
```

### When to Use Each

- **Story points** work well for iterative development where the team tracks
  velocity over multiple sprints and uses historical data to forecast.
- **Time-based estimates** are necessary when coordinating with external
  dependencies (contracts, regulatory deadlines, hardware procurement).

## Planning Poker Process

Planning poker is a consensus-based estimation technique that reduces anchoring
bias by having everyone reveal their estimate simultaneously.

```
PLANNING POKER PROCESS

  Step 1: Present        Step 2: Individual     Step 3: Reveal
  the item               estimation             simultaneously

  +-------------+        +---------+            +---+---+---+---+
  | "As a user, |        | Each    |            | 3 | 5 | 5 | 8 |
  |  I want to  |        | person  |            +---+---+---+---+
  |  reset my   |        | picks a |              ^           ^
  |  password"  |        | card    |              |           |
  +-------------+        | secretly|            Low         High
                         +---------+            must explain

  Step 4: Discuss        Step 5: Re-estimate    Step 6: Converge
  outliers               after discussion       on consensus

  "Why 3?"               +---------+            +---+---+---+---+
  "I assumed we          | Repeat  |            | 5 | 5 | 5 | 5 |
   reuse the existing    | steps   |            +---+---+---+---+
   email module"         | 2 - 4   |
                         +---------+            Consensus reached
  "Why 8?"
  "What about the
   security audit
   requirement?"
```

### Common Point Scales

- **Fibonacci**: 1, 2, 3, 5, 8, 13, 21 -- gaps grow with uncertainty
- **Powers of 2**: 1, 2, 4, 8, 16 -- simple doubling
- **T-shirt sizes**: XS, S, M, L, XL -- avoids numeric precision entirely

## Velocity Tracking and Forecasting

Velocity is the amount of work a team completes per iteration, measured in the
same units used for estimation (typically story points).

```
VELOCITY OVER TIME

  Points
  Completed
    |
 40 |              *
    |        *           *     *
 30 |  *           *                 *
    |        *                 *
 20 |                                      Average: 30
    |  - - - - - - - - - - - - - - - - -
 10 |
    |
  0 +--+--+--+--+--+--+--+--+--+--+--> Sprint
     1  2  3  4  5  6  7  8  9  10

  Use a rolling average (last 3-5 sprints) for forecasting.
  DO NOT use best-ever or worst-ever velocity.
```

### Forecasting with Velocity

```
FUNCTION ForecastCompletion(backlog_points, velocity_history)
    SET recent_sprints TO LastN(velocity_history, 5)

    SET optimistic_velocity TO Maximum(recent_sprints)
    SET pessimistic_velocity TO Minimum(recent_sprints)
    SET average_velocity TO Sum(recent_sprints) / Length(recent_sprints)

    IF average_velocity = 0 THEN
        RETURN "Insufficient data to forecast"
    END IF

    SET optimistic_sprints TO Ceiling(backlog_points / optimistic_velocity)
    SET expected_sprints TO Ceiling(backlog_points / average_velocity)
    SET pessimistic_sprints TO Ceiling(backlog_points / pessimistic_velocity)

    RETURN {
        "best_case": optimistic_sprints,
        "expected": expected_sprints,
        "worst_case": pessimistic_sprints
    }
END FUNCTION
```

**Example**: 150 points remaining, velocity history [28, 35, 30, 32, 25].
- Best case: 150 / 35 = 5 sprints
- Expected: 150 / 30 = 5 sprints
- Worst case: 150 / 25 = 6 sprints

Always communicate the range, not a single number.

## PERT Estimation

The Program Evaluation and Review Technique (PERT) produces a weighted estimate
from three scenarios, giving extra weight to the most likely case.

```
PERT FORMULA

                Optimistic + (4 * Most Likely) + Pessimistic
  Estimate  =  -----------------------------------------------
                                    6

  Standard              Pessimistic - Optimistic
  Deviation  =          -------------------------
                                  6
```

```
FUNCTION PERTEstimate(optimistic, most_likely, pessimistic)
    SET estimate TO (optimistic + (4 * most_likely) + pessimistic) / 6
    SET std_dev TO (pessimistic - optimistic) / 6

    RETURN {
        "estimate": estimate,
        "std_deviation": std_dev,
        "confidence_68": estimate + std_dev,
        "confidence_95": estimate + (2 * std_dev)
    }
END FUNCTION
```

**Example**: A task estimated at 2 days optimistic, 5 days most likely,
14 days pessimistic:
- PERT estimate: (2 + 20 + 14) / 6 = 6 days
- Standard deviation: (14 - 2) / 6 = 2 days
- 95% confidence: 6 + 4 = 10 days

## Reference-Class Forecasting

Instead of estimating from first principles, reference-class forecasting asks:
"How long did similar things take in the past?"

```
REFERENCE-CLASS FORECASTING

  Step 1: Identify the reference class
  +--------------------------------------------------+
  | "What category does this work fall into?"         |
  | Examples: new API endpoint, database migration,   |
  |           third-party integration, UI form        |
  +--------------------------------------------------+
           |
           v
  Step 2: Collect historical data
  +--------------------------------------------------+
  | "How long did past items in this class take?"     |
  | Find 5-10 comparable completed items              |
  +--------------------------------------------------+
           |
           v
  Step 3: Use the distribution
  +--------------------------------------------------+
  | Median = expected duration                        |
  | 80th percentile = buffer for uncertainty          |
  | Maximum = worst-case planning                     |
  +--------------------------------------------------+
```

This technique is powerful because it sidesteps the cognitive biases that
plague bottom-up estimation.

## Estimation Anti-Patterns

### Anchoring

The first number mentioned disproportionately influences subsequent estimates.

**Example**: A manager says "I think this is about 2 weeks" before the team
estimates. The team now anchors around 2 weeks regardless of actual complexity.

**Fix**: Use planning poker (simultaneous reveal) to prevent anchoring.

### Padding

Adding a hidden buffer to every estimate "just in case."

**Problems**: Padding compounds at each level of aggregation. If developers pad
by 20%, leads pad by 20%, and managers pad by 20%, a 10-day task becomes 17 days.
Also, Parkinson's Law dictates that work expands to fill the padded time.

**Fix**: Estimate honestly and manage uncertainty explicitly through ranges.

### Optimism Bias

Estimating based on the best-case scenario while ignoring likely complications.

**Example**: "If nothing goes wrong, we can do this in 3 days." Something almost
always goes wrong.

**Fix**: Use PERT estimation, which forces you to consider pessimistic scenarios.

### Precision Fallacy

Presenting estimates with unjustified precision.

**Example**: "This will take 11.5 days." This implies an accuracy that does not
exist for creative work.

**Fix**: Use ranges ("8-15 days") or relative sizing (story points).

### Sunk Cost Anchoring

Refusing to re-estimate when new information reveals the original estimate was
wrong.

**Example**: "We estimated 3 sprints so we must finish in 3 sprints" even after
discovering major unforeseen complexity.

**Fix**: Re-estimate regularly. Past estimates are predictions, not commitments.

## The No-Estimates Movement

Some practitioners argue that estimation is waste -- the effort spent estimating
could be spent building. The no-estimates approach proposes alternatives:

```
NO-ESTIMATES ALTERNATIVES

  Instead of...              Try...
  +-----------------------+  +----------------------------------+
  | "How many points is   |  | "Can we slice this into pieces  |
  |  this?"               |  |  that each take 1-2 days?"      |
  +-----------------------+  +----------------------------------+
  | "When will the        |  | "How many items can we finish   |
  |  project be done?"    |  |  per week? Count remaining      |
  +-----------------------+  |  items."                         |
  | "Estimate every item  |  +----------------------------------+
  |  in the backlog"      |  | "Make items roughly equal size  |
  +-----------------------+  |  and count them."               |
                             +----------------------------------+
```

The key insight: if items are roughly uniform in size, counting items and
measuring throughput can be more accurate than estimating and measuring velocity.

## Summary

```
ESTIMATION DECISION TREE

  Do you have historical data?
       |              |
      YES             NO
       |              |
       v              v
  Reference-class    PERT estimation
  forecasting        (3-point)
       |              |
       v              v
  Can you slice     Planning poker
  items uniformly?  for relative
       |            sizing
      YES
       |
       v
  Consider throughput-
  based forecasting
  (no-estimates)
```

The best estimation practice is the one that produces honest forecasts your
team and stakeholders can act on. Whatever technique you choose, always
communicate uncertainty explicitly and re-estimate as you learn more.

---

## Practice Problems

1. **PERT calculation**: A team estimates a feature at 3 days optimistic, 7 days
   most likely, and 18 days pessimistic. Calculate the PERT estimate, standard
   deviation, and 95% confidence bound.

2. **Velocity forecasting**: A team's velocity over the last 6 sprints was:
   [22, 28, 25, 30, 27, 24]. They have 200 story points remaining. Calculate
   the best-case, expected, and worst-case number of sprints to completion
   using the last 5 sprints.

3. **Anti-pattern identification**: A project manager tells the team: "The client
   needs this in exactly 6 weeks. Please estimate accordingly." Identify the
   anti-patterns present and explain why this approach is problematic.

4. **Reference-class exercise**: You need to estimate a "user password reset"
   feature. From past data, similar features took: 4, 6, 5, 8, 5, 12, 6, 7
   days. What would you use as your expected estimate? What would you communicate
   as the upper bound?

5. **Slicing exercise**: The following item is estimated at 21 story points:
   "Build a reporting dashboard that shows sales by region, time period, and
   product category with export capability." Slice this into at least 5 smaller
   items that could each be completed in 1-3 points.
