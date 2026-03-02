# Version Control

## Overview

Version control (also called source control or revision control) is the
practice of tracking and managing changes to files over time. It enables
collaboration, history tracking, branching, and recovery from mistakes.

---

## Core Concepts

### Repository

A repository is the database that stores all versions of all files in a
project, along with metadata about each change.

```
Repository Structure:

  [Repository]
    |
    +-- [Commit History]
    |     |-- Commit A (initial)
    |     |-- Commit B
    |     |-- Commit C (latest)
    |
    +-- [Branches]
    |     |-- main
    |     |-- feature/login
    |     |-- bugfix/null-check
    |
    +-- [Tags]
          |-- v1.0.0
          |-- v1.1.0
```

### Commit

A commit is a snapshot of the project at a point in time. Each commit has:

```
+---------------------------------------------+
| Commit: a7f3b2c                             |
+---------------------------------------------+
| Author:  Developer Name                     |
| Date:    2026-01-15 14:32:00                |
| Parent:  b8e4d1a                            |
| Message: Add input validation to login form |
+---------------------------------------------+
| Changes:                                    |
|   Modified: auth/validator.src              |
|   Added:    auth/tests/validator_test.src   |
+---------------------------------------------+
```

### Branch

A branch is an independent line of development. Branches allow parallel work
without interference.

```
  main:      A---B---C---D---E
                  \
  feature:         F---G---H

  - "main" continues with commits D and E
  - "feature" diverges at B with commits F, G, H
  - Both lines of work are isolated
```

### Merge

Merging combines changes from one branch into another.

```
  Before merge:
  main:      A---B---C---D
                  \
  feature:         E---F---G

  After merging feature into main:
  main:      A---B---C---D---M  (M = merge commit)
                  \         /
  feature:         E---F---G
```

### Conflict

A conflict occurs when two branches modify the same part of the same file.
The developer must manually resolve which changes to keep.

```
  Conflict Example:

  main changed line 10 to:     SET timeout = 30
  feature changed line 10 to:  SET timeout = 60

  Resolution options:
  1. Keep main's version:      SET timeout = 30
  2. Keep feature's version:   SET timeout = 60
  3. Combine both:             SET timeout = 45  (or any other resolution)
```

---

## Branching Workflows

### Feature Branch Workflow

Every feature is developed in a dedicated branch, then merged to main.

```
  main:        A---B-------C-------D-------E
                    \     / \     / \     /
  feature-1:         F---G   |   |   |   |
                             |   |   |   |
  feature-2:          H---I--J   |   |   |
                                 |   |   |
  feature-3:            K---L----M   |   |
                                     |   |
  bugfix-1:               N----------O   |
                                         |
  feature-4:                P---Q---R----S

  Rules:
  - main is always deployable
  - Features branch from main
  - Features merge back via pull request
  - Branch is deleted after merge
```

### GitFlow Workflow

A more structured branching model with dedicated branch types.

```
                                          HOTFIX
                                        /       \
  main:     ----*-----------*-----------*---------*----
                |           |                     |
                |           |                     |
  release:      |       /---*---\                 |
                |      /         \                |
                |     /           \               |
  develop:  ----*----*---*---*----*----*---*---*---*----
                     |       |        |       |
  feature-A:         +--*--*-+        |       |
                                      |       |
  feature-B:                   +--*---+       |
                                              |
  feature-C:                          +--*--*-+

  Branch Types:
  +----------+-------------------------------------------------+
  | main     | Production-ready code only                      |
  | develop  | Integration branch for features                 |
  | feature/ | Individual features (branch from develop)       |
  | release/ | Preparation for production release              |
  | hotfix/  | Emergency fixes for production (branch from main)|
  +----------+-------------------------------------------------+
```

### Trunk-Based Development

All developers commit to a single branch (trunk/main) with short-lived
feature branches (hours or 1-2 days maximum).

```
  main:    A--B--C--D--E--F--G--H--I--J--K--L--M--N
               \ /      \ /         \ /
  short-1:      X        |           |
                         |           |
  short-2:               Y           |
                                     |
  short-3:                           Z

  Rules:
  - Main branch is always releasable
  - Feature branches live < 2 days
  - Feature flags hide incomplete work
  - Continuous integration runs on every commit
  - No long-lived branches
```

### Workflow Comparison

```
+-----------------------+------------+----------+-------------------+
| Aspect                | Feature Br.| GitFlow  | Trunk-Based       |
+-----------------------+------------+----------+-------------------+
| Branch lifetime       | Days-weeks | Variable | Hours-1 day       |
| Complexity            | Low        | High     | Very Low          |
| Release frequency     | Moderate   | Scheduled| Continuous        |
| Merge conflicts       | Moderate   | High     | Low (short-lived) |
| Best for              | Most teams | Large/   | High-performing   |
|                       |            | versioned| CI/CD teams       |
+-----------------------+------------+----------+-------------------+
```

---

## Merge vs Rebase

Two strategies for integrating changes from one branch into another.

### Merge

Creates a merge commit that preserves both branch histories.

```
  Before:
  main:      A---B---C
                  \
  feature:         D---E

  After merge:
  main:      A---B---C---M
                  \      /
  feature:         D---E

  Pros: Preserves complete history; non-destructive
  Cons: Creates extra merge commits; history can be hard to read
```

### Rebase

Replays feature commits on top of the target branch, creating a linear history.

```
  Before:
  main:      A---B---C
                  \
  feature:         D---E

  After rebase (feature onto main):
  main:      A---B---C
                      \
  feature:             D'---E'  (D and E are re-applied)

  After fast-forward merge:
  main:      A---B---C---D'---E'

  Pros: Clean, linear history; easy to follow
  Cons: Rewrites history; dangerous on shared branches
```

### Golden Rule of Rebase

```
+----------------------------------------------------------+
| NEVER rebase commits that have been shared with others.  |
| Only rebase your own local, unpushed commits.            |
+----------------------------------------------------------+
```

---

## Pull Request Workflow

Pull requests (PRs) or merge requests (MRs) are a code review mechanism.

```
Pull Request Lifecycle:

  1. Developer creates feature branch
  2. Developer pushes commits to feature branch
  3. Developer opens Pull Request
     +----------------------------------------------+
     | PR #42: Add password strength validation     |
     |----------------------------------------------|
     | Author: developer-name                       |
     | Branch: feature/password-strength -> main    |
     | Changes: 3 files, +85 lines, -12 lines      |
     |----------------------------------------------|
     | Description:                                 |
     |   Adds real-time password strength meter     |
     |   with entropy calculation.                  |
     |----------------------------------------------|
     | Reviewers: reviewer-a, reviewer-b            |
     | Status: Review Required                      |
     +----------------------------------------------+
  4. Reviewers examine code, leave comments
  5. Developer addresses feedback with new commits
  6. Reviewers approve
  7. Branch is merged (merge, squash, or rebase)
  8. Branch is deleted
```

### Review Checklist

```
+----------------------------------------------------------+
| Code Review Checklist                                    |
+----------------------------------------------------------+
| [ ] Code is correct and handles edge cases               |
| [ ] Tests are included and passing                       |
| [ ] No security vulnerabilities introduced               |
| [ ] Code follows project conventions                     |
| [ ] No unnecessary complexity added                      |
| [ ] Documentation updated if needed                      |
| [ ] No unrelated changes included                        |
| [ ] Commit messages are clear and descriptive            |
+----------------------------------------------------------+
```

---

## Semantic Versioning

Semantic versioning (SemVer) uses a three-part version number to communicate
the nature of changes.

```
  MAJOR.MINOR.PATCH

  Format: X.Y.Z

  +-------+---------------------------------------------------+
  | MAJOR | Incremented for incompatible (breaking) changes   |
  | MINOR | Incremented for backward-compatible new features  |
  | PATCH | Incremented for backward-compatible bug fixes     |
  +-------+---------------------------------------------------+

  Examples:
  1.0.0 -> 1.0.1  (bug fix)
  1.0.1 -> 1.1.0  (new feature, backward compatible)
  1.1.0 -> 2.0.0  (breaking change)

  Pre-release:  1.0.0-alpha.1, 1.0.0-beta.3, 1.0.0-rc.1
  Build meta:   1.0.0+build.123
```

### Version Lifecycle

```
  0.1.0 --> 0.2.0 --> 0.9.0 --> 1.0.0 (stable API)
                                  |
                         1.0.1 (patch)
                         1.1.0 (minor feature)
                         1.2.0 (another feature)
                         2.0.0 (breaking change)
```

---

## Monorepo vs Polyrepo

### Monorepo

All projects live in a single repository.

```
  /monorepo
    /packages
      /frontend
      /backend
      /shared-lib
      /mobile-app
    /tools
    /docs

  Pros:
  + Atomic cross-project changes
  + Shared tooling and configuration
  + Easy code sharing
  + Single source of truth

  Cons:
  - Repository can become very large
  - Build times increase
  - Access control is coarser
  - Requires specialized tooling at scale
```

### Polyrepo

Each project has its own repository.

```
  /repo-frontend      (separate repository)
  /repo-backend       (separate repository)
  /repo-shared-lib    (separate repository)
  /repo-mobile-app    (separate repository)

  Pros:
  + Clear ownership boundaries
  + Independent release cycles
  + Fine-grained access control
  + Smaller, faster repositories

  Cons:
  - Cross-project changes require coordination
  - Dependency version management is complex
  - Code sharing requires publishing packages
  - Tooling must be duplicated or centralized
```

### Decision Guide

```
+-----------------------------+----------+-----------+
| Factor                      | Monorepo | Polyrepo  |
+-----------------------------+----------+-----------+
| Small team, shared code     | Better   |           |
| Large org, many teams       |          | Better    |
| Frequent cross-project deps | Better   |           |
| Independent release cycles  |          | Better    |
| Strong code ownership       |          | Better    |
| Atomic refactoring          | Better   |           |
+-----------------------------+----------+-----------+
```

---

## Best Practices

```
+----------------------------------------------------------+
| Version Control Best Practices                           |
+----------------------------------------------------------+
| [ ] Commit frequently with clear, descriptive messages   |
| [ ] Keep commits focused (one logical change per commit) |
| [ ] Never commit secrets, credentials, or API keys       |
| [ ] Use branches for all non-trivial changes             |
| [ ] Review code before merging (pull requests)           |
| [ ] Keep main/trunk always deployable                    |
| [ ] Tag releases with semantic versions                  |
| [ ] Write meaningful commit messages (what and why)      |
| [ ] Delete branches after merging                        |
| [ ] Use a .gitignore (or equivalent) to exclude          |
|     build artifacts and temporary files                   |
+----------------------------------------------------------+
```

### Commit Message Format

```
  <type>: <short summary in imperative mood>

  <optional body: explain WHAT and WHY, not HOW>

  <optional footer: references, breaking changes>

  Types: feat, fix, docs, refactor, test, chore, perf

  Example:
  feat: add password strength meter to registration form

  Implements real-time entropy calculation to give users
  feedback on password quality during account creation.

  Closes #142
```

Version control is the foundation of collaborative software development. Every
team should have clear conventions for branching, committing, reviewing, and
releasing code.
