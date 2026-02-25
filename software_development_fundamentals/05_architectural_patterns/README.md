# 05 - Architectural Patterns

## Overview

Architectural patterns are reusable solutions to commonly occurring problems in
software architecture. They define the fundamental structural organization of a
system, specifying the roles and relationships of its components, the rules
governing their interactions, and the constraints on their behavior.

Choosing the right architectural pattern is one of the most consequential
decisions in a software project. It shapes how teams organize code, how
systems scale, how faults propagate, and how easily the system can evolve
over time. No single pattern is universally superior; each embodies a set
of trade-offs suited to specific contexts, team sizes, and business needs.

This section covers 18 foundational architectural patterns organized into
four categories: structural, distributed, domain-centric, and specialized.

---

## Pattern Catalog

### Structural Patterns

| # | Pattern | Summary |
|---|---------|---------|
| 01 | [Layered Architecture](01_layered_architecture.md) | Horizontal layers with strict dependency rules |
| 02 | [Client-Server](02_client_server.md) | Request/response between consumers and providers |
| 03 | [Monolithic Architecture](03_monolithic_architecture.md) | Single deployable unit with internal modularity |
| 10 | [Model-View-Controller](10_mvc.md) | Separation of data, presentation, and control flow |
| 11 | [Model-View-ViewModel](11_mvvm.md) | Data binding between views and presentation logic |
| 12 | [Pipe and Filter](12_pipe_and_filter.md) | Sequential data transformation through stages |

### Distributed Patterns

| # | Pattern | Summary |
|---|---------|---------|
| 04 | [Microservices](04_microservices.md) | Independently deployable services with bounded scope |
| 05 | [Event-Driven](05_event_driven.md) | Asynchronous communication through events |
| 06 | [Service-Oriented](06_service_oriented.md) | Enterprise service bus with formal contracts |
| 13 | [Serverless](13_serverless.md) | Function-as-a-Service with event triggers |
| 14 | [Space-Based](14_space_based.md) | In-memory data grids with processing units |
| 15 | [Peer-to-Peer](15_peer_to_peer.md) | Decentralized nodes with equal capabilities |
| 16 | [Broker](16_broker.md) | Intermediary for service discovery and routing |

### Domain-Centric Patterns

| # | Pattern | Summary |
|---|---------|---------|
| 07 | [Hexagonal Architecture](07_hexagonal_architecture.md) | Ports and adapters with dependency inversion |
| 08 | [Clean Architecture](08_clean_architecture.md) | Concentric dependency circles around use cases |
| 09 | [Domain-Driven Design](09_domain_driven_design.md) | Bounded contexts with rich domain models |

### Specialized Patterns

| # | Pattern | Summary |
|---|---------|---------|
| 17 | [CQRS](17_cqrs.md) | Separate models for reading and writing data |
| 18 | [Event Sourcing](18_event_sourcing.md) | Persisting state as an append-only event log |

---

## Comparison Matrix

### Scalability

| Pattern | Horizontal | Vertical | Data Partitioning | Rating |
|---------|-----------|----------|-------------------|--------|
| Layered | Low | Medium | Low | ** |
| Client-Server | Medium | Medium | Low | ** |
| Monolithic | Low | Medium | Low | * |
| Microservices | High | High | High | ***** |
| Event-Driven | High | Medium | High | **** |
| Service-Oriented | Medium | Medium | Medium | *** |
| Hexagonal | Low | Medium | Low | ** |
| Clean | Low | Medium | Low | ** |
| DDD | Medium | Medium | Medium | *** |
| MVC | Low | Medium | Low | ** |
| MVVM | Low | Medium | Low | ** |
| Pipe and Filter | High | Medium | High | **** |
| Serverless | High | N/A | High | ***** |
| Space-Based | High | High | High | ***** |
| Peer-to-Peer | High | Low | High | **** |
| Broker | High | Medium | Medium | **** |
| CQRS | High | High | High | ***** |
| Event Sourcing | High | Medium | High | **** |

### Complexity

| Pattern | Learning Curve | Implementation | Operations | Rating |
|---------|---------------|----------------|------------|--------|
| Layered | Low | Low | Low | * |
| Client-Server | Low | Low | Low | * |
| Monolithic | Low | Low | Low | * |
| Microservices | High | High | High | ***** |
| Event-Driven | Medium | Medium | High | **** |
| Service-Oriented | High | High | High | ***** |
| Hexagonal | Medium | Medium | Low | *** |
| Clean | Medium | Medium | Low | *** |
| DDD | High | High | Medium | **** |
| MVC | Low | Low | Low | * |
| MVVM | Medium | Medium | Low | ** |
| Pipe and Filter | Low | Low | Low | * |
| Serverless | Medium | Medium | Medium | *** |
| Space-Based | High | High | High | ***** |
| Peer-to-Peer | High | High | High | ***** |
| Broker | Medium | Medium | Medium | *** |
| CQRS | Medium | High | Medium | **** |
| Event Sourcing | High | High | High | ***** |

### Testability

| Pattern | Unit | Integration | End-to-End | Rating |
|---------|------|-------------|------------|--------|
| Layered | Medium | Medium | Medium | *** |
| Client-Server | Medium | Medium | Medium | *** |
| Monolithic | Medium | Low | Low | ** |
| Microservices | High | Low | Low | *** |
| Event-Driven | Medium | Low | Low | ** |
| Service-Oriented | Medium | Low | Low | ** |
| Hexagonal | High | High | Medium | ***** |
| Clean | High | High | Medium | ***** |
| DDD | High | Medium | Medium | **** |
| MVC | High | Medium | Medium | **** |
| MVVM | High | Medium | Medium | **** |
| Pipe and Filter | High | High | Medium | ***** |
| Serverless | Medium | Low | Low | ** |
| Space-Based | Low | Low | Low | * |
| Peer-to-Peer | Low | Low | Low | * |
| Broker | Medium | Medium | Medium | *** |
| CQRS | High | Medium | Low | *** |
| Event Sourcing | High | Medium | Low | *** |

### Deployment

| Pattern | Ease | Independence | Rollback | Rating |
|---------|------|-------------|----------|--------|
| Layered | High | Low | Medium | ** |
| Client-Server | High | Medium | Medium | *** |
| Monolithic | High | Low | Medium | ** |
| Microservices | Low | High | High | **** |
| Event-Driven | Medium | High | Medium | **** |
| Service-Oriented | Low | Medium | Medium | ** |
| Hexagonal | High | Low | Medium | ** |
| Clean | High | Low | Medium | ** |
| DDD | Medium | Medium | Medium | *** |
| MVC | High | Low | Medium | ** |
| MVVM | High | Low | Medium | ** |
| Pipe and Filter | Medium | High | High | **** |
| Serverless | High | High | High | ***** |
| Space-Based | Low | Medium | Medium | ** |
| Peer-to-Peer | Medium | High | Medium | *** |
| Broker | Medium | Medium | Medium | *** |
| CQRS | Medium | Medium | Medium | *** |
| Event Sourcing | Medium | Medium | High | **** |

### Team Size Suitability

| Pattern | Solo/Small (1-5) | Medium (5-20) | Large (20+) |
|---------|------------------|---------------|-------------|
| Layered | Excellent | Good | Fair |
| Client-Server | Excellent | Good | Fair |
| Monolithic | Excellent | Good | Poor |
| Microservices | Poor | Good | Excellent |
| Event-Driven | Fair | Good | Excellent |
| Service-Oriented | Poor | Fair | Excellent |
| Hexagonal | Good | Excellent | Good |
| Clean | Good | Excellent | Good |
| DDD | Fair | Good | Excellent |
| MVC | Excellent | Good | Fair |
| MVVM | Excellent | Good | Fair |
| Pipe and Filter | Excellent | Good | Good |
| Serverless | Good | Excellent | Good |
| Space-Based | Poor | Fair | Good |
| Peer-to-Peer | Poor | Fair | Good |
| Broker | Fair | Good | Excellent |
| CQRS | Fair | Good | Excellent |
| Event Sourcing | Fair | Good | Excellent |

---

## How to Use This Section

1. **Start with the comparison matrix** above to narrow your choices based
   on your constraints (team size, scale requirements, complexity budget).
2. **Read the pattern files** for your top two or three candidates. Focus
   on the "When to Use" and "When NOT to Use" sections.
3. **Study the trade-offs** tables in each pattern file to understand what
   you gain and what you give up.
4. **Combine patterns** where appropriate. Many real systems use multiple
   patterns at different levels of abstraction (for example, microservices
   with hexagonal architecture inside each service, and CQRS for the
   data layer).
5. **Revisit decisions** as your system evolves. The right architecture for
   a prototype is rarely the right architecture at scale.

---

## Prerequisites

Before studying these patterns, you should be comfortable with:

- Basic data structures and algorithms (Section 01)
- Object-oriented and functional design principles (Section 02)
- Design patterns at the class/object level (Section 04)
- Network fundamentals: request/response, messaging, serialization

---

## Further Reading

- "Pattern-Oriented Software Architecture" (Buschmann et al.)
- "Software Architecture in Practice" (Bass, Clements, Kazman)
- "Fundamentals of Software Architecture" (Richards, Ford)
- "Building Evolutionary Architectures" (Ford, Parsons, Kua)
- "Designing Data-Intensive Applications" (Kleppmann)
