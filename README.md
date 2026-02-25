# Software Development Fundamentals

A comprehensive, **language-agnostic** reference covering the full breadth of software
engineering knowledge — from first principles through distributed systems, DevOps, and
professional practices.

Every concept is explained with **pseudocode** that maps to any language, illustrated
with **ASCII diagrams**, and grounded in **complexity analysis** where applicable.

---

## At a Glance

| | |
|---|---|
| **Topic files** | 77 in-depth references |
| **Sections** | 10, ordered for progressive learning |
| **Architectural patterns** | 18 fully documented (layered through event sourcing) |
| **GoF design patterns** | All 23 — creational, structural, behavioral |
| **Refactoring techniques** | 20+ with before/after pseudocode |
| **Pseudocode examples** | 120+ implementations |
| **ASCII diagrams** | 80+ structural and execution-trace diagrams |
| **Total content** | 34,500+ lines |
| **Language dependencies** | None |

---

## Sections

### [1 — Core Programming](software_development_fundamentals/01_core_programming/)

Variables and types, control flow, functions and scope, error handling, object-oriented
programming, functional programming.

### [2 — Data Structures](software_development_fundamentals/02_data_structures/)

Arrays, linked lists, stacks, queues, hash tables, trees (BST, AVL), heaps, and graphs
— with pseudocode implementations and Big-O analysis for every operation.

### [3 — Algorithms](software_development_fundamentals/03_algorithms/)

Complexity analysis, searching, comparison and non-comparison sorting, recursion and
backtracking, dynamic programming, greedy algorithms, graph algorithms, string algorithms
(KMP, Rabin-Karp, tries), and bit manipulation.

### [4 — Software Design](software_development_fundamentals/04_software_design/)

SOLID principles, DRY, KISS, YAGNI, all 23 Gang of Four design patterns, design quality
metrics (cohesion, coupling, code smells), and a 20+ entry refactoring catalog.

### [5 — Architectural Patterns](software_development_fundamentals/05_architectural_patterns/)

Eighteen patterns from monolithic through distributed: layered, client-server, microservices,
event-driven, hexagonal, clean architecture, DDD, MVC, MVVM, pipe-and-filter, serverless,
space-based, peer-to-peer, broker, CQRS, and event sourcing.

### [6 — Development Processes](software_development_fundamentals/06_development_processes/)

Waterfall, Agile (Scrum, Kanban, user stories, velocity), and version control (branching
strategies, collaborative workflows, merge vs. rebase).

### [7 — Testing and Quality Assurance](software_development_fundamentals/07_testing_and_qa/)

Unit testing (Arrange-Act-Assert, test doubles), integration testing, system testing,
user acceptance testing, test-driven development (Red-Green-Refactor), and systematic
debugging strategies.

### [8 — Advanced Topics](software_development_fundamentals/08_advanced_topics/)

Memory management, concurrency and parallelism (threads, locks, actor model), database
fundamentals (relational, NoSQL, ACID, CAP), networking (OSI model, TCP/UDP, DNS, TLS),
API design, security (OWASP Top Ten, cryptography), system design (scaling, caching,
sharding), and distributed systems (Paxos, Raft, 2PC, Saga, CRDTs, vector clocks).

### [9 — DevOps](software_development_fundamentals/09_devops/)

CI/CD pipelines and deployment strategies (blue-green, canary, rolling), containerization
and orchestration, infrastructure as code, monitoring and observability (logs, metrics,
traces, SLI/SLO/SLA), site reliability engineering (error budgets, incident management,
chaos engineering), and configuration management (feature flags, secrets, twelve-factor).

### [10 — Professional Practices](software_development_fundamentals/10_professional_practices/)

Code review processes and feedback, technical debt management (debt quadrant, pay-down
planning), estimation and planning (story points, PERT, cone of uncertainty), technical
documentation (Diataxis framework, ADRs, docs-as-code), and communication and collaboration
(written communication, blameless culture, stakeholder management).

---

## How to Use This Reference

**Learning from scratch** — Work through the sections in order. Each builds on the previous:

```
Core Programming → Data Structures → Algorithms → Design → Architecture
    → Processes → Testing → Advanced Topics → DevOps → Professional Practices
```

**Quick reference** — Jump to any topic directly. Every file is self-contained with
cross-references where concepts overlap.

**Interview preparation** — Focus on Sections 2–3 for coding interviews, Sections 5
and 8 for system design rounds, Section 4 for design discussions.

**Team onboarding** — Sections 4, 6, 7, 9, and 10 cover the practices and standards
that align development teams.

---

## Conventions

All content follows these rules:

- **Pseudocode only** — no language-specific syntax. The pseudocode conventions
  (`FUNCTION`, `SET`, `IF/THEN/END IF`, `FOR EACH/DO/END FOR`, `RETURN`) are documented
  in [CONTRIBUTING.md](CONTRIBUTING.md).
- **ASCII diagrams** — all visuals are plain text so they render anywhere without
  external tools.
- **Complexity analysis** — every data structure and algorithm includes Big-O for time
  and space.
- **Cross-references** — related topics in other sections are linked where relevant.

---

## Repository Structure

```
software-development-fundamentals/
├── README.md                              ← You are here
├── CONTRIBUTING.md                        ← Content standards and pseudocode conventions
├── LICENSE                                ← MIT License
└── software_development_fundamentals/
    ├── README.md                          ← Detailed table of contents
    ├── 01_core_programming/          (6 topic files)
    ├── 02_data_structures/           (8 topic files)
    ├── 03_algorithms/               (10 topic files)
    ├── 04_software_design/           (7 topic files)
    ├── 05_architectural_patterns/   (18 pattern files)
    ├── 06_development_processes/     (3 topic files)
    ├── 07_testing_and_qa/            (6 topic files)
    ├── 08_advanced_topics/           (8 topic files)
    ├── 09_devops/                    (6 topic files)
    └── 10_professional_practices/    (5 topic files)
```

---

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for content standards,
pseudocode conventions, and the pull request workflow.

Key rules:

1. **Language-agnostic only** — use pseudocode, never language-specific syntax.
2. **One concept per file** — keep topics focused at 300–600 lines.
3. **Include diagrams** — ASCII art, no external image dependencies.
4. **Show complexity** — Big-O for all data structures and algorithms.
5. **Cross-reference** — link to related topics across sections.

---

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
