# Software Development Fundamentals

A comprehensive, language-agnostic reference covering the core knowledge every software
developer needs -- from basic programming concepts through data structures, algorithms,
design principles, architectural patterns, development processes, testing, and advanced
systems topics.

## Overview

This reference is designed for developers at any stage of their career. Whether you are
learning to program for the first time, preparing for technical interviews, or looking
for a structured refresher on fundamentals, this material provides clear explanations,
pseudocode implementations, and visual diagrams for every major topic in software
development.

**What makes this reference different:**

- **Language-agnostic.** All examples use pseudocode that maps cleanly to any programming
  language. There is no dependency on any specific runtime, framework, or toolchain.
- **Visual.** Over 60 ASCII diagrams illustrate data structure layouts, algorithm
  execution traces, architectural topologies, and system interactions.
- **Comprehensive.** 73 topic files organized across 10 sections cover the full breadth
  of software development fundamentals.
- **Self-contained.** Each file can be read independently, but the sections are ordered
  to build on each other for a structured learning path.

---

## Key Features

| Feature                        | Details                                             |
| ------------------------------ | --------------------------------------------------- |
| Topic files                    | 73 in-depth references                              |
| Sections                       | 10 organized by domain                              |
| Architectural patterns         | 18 fully documented patterns                        |
| Gang of Four design patterns   | 23 patterns cataloged                               |
| Pseudocode examples            | 100+ implementations across all topics              |
| ASCII diagrams                 | 60+ structural and trace diagrams                   |
| Total content                  | 34,500+ lines of technical reference material       |
| Language dependencies          | None -- entirely language-agnostic                  |

---

## Table of Contents

### [Section 1: Core Programming](01_core_programming/)

The foundational building blocks of programming: how data is represented, how execution
flows, how code is organized into functions, and how errors are handled. Covers both
object-oriented and functional paradigms.

| # | File | Topic |
|---|------|-------|
| 1 | [variables_and_data_types.md](01_core_programming/variables_and_data_types.md) | Primitive and composite types, type systems, type conversion |
| 2 | [control_flow.md](01_core_programming/control_flow.md) | Conditionals, loops, branching, short-circuit evaluation |
| 3 | [functions_and_scope.md](01_core_programming/functions_and_scope.md) | Parameters, return values, scope rules, closures, call stack |
| 4 | [error_handling.md](01_core_programming/error_handling.md) | Exceptions, return codes, defensive programming, error propagation |
| 5 | [object_oriented_programming.md](01_core_programming/object_oriented_programming.md) | Classes, inheritance, polymorphism, encapsulation, composition |
| 6 | [functional_programming.md](01_core_programming/functional_programming.md) | Pure functions, immutability, higher-order functions, map/filter/reduce |

### [Section 2: Data Structures](02_data_structures/)

The essential data organization schemes, from linear collections through trees and graphs.
Each structure includes pseudocode implementations and complexity analysis.

| # | File | Topic |
|---|------|-------|
| 1 | [arrays.md](02_data_structures/arrays.md) | Contiguous storage, dynamic resizing, multidimensional arrays |
| 2 | [linked_lists.md](02_data_structures/linked_lists.md) | Singly/doubly linked, circular lists, sentinel nodes |
| 3 | [stacks.md](02_data_structures/stacks.md) | LIFO operations, array and linked-list implementations |
| 4 | [queues.md](02_data_structures/queues.md) | FIFO operations, circular queues, deques, priority queues |
| 5 | [hash_tables.md](02_data_structures/hash_tables.md) | Hash functions, collision resolution, load factors |
| 6 | [trees.md](02_data_structures/trees.md) | Binary trees, BSTs, AVL trees, traversal algorithms |
| 7 | [heaps.md](02_data_structures/heaps.md) | Min/max heaps, heapify, heap sort, priority queue implementation |
| 8 | [graphs.md](02_data_structures/graphs.md) | Adjacency matrix/list, directed/undirected, weighted graphs |

### [Section 3: Algorithms](03_algorithms/)

The major algorithmic paradigms and techniques, with formal complexity analysis, pseudocode
implementations, and step-by-step execution traces.

| # | File | Topic |
|---|------|-------|
| 1 | [complexity_analysis.md](03_algorithms/complexity_analysis.md) | Big-O/Omega/Theta, amortized analysis, space complexity |
| 2 | [searching_algorithms.md](03_algorithms/searching_algorithms.md) | Linear search, binary search, interpolation search |
| 3 | [comparison_sorting.md](03_algorithms/comparison_sorting.md) | Bubble, selection, insertion, merge, quick, heap sort |
| 4 | [non_comparison_sorting.md](03_algorithms/non_comparison_sorting.md) | Counting sort, radix sort, bucket sort |
| 5 | [recursion_and_backtracking.md](03_algorithms/recursion_and_backtracking.md) | Recursive thinking, memoization, N-Queens, pruning |
| 6 | [dynamic_programming.md](03_algorithms/dynamic_programming.md) | Memoization, tabulation, state transitions, space optimization |
| 7 | [greedy_algorithms.md](03_algorithms/greedy_algorithms.md) | Greedy choice property, Huffman coding, activity selection |
| 8 | [graph_algorithms.md](03_algorithms/graph_algorithms.md) | BFS, DFS, Dijkstra, Bellman-Ford, Kruskal, Prim, topological sort |
| 9 | [string_algorithms.md](03_algorithms/string_algorithms.md) | Pattern matching (KMP, Rabin-Karp), trie operations, edit distance, suffix arrays |
| 10 | [bit_manipulation.md](03_algorithms/bit_manipulation.md) | Binary representation, bitwise operators, common bit tricks, bit masking |

### [Section 4: Software Design](04_software_design/)

Design principles, patterns, and quality metrics that determine how maintainable and
extensible a codebase will be over its lifetime.

| # | File | Topic |
|---|------|-------|
| 1 | [solid_principles.md](04_software_design/solid_principles.md) | SRP, OCP, LSP, ISP, DIP with violation/correction examples |
| 2 | [dry.md](04_software_design/dry.md) | Eliminating knowledge duplication across abstractions |
| 3 | [kiss.md](04_software_design/kiss.md) | Designing for simplicity without sacrificing capability |
| 4 | [yagni.md](04_software_design/yagni.md) | Avoiding speculative generality and premature abstraction |
| 5 | [design_patterns.md](04_software_design/design_patterns.md) | All 23 Gang of Four patterns: Creational, Structural, Behavioral |
| 6 | [design_quality.md](04_software_design/design_quality.md) | Cohesion, coupling, code smells, refactoring techniques |
| 7 | [refactoring_catalog.md](04_software_design/refactoring_catalog.md) | 20+ refactoring techniques with before/after pseudocode by category |

### [Section 5: Architectural Patterns](05_architectural_patterns/)

Eighteen software architecture patterns covering monolithic through distributed systems,
with structure diagrams, trade-off analysis, and implementation guidance.

| #  | File | Pattern |
|----|------|---------|
| 1  | [01_layered_architecture.md](05_architectural_patterns/01_layered_architecture.md) | Presentation/Business/Data layering |
| 2  | [02_client_server.md](05_architectural_patterns/02_client_server.md) | Client-server communication model |
| 3  | [03_monolithic_architecture.md](05_architectural_patterns/03_monolithic_architecture.md) | Single-deployment-unit architecture |
| 4  | [04_microservices.md](05_architectural_patterns/04_microservices.md) | Independently deployable service decomposition |
| 5  | [05_event_driven.md](05_architectural_patterns/05_event_driven.md) | Event producers, consumers, and channels |
| 6  | [06_service_oriented.md](05_architectural_patterns/06_service_oriented.md) | Service contracts and enterprise service bus |
| 7  | [07_hexagonal_architecture.md](05_architectural_patterns/07_hexagonal_architecture.md) | Ports and adapters for testable domain logic |
| 8  | [08_clean_architecture.md](05_architectural_patterns/08_clean_architecture.md) | Dependency rule and concentric layers |
| 9  | [09_domain_driven_design.md](05_architectural_patterns/09_domain_driven_design.md) | Bounded contexts, aggregates, ubiquitous language |
| 10 | [10_mvc.md](05_architectural_patterns/10_mvc.md) | Model-View-Controller separation |
| 11 | [11_mvvm.md](05_architectural_patterns/11_mvvm.md) | Model-View-ViewModel with data binding |
| 12 | [12_pipe_and_filter.md](05_architectural_patterns/12_pipe_and_filter.md) | Data transformation pipelines |
| 13 | [13_serverless.md](05_architectural_patterns/13_serverless.md) | Function-as-a-Service and managed infrastructure |
| 14 | [14_space_based.md](05_architectural_patterns/14_space_based.md) | In-memory data grids and processing units |
| 15 | [15_peer_to_peer.md](05_architectural_patterns/15_peer_to_peer.md) | Decentralized node networks |
| 16 | [16_broker.md](05_architectural_patterns/16_broker.md) | Message broker mediated communication |
| 17 | [17_cqrs.md](05_architectural_patterns/17_cqrs.md) | Command Query Responsibility Segregation |
| 18 | [18_event_sourcing.md](05_architectural_patterns/18_event_sourcing.md) | Immutable event log as source of truth |

### [Section 6: Development Processes](06_development_processes/)

Organizational frameworks for planning, building, and delivering software, plus the
version control practices that support them.

| # | File | Topic |
|---|------|-------|
| 1 | [waterfall.md](06_development_processes/waterfall.md) | Sequential phase-based development model |
| 2 | [agile.md](06_development_processes/agile.md) | Iterative development, Scrum, Kanban, user stories |
| 3 | [version_control.md](06_development_processes/version_control.md) | Repositories, branching, merging, collaborative workflows |

### [Section 7: Testing and Quality Assurance](07_testing_and_qa/)

The full testing pyramid from unit tests through acceptance testing, plus test-driven
development and systematic debugging techniques.

| # | File | Topic |
|---|------|-------|
| 1 | [unit_testing.md](07_testing_and_qa/unit_testing.md) | Arrange-Act-Assert, test doubles, boundary analysis |
| 2 | [integration_testing.md](07_testing_and_qa/integration_testing.md) | Top-down, bottom-up, sandwich integration strategies |
| 3 | [system_testing.md](07_testing_and_qa/system_testing.md) | Functional/non-functional testing, regression, smoke tests |
| 4 | [user_acceptance_testing.md](07_testing_and_qa/user_acceptance_testing.md) | UAT planning, acceptance criteria, alpha/beta testing |
| 5 | [test_driven_development.md](07_testing_and_qa/test_driven_development.md) | Red-Green-Refactor cycle, test-first design |
| 6 | [debugging.md](07_testing_and_qa/debugging.md) | Systematic debugging strategies, logging, root cause analysis |

### [Section 8: Advanced Topics](08_advanced_topics/)

Systems-level knowledge covering memory, concurrency, databases, networking, APIs,
security, and holistic system design.

| # | File | Topic |
|---|------|-------|
| 1 | [memory_management.md](08_advanced_topics/memory_management.md) | Stack vs. heap, garbage collection, memory leaks |
| 2 | [concurrency_and_parallelism.md](08_advanced_topics/concurrency_and_parallelism.md) | Threads, race conditions, mutexes, deadlock, actor model |
| 3 | [database_fundamentals.md](08_advanced_topics/database_fundamentals.md) | Relational/NoSQL, ACID, indexing, CAP theorem |
| 4 | [networking_fundamentals.md](08_advanced_topics/networking_fundamentals.md) | OSI model, TCP/UDP, HTTP, TLS, DNS, sockets |
| 5 | [api_design_principles.md](08_advanced_topics/api_design_principles.md) | REST design, versioning, pagination, rate limiting |
| 6 | [security_fundamentals.md](08_advanced_topics/security_fundamentals.md) | CIA triad, OWASP Top Ten, cryptography, secure coding |
| 7 | [system_design_basics.md](08_advanced_topics/system_design_basics.md) | Scaling, caching, sharding, load balancing, message queues |
| 8 | [distributed_systems.md](08_advanced_topics/distributed_systems.md) | Consensus algorithms (Paxos, Raft), distributed transactions, CRDTs, vector clocks |

### [Section 9: DevOps](09_devops/)

Practices and disciplines for reliable, repeatable, and observable software delivery,
from CI/CD pipelines through infrastructure management and operational reliability.

| # | File | Topic |
|---|------|-------|
| 1 | [ci_cd_pipelines.md](09_devops/ci_cd_pipelines.md) | Build, test, deploy automation, deployment patterns, rollback procedures |
| 2 | [containerization.md](09_devops/containerization.md) | Containers vs. VMs, image layering, orchestration, service mesh |
| 3 | [infrastructure_as_code.md](09_devops/infrastructure_as_code.md) | Declarative infrastructure, state management, drift detection, immutable infrastructure |
| 4 | [monitoring_and_observability.md](09_devops/monitoring_and_observability.md) | Logs, metrics, traces, alerting strategies, SLI/SLO/SLA definitions |
| 5 | [site_reliability_engineering.md](09_devops/site_reliability_engineering.md) | Error budgets, toil reduction, incident management, chaos engineering |
| 6 | [configuration_management.md](09_devops/configuration_management.md) | Feature flags, secrets management, twelve-factor principles, drift reconciliation |

### [Section 10: Professional Practices](10_professional_practices/)

The disciplines, habits, and interpersonal skills that distinguish high-performing
development teams, from code review through estimation, documentation, and collaboration.

| # | File | Topic |
|---|------|-------|
| 1 | [code_review.md](10_professional_practices/code_review.md) | Review processes, giving/receiving feedback, quality standards |
| 2 | [technical_debt.md](10_professional_practices/technical_debt.md) | Measuring, managing, and paying down accumulated shortcuts |
| 3 | [estimation_and_planning.md](10_professional_practices/estimation_and_planning.md) | Effort forecasting, uncertainty management, realistic commitments |
| 4 | [technical_documentation.md](10_professional_practices/technical_documentation.md) | READMEs, API references, architecture decision records |
| 5 | [communication_and_collaboration.md](10_professional_practices/communication_and_collaboration.md) | Clear writing, productive meetings, knowledge sharing, blameless culture |

---

## How to Use This Reference

### Structured learning path (beginners)

Work through the sections in order. Each section builds on the previous:

```
Section 1: Core Programming
    |
    v
Section 2: Data Structures
    |
    v
Section 3: Algorithms
    |
    v
Section 4: Software Design
    |
    v
Section 5: Architectural Patterns
    |
    v
Section 6: Development Processes
    |
    v
Section 7: Testing and QA
    |
    v
Section 8: Advanced Topics
    |
    v
Section 9: DevOps
    |
    v
Section 10: Professional Practices
```

### Topic-based lookup (experienced developers)

Each file is self-contained. Jump directly to any topic using the table of contents above.
Cross-references between files are noted where relevant.

### Interview preparation

Focus on Sections 2-3 (data structures and algorithms) for coding interviews, Sections 5
and 8 for system design interviews, and Section 4 for design discussions.

### Team onboarding

Sections 4, 6, 7, and 10 (design, process, testing, professional practices) are
particularly valuable for aligning new team members on practices and standards.

---

## Project Structure

```
software_development_fundamentals/
  README.md                          <-- You are here
  01_core_programming/               (6 files + README)
  02_data_structures/                (8 files + README)
  03_algorithms/                     (10 files + README)
  04_software_design/                (7 files + README)
  05_architectural_patterns/         (18 files + README)
  06_development_processes/          (3 files + README)
  07_testing_and_qa/                 (6 files + README)
  08_advanced_topics/                (8 files + README)
  09_devops/                         (6 files + README)
  10_professional_practices/         (5 files + README)
```

---

## Contributing

Contributions are welcome. Please follow these guidelines:

1. **Language-agnostic only.** Do not add examples tied to a specific programming language.
   Use pseudocode that any developer can read and translate.
2. **One concept per file.** Each file should cover a single, well-scoped topic.
3. **Include diagrams.** Use ASCII art for diagrams so they render in any text viewer
   without external dependencies.
4. **Show complexity.** For data structures and algorithms, always include Big-O analysis.
5. **Keep files focused.** Aim for 300-600 lines per topic file. If a file grows beyond
   that, consider splitting it into sub-topics.
6. **Cross-reference.** Link to related topics in other sections where appropriate.

To contribute:
- Fork the repository
- Create a feature branch
- Add or modify content following the conventions above
- Submit a pull request with a clear description of changes

---

## License

This project is licensed under the MIT License. See [LICENSE](../LICENSE) for details.

---

*Software Development Fundamentals -- a structured, language-agnostic reference for
building strong foundations in software engineering.*
