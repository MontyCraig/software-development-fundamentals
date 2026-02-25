# Professional Practices

## Overview

Software development is as much a human endeavor as a technical one. Writing code
that compiles and passes tests is necessary but insufficient. Professional practices
encompass the disciplines, habits, and interpersonal skills that distinguish a
functioning team from a high-performing one. This section covers the practices that
experienced developers rely on to build sustainable software over time.

```
+---------------------------------------------------------------+
|                   PROFESSIONAL PRACTICES                       |
|                                                               |
|   +--------------+    +--------------+    +--------------+    |
|   | Code Review  |    |  Technical   |    | Estimation & |    |
|   |              |--->|    Debt      |--->|  Planning    |    |
|   +--------------+    +--------------+    +--------------+    |
|          |                   |                   |            |
|          v                   v                   v            |
|   +--------------+    +--------------+    +--------------+    |
|   |  Technical   |    |Communication |    |  Continuous  |    |
|   |Documentation |    |& Collaboration|   | Improvement  |    |
|   +--------------+    +--------------+    +--------------+    |
+---------------------------------------------------------------+
```

## Table of Contents

| # | Topic | Description |
|---|-------|-------------|
| 1 | [Code Review](code_review.md) | How teams inspect each other's work to catch defects, share knowledge, and maintain quality standards across a codebase. |
| 2 | [Technical Debt](technical_debt.md) | Understanding, measuring, and managing the accumulated cost of shortcuts and deferred work in software systems. |
| 3 | [Estimation and Planning](estimation_and_planning.md) | Techniques for forecasting effort, managing uncertainty, and making commitments the team can actually keep. |
| 4 | [Technical Documentation](technical_documentation.md) | Writing and maintaining documentation that serves its audience, from READMEs and API references to architecture decision records. |
| 5 | [Communication and Collaboration](communication_and_collaboration.md) | The interpersonal practices that make teams effective: clear writing, productive meetings, knowledge sharing, and blameless culture. |

## Recommended Reading Order

1. **Code Review** -- Start here. Review is the most frequent professional
   interaction most developers have. Understanding how to give and receive
   feedback well is foundational.

2. **Technical Debt** -- Once you can evaluate code through review, learn to
   recognize and quantify the systemic quality issues that accumulate over time.

3. **Estimation and Planning** -- With an understanding of quality trade-offs,
   learn how teams forecast work and make realistic commitments.

4. **Technical Documentation** -- Good documentation is the bridge between what
   the team knows and what the team can do. Learn the frameworks and habits that
   keep docs useful.

5. **Communication and Collaboration** -- This ties everything together. All
   prior topics depend on clear communication and healthy team dynamics.

## Prerequisites

This section assumes familiarity with version control concepts (branching,
merging, pull requests) and basic software development workflow. No specific
programming language knowledge is required.

## Key Themes

- **Sustainability** -- Practices that keep a codebase healthy over months and years
- **Empathy** -- Understanding your audience, whether they are reviewers, users, or future maintainers
- **Measurement** -- Quantifying quality so decisions are grounded in evidence, not opinion
- **Continuous improvement** -- Small, consistent investments compound into large gains
