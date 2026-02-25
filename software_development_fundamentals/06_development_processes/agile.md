# Agile Development

## The Agile Manifesto (2001)

### Four Values

```
+---------------------------------------------------------------+
|                      AGILE MANIFESTO                          |
+---------------------------------------------------------------+
|                                                               |
|  Individuals and interactions   OVER  Processes and tools     |
|                                                               |
|  Working software               OVER  Comprehensive docs     |
|                                                               |
|  Customer collaboration         OVER  Contract negotiation   |
|                                                               |
|  Responding to change           OVER  Following a plan       |
|                                                               |
|  While there is value in the items on the right,             |
|  we value the items on the left more.                        |
+---------------------------------------------------------------+
```

### Twelve Principles

```
+---+---------------------------------------------------------------+
| 1 | Highest priority: satisfy the customer through early and      |
|   | continuous delivery of valuable software                      |
+---+---------------------------------------------------------------+
| 2 | Welcome changing requirements, even late in development       |
+---+---------------------------------------------------------------+
| 3 | Deliver working software frequently (weeks, not months)       |
+---+---------------------------------------------------------------+
| 4 | Business people and developers must work together daily       |
+---+---------------------------------------------------------------+
| 5 | Build projects around motivated individuals; give them the    |
|   | environment and support they need, and trust them             |
+---+---------------------------------------------------------------+
| 6 | Face-to-face conversation is the most efficient and effective |
|   | method of conveying information                               |
+---+---------------------------------------------------------------+
| 7 | Working software is the primary measure of progress           |
+---+---------------------------------------------------------------+
| 8 | Sustainable development pace -- maintain it indefinitely      |
+---+---------------------------------------------------------------+
| 9 | Continuous attention to technical excellence and good design  |
+---+---------------------------------------------------------------+
|10 | Simplicity -- maximizing the amount of work not done          |
+---+---------------------------------------------------------------+
|11 | Best architectures, requirements, and designs emerge from     |
|   | self-organizing teams                                        |
+---+---------------------------------------------------------------+
|12 | Regularly reflect and adjust behavior for effectiveness       |
+---+---------------------------------------------------------------+
```

---

## Scrum

Scrum is an agile framework for managing work in fixed-length iterations
called Sprints (typically 1-4 weeks).

### Roles

```
+------------------+-----------------------------------------------+
| Role             | Responsibility                                |
+------------------+-----------------------------------------------+
| Product Owner    | Defines what to build (manages backlog),      |
|                  | maximizes product value, represents customers |
+------------------+-----------------------------------------------+
| Scrum Master     | Facilitates process, removes impediments,     |
|                  | coaches the team on Scrum practices           |
+------------------+-----------------------------------------------+
| Development Team | Self-organizing group (3-9 people) that       |
|                  | builds the product increment                  |
+------------------+-----------------------------------------------+
```

### Ceremonies (Events)

```
+---------------------+----------+---------------------------------------+
| Ceremony            | Duration | Purpose                               |
+---------------------+----------+---------------------------------------+
| Sprint Planning     | 2-4 hrs  | Select backlog items, define sprint    |
|                     |          | goal, plan tasks                      |
+---------------------+----------+---------------------------------------+
| Daily Standup       | 15 min   | Synchronize: What did I do? What      |
|                     |          | will I do? Any blockers?              |
+---------------------+----------+---------------------------------------+
| Sprint Review       | 1-2 hrs  | Demo working software to stakeholders |
+---------------------+----------+---------------------------------------+
| Sprint Retrospective| 1-2 hrs  | Reflect: What went well? What to      |
|                     |          | improve? Action items.                |
+---------------------+----------+---------------------------------------+
| Backlog Refinement  | Ongoing  | Clarify, estimate, and prioritize     |
|                     |          | upcoming backlog items                |
+---------------------+----------+---------------------------------------+
```

### Artifacts

```
+-------------------+----------------------------------------------+
| Artifact          | Description                                  |
+-------------------+----------------------------------------------+
| Product Backlog   | Ordered list of all desired features,        |
|                   | changes, and fixes                           |
+-------------------+----------------------------------------------+
| Sprint Backlog    | Subset of product backlog selected for the   |
|                   | current sprint plus the plan for delivery    |
+-------------------+----------------------------------------------+
| Product Increment | Working, tested software delivered at the    |
|                   | end of each sprint                           |
+-------------------+----------------------------------------------+
```

### Sprint Board (ASCII)

```
+------------------------------------------------------------------+
|                     SPRINT BOARD                                 |
+------------------------------------------------------------------+
| TO DO          | IN PROGRESS      | IN REVIEW    | DONE         |
|----------------|------------------|--------------|--------------|
| [Story #5]     | [Story #2] (Bob) | [Story #1]   | [Story #4]   |
|   Task 5a      |   Task 2c        |   (Alice     |   All tasks  |
|   Task 5b      |                  |    reviewing) |   completed  |
|                |                  |              |              |
| [Story #7]     | [Story #3] (Eve) |              | [Story #6]   |
|   Task 7a      |   Task 3b        |              |   All tasks  |
|   Task 7b      |                  |              |   completed  |
|   Task 7c      |                  |              |              |
+------------------------------------------------------------------+
| Sprint Goal: Complete user authentication module                 |
| Velocity: 34 story points | Days Remaining: 6                   |
+------------------------------------------------------------------+
```

---

## Kanban

Kanban focuses on continuous flow rather than fixed iterations. Work items
flow through stages with explicit work-in-progress (WIP) limits.

### Core Practices

1. Visualize the workflow
2. Limit work in progress (WIP)
3. Manage flow
4. Make process policies explicit
5. Implement feedback loops
6. Improve collaboratively

### Kanban Board (ASCII)

```
+------------------------------------------------------------------+
|                     KANBAN BOARD                                 |
+------------------------------------------------------------------+
| BACKLOG     | ANALYSIS  | DEV       | TEST      | DEPLOY | DONE |
| (no limit)  | WIP: 3    | WIP: 4   | WIP: 3    | WIP: 2 |      |
|-------------|-----------|----------|-----------|--------|------|
| [Item 12]   | [Item 8]  | [Item 5] | [Item 3]  |[Item 1]|[#10] |
| [Item 13]   | [Item 9]  | [Item 6] | [Item 4]  |        |[#11] |
| [Item 14]   | [Item 10] | [Item 7] |           |        |[#9]  |
| [Item 15]   |           |          |           |        |[#2]  |
| [Item 16]   |           |          |           |        |      |
+------------------------------------------------------------------+
| Lead Time: avg 8 days  | Cycle Time: avg 5 days | Throughput: 3/wk |
+------------------------------------------------------------------+
```

### WIP Limits and Flow

```
Why WIP Limits Matter:

  Without limits:          With limits (WIP: 3):
  +----+                   +----+
  |AAAA|  Start many,      |AAA |  Start few,
  |BBBB|  finish few       |    |  finish more
  |CCCC|                   +----+
  |DDDD|                   Throughput: HIGH
  +----+                   Lead time: SHORT
  Throughput: LOW
  Lead time: LONG

  Little's Law: Lead Time = WIP / Throughput
  Reducing WIP reduces lead time.
```

---

## Extreme Programming (XP)

XP emphasizes technical practices that improve code quality and team
responsiveness.

### Core Practices

```
+---+---------------------------+--------------------------------------+
| # | Practice                  | Description                          |
+---+---------------------------+--------------------------------------+
| 1 | Pair Programming          | Two developers work at one station   |
| 2 | Test-Driven Development   | Write tests before implementation    |
| 3 | Continuous Integration    | Merge and test code multiple times   |
|   |                           | per day                              |
| 4 | Refactoring               | Continuously improve code structure  |
| 5 | Simple Design             | The simplest thing that could work   |
| 6 | Collective Code Ownership | Anyone can modify any code           |
| 7 | Coding Standards          | Agreed-upon conventions              |
| 8 | Sustainable Pace          | 40-hour weeks; no burnout            |
| 9 | On-Site Customer          | Direct access to business expertise  |
|10 | Small Releases            | Frequent, incremental deliveries     |
|11 | Planning Game             | Collaborative estimation sessions    |
|12 | Metaphor                  | Shared vocabulary for the system     |
+---+---------------------------+--------------------------------------+
```

---

## User Stories

### Format

```
As a [type of user],
I want [some goal],
So that [some reason/benefit].
```

### INVEST Criteria

```
+---+----------------+----------------------------------------------+
| I | Independent    | Stories can be developed in any order         |
| N | Negotiable     | Details can be discussed and refined          |
| V | Valuable       | Each story delivers value to the user         |
| E | Estimable      | The team can estimate the effort              |
| S | Small          | Fits within a single sprint                  |
| T | Testable       | Acceptance criteria can be verified           |
+---+----------------+----------------------------------------------+
```

### Story Example with Acceptance Criteria

```
STORY: User Login
  As a registered user,
  I want to log in with my credentials,
  So that I can access my personal dashboard.

  ACCEPTANCE CRITERIA:
    GIVEN a registered user with valid credentials
    WHEN they submit the login form
    THEN they are redirected to their dashboard

    GIVEN a user with invalid credentials
    WHEN they submit the login form
    THEN an error message is displayed

    GIVEN a user who fails login 5 times
    WHEN they attempt a 6th login
    THEN their account is temporarily locked for 15 minutes
```

---

## Estimation Techniques

### Planning Poker

```
Process:
  1. Product Owner presents a story
  2. Team discusses to understand scope
  3. Each member privately selects a card
  4. All cards revealed simultaneously
  5. Discuss outliers (highest and lowest)
  6. Re-vote if needed until consensus

  Card Values (Fibonacci-like):
  +---+---+---+---+---+---+---+---+---+
  | 1 | 2 | 3 | 5 | 8 |13 |20 |40 | ? |
  +---+---+---+---+---+---+---+---+---+
  small         medium        large  unknown
```

### T-Shirt Sizing

```
  +----+--------+-------------------+
  | XS | 1 day  | Trivial change    |
  | S  | 2-3 d  | Small feature     |
  | M  | 1 week | Standard feature  |
  | L  | 2 week | Complex feature   |
  | XL | 1 month| Epic (split it!)  |
  +----+--------+-------------------+
```

---

## Velocity Tracking

Velocity is the average number of story points completed per sprint.

```
Velocity Chart:

  Points
  50 |
  45 |                          *
  40 |              *     *          *
  35 |        *                           *
  30 |  *                                      Average: 38
  25 |
  20 |
     +--+-----+-----+-----+-----+-----+-----+---
        S1    S2    S3    S4    S5    S6    S7

  Use: After 3-4 sprints, velocity stabilizes and becomes
  a reliable forecasting tool.

  Capacity = Average Velocity = ~38 points/sprint
  If remaining backlog = 190 points
  Estimated sprints remaining = 190 / 38 = 5 sprints
```

---

## Scrum vs Kanban Comparison

```
+---------------------+------------------------+------------------------+
| Aspect              | Scrum                  | Kanban                 |
+---------------------+------------------------+------------------------+
| Cadence             | Fixed sprints          | Continuous flow        |
| Roles               | PO, SM, Dev Team       | No prescribed roles    |
| Change policy       | After sprint ends      | Anytime                |
| Metrics             | Velocity               | Lead time, throughput  |
| Work commitment     | Sprint backlog         | WIP limits             |
| Planning            | Sprint planning events | On-demand              |
| Best for            | Product development    | Support, maintenance   |
+---------------------+------------------------+------------------------+
```

---

## Common Agile Anti-Patterns

```
+---+---------------------------+--------------------------------------+
| # | Anti-Pattern              | Problem                              |
+---+---------------------------+--------------------------------------+
| 1 | Cargo Cult Agile          | Following rituals without            |
|   |                           | understanding the purpose            |
+---+---------------------------+--------------------------------------+
| 2 | Sprint Zero Forever       | Never-ending "setup" sprints         |
+---+---------------------------+--------------------------------------+
| 3 | Velocity as Target        | Optimizing the metric, not value     |
+---+---------------------------+--------------------------------------+
| 4 | Absent Product Owner      | No one available to make decisions   |
+---+---------------------------+--------------------------------------+
| 5 | Standup as Status Report  | Reporting to manager instead of      |
|   |                           | synchronizing with peers             |
+---+---------------------------+--------------------------------------+
| 6 | No Retrospective Actions  | Reflecting without changing          |
+---+---------------------------+--------------------------------------+
| 7 | Zombie Scrum              | All the ceremonies, no soul          |
+---+---------------------------+--------------------------------------+
```

Agile is a mindset, not a checklist. The practices exist to serve the values
and principles, not the other way around.
