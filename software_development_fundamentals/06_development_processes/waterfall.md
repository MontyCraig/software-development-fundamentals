# Waterfall Development Model

## Overview

The Waterfall model is a sequential, phase-based approach to software
development. Each phase must be completed before the next begins, and there
is minimal backtracking. Introduced by Winston Royce in 1970 (who, ironically,
described it as flawed), it remains relevant for certain types of projects.

---

## Sequential Phases

```
+------------------------------------------------------------------+
|                    WATERFALL MODEL                                |
+------------------------------------------------------------------+
|                                                                  |
|  +------------------+                                            |
|  |  1. REQUIREMENTS |                                            |
|  +--------+---------+                                            |
|           |                                                      |
|           v                                                      |
|  +------------------+                                            |
|  |  2. DESIGN       |                                            |
|  +--------+---------+                                            |
|           |                                                      |
|           v                                                      |
|  +------------------+                                            |
|  |  3. IMPLEMENT    |                                            |
|  +--------+---------+                                            |
|           |                                                      |
|           v                                                      |
|  +------------------+                                            |
|  |  4. TESTING      |                                            |
|  +--------+---------+                                            |
|           |                                                      |
|           v                                                      |
|  +------------------+                                            |
|  |  5. DEPLOYMENT   |                                            |
|  +--------+---------+                                            |
|           |                                                      |
|           v                                                      |
|  +------------------+                                            |
|  |  6. MAINTENANCE  |                                            |
|  +------------------+                                            |
|                                                                  |
|  Flow: strictly downward (like a waterfall)                      |
+------------------------------------------------------------------+
```

---

## Phase Details

### Phase 1: Requirements

All requirements are gathered, documented, and frozen before design begins.

```
Deliverable: Software Requirements Specification (SRS)

Contents:
  +---+--------------------------------------------+
  | 1 | Functional requirements                    |
  | 2 | Non-functional requirements (performance,  |
  |   | security, scalability)                     |
  | 3 | System constraints                         |
  | 4 | User interface specifications              |
  | 5 | Data requirements                          |
  | 6 | Acceptance criteria                        |
  +---+--------------------------------------------+

  Sign-off required from all stakeholders before proceeding.
```

### Phase 2: Design

Translate requirements into system architecture and detailed design.

```
Deliverables:
  - High-Level Design (HLD): System architecture, module decomposition
  - Low-Level Design (LLD): Module internals, data structures, algorithms

Architecture Diagram Example:

  +-------------------+
  | Presentation Layer|
  +--------+----------+
           |
  +--------+----------+
  | Business Logic    |
  +--------+----------+
           |
  +--------+----------+
  | Data Access Layer |
  +--------+----------+
           |
  +--------+----------+
  | Database          |
  +-------------------+
```

### Phase 3: Implementation

Developers write code according to the design documents. No design changes
are allowed without a formal change request process.

```
Activities:
  - Code modules according to Low-Level Design
  - Conduct code reviews against design specs
  - Perform unit testing for individual modules
  - Document code according to standards
  - Track progress against implementation plan
```

### Phase 4: Testing

Dedicated testing phase after all implementation is complete. Testers verify
the system against the requirements specification.

```
Testing Levels:
  +---+------------------+-----------------------------------------+
  | 1 | Unit Testing     | Individual modules (often during impl.) |
  | 2 | Integration Test | Modules working together                |
  | 3 | System Testing   | Complete system against requirements    |
  | 4 | UAT              | Customer validates acceptance criteria  |
  +---+------------------+-----------------------------------------+

  Defects are logged, prioritized, fixed, and retested.
  All critical and high-severity defects must be resolved.
```

### Phase 5: Deployment

The completed, tested system is released to the production environment.

```
Activities:
  - Environment preparation
  - Data migration
  - User training
  - System installation
  - Go-live support
  - Post-deployment verification
```

### Phase 6: Maintenance

Ongoing support after deployment: bug fixes, patches, enhancements.

```
Maintenance Types:
  +---+-------------------+----------------------------------------+
  | 1 | Corrective        | Fixing defects found in production     |
  | 2 | Adaptive          | Adapting to environment changes        |
  | 3 | Perfective        | Enhancing functionality or performance |
  | 4 | Preventive        | Improving maintainability              |
  +---+-------------------+----------------------------------------+
```

---

## Documentation Requirements

Waterfall is documentation-heavy by design. Each phase produces formal
deliverables that serve as the contract for the next phase.

```
Phase-to-Document Mapping:

  Requirements  --> Software Requirements Specification (SRS)
  Design        --> High-Level Design (HLD)
                --> Low-Level Design (LLD)
                --> Database Design Document
  Implementation --> Code + Code Review Reports
  Testing       --> Test Plan
                --> Test Cases
                --> Test Results Report
                --> Defect Report
  Deployment    --> Deployment Plan
                --> User Manual
                --> Training Materials
  Maintenance   --> Change Request Log
                --> Release Notes
```

---

## V-Model (Verification and Validation)

The V-Model extends Waterfall by explicitly pairing each development phase
with a corresponding testing phase.

```
  Requirements --------+                  +-------- Acceptance Testing
       |               |                  |               |
       v               |   Validation     |               ^
  System Design -------+--+           +---+------ System Testing
       |                  |           |                   |
       v                  |           |                   ^
  Module Design ----------+-----------+-------Integration Testing
       |                                                  |
       v                                                  ^
  Implementation ---- (code) ----> Unit Testing ----------+

  Left side: Development (progressive detail)
  Right side: Testing (progressive integration)
  Bottom: Implementation is the pivot point
```

Each design phase on the left maps directly to a testing phase on the right:

```
+--------------------+----------------------+
| Development Phase  | Corresponding Test   |
+--------------------+----------------------+
| Requirements       | Acceptance Testing   |
| System Design      | System Testing       |
| Module Design      | Integration Testing  |
| Implementation     | Unit Testing         |
+--------------------+----------------------+
```

---

## When Waterfall Works

Waterfall is appropriate for specific project types:

```
+----+-----------------------------------------------+
| 1  | Fixed, well-understood requirements            |
|    | (e.g., regulatory systems, standards)          |
+----+-----------------------------------------------+
| 2  | Regulatory or compliance requirements          |
|    | demand full documentation and traceability     |
+----+-----------------------------------------------+
| 3  | Safety-critical systems where formal           |
|    | verification is mandatory                      |
+----+-----------------------------------------------+
| 4  | Contract-based development with fixed          |
|    | deliverables and sign-offs                     |
+----+-----------------------------------------------+
| 5  | Hardware-integrated systems where changes      |
|    | after manufacturing are extremely expensive    |
+----+-----------------------------------------------+
| 6  | Small, well-defined projects with              |
|    | experienced teams                              |
+----+-----------------------------------------------+
```

### When Waterfall Fails

```
+----+-----------------------------------------------+
| 1  | Requirements are uncertain or evolving         |
| 2  | Customer feedback is needed during development |
| 3  | Time-to-market pressure requires early releases|
| 4  | The technology is new and requires exploration |
| 5  | The project is long (requirements drift)       |
+----+-----------------------------------------------+
```

---

## Waterfall vs Agile Comparison

```
+---------------------+-----------------------+-----------------------+
| Aspect              | Waterfall             | Agile                 |
+---------------------+-----------------------+-----------------------+
| Requirements        | Fixed at start        | Evolving              |
| Delivery            | Single final delivery | Incremental releases  |
| Customer involvement| Beginning and end     | Continuous            |
| Change tolerance    | Low (costly)          | High (expected)       |
| Risk discovery      | Late (testing phase)  | Early (each sprint)   |
| Documentation       | Heavy, formal         | Lightweight, living   |
| Team structure      | Specialized phases    | Cross-functional      |
| Planning            | Upfront, detailed     | Iterative, adaptive   |
| Testing             | After implementation  | Continuous            |
| Feedback loop       | Long (months)         | Short (weeks)         |
+---------------------+-----------------------+-----------------------+
```

---

## Hybrid Approaches

In practice, many organizations combine elements of both models.

### Water-Scrum-Fall

```
  +------------------+     +------------------+     +------------------+
  | WATERFALL        |     | AGILE            |     | WATERFALL        |
  | Requirements     | --> | Iterative        | --> | Testing &        |
  | & Design         |     | Development      |     | Deployment       |
  | (upfront)        |     | (sprints)        |     | (formal)         |
  +------------------+     +------------------+     +------------------+
```

### Spiral Model

Combines iterative development with Waterfall's systematic approach. Each
cycle includes: planning, risk analysis, engineering, and evaluation.

```
          Planning
             |
    +--------+---------+
    |                  |
  Evaluation    Risk Analysis
    |                  |
    +--------+---------+
             |
         Engineering

  Repeated in expanding spirals, each adding more functionality.
```

### Rational Unified Process (RUP)

Iterative, but with defined phases (Inception, Elaboration, Construction,
Transition) and formal deliverables.

---

## Waterfall Risk Profile

```
  Risk / Cost of Change
  ^
  |                                    *
  |                               *
  |                          *
  |                     *
  |                *
  |           *
  |      *
  |  *
  +--+----+----+----+----+----+----->
     Req  Des  Impl Test Depl Maint
                 Phase

  Key insight: The cost of discovering a requirements error
  in the testing phase is 10-100x more expensive than
  discovering it during the requirements phase.

  This is Waterfall's biggest risk: late feedback.
```

---

## Summary

```
+----------------------------------------------------------+
| Waterfall Checklist                                      |
+----------------------------------------------------------+
| [ ] Requirements are stable and well-documented          |
| [ ] Stakeholders agree on scope before development       |
| [ ] Each phase has clear entry/exit criteria             |
| [ ] Formal reviews at each phase gate                    |
| [ ] Change control process is defined                    |
| [ ] Testing strategy maps to V-Model                     |
| [ ] Documentation is maintained throughout               |
| [ ] Risk of late-discovered issues is acknowledged       |
+----------------------------------------------------------+
```

Waterfall is neither obsolete nor universally applicable. Understanding its
strengths and limitations allows teams to choose it when appropriate and avoid
it when flexibility and early feedback are paramount.
