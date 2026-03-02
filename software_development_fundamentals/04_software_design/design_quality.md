# Design Quality: Cohesion, Coupling, Code Smells, and Refactoring

## Overview

Software design quality is measured by how well modules are organized internally
(cohesion) and how they relate to each other externally (coupling). High
cohesion and low coupling are the two most important goals of modular design.

---

## Cohesion

Cohesion measures how strongly the elements within a module belong together.
Higher cohesion means the module's responsibilities are focused and related.

### Cohesion Spectrum

```
WORST                                                          BEST
  |                                                              |
  v                                                              v
+----------+--------+----------+----------+----------+-----------+----------+
|Coincident|Logical |Temporal  |Procedural|Communic- |Sequential |Functional|
|  al      |        |          |          |  ational |           |          |
+----------+--------+----------+----------+----------+-----------+----------+
  Avoid      Avoid    Tolerate   Tolerate   Accept     Good        Ideal
```

### 1. Coincidental Cohesion (Worst)

Elements are grouped arbitrarily with no meaningful relationship.

```
MODULE Utilities:
    FUNCTION formatDate(d: Date) -> Text: ...
    FUNCTION calculateTax(amount: Number) -> Number: ...
    FUNCTION sendEmail(to: Text, body: Text) -> Void: ...
    FUNCTION compressImage(img: Image) -> Image: ...
END MODULE
```

These have nothing in common. This is a "junk drawer" module.

### 2. Logical Cohesion

Elements perform similar operations but on unrelated data or for unrelated
reasons. Grouped by "kind of activity" rather than purpose.

```
MODULE AllValidators:
    FUNCTION validateEmail(email: Text) -> Boolean: ...
    FUNCTION validateDNA(sequence: Text) -> Boolean: ...
    FUNCTION validateChessMove(move: Move) -> Boolean: ...
END MODULE
```

All validate, but for completely different domains.

### 3. Temporal Cohesion

Elements are grouped because they execute at the same time.

```
MODULE StartupRoutine:
    FUNCTION initializeLogger() -> Void: ...
    FUNCTION loadConfiguration() -> Void: ...
    FUNCTION warmCache() -> Void: ...
    FUNCTION connectToDatabase() -> Void: ...
END MODULE
```

Grouped by "runs at startup" rather than by responsibility.

### 4. Procedural Cohesion

Elements are grouped because they follow a specific sequence of execution.

```
MODULE UserRegistration:
    FUNCTION validateInput(data: Map) -> Boolean: ...
    FUNCTION createUser(data: Map) -> User: ...
    FUNCTION sendWelcomeEmail(user: User) -> Void: ...
    FUNCTION logRegistration(user: User) -> Void: ...
END MODULE
```

Better -- these form a procedure, but the module mixes validation, persistence,
email, and logging concerns.

### 5. Communicational Cohesion

Elements operate on the same data or contribute to the same output.

```
MODULE CustomerReport:
    FUNCTION loadCustomerData(id: Text) -> Customer: ...
    FUNCTION calculateLifetimeValue(c: Customer) -> Number: ...
    FUNCTION formatReport(c: Customer, ltv: Number) -> Text: ...
END MODULE
```

All work with customer data to produce a report. Reasonable cohesion.

### 6. Sequential Cohesion

Output of one element serves as input to the next.

```
MODULE ImageProcessor:
    FUNCTION decode(raw: Bytes) -> Image: ...
    FUNCTION resize(img: Image, w: Number, h: Number) -> Image: ...
    FUNCTION applyFilter(img: Image, filter: Filter) -> Image: ...
    FUNCTION encode(img: Image, format: Text) -> Bytes: ...
END MODULE
```

Each step feeds the next. Strong cohesion.

### 7. Functional Cohesion (Best)

Every element contributes to a single, well-defined task.

```
MODULE PasswordHasher:
    SET algorithm: HashAlgorithm
    SET saltLength: Number

    FUNCTION hash(password: Text) -> HashedPassword:
        SET salt = CALL generateSalt(saltLength)
        SET hashed = CALL algorithm.compute(password + salt)
        RETURN NEW HashedPassword(hashed, salt)
    END FUNCTION

    FUNCTION verify(password: Text, stored: HashedPassword) -> Boolean:
        SET computed = CALL algorithm.compute(password + stored.salt)
        RETURN computed = stored.hash
    END FUNCTION
END MODULE
```

Everything in this module serves one purpose: password hashing.

---

## Coupling

Coupling measures the degree of interdependence between modules. Lower coupling
means modules can change independently.

### Coupling Spectrum

```
WORST                                                    BEST
  |                                                        |
  v                                                        v
+--------+---------+--------+---------+---------+----------+
|Content |Common   |Control |Stamp    |Data     |Message   |
|        |         |        |         |         |(No       |
|        |         |        |         |         | coupling)|
+--------+---------+--------+---------+---------+----------+
  Avoid    Avoid    Minimize  Tolerate  Good      Ideal
```

### 1. Content Coupling (Worst)

One module directly accesses or modifies the internals of another.

```
// Module B reaches into Module A's private data
MODULE ModuleB:
    FUNCTION hack() -> Void:
        SET ModuleA.internalCounter = 0    // Directly modifying internals
        CALL ModuleA.privateHelper()       // Calling private function
    END FUNCTION
END MODULE
```

### 2. Common Coupling

Multiple modules share global mutable state.

```
GLOBAL sharedState = { userCount: 0, lastError: NULL }

MODULE ModuleA:
    FUNCTION process() -> Void:
        SET sharedState.userCount = sharedState.userCount + 1
    END FUNCTION
END MODULE

MODULE ModuleB:
    FUNCTION report() -> Text:
        RETURN "Users: " + sharedState.userCount    // Depends on A's mutation
    END FUNCTION
END MODULE
```

### 3. Control Coupling

One module controls the behavior of another by passing control flags.

```
FUNCTION formatOutput(data: Text, useHTML: Boolean) -> Text:
    IF useHTML THEN
        RETURN "<p>" + data + "</p>"
    ELSE
        RETURN data
    END IF
END FUNCTION
```

The caller controls internal branching. Better to have two separate functions.

### 4. Stamp Coupling

Modules share a composite data structure but each uses only part of it.

```
FUNCTION printUserName(user: UserRecord) -> Void:
    OUTPUT user.name
    // Receives entire UserRecord but only needs name
END FUNCTION
```

### 5. Data Coupling (Good)

Modules communicate through simple, well-defined parameters.

```
FUNCTION calculateArea(width: Number, height: Number) -> Number:
    RETURN width * height
END FUNCTION
```

Only the necessary data is passed. No unnecessary dependencies.

### 6. Message Coupling (Best)

Modules communicate through messages or events with no direct knowledge of
each other.

```
MODULE OrderProcessor:
    FUNCTION complete(order: Order) -> Void:
        CALL process(order)
        CALL eventBus.publish("OrderCompleted", { orderId: order.id })
    END FUNCTION
END MODULE

MODULE InventoryUpdater:
    ON EVENT "OrderCompleted":
        CALL updateStock(event.orderId)
    END ON
END MODULE
```

Neither module knows the other exists.

---

## Code Smells Catalog

Code smells are indicators of potential design problems. They are not bugs --
the code works -- but they suggest that refactoring would improve quality.

### Bloaters

```
+---+----------------------------+----------------------------------+
| # | Smell                      | Description                      |
+---+----------------------------+----------------------------------+
| 1 | Long Method                | Function exceeds 20-30 lines     |
| 2 | Large Module               | Module has too many               |
|   |                            | responsibilities                 |
| 3 | Long Parameter List        | Function takes > 3-4 parameters  |
| 4 | Data Clumps                | Groups of data that always       |
|   |                            | appear together                  |
| 5 | Primitive Obsession        | Using primitives instead of      |
|   |                            | small domain types               |
+---+----------------------------+----------------------------------+
```

### Object-Orientation Abusers

```
+---+----------------------------+----------------------------------+
| # | Smell                      | Description                      |
+---+----------------------------+----------------------------------+
| 6 | Switch/Case Chains         | Long conditional chains that     |
|   |                            | should be polymorphism           |
| 7 | Refused Bequest            | Subtype does not use inherited   |
|   |                            | behavior                         |
| 8 | Temporary Field            | Fields only used in certain      |
|   |                            | circumstances                    |
+---+----------------------------+----------------------------------+
```

### Change Preventers

```
+---+----------------------------+----------------------------------+
| # | Smell                      | Description                      |
+---+----------------------------+----------------------------------+
| 9 | Divergent Change           | One module changes for many      |
|   |                            | different reasons                |
|10 | Shotgun Surgery            | One change requires edits        |
|   |                            | across many modules              |
|11 | Parallel Inheritance       | Adding a subtype requires        |
|   |                            | adding subtypes elsewhere too    |
+---+----------------------------+----------------------------------+
```

### Dispensables

```
+---+----------------------------+----------------------------------+
| # | Smell                      | Description                      |
+---+----------------------------+----------------------------------+
|12 | Dead Code                  | Unreachable or unused code       |
|13 | Speculative Generality     | Abstractions for hypothetical    |
|   |                            | future use                       |
|14 | Duplicate Code             | Same logic in multiple places    |
+---+----------------------------+----------------------------------+
```

### Couplers

```
+---+----------------------------+----------------------------------+
| # | Smell                      | Description                      |
+---+----------------------------+----------------------------------+
|15 | Feature Envy               | Method uses another module's     |
|   |                            | data more than its own           |
|16 | Inappropriate Intimacy     | Modules know too much about      |
|   |                            | each other's internals           |
|17 | Message Chains             | a.getB().getC().getD().doThing() |
|18 | Middle Man                 | Module delegates everything      |
|   |                            | without adding value             |
+---+----------------------------+----------------------------------+
```

---

## Refactoring Strategies

### Extract Function

Move a block of code into its own named function.

```
BEFORE:
    // ... 50 lines of order processing
    SET tax = subtotal * taxRate
    SET shipping = weight * shippingRate
    SET total = subtotal + tax + shipping

AFTER:
    SET total = CALL calculateTotal(subtotal, taxRate, weight, shippingRate)
```

### Extract Module

Move related functions and data into a dedicated module.

### Inline Function

Replace a trivial function call with its body when the name adds no clarity.

### Move Function

Relocate a function to the module where it belongs (fixes Feature Envy).

### Replace Conditional with Polymorphism

```
BEFORE:
    IF shape.type = "circle" THEN
        SET area = PI * shape.radius ^ 2
    ELSE IF shape.type = "square" THEN
        SET area = shape.side ^ 2
    END IF

AFTER:
    SET area = CALL shape.area()    // Each shape type implements its own
```

### Introduce Parameter Object

```
BEFORE:
    FUNCTION createEvent(name: Text, date: Date, startTime: Time,
                         endTime: Time, location: Text, capacity: Number)

AFTER:
    FUNCTION createEvent(details: EventDetails)
```

### Replace Magic Numbers with Constants

```
BEFORE:
    IF age >= 18 THEN ...

AFTER:
    CONSTANT MINIMUM_VOTING_AGE = 18
    IF age >= MINIMUM_VOTING_AGE THEN ...
```

---

## Design Quality Metrics

```
+---------------------------+------------------------------------------+
| Metric                    | Description                              |
+---------------------------+------------------------------------------+
| Cyclomatic Complexity     | Number of independent execution paths    |
| Afferent Coupling (Ca)    | Number of modules that depend on this    |
| Efferent Coupling (Ce)    | Number of modules this depends on        |
| Instability (I)           | Ce / (Ca + Ce) -- ratio 0 (stable)       |
|                           | to 1 (unstable)                          |
| Abstractness (A)          | Ratio of abstract types to total types   |
| Distance from Main Seq.   | |A + I - 1| -- closer to 0 is better    |
| Depth of Inheritance      | Levels in type hierarchy (keep < 3)      |
| Lines per Module          | Module size (target: < 200-300)          |
| Lines per Function        | Function size (target: < 20)             |
+---------------------------+------------------------------------------+
```

### The Main Sequence

```
  Abstractness (A)
  1.0 |  Zone of        .
      |  Uselessness  .
      |             .
  0.5 |           *  <-- Main Sequence (ideal)
      |         .
      |       .
      |     .   Zone of
  0.0 |   .    Pain
      +---+---+---+---+
     0.0 0.25 0.5 0.75 1.0
                Instability (I)

  Zone of Pain:    Stable + Concrete = hard to change
  Zone of Useless: Unstable + Abstract = no one uses it
  Main Sequence:   Balanced abstractness and stability
```

---

## Summary

```
Quality Design Checklist:

  [ ] Each module has functional or sequential cohesion
  [ ] Modules communicate through data or message coupling
  [ ] No global mutable state is shared between modules
  [ ] Functions are under 20 lines
  [ ] Modules are under 300 lines
  [ ] Parameter lists have 4 or fewer parameters
  [ ] No code smells remain unaddressed
  [ ] Cyclomatic complexity per function is under 10
  [ ] Dependencies flow toward stable abstractions
```

Good design is not about following rules blindly. It is about understanding the
trade-offs and making informed decisions that keep the system maintainable,
testable, and adaptable over its lifetime.
