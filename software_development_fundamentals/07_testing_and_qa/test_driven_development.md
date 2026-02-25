# Test-Driven Development (TDD)

## Overview

Test-Driven Development is a software development discipline where tests are
written before the production code. The developer writes a failing test first,
then writes the minimum code to make it pass, then refactors for quality. This
cycle is called Red-Green-Refactor.

```
+-------------------------------------------------------+
| "Test-Driven Development is not about testing.        |
|  It is about designing software guided by tests."     |
+-------------------------------------------------------+
```

---

## The Red-Green-Refactor Cycle

```
         +----------+
         |   RED    |  Write a failing test
         | (test    |  that defines desired behavior
         |  fails)  |
         +----+-----+
              |
              v
         +----------+
         |  GREEN   |  Write the minimum code
         | (test    |  to make the test pass
         |  passes) |
         +----+-----+
              |
              v
         +----------+
         | REFACTOR |  Improve code quality
         | (tests   |  without changing behavior
         |  still   |  (all tests still pass)
         |  pass)   |
         +----+-----+
              |
              +--------> Return to RED
```

### Phase Details

```
+----------+----------------------------------------------------------+
| RED      | 1. Write a test for the NEXT piece of behavior          |
|          | 2. Run the test -- it MUST fail                          |
|          | 3. The failure confirms the test is valid                |
|          | (A test that passes immediately proves nothing)          |
+----------+----------------------------------------------------------+
| GREEN    | 1. Write the SIMPLEST code that makes the test pass     |
|          | 2. Do not write more than is needed                      |
|          | 3. Hardcode if necessary (you will generalize later)     |
|          | 4. Run ALL tests -- they must all pass                   |
+----------+----------------------------------------------------------+
| REFACTOR | 1. Remove duplication                                    |
|          | 2. Improve naming and structure                          |
|          | 3. Extract functions or modules if needed                |
|          | 4. Run ALL tests after each change -- they must pass    |
+----------+----------------------------------------------------------+
```

---

## TDD Walkthrough: Building a Stack

Let us build a stack data structure step by step using TDD.

### Iteration 1: A New Stack Is Empty

```
// RED: Write a failing test
FUNCTION test_new_stack_is_empty():
    SET stack = NEW Stack()
    ASSERT stack.isEmpty() = true
END FUNCTION
// This fails because Stack does not exist yet.

// GREEN: Minimal implementation
MODULE Stack:
    FUNCTION isEmpty() -> Boolean:
        RETURN true
    END FUNCTION
END MODULE
// Test passes. (Yes, hardcoding is fine at this stage.)

// REFACTOR: Nothing to refactor yet.
```

### Iteration 2: Stack With One Item Is Not Empty

```
// RED: Write a failing test
FUNCTION test_stack_with_one_item_is_not_empty():
    SET stack = NEW Stack()
    CALL stack.push("item")
    ASSERT stack.isEmpty() = false
END FUNCTION
// This fails because push() does not exist.

// GREEN: Minimal implementation
MODULE Stack:
    SET count = 0

    FUNCTION push(item: Any) -> Void:
        SET count = count + 1
    END FUNCTION

    FUNCTION isEmpty() -> Boolean:
        RETURN count = 0
    END FUNCTION
END MODULE
// Both tests pass.

// REFACTOR: Nothing needed yet.
```

### Iteration 3: Pop Returns the Last Pushed Item

```
// RED: Write a failing test
FUNCTION test_pop_returns_last_pushed_item():
    SET stack = NEW Stack()
    CALL stack.push("hello")
    SET result = CALL stack.pop()
    ASSERT result = "hello"
END FUNCTION
// Fails: pop() does not exist.

// GREEN: Minimal implementation
MODULE Stack:
    SET items: List<Any> = EMPTY LIST

    FUNCTION push(item: Any) -> Void:
        CALL items.addLast(item)
    END FUNCTION

    FUNCTION pop() -> Any:
        RETURN items.removeLast()
    END FUNCTION

    FUNCTION isEmpty() -> Boolean:
        RETURN LENGTH(items) = 0
    END FUNCTION
END MODULE
// All three tests pass.

// REFACTOR: Removed the "count" variable (replaced by items list).
```

### Iteration 4: Pop From Empty Stack Raises Error

```
// RED: Write a failing test
FUNCTION test_pop_from_empty_stack_raises_error():
    SET stack = NEW Stack()
    ASSERT CALL stack.pop() RAISES EmptyStackError
END FUNCTION
// Fails: currently crashes with a different error.

// GREEN: Add guard clause
MODULE Stack:
    SET items: List<Any> = EMPTY LIST

    FUNCTION push(item: Any) -> Void:
        CALL items.addLast(item)
    END FUNCTION

    FUNCTION pop() -> Any:
        IF this.isEmpty() THEN
            RAISE EmptyStackError("Cannot pop from empty stack")
        END IF
        RETURN items.removeLast()
    END FUNCTION

    FUNCTION isEmpty() -> Boolean:
        RETURN LENGTH(items) = 0
    END FUNCTION
END MODULE
// All four tests pass.
```

### Iteration 5: LIFO Order

```
// RED: Write a failing test
FUNCTION test_stack_follows_lifo_order():
    SET stack = NEW Stack()
    CALL stack.push("first")
    CALL stack.push("second")
    CALL stack.push("third")
    ASSERT CALL stack.pop() = "third"
    ASSERT CALL stack.pop() = "second"
    ASSERT CALL stack.pop() = "first"
END FUNCTION
// This passes immediately! Our implementation already handles LIFO.
// This is a CONFIRMATION test -- good to have, but we did not need
// to change code. Move on.
```

### Iteration 6: Peek Without Removing

```
// RED: Write a failing test
FUNCTION test_peek_returns_top_without_removing():
    SET stack = NEW Stack()
    CALL stack.push("item")
    ASSERT CALL stack.peek() = "item"
    ASSERT stack.isEmpty() = false   // Still there
END FUNCTION
// Fails: peek() does not exist.

// GREEN: Add peek
FUNCTION peek() -> Any:
    IF this.isEmpty() THEN
        RAISE EmptyStackError("Cannot peek empty stack")
    END IF
    RETURN items[LENGTH(items) - 1]
END FUNCTION
// All six tests pass.

// REFACTOR: Extract "top index" to reduce duplication
// between pop() and peek().
```

---

## Behavior-Driven Development (BDD)

BDD extends TDD by using natural-language specifications that describe
behavior from the user's perspective. BDD uses the Given-When-Then format.

### Given-When-Then Structure

```
FEATURE: Shopping Cart

  SCENARIO: Adding an item to an empty cart
    GIVEN an empty shopping cart
    WHEN the customer adds a product with price 29.99
    THEN the cart contains 1 item
    AND the cart total is 29.99

  SCENARIO: Removing an item from the cart
    GIVEN a cart containing 2 items
    WHEN the customer removes one item
    THEN the cart contains 1 item

  SCENARIO: Applying a discount code
    GIVEN a cart with total 100.00
    WHEN the customer applies discount code "SAVE10"
    THEN the cart total is 90.00
    AND the discount "SAVE10" is shown as applied
```

### BDD Process

```
  1. Business and development collaborate on scenarios
  2. Scenarios are written in Given-When-Then format
  3. Step definitions map scenarios to executable tests
  4. Tests drive the implementation (same as TDD)
  5. Scenarios serve as living documentation
```

```
BDD vs TDD:

  TDD:  Developer writes technical unit tests
  BDD:  Team writes behavior specifications in business language
        that are ALSO executable tests

  BDD is TDD applied at the feature/acceptance level.
```

---

## Acceptance Test-Driven Development (ATDD)

ATDD applies TDD at the acceptance test level. The team writes acceptance
tests before development begins, and those tests define "done."

```
ATDD Workflow:

  1. [Business] defines acceptance criteria in Given-When-Then
  2. [Team] automates acceptance tests (they fail -- RED)
  3. [Developer] uses TDD to implement features
  4. [Developer] runs acceptance tests (they pass -- GREEN)
  5. [Team] refactors and reviews

  Outer loop: Acceptance tests (ATDD)
  Inner loop: Unit tests (TDD)

  +------------------------------------------+
  | Acceptance Test: "User can log in"        |
  |                                           |
  |   +------------------------------------+  |
  |   | Unit Test: validateCredentials()   |  |
  |   +------------------------------------+  |
  |   | Unit Test: createSession()         |  |
  |   +------------------------------------+  |
  |   | Unit Test: redirectToDashboard()   |  |
  |   +------------------------------------+  |
  |                                           |
  +------------------------------------------+
```

---

## The Test Pyramid in TDD Context

```
           /\
          /  \       Acceptance tests (ATDD/BDD)
         / AT \      Written first, pass last
        /------\
       / Integr-\    Integration tests
      /  ation   \   Verify module interactions
     /------------\
    /  Unit Tests  \  Written per TDD cycle
   /________________\ Fast, many, specific

  In TDD, you work bottom-up:
  Unit tests drive individual module design.
  Integration tests verify module combinations.
  Acceptance tests confirm the feature works end-to-end.
```

---

## Benefits of TDD

```
+---+-----------------------------------------------------------+
| 1 | Design feedback: Tests force you to think about           |
|   | interfaces and usability BEFORE implementation            |
+---+-----------------------------------------------------------+
| 2 | Regression safety: Comprehensive test suite catches       |
|   | regressions immediately                                   |
+---+-----------------------------------------------------------+
| 3 | Documentation: Tests describe what the code does          |
|   | and serve as executable examples                          |
+---+-----------------------------------------------------------+
| 4 | Confidence: Refactoring is safe because tests verify      |
|   | behavior is preserved                                     |
+---+-----------------------------------------------------------+
| 5 | Simpler design: The pressure to write testable code       |
|   | naturally leads to better modularity and lower coupling  |
+---+-----------------------------------------------------------+
| 6 | Incremental progress: Small, proven steps toward          |
|   | a working solution                                       |
+---+-----------------------------------------------------------+
| 7 | Reduced debugging time: Bugs are caught immediately       |
|   | after being introduced                                   |
+---+-----------------------------------------------------------+
```

---

## Criticisms of TDD

```
+---+-----------------------------------------------------------+
| 1 | Slower initial development: Writing tests first takes     |
|   | more time upfront (pays off later in fewer bugs)          |
+---+-----------------------------------------------------------+
| 2 | False confidence: Passing tests do not guarantee          |
|   | correctness -- they only verify what was tested           |
+---+-----------------------------------------------------------+
| 3 | Over-testing: Can lead to testing implementation          |
|   | details, creating fragile tests                           |
+---+-----------------------------------------------------------+
| 4 | Difficult for exploration: When the design is unknown,    |
|   | writing tests first can feel constraining                |
+---+-----------------------------------------------------------+
| 5 | Maintenance burden: Large test suites require ongoing     |
|   | maintenance as the system evolves                         |
+---+-----------------------------------------------------------+
| 6 | Not suitable for all domains: UI, data science, and      |
|   | prototype code may not benefit as much                   |
+---+-----------------------------------------------------------+
```

---

## When TDD Works Best

```
+----------------------------------------------------------+
| TDD Excels When:                                         |
+----------------------------------------------------------+
| [ ] Requirements are well-understood                     |
| [ ] The domain has clear business rules                  |
| [ ] Code must be reliable (financial, medical, safety)   |
| [ ] The team values long-term maintainability            |
| [ ] Refactoring is expected and frequent                 |
| [ ] The project has a long lifespan                      |
+----------------------------------------------------------+

+----------------------------------------------------------+
| TDD May Not Be Ideal When:                               |
+----------------------------------------------------------+
| [ ] Prototyping or exploring an unknown domain           |
| [ ] Building throwaway code (spikes, proof of concept)   |
| [ ] Heavy UI work where visual testing is more relevant  |
| [ ] Extremely tight deadlines with no future maintenance |
+----------------------------------------------------------+
```

---

## TDD Rules (Robert C. Martin's Three Laws)

```
+---+-----------------------------------------------------------+
| 1 | You may not write production code until you have written  |
|   | a failing unit test.                                      |
+---+-----------------------------------------------------------+
| 2 | You may not write more of a unit test than is sufficient  |
|   | to fail (and not compiling counts as failing).            |
+---+-----------------------------------------------------------+
| 3 | You may not write more production code than is sufficient |
|   | to pass the currently failing test.                       |
+---+-----------------------------------------------------------+
```

These rules enforce the discipline of small, incremental steps. They prevent
writing code that is not validated by a test and prevent writing tests that
test too much at once.

---

## TDD Workflow Summary

```
+----------------------------------------------------------+
| TDD Workflow Checklist                                   |
+----------------------------------------------------------+
| [ ] Write a small, focused test for the next behavior    |
| [ ] Run the test -- verify it fails (RED)                |
| [ ] Write the simplest code to make it pass              |
| [ ] Run all tests -- verify they all pass (GREEN)        |
| [ ] Refactor code while keeping tests green              |
| [ ] Repeat for the next behavior                         |
| [ ] Each cycle takes 1-10 minutes                        |
| [ ] Tests cover behavior, not implementation details     |
| [ ] Test names clearly describe scenarios                |
| [ ] Refactoring is done with confidence (tests protect)  |
+----------------------------------------------------------+
```

TDD is a discipline, not a dogma. It works best when applied thoughtfully,
adapted to the team's context, and combined with good judgment about what
to test and at what level.
