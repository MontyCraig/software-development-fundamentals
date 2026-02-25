# Refactoring Catalog

## Overview

Refactoring is the process of restructuring existing code without changing its
external behavior. The goal is to improve readability, maintainability, and
extensibility while preserving exactly the same observable functionality.

Good refactoring is systematic, incremental, and safe. Each step is small enough
that if something breaks, the cause is immediately obvious. Tests serve as the
safety net: run them before, refactor, run them after. Green to green is the rule.

## When to Refactor

```
+-----------------------------------+-----------------------------------+
|        REFACTOR WHEN              |      DO NOT REFACTOR WHEN         |
+-----------------------------------+-----------------------------------+
| Adding a feature is hard because  | The code works and will never be  |
| the structure resists change      | modified again                    |
|                                   |                                   |
| You see the same logic in three   | You are close to a deadline and   |
| or more places                    | refactoring is not scoped         |
|                                   |                                   |
| A function is too long to         | You lack tests to verify behavior |
| understand at a glance            | is preserved (write tests first!) |
|                                   |                                   |
| A name no longer describes what   | The codebase is being replaced    |
| the code actually does            | or rewritten entirely             |
+-----------------------------------+-----------------------------------+
```

## The Refactoring Workflow

```
    +------------------+
    | Identify a smell |
    +--------+---------+
             |
             v
    +------------------+
    | Ensure tests     |<--- If missing, write tests FIRST
    | exist and pass   |
    +--------+---------+
             |
             v
    +------------------+
    | Apply ONE small  |     Keep each step reversible
    | refactoring step |
    +--------+---------+
             |
             v
    +------------------+
    | Run all tests    |---> RED? Undo and try smaller
    | (must be GREEN)  |
    +--------+---------+
             |
             v
    +------------------+
    | Commit working   |     Small, frequent commits
    | state            |
    +--------+---------+
             |
             v
    +------------------+
    | More to improve? |---> Yes: return to top
    +------------------+---> No: done
```

## Refactoring Techniques by Category

### Category 1: Composing Methods

#### 1.1 Extract Function -- Pull a coherent block into its own named function.

**BEFORE:**
```
FUNCTION print_invoice(invoice: Invoice) -> Void:
    print_header(invoice)
    SET subtotal = 0
    FOR EACH item IN invoice.items DO
        SET subtotal = subtotal + item.price * item.quantity
    END FOR
    print("Total: " + FORMAT_CURRENCY(subtotal + subtotal * invoice.tax_rate))
END FUNCTION
```

**AFTER:**
```
FUNCTION print_invoice(invoice: Invoice) -> Void:
    print_header(invoice)
    print("Total: " + FORMAT_CURRENCY(calculate_total(invoice)))
END FUNCTION

FUNCTION calculate_total(invoice: Invoice) -> Number:
    SET subtotal = 0
    FOR EACH item IN invoice.items DO
        SET subtotal = subtotal + item.price * item.quantity
    END FOR
    RETURN subtotal + subtotal * invoice.tax_rate
END FUNCTION
```

#### 1.2 Inline Function -- Remove trivial indirection.

**BEFORE:** `get_rating()` calls `more_than_five_late()` which just returns `driver.late_deliveries > 5`.
**AFTER:** `get_rating()` contains the comparison directly.

#### 1.3 Replace Temp with Query -- Replace a temp with a named function call.

**BEFORE:** `SET base_price = order.quantity * order.unit_price` then use `base_price`.
**AFTER:** Call `base_price(order)` function directly. Computation is reusable and named.

#### 1.4 Introduce Explaining Variable -- Name complex expressions.

**BEFORE:**
```
IF price * qty > 500 AND loyalty > 2 AND NOT flagged THEN apply_discount() END IF
```

**AFTER:**
```
SET is_large_order = price * qty > 500
SET is_loyal = loyalty > 2
IF is_large_order AND is_loyal AND NOT flagged THEN apply_discount() END IF
```

#### 1.5 Split Temporary Variable -- One variable per purpose.

**BEFORE:** `SET temp = perimeter ... SET temp = area` (reused for different meanings).
**AFTER:** `SET perimeter = ...` and `SET area = ...` (separate, descriptive names).

### Category 2: Moving Features

#### 2.1 Move Function -- Move to the module that uses the data most.

**BEFORE:** `Account.overdraft_charge()` accesses `account.type.is_premium` extensively.
**AFTER:** Logic moves to `AccountType.overdraft_charge(days_overdrawn)`. Account delegates.

#### 2.2 Extract Module -- Split a module with multiple responsibilities.

**BEFORE:** Employee has name, department, salary, tax, and functions for all of them.
**AFTER:** Employee holds identity; EmployeePayroll holds salary and tax logic.

#### 2.3 Hide Delegate -- Add a method to hide a chain of calls.

**BEFORE:** `SET manager = employee.department.manager`
**AFTER:** `SET manager = employee.get_manager()` (delegates internally)

#### 2.4 Remove Middle Man -- Inverse of Hide Delegate.

**BEFORE:** `employee.get_manager()`, `employee.get_budget()`, ... (many trivial delegates)
**AFTER:** `employee.get_department().manager` (client calls delegate directly)

### Category 3: Organizing Data

#### 3.1 Replace Magic Number with Named Constant

**BEFORE:** `RETURN mass * 9.81 * height`
**AFTER:**
```
CONSTANT GRAVITATIONAL_ACCELERATION = 9.81
RETURN mass * GRAVITATIONAL_ACCELERATION * height
```

#### 3.2 Encapsulate Collection

Never expose a raw mutable collection. Provide add/remove/query methods.

**BEFORE:**
```
FUNCTION get_students() -> List: RETURN self.students  // caller can mutate!
```

**AFTER:**
```
FUNCTION get_students() -> List: RETURN COPY(self.students)
FUNCTION add_student(s: Student): APPEND s TO self.students
FUNCTION remove_student(s: Student): REMOVE s FROM self.students
```

#### 3.3 Replace Type Code with Polymorphism

Replace conditional logic on a type code with polymorphic dispatch.

**BEFORE:**
```
FUNCTION area(shape: Shape) -> Number:
    IF shape.type = "circle" THEN RETURN 3.14 * r * r
    ELSE IF shape.type = "rectangle" THEN RETURN w * h END IF
END FUNCTION
```

**AFTER:**
```
INTERFACE Shape:
    FUNCTION area() -> Number

MODULE Circle IMPLEMENTS Shape:
    FUNCTION area() -> Number: RETURN 3.14 * self.radius * self.radius
END MODULE

MODULE Rectangle IMPLEMENTS Shape:
    FUNCTION area() -> Number: RETURN self.width * self.height
END MODULE
```

### Category 4: Simplifying Conditionals

#### 4.1 Decompose Conditional

Replace complex conditionals with well-named function calls.

**BEFORE:**
```
IF date >= SUMMER_START AND date <= SUMMER_END THEN
    SET charge = quantity * summer_rate + service_charge
ELSE
    SET charge = quantity * winter_rate
END IF
```

**AFTER:**
```
IF is_summer(date) THEN
    SET charge = summer_charge(quantity)
ELSE
    SET charge = winter_charge(quantity)
END IF
```

#### 4.2 Consolidate Conditional

Merge multiple conditions producing the same result into one named check.

**BEFORE:**
```
IF employee.seniority < 2 THEN RETURN 0 END IF
IF employee.months_disabled > 12 THEN RETURN 0 END IF
IF employee.is_part_time THEN RETURN 0 END IF
```

**AFTER:**
```
IF is_not_eligible(employee) THEN RETURN 0 END IF

FUNCTION is_not_eligible(e: Employee) -> Boolean:
    RETURN e.seniority < 2 OR e.months_disabled > 12 OR e.is_part_time
END FUNCTION
```

#### 4.3 Replace Nested Conditional with Guard Clauses

Flatten deep nesting by handling special cases early with returns.

**BEFORE:**
```
FUNCTION calculate_pay(emp: Employee) -> Number:
    SET result = 0
    IF emp.is_active THEN
        IF emp.is_separated THEN
            SET result = separated_amount(emp)
        ELSE
            IF emp.is_retired THEN
                SET result = retired_amount(emp)
            ELSE
                SET result = normal_pay(emp)
            END IF
        END IF
    END IF
    RETURN result
END FUNCTION
```

**AFTER:**
```
FUNCTION calculate_pay(emp: Employee) -> Number:
    IF NOT emp.is_active THEN RETURN 0 END IF
    IF emp.is_separated THEN RETURN separated_amount(emp) END IF
    IF emp.is_retired THEN RETURN retired_amount(emp) END IF
    RETURN normal_pay(emp)
END FUNCTION
```

#### 4.4 Replace Conditional with Polymorphism

When a conditional selects behavior based on type, use polymorphic dispatch through
an interface instead. See Section 3.3 for a full example.

### Category 5: Dealing with Generalization

#### 5.1 Pull Up Method

Move identical methods from sibling modules into their shared parent.

**BEFORE:** Both FullTimeEmployee and PartTimeEmployee define identical `annual_cost()`.
**AFTER:** `annual_cost()` lives in parent Employee; children inherit it.

#### 5.2 Push Down Method

Move a method from parent to only the child that needs it.

**BEFORE:** Employee has `commission_rate()` but only SalesEmployee uses it.
**AFTER:** `commission_rate()` moves to SalesEmployee only.

#### 5.3 Extract Interface

Define a shared interface when unrelated modules support the same operations.

**BEFORE:** DatabaseLogger and FileLogger both have `log()` but no shared type.
**AFTER:**
```
INTERFACE Logger:
    FUNCTION log(message: Text) -> Void

MODULE DatabaseLogger IMPLEMENTS Logger: ...
MODULE FileLogger IMPLEMENTS Logger: ...
```

#### 5.4 Collapse Hierarchy

Merge parent and child when the child adds nothing meaningful.

**BEFORE:** SalariedEmployee extends Employee but adds no behavior or data.
**AFTER:** Eliminate SalariedEmployee; update all references to Employee.

## Code Smell to Refactoring Mapping

```
+----------------------------------+---------------------------------------------+
| Code Smell                       | Suggested Refactoring(s)                    |
+----------------------------------+---------------------------------------------+
| Long Function                    | Extract Function, Decompose Conditional     |
| Duplicated Logic                 | Extract Function, Pull Up Method            |
| Long Parameter List              | Introduce Explaining Var, Extract Module    |
| Divergent Change (1 module,      | Extract Module                              |
|   many reasons to change)        |                                             |
| Shotgun Surgery (1 change,       | Move Function, Inline Function              |
|   many modules affected)         |                                             |
| Feature Envy                     | Move Function                               |
| Data Clumps                      | Extract Module                              |
| Primitive Obsession              | Replace Type Code with Polymorphism         |
| Switch/If-Type Statements        | Replace Conditional with Polymorphism       |
| Speculative Generality           | Collapse Hierarchy, Inline Function         |
| Message Chains                   | Hide Delegate                               |
| Middle Man                       | Remove Middle Man                           |
| Magic Numbers                    | Replace Magic Number with Named Constant    |
| Excessive Comments               | Extract Function, Introduce Explaining Var  |
+----------------------------------+---------------------------------------------+
```

## When Refactoring Goes Wrong

### 1. Refactoring Without Tests
Changing structure without a safety net. If no tests exist, write them first.

### 2. Big Bang Refactoring
Restructuring everything at once creates an untestable, unreviewable delta.
Refactor in small steps. Commit after each.

### 3. Speculative Refactoring
Restructuring "because we might need flexibility later" violates YAGNI. Refactor
to solve a problem you have today, not one you imagine.

### 4. Refactoring Under Deadline Pressure
Half-finished restructuring is worse than the original. Refactor when you have time
to finish properly, or not at all.

### 5. Over-Extracting
Too many tiny single-use functions create a maze of indirection. Each extraction
should earn its existence through clarity, reuse, or testability.

### 6. Renaming Without Communication
Renaming across module boundaries without coordinating creates merge conflicts and
broken integrations. Always communicate shared refactorings.

## Summary

```
+---------------------------------------------+
|   Code that works is not finished.           |
|   Code that is clean is not perfect.         |
|   Refactoring is a continuous practice,      |
|   not a one-time event.                      |
|                                              |
|   Small steps. Frequent tests. Clear names.  |
|   Each commit leaves the code better than    |
|   you found it.                              |
+---------------------------------------------+
```

## Practice Problems

1. **Smell Identification:** Identify at least three code smells and the refactoring
   for each:
   ```
   FUNCTION process(t, q, r, flag, mode, user_id, dept_code) -> Number:
       SET x = 0
       IF flag = 1 THEN
           SET x = q * 0.08
           IF r > 100 THEN SET x = x + 15 END IF
       ELSE IF flag = 2 THEN
           SET x = q * 0.12
           IF r > 100 THEN SET x = x + 15 END IF
       ELSE IF flag = 3 THEN
           SET x = q * 0.05
       END IF
       RETURN x
   END FUNCTION
   ```

2. **Extract and Name:** Refactor the function above using Extract Function for the
   surcharge logic, Introduce Explaining Variable for `q * rate`, and Replace Magic
   Number for the constants.

3. **Guard Clause Conversion:** Rewrite using guard clauses:
   ```
   FUNCTION get_payment(order: Order) -> Number:
       SET result = 0
       IF order IS NOT NULL THEN
           IF order.is_paid THEN
               IF order.has_receipt THEN SET result = order.amount END IF
           END IF
       END IF
       RETURN result
   END FUNCTION
   ```

4. **Polymorphism Refactoring:** A notification system uses type codes ("email",
   "sms", "push") with a conditional. Design an interface and three modules that
   eliminate it. Show both versions.

5. **Refactoring Plan:** You inherit a 200-line function that validates input,
   calculates pricing, applies discounts, generates an invoice, and sends
   confirmation. List the specific techniques you would apply in order.

6. **Identify the Anti-Pattern:** Name the anti-pattern and correct approach:
   - Rewriting the data layer "in case we switch databases."
   - Refactoring the auth module the night before release.
   - Splitting a function into 12 single-line helpers each called once.
   - Pushing a major rename without telling the frontend team.
