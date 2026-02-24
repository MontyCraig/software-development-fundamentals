# Section 1: Core Programming

## Overview

This section covers the foundational building blocks of programming that every developer
must master. The topics progress from basic data representation through control structures,
modular code organization, and into the major programming paradigms. All concepts are
presented in a language-agnostic manner using pseudocode, making them applicable regardless
of which programming language you work with.

Understanding these fundamentals deeply -- not just syntactically, but conceptually -- is
what separates developers who can adapt to any technology from those who are tied to a
single tool.

---

## Table of Contents

### 1. [Variables and Data Types](variables_and_data_types.md)
An introduction to how programs store and represent information. Covers primitive types
(integers, floating-point numbers, booleans, characters), composite types, type systems
(static vs. dynamic, strong vs. weak), and type conversion. Includes memory layout
diagrams and pseudocode examples for type operations.

### 2. [Control Flow](control_flow.md)
Explains the mechanisms that determine execution order within a program. Covers conditional
branching (if/else, switch/case), loop constructs (for, while, do-while, for-each),
short-circuit evaluation, and nested control structures. Includes flowchart-style ASCII
diagrams for each construct.

### 3. [Functions and Scope](functions_and_scope.md)
Explores how to decompose programs into reusable, named blocks of logic. Covers function
declaration, parameters and return values, pass-by-value vs. pass-by-reference, scope
rules (local, global, block), closures, and the call stack. Includes stack frame diagrams
and pseudocode demonstrations.

### 4. [Error Handling](error_handling.md)
Covers strategies for anticipating, detecting, and responding to runtime failures. Explores
return-code-based error handling, exception mechanisms (try/catch/finally), custom error
types, error propagation, and defensive programming. Includes decision trees for choosing
error-handling strategies.

### 5. [Object-Oriented Programming](object_oriented_programming.md)
A comprehensive guide to the OOP paradigm: classes, objects, encapsulation, inheritance,
polymorphism, and abstraction. Covers access modifiers, constructors, method overriding,
abstract classes, interfaces, and composition vs. inheritance. Includes UML-style ASCII
class diagrams.

### 6. [Functional Programming](functional_programming.md)
Introduces the functional paradigm: pure functions, immutability, first-class and
higher-order functions, closures, map/filter/reduce, function composition, currying, and
recursion. Contrasts imperative and functional approaches with side-by-side pseudocode
examples.

---

## Recommended Reading Order

For those new to programming, follow the numbered order above. Each topic builds on the
previous one:

1. **Variables and Data Types** -- You need to understand data before you can manipulate it.
2. **Control Flow** -- Once you have data, you need to make decisions and repeat operations.
3. **Functions and Scope** -- Organizing code into reusable units is the next step.
4. **Error Handling** -- Real programs must handle failure gracefully.
5. **Object-Oriented Programming** -- The dominant paradigm for structuring larger programs.
6. **Functional Programming** -- A complementary paradigm that changes how you think about computation.

Experienced developers may jump directly to topics 5 and 6 for a paradigm refresher, or
to topic 4 for error-handling patterns.

## Prerequisites

- No prior programming experience is strictly required, though familiarity with basic
  mathematical concepts (variables, expressions, logic) is helpful.
- A willingness to read and trace through pseudocode examples by hand.

## What You Will Learn

After completing this section, you will be able to:
- Explain how data is represented and stored in programs
- Trace the execution flow of conditional and iterative constructs
- Design functions with clear interfaces and appropriate scope management
- Implement robust error-handling strategies
- Model real-world problems using object-oriented design
- Apply functional programming techniques to write concise, composable code
