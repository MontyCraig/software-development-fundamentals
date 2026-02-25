# DevOps

## Overview

DevOps is a set of practices that unifies software development and operations,
aiming to shorten the development lifecycle while delivering high-quality
software continuously. This section covers the foundational concepts, patterns,
and disciplines that make modern software delivery reliable, repeatable, and
observable.

The material here is **language-agnostic and tool-agnostic**. Principles are
presented through universal pseudocode, ASCII diagrams, and conceptual models
that apply regardless of the specific technologies you choose.

## Prerequisites

Before diving into this section, you should be comfortable with:

- Version control fundamentals (branching, merging, pull requests)
- Basic networking concepts (ports, protocols, DNS)
- Command-line proficiency and scripting basics
- Software testing principles (unit, integration, end-to-end)
- General understanding of operating system processes and file systems

## Table of Contents

| # | File | Description |
|---|------|-------------|
| 1 | [CI/CD Pipelines](ci_cd_pipelines.md) | Build, test, and deploy automation. Covers pipeline stages, branch strategies, deployment patterns (blue-green, canary, rolling), artifact management, rollback procedures, and pipeline security. |
| 2 | [Containerization](containerization.md) | Lightweight process isolation and orchestration. Covers containers versus virtual machines, image layering, container lifecycle, orchestration concepts, service mesh, health checks, resource management, and networking. |
| 3 | [Infrastructure as Code](infrastructure_as_code.md) | Managing infrastructure through declarative and imperative definitions. Covers state management, drift detection, dependency graphs, immutable infrastructure, environment promotion, module reuse, and testing strategies. |
| 4 | [Monitoring and Observability](monitoring_and_observability.md) | Understanding system behavior in production. Covers the three pillars (logs, metrics, traces), structured logging, metric types, distributed tracing, alerting strategies, SLI/SLO/SLA definitions, and dashboarding principles. |
| 5 | [Site Reliability Engineering](site_reliability_engineering.md) | Applying engineering discipline to operations. Covers error budgets, toil reduction, incident management, on-call practices, blameless postmortems, chaos engineering, capacity planning, and runbooks. |
| 6 | [Configuration Management](configuration_management.md) | Managing application and environment configuration safely. Covers environment-specific patterns, feature flags, secrets management, configuration validation, twelve-factor principles, and drift reconciliation. |

## Recommended Reading Order

```
┌─────────────────────────┐
│  1. CI/CD Pipelines     │──── Start here: the backbone of delivery
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│  2. Containerization    │──── How artifacts are packaged and run
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│  3. Infrastructure as   │──── How environments are provisioned
│     Code                │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│  4. Monitoring &        │──── How you see what is happening
│     Observability       │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│  5. Site Reliability    │──── How you keep it running reliably
│     Engineering         │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│  6. Configuration       │──── How you manage settings and secrets
│     Management          │
└─────────────────────────┘
```

Files 4, 5, and 6 can be read in any order once you have completed the first
three. However, the order above builds concepts progressively and is recommended
for newcomers to DevOps.
