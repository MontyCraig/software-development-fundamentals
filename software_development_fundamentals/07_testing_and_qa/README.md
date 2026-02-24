# Section 7: Testing and Quality Assurance

## Overview

Testing is the systematic practice of verifying that software behaves correctly and meets
its requirements. Quality assurance goes beyond testing to encompass the processes and
disciplines that prevent defects from being introduced in the first place.

This section covers the full testing pyramid -- from unit tests that verify individual
components in isolation, through integration and system tests that verify assembled
systems, to user acceptance testing that validates business requirements. It also covers
test-driven development as a design discipline and debugging as the systematic art of
finding and resolving defects.

All examples use language-agnostic pseudocode and ASCII diagrams.

---

## Table of Contents

### 1. [Unit Testing](unit_testing.md)
Testing the smallest testable units of software in isolation. Covers what constitutes a
"unit," the Arrange-Act-Assert pattern, test doubles (mocks, stubs, fakes, spies), test
naming conventions, boundary value analysis, and equivalence partitioning. Includes
pseudocode test examples and ASCII diagrams of test structure.

### 2. [Integration Testing](integration_testing.md)
Verifying that independently developed modules work correctly when combined. Covers
integration strategies (top-down, bottom-up, sandwich/hybrid, big-bang), test harnesses,
interface contract testing, and database integration testing. Includes ASCII diagrams of
integration topologies and stub/driver relationships.

### 3. [System Testing](system_testing.md)
Validating the complete, integrated system against its specified requirements. Covers
functional testing, non-functional testing (performance, security, usability, reliability),
regression testing, smoke testing, and sanity testing. Includes test plan templates and
traceability matrices in ASCII format.

### 4. [User Acceptance Testing (UAT)](user_acceptance_testing.md)
The final phase of testing where real users or stakeholders validate the system against
business requirements. Covers UAT planning, test scenario design, acceptance criteria,
alpha and beta testing, sign-off procedures, and the relationship between UAT and
business requirements. Includes acceptance criteria templates.

### 5. [Test-Driven Development (TDD)](test_driven_development.md)
A development discipline where tests are written before implementation code. Covers the
Red-Green-Refactor cycle, test-first design, the relationship between TDD and emergent
design, common TDD patterns, and when TDD is most (and least) effective. Includes
step-by-step pseudocode walkthroughs of the TDD cycle.

### 6. [Debugging](debugging.md)
The systematic process of finding and resolving software defects. Covers debugging
strategies (binary search, divide and conquer, cause elimination), common bug categories,
reading error messages effectively, logging and tracing, rubber duck debugging, and
post-mortem analysis. Includes decision-tree diagrams for debugging workflows.

---

## Recommended Reading Order

The section follows the testing pyramid from bottom to top, then covers complementary
disciplines:

1. **Unit Testing** -- The foundation of all testing; tests are fast, focused, and numerous.
2. **Integration Testing** -- The next level up; verifies that units work together.
3. **System Testing** -- Full system validation; builds on integration testing concepts.
4. **User Acceptance Testing** -- Business-level validation; completes the testing pyramid.
5. **Test-Driven Development** -- A discipline that changes how you write code, not just tests.
6. **Debugging** -- The skill you need when tests reveal failures that need investigation.

Alternatively, experienced developers may start with TDD (topic 5) to adopt a test-first
mindset before studying the individual test levels.

## Prerequisites

- [Section 1: Core Programming](../01_core_programming/) -- especially functions and error handling.
- [Section 4: Software Design](../04_software_design/) -- design patterns and principles
  help you write testable code.
- Basic familiarity with building multi-component systems is helpful for integration and
  system testing topics.

## What You Will Learn

After completing this section, you will be able to:
- Write effective unit tests using established patterns and test doubles
- Design integration test strategies for multi-component systems
- Plan and execute system-level and acceptance-level test activities
- Apply test-driven development to guide implementation through tests
- Systematically debug defects using structured approaches
- Understand the testing pyramid and how each level contributes to quality
