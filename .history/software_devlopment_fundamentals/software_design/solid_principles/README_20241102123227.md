# SOLID Principles

## Overview

SOLID is an acronym for five design principles that help create maintainable and scalable software.

## Principles

### 1. Single Responsibility Principle (SRP)

* Definition: A class should have only one reason to change
* Implementation:
  * Focused class responsibilities
  * Clear separation of concerns
  * Cohesive functionality
* Examples:
  * `srp_example.py`
  * `srp_violations.py`
  * `srp_refactoring.py`

### 2. Open/Closed Principle (OCP)

* Definition: Software entities should be open for extension but closed for modification
* Implementation:
  * Abstract base classes
  * Interface definitions
  * Plugin architectures
* Examples:
  * `ocp_example.py`
  * `ocp_violations.py`
  * `ocp_refactoring.py`

### 3. Liskov Substitution Principle (LSP)

* Definition: Subtypes must be substitutable for their base types
* Implementation:
  * Contract adherence
  * Inheritance rules
  * Type checking
* Examples:
  * `lsp_example.py`
  * `lsp_violations.py`
  * `lsp_refactoring.py`

### 4. Interface Segregation Principle (ISP)

* Definition: Clients should not be forced to depend on interfaces they don't use
* Implementation:
  * Interface granularity
  * Role interfaces
  * Minimal interfaces
* Examples:
  * `isp_example.py`
  * `isp_violations.py`
  * `isp_refactoring.py`

### 5. Dependency Inversion Principle (DIP)

* Definition: High-level modules should not depend on low-level modules
* Implementation:
  * Dependency injection
  * Abstract dependencies
  * Inversion of control
* Examples:
  * `dip_example.py`
  * `dip_violations.py`
  * `dip_refactoring.py`

## Testing

* Unit tests for each principle
* Integration tests for combined principles
* Violation detection tests


