# Section 6: Development Processes

## Overview

Software development processes define the organizational frameworks that teams use to
plan, build, deliver, and maintain software. The choice of process profoundly affects
how work is structured, how quality is assured, how changes are managed, and how teams
collaborate.

This section covers the two dominant process families -- plan-driven (Waterfall) and
adaptive (Agile) -- along with version control, the foundational tool that enables all
modern development workflows. Understanding these processes is essential for working
effectively in any team environment.

---

## Table of Contents

### 1. [Waterfall Development Model](waterfall.md)
The classic sequential, phase-based approach to software development. Covers the five
phases (Requirements, Design, Implementation, Verification, Maintenance), phase gates
and deliverables, the rationale behind the model, its strengths in regulated and
well-understood domains, and its limitations when requirements are uncertain. Includes
ASCII phase-flow diagrams and comparison with iterative approaches.

### 2. [Agile Development](agile.md)
The adaptive, iterative approach to software development. Covers the Agile Manifesto's
four values and twelve principles, Scrum (roles, ceremonies, artifacts), Kanban (boards,
WIP limits, flow metrics), sprint planning, retrospectives, and user stories. Includes
ASCII diagrams of sprint cycles and Kanban boards. Contrasts Agile with Waterfall to
clarify when each approach is appropriate.

### 3. [Version Control](version_control.md)
The practice and tooling for tracking changes to source code over time. Covers the
concepts of repositories, commits, branches, merges, and tags. Explains centralized vs.
distributed version control models, branching strategies (feature branches, release
branches, trunk-based development), merge conflict resolution, and collaborative
workflows (pull requests, code review). Includes ASCII diagrams of branch topologies
and merge scenarios.

---

## Recommended Reading Order

These three topics are relatively independent but benefit from this sequence:

1. **Waterfall** -- Understanding the plan-driven approach first provides context for
   why Agile emerged as a response to its limitations.
2. **Agile** -- The dominant modern methodology; contrasts with Waterfall illuminate
   the trade-offs each approach makes.
3. **Version Control** -- The tool that makes both Waterfall and Agile workflows
   practical; best understood after grasping why teams need structured collaboration.

Alternatively, if you are already working in a team, start with Version Control for
immediate practical benefit, then explore the process models.

## Prerequisites

- [Section 1: Core Programming](../01_core_programming/) -- basic programming knowledge
  to understand what is being managed by these processes.
- No specific technical prerequisites; these topics are more organizational than technical.

## What You Will Learn

After completing this section, you will be able to:
- Compare plan-driven and adaptive development methodologies
- Identify which process model suits a given project context
- Apply Scrum and Kanban practices in team settings
- Use version control concepts effectively (branching, merging, collaboration)
- Understand the role of process discipline in software quality
