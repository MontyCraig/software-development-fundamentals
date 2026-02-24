# Section 4: Software Design

## Overview

Software design is the discipline of making structural decisions that determine how
maintainable, extensible, and understandable a codebase will be over its lifetime. While
algorithms and data structures solve computational problems, design principles and patterns
solve organizational problems -- how to structure code so that it can evolve without
collapsing under its own complexity.

This section covers the foundational design principles (SOLID, DRY, KISS, YAGNI), the
classic Gang of Four design patterns, and the metrics and techniques for evaluating and
improving design quality. All concepts are presented with language-agnostic pseudocode
and ASCII diagrams.

---

## Table of Contents

### 1. [SOLID Principles](solid_principles.md)
The five principles that form the backbone of maintainable object-oriented design: Single
Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, and Dependency
Inversion. Each principle is explained with motivation, pseudocode examples showing
violations and corrections, and ASCII diagrams of dependency relationships.

### 2. [DRY -- Don't Repeat Yourself](dry.md)
The principle that every piece of knowledge should have a single, unambiguous, authoritative
representation. Covers what constitutes true duplication vs. coincidental similarity, when
extraction helps vs. when it hurts, and techniques for eliminating repetition at different
levels of abstraction.

### 3. [KISS -- Keep It Simple, Stupid](kiss.md)
The principle that simplicity should be a key goal in design. Covers how to recognize
unnecessary complexity, the tension between flexibility and simplicity, and strategies for
achieving simple designs without sacrificing capability. Includes before/after pseudocode
refactoring examples.

### 4. [YAGNI -- You Aren't Gonna Need It](yagni.md)
The principle of not implementing functionality until it is actually required. Covers the
costs of speculative generality, how YAGNI interacts with other principles (especially
Open/Closed), and guidelines for deciding when to build for the future vs. build for today.

### 5. [Design Patterns (Gang of Four)](design_patterns.md)
A comprehensive catalog of the 23 classic design patterns organized by category: Creational
(Factory, Abstract Factory, Builder, Singleton, Prototype), Structural (Adapter, Bridge,
Composite, Decorator, Facade, Flyweight, Proxy), and Behavioral (Observer, Strategy,
Command, State, Template Method, Iterator, Mediator, Chain of Responsibility, Visitor,
Memento, Interpreter). Each pattern includes intent, structure diagrams, and pseudocode.

### 6. [Design Quality](design_quality.md)
How to measure and improve the quality of a software design. Covers cohesion (functional,
sequential, communicational, temporal), coupling (data, stamp, control, common, content),
code smells and their refactorings, and metrics for evaluating design health. Includes
smell-to-refactoring mapping tables.

---

## Recommended Reading Order

The principles should be learned before the patterns, as patterns are applications of
principles:

1. **SOLID Principles** -- The foundational design principles; everything else builds on these.
2. **DRY** -- Eliminating duplication is one of the first design improvements to internalize.
3. **KISS** -- Balances the temptation to over-engineer that sometimes accompanies DRY.
4. **YAGNI** -- Completes the trio of pragmatic principles (DRY + KISS + YAGNI).
5. **Design Patterns** -- Reusable solutions that embody the above principles.
6. **Design Quality** -- The analytical lens for evaluating whether your design is working.

Alternatively, experienced developers may read Design Quality first to establish a
vocabulary for evaluating designs, then revisit the principles and patterns with that
framework in mind.

## Prerequisites

- [Section 1: Core Programming](../01_core_programming/) -- especially OOP and functional concepts.
- Familiarity with building multi-module programs is helpful for appreciating design concerns.

## What You Will Learn

After completing this section, you will be able to:
- Apply SOLID principles to create flexible, maintainable code
- Recognize and eliminate unnecessary duplication, complexity, and speculation
- Select and implement appropriate design patterns for common problems
- Evaluate design quality using cohesion, coupling, and code smell analysis
- Refactor existing code to improve its structural integrity
