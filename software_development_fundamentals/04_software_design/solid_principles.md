# SOLID Principles

## Overview

SOLID is an acronym for five design principles that make software systems more
understandable, flexible, and maintainable. These principles were promoted by
Robert C. Martin and form the foundation of good object-oriented design.

```
+-----------------------------------------------------------+
|                    SOLID PRINCIPLES                        |
+-----------------------------------------------------------+
|  S  |  Single Responsibility Principle (SRP)              |
|  O  |  Open/Closed Principle (OCP)                        |
|  L  |  Liskov Substitution Principle (LSP)                |
|  I  |  Interface Segregation Principle (ISP)              |
|  D  |  Dependency Inversion Principle (DIP)               |
+-----------------------------------------------------------+
```

Each principle addresses a specific aspect of software design and together they
guide developers toward systems that are resilient to change.

---

## S - Single Responsibility Principle (SRP)

### Definition

A module should have one, and only one, reason to change. Each module should be
responsible to one, and only one, actor (stakeholder or user group).

### Real-World Analogy

Consider a restaurant. The chef cooks, the waiter serves, and the cashier
handles payments. If one person did all three jobs, a change in payment policy
would risk affecting how food is prepared. Separation of roles limits the
blast radius of change.

### Violation

```
MODULE UserManager:
    FUNCTION createUser(name: Text, email: Text) -> User:
        SET user = NEW User(name, email)
        CALL database.save(user)
        SET html = "<h1>Welcome " + name + "</h1>"
        CALL emailService.send(email, html)
        CALL logger.write("User created: " + name)
        RETURN user
    END FUNCTION

    FUNCTION generateReport(users: List<User>) -> Text:
        SET csv = "Name,Email\n"
        FOR EACH user IN users DO
            SET csv = csv + user.name + "," + user.email + "\n"
        END FOR
        RETURN csv
    END FUNCTION
END MODULE
```

This module has three reasons to change: user creation logic, email formatting,
and report generation. Three different actors could request changes.

### Correct Implementation

```
MODULE UserCreator:
    FUNCTION create(name: Text, email: Text) -> User:
        SET user = NEW User(name, email)
        CALL userRepository.save(user)
        RETURN user
    END FUNCTION
END MODULE

MODULE WelcomeNotifier:
    FUNCTION notify(user: User) -> Void:
        SET message = CALL templateEngine.render("welcome", user)
        CALL emailService.send(user.email, message)
    END FUNCTION
END MODULE

MODULE UserReportGenerator:
    FUNCTION generate(users: List<User>) -> Text:
        SET csv = "Name,Email\n"
        FOR EACH user IN users DO
            SET csv = csv + user.name + "," + user.email + "\n"
        END FOR
        RETURN csv
    END FUNCTION
END MODULE
```

```
Dependency Diagram (SRP Applied):

  [UserCreator] ----> [UserRepository]
        |
        v
  [WelcomeNotifier] --> [EmailService]
                    --> [TemplateEngine]

  [UserReportGenerator]  (independent)
```

Each module now has exactly one reason to change and serves exactly one actor.

---

## O - Open/Closed Principle (OCP)

### Definition

Software entities should be open for extension but closed for modification.
You should be able to add new behavior without changing existing code.

### Real-World Analogy

A power strip is open for extension (plug in new devices) but closed for
modification (you do not rewire it for each new device). New devices conform
to the existing plug interface.

### Violation

```
FUNCTION calculateDiscount(customer: Customer) -> Number:
    IF customer.type = "regular" THEN
        RETURN customer.total * 0.05
    ELSE IF customer.type = "premium" THEN
        RETURN customer.total * 0.10
    ELSE IF customer.type = "vip" THEN
        RETURN customer.total * 0.20
    END IF
    RETURN 0
END FUNCTION
```

Adding a new customer type requires modifying this function every time.

### Correct Implementation

```
INTERFACE DiscountStrategy:
    FUNCTION calculate(total: Number) -> Number
END INTERFACE

MODULE RegularDiscount IMPLEMENTS DiscountStrategy:
    FUNCTION calculate(total: Number) -> Number:
        RETURN total * 0.05
    END FUNCTION
END MODULE

MODULE PremiumDiscount IMPLEMENTS DiscountStrategy:
    FUNCTION calculate(total: Number) -> Number:
        RETURN total * 0.10
    END FUNCTION
END MODULE

MODULE VIPDiscount IMPLEMENTS DiscountStrategy:
    FUNCTION calculate(total: Number) -> Number:
        RETURN total * 0.20
    END FUNCTION
END MODULE

FUNCTION calculateDiscount(strategy: DiscountStrategy, total: Number) -> Number:
    RETURN strategy.calculate(total)
END FUNCTION
```

```
Extension Diagram (OCP Applied):

  [DiscountStrategy]  <--- interface
       ^   ^   ^
       |   |   |
  [Regular] [Premium] [VIP]  <--- extend without modifying

  To add "Employee" discount:
  Just create [EmployeeDiscount] implementing [DiscountStrategy]
  No existing code changes needed.
```

---

## L - Liskov Substitution Principle (LSP)

### Definition

Objects of a supertype should be replaceable with objects of a subtype without
altering the correctness of the program. Subtypes must honor the contracts
established by their parent types.

### Real-World Analogy

If you order a "vehicle" for transportation, receiving a car, bus, or truck
should all work. But receiving a toy car would violate your expectations --
it looks like a vehicle but cannot actually transport you.

### Violation

```
MODULE Rectangle:
    SET width = 0
    SET height = 0

    FUNCTION setWidth(w: Number) -> Void:
        SET width = w
    END FUNCTION

    FUNCTION setHeight(h: Number) -> Void:
        SET height = h
    END FUNCTION

    FUNCTION area() -> Number:
        RETURN width * height
    END FUNCTION
END MODULE

MODULE Square EXTENDS Rectangle:
    FUNCTION setWidth(w: Number) -> Void:
        SET width = w
        SET height = w    // Forces height to match width
    END FUNCTION

    FUNCTION setHeight(h: Number) -> Void:
        SET height = h
        SET width = h     // Forces width to match height
    END FUNCTION
END MODULE

// Client code breaks:
FUNCTION testArea(rect: Rectangle) -> Void:
    CALL rect.setWidth(5)
    CALL rect.setHeight(4)
    ASSERT rect.area() = 20    // Fails for Square! Gets 16.
END FUNCTION
```

Square changes the postcondition of setWidth/setHeight, breaking substitution.

### Correct Implementation

```
INTERFACE Shape:
    FUNCTION area() -> Number
END INTERFACE

MODULE Rectangle IMPLEMENTS Shape:
    CONSTRUCTOR(width: Number, height: Number):
        SET this.width = width
        SET this.height = height
    END CONSTRUCTOR

    FUNCTION area() -> Number:
        RETURN this.width * this.height
    END FUNCTION
END MODULE

MODULE Square IMPLEMENTS Shape:
    CONSTRUCTOR(side: Number):
        SET this.side = side
    END CONSTRUCTOR

    FUNCTION area() -> Number:
        RETURN this.side * this.side
    END FUNCTION
END MODULE

// Client code works with any Shape:
FUNCTION printArea(shape: Shape) -> Void:
    OUTPUT "Area: " + shape.area()
END FUNCTION
```

```
Type Hierarchy (LSP Compliant):

        [Shape]  <--- interface with area() contract
        /     \
  [Rectangle] [Square]   <--- both fulfill contract independently

  Any Shape can be passed to printArea() without surprise.
```

### LSP Rules Summary

```
+--------------------------------------------------+
| LSP Contract Rules                               |
+--------------------------------------------------+
| 1. Preconditions cannot be strengthened           |
| 2. Postconditions cannot be weakened              |
| 3. Invariants of parent must be preserved         |
| 4. History constraint: subtypes cannot add        |
|    state mutations the parent forbids             |
+--------------------------------------------------+
```

---

## I - Interface Segregation Principle (ISP)

### Definition

No client should be forced to depend on methods it does not use. Prefer many
small, specific interfaces over one large, general-purpose interface.

### Real-World Analogy

A universal remote with 200 buttons is overwhelming when you only need
volume and channel. A simple remote with just the buttons you need is better.
Different users get different remotes suited to their needs.

### Violation

```
INTERFACE Worker:
    FUNCTION work() -> Void
    FUNCTION eat() -> Void
    FUNCTION sleep() -> Void
    FUNCTION attendMeeting() -> Void
    FUNCTION writeReport() -> Void
END INTERFACE

MODULE Robot IMPLEMENTS Worker:
    FUNCTION work() -> Void:
        // Robot works
    END FUNCTION

    FUNCTION eat() -> Void:
        // ERROR: Robots do not eat!
        RAISE NotApplicableError
    END FUNCTION

    FUNCTION sleep() -> Void:
        // ERROR: Robots do not sleep!
        RAISE NotApplicableError
    END FUNCTION

    FUNCTION attendMeeting() -> Void:
        // ERROR: Robots do not attend meetings!
        RAISE NotApplicableError
    END FUNCTION

    FUNCTION writeReport() -> Void:
        // ERROR: Robots do not write reports!
        RAISE NotApplicableError
    END FUNCTION
END MODULE
```

Robot is forced to implement methods that make no sense for it.

### Correct Implementation

```
INTERFACE Workable:
    FUNCTION work() -> Void
END INTERFACE

INTERFACE Feedable:
    FUNCTION eat() -> Void
END INTERFACE

INTERFACE Restable:
    FUNCTION sleep() -> Void
END INTERFACE

INTERFACE MeetingAttendee:
    FUNCTION attendMeeting() -> Void
END INTERFACE

MODULE HumanWorker IMPLEMENTS Workable, Feedable, Restable, MeetingAttendee:
    FUNCTION work() -> Void:
        // Human works
    END FUNCTION
    FUNCTION eat() -> Void:
        // Human eats
    END FUNCTION
    FUNCTION sleep() -> Void:
        // Human sleeps
    END FUNCTION
    FUNCTION attendMeeting() -> Void:
        // Human attends meeting
    END FUNCTION
END MODULE

MODULE Robot IMPLEMENTS Workable:
    FUNCTION work() -> Void:
        // Robot works -- only implements what it needs
    END FUNCTION
END MODULE
```

```
Interface Segregation Diagram:

  BEFORE (fat interface):
  +------------------+
  |     Worker       |
  | work()           |
  | eat()            |  <-- Robot forced to implement all
  | sleep()          |
  | attendMeeting()  |
  | writeReport()    |
  +------------------+

  AFTER (segregated interfaces):
  [Workable]   [Feedable]   [Restable]   [MeetingAttendee]
     |  |          |             |               |
  [Robot] [HumanWorker]-----+---+---------------+
          (implements all four)
```

---

## D - Dependency Inversion Principle (DIP)

### Definition

High-level modules should not depend on low-level modules. Both should depend
on abstractions. Abstractions should not depend on details. Details should
depend on abstractions.

### Real-World Analogy

A lamp does not depend on a specific brand of light bulb. It depends on a
standard socket (abstraction). Any bulb fitting that socket works. The high-
level device (lamp) and low-level device (bulb) both depend on the socket
standard.

### Violation

```
MODULE MySQLDatabase:
    FUNCTION query(sql: Text) -> List<Record>:
        // Directly queries MySQL
    END FUNCTION
END MODULE

MODULE OrderService:
    SET db = NEW MySQLDatabase()       // Direct dependency on concrete type

    FUNCTION getOrders() -> List<Order>:
        SET records = CALL db.query("SELECT * FROM orders")
        RETURN mapToOrders(records)
    END FUNCTION
END MODULE
```

```
Dependency Direction (WRONG):

  [OrderService]  --->  [MySQLDatabase]
   (high-level)          (low-level)

  High-level depends directly on low-level detail.
```

### Correct Implementation

```
INTERFACE DataRepository:
    FUNCTION findAll(entity: Text) -> List<Record>
    FUNCTION findById(entity: Text, id: Text) -> Record
    FUNCTION save(entity: Text, data: Record) -> Void
END INTERFACE

MODULE MySQLRepository IMPLEMENTS DataRepository:
    FUNCTION findAll(entity: Text) -> List<Record>:
        RETURN query("SELECT * FROM " + entity)
    END FUNCTION
    FUNCTION findById(entity: Text, id: Text) -> Record:
        RETURN query("SELECT * FROM " + entity + " WHERE id = " + id)
    END FUNCTION
    FUNCTION save(entity: Text, data: Record) -> Void:
        // Insert into MySQL
    END FUNCTION
END MODULE

MODULE OrderService:
    SET repository: DataRepository     // Depends on abstraction

    CONSTRUCTOR(repo: DataRepository):
        SET repository = repo
    END CONSTRUCTOR

    FUNCTION getOrders() -> List<Order>:
        SET records = CALL repository.findAll("orders")
        RETURN mapToOrders(records)
    END FUNCTION
END MODULE

// Usage -- injection at composition root:
SET repo = NEW MySQLRepository()
SET service = NEW OrderService(repo)
```

```
Dependency Direction (CORRECT):

  [OrderService]  --->  [DataRepository]  <---  [MySQLRepository]
   (high-level)          (abstraction)           (low-level)

  Both depend on the abstraction. Neither depends on the other.

  Swap to another implementation:
  [OrderService]  --->  [DataRepository]  <---  [PostgresRepository]
                                          <---  [InMemoryRepository]
```

### DIP and Dependency Injection

```
+------------------------------------------------------+
| Dependency Injection Styles                          |
+------------------------------------------------------+
| Constructor Injection:                               |
|   SET service = NEW Service(dependency)              |
|                                                      |
| Setter Injection:                                    |
|   SET service = NEW Service()                        |
|   CALL service.setDependency(dependency)             |
|                                                      |
| Interface Injection:                                 |
|   MODULE Service IMPLEMENTS Injectable:              |
|       FUNCTION inject(dep: Dependency) -> Void       |
+------------------------------------------------------+
```

---

## Principles Working Together

```
Relationship Map:

  SRP: "Each module has one reason to change"
   |
   +--> OCP: "Extend behavior without modifying existing code"
   |     |
   |     +--> DIP: "Depend on abstractions to enable extension"
   |           |
   |           +--> ISP: "Keep abstractions focused and small"
   |
   +--> LSP: "Subtypes must honor parent contracts"
              |
              +--> ISP: "Small interfaces are easier to implement correctly"
```

### When Principles Conflict

No principle should be applied dogmatically. Consider trade-offs:

```
+----------------------------------------------+
| Situation              | Prioritize           |
+----------------------------------------------+
| Small, stable module   | Simplicity over SRP  |
| Rapidly changing code  | OCP and DIP          |
| Deep type hierarchies  | LSP carefully         |
| Few implementors       | ISP may be overkill   |
| Prototype / MVP        | YAGNI over all SOLID  |
+----------------------------------------------+
```

---

## Summary Checklist

```
+---+-------------------------------------------+------------------+
| # | Principle                                 | Key Question     |
+---+-------------------------------------------+------------------+
| S | Single Responsibility                     | Does this module |
|   |                                           | have >1 reason   |
|   |                                           | to change?       |
+---+-------------------------------------------+------------------+
| O | Open/Closed                               | Can I add new    |
|   |                                           | behavior without |
|   |                                           | editing this?    |
+---+-------------------------------------------+------------------+
| L | Liskov Substitution                       | Can subtypes     |
|   |                                           | replace parents  |
|   |                                           | without surprise?|
+---+-------------------------------------------+------------------+
| I | Interface Segregation                     | Are clients      |
|   |                                           | forced to depend |
|   |                                           | on unused methods|
+---+-------------------------------------------+------------------+
| D | Dependency Inversion                      | Do high-level    |
|   |                                           | modules depend on|
|   |                                           | low-level ones?  |
+---+-------------------------------------------+------------------+
```

Applying SOLID principles consistently leads to systems that are easier to
test, extend, and maintain over their entire lifecycle.
