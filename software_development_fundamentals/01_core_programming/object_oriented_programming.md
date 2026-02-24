# Object-Oriented Programming

## Overview

Object-Oriented Programming (OOP) is a paradigm that organizes software around objects --
data structures that bundle state (fields/attributes) and behavior (methods/operations)
into cohesive units. Instead of thinking about a program as a sequence of procedures
operating on separate data, OOP models the problem domain as interacting objects that
communicate through well-defined interfaces.

OOP emerged to manage complexity in large software systems. By grouping related data and
behavior together, it reduces the cognitive load of understanding any single part of the
system. The four foundational pillars -- encapsulation, inheritance, polymorphism, and
abstraction -- provide a framework for building systems that are modular, extensible,
and maintainable.

While OOP is not the only way to structure software, its principles have shaped the design
of countless systems, frameworks, and architectures. Understanding OOP deeply is essential
for working with most modern codebases and for making informed decisions about when to use
(and when not to use) object-oriented design.

## Key Concepts

### Classes and Objects

A class is a blueprint that defines the structure and behavior of objects. An object is
a specific instance created from that blueprint.

```
DEFINE CLASS Dog:
    // Fields (state)
    name: String
    breed: String
    age: Integer

    // Constructor
    CONSTRUCTOR(name: String, breed: String, age: Integer):
        SET this.name = name
        SET this.breed = breed
        SET this.age = age
    END CONSTRUCTOR

    // Methods (behavior)
    METHOD bark() -> VOID:
        PRINT this.name + " says: Woof!"
    END METHOD

    METHOD ageInHumanYears() -> Integer:
        RETURN this.age * 7
    END METHOD
END CLASS

// Creating objects (instances)
SET myDog = NEW Dog("Rex", "Labrador", 3)
myDog.bark()                          // "Rex says: Woof!"
PRINT myDog.ageInHumanYears()         // 21
```

## The Four Pillars of OOP

### Pillar 1: Encapsulation

Encapsulation bundles data and the methods that operate on it together, and restricts
direct access to internal state. External code interacts only through a public interface.

```
DEFINE CLASS BankAccount:
    PRIVATE balance: Float
    PRIVATE accountNumber: String

    CONSTRUCTOR(accountNumber: String, initialBalance: Float):
        SET this.accountNumber = accountNumber
        SET this.balance = initialBalance
    END CONSTRUCTOR

    // Public interface
    PUBLIC METHOD deposit(amount: Float) -> VOID:
        IF amount <= 0 THEN
            RAISE Error("Deposit amount must be positive")
        END IF
        SET this.balance = this.balance + amount
    END METHOD

    PUBLIC METHOD withdraw(amount: Float) -> Boolean:
        IF amount > this.balance THEN
            RETURN FALSE
        END IF
        SET this.balance = this.balance - amount
        RETURN TRUE
    END METHOD

    PUBLIC METHOD getBalance() -> Float:
        RETURN this.balance
    END METHOD

    // Private helper -- not accessible from outside
    PRIVATE METHOD logTransaction(type: String, amount: Float) -> VOID:
        WRITE_LOG(this.accountNumber + ": " + type + " " + TO_STRING(amount))
    END METHOD
END CLASS

SET account = NEW BankAccount("ACC-001", 1000.00)
account.deposit(500.00)              // OK
PRINT account.getBalance()           // 1500.00
PRINT account.balance                // ERROR: balance is PRIVATE
```

**Why it matters**: Encapsulation protects invariants. The account balance can never
become negative because all modifications go through controlled methods.

### Pillar 2: Inheritance

Inheritance allows a new class (child/subclass) to inherit fields and methods from an
existing class (parent/superclass), enabling code reuse and hierarchical modeling.

```
DEFINE CLASS Animal:
    name: String
    sound: String

    CONSTRUCTOR(name: String, sound: String):
        SET this.name = name
        SET this.sound = sound
    END CONSTRUCTOR

    METHOD speak() -> String:
        RETURN this.name + " says " + this.sound
    END METHOD

    METHOD eat(food: String) -> VOID:
        PRINT this.name + " eats " + food
    END METHOD
END CLASS

DEFINE CLASS Cat EXTENDS Animal:
    indoor: Boolean

    CONSTRUCTOR(name: String, indoor: Boolean):
        SUPER("Cat: " + name, "Meow")
        SET this.indoor = indoor
    END CONSTRUCTOR

    METHOD purr() -> VOID:
        PRINT this.name + " purrs softly"
    END METHOD
END CLASS

SET whiskers = NEW Cat("Whiskers", TRUE)
PRINT whiskers.speak()    // "Cat: Whiskers says Meow" (inherited)
whiskers.eat("tuna")      // "Cat: Whiskers eats tuna" (inherited)
whiskers.purr()           // "Cat: Whiskers purrs softly" (own method)
```

### Pillar 3: Polymorphism

Polymorphism means "many forms" -- the same interface can behave differently depending on
the underlying type. This enables writing code that works with any object that conforms
to an expected interface.

```
DEFINE CLASS Shape:
    METHOD area() -> Float:
        RETURN 0.0
    END METHOD

    METHOD describe() -> String:
        RETURN "Shape with area " + TO_STRING(this.area())
    END METHOD
END CLASS

DEFINE CLASS Circle EXTENDS Shape:
    radius: Float

    CONSTRUCTOR(radius: Float):
        SET this.radius = radius
    END CONSTRUCTOR

    OVERRIDE METHOD area() -> Float:
        RETURN 3.14159 * this.radius * this.radius
    END METHOD
END CLASS

DEFINE CLASS Rectangle EXTENDS Shape:
    width: Float
    height: Float

    CONSTRUCTOR(width: Float, height: Float):
        SET this.width = width
        SET this.height = height
    END CONSTRUCTOR

    OVERRIDE METHOD area() -> Float:
        RETURN this.width * this.height
    END METHOD
END CLASS

// Polymorphism in action
FUNCTION totalArea(shapes: List<Shape>) -> Float:
    SET sum = 0.0
    FOR EACH shape IN shapes DO
        SET sum = sum + shape.area()    // calls the correct override
    END FOR
    RETURN sum
END FUNCTION

SET shapes = [NEW Circle(5.0), NEW Rectangle(4.0, 6.0), NEW Circle(3.0)]
PRINT totalArea(shapes)   // 78.54 + 24.0 + 28.27 = 130.81
```

### Pillar 4: Abstraction

Abstraction hides complex implementation details behind simple interfaces. Abstract
classes and interfaces define contracts without providing full implementations.

```
DEFINE ABSTRACT CLASS PaymentProcessor:
    // Abstract method -- must be implemented by subclasses
    ABSTRACT METHOD processPayment(amount: Float) -> Boolean

    // Concrete method -- shared by all subclasses
    METHOD formatReceipt(amount: Float) -> String:
        RETURN "Payment of $" + TO_STRING(amount) + " processed"
    END METHOD
END CLASS

DEFINE CLASS CreditCardProcessor EXTENDS PaymentProcessor:
    OVERRIDE METHOD processPayment(amount: Float) -> Boolean:
        // Credit card specific logic
        SET authorized = AUTHORIZE_CARD(this.cardNumber, amount)
        RETURN authorized
    END METHOD
END CLASS

DEFINE CLASS DigitalWalletProcessor EXTENDS PaymentProcessor:
    OVERRIDE METHOD processPayment(amount: Float) -> Boolean:
        // Digital wallet specific logic
        SET sufficient = CHECK_WALLET_BALANCE(this.walletId, amount)
        IF sufficient THEN
            DEDUCT_WALLET(this.walletId, amount)
        END IF
        RETURN sufficient
    END METHOD
END CLASS
```

## Class Hierarchy Diagram

```
                    +-------------------+
                    |    <<abstract>>   |
                    |      Vehicle      |
                    +-------------------+
                    | - make: String    |
                    | - model: String   |
                    | - year: Integer   |
                    +-------------------+
                    | + start(): VOID   |
                    | + stop(): VOID    |
                    | + fuelType():Str  | <<abstract>>
                    +--------+----------+
                             |
              +--------------+--------------+
              |                             |
    +---------+----------+      +-----------+---------+
    |     Car            |      |     Truck           |
    +--------------------+      +---------------------+
    | - doors: Integer   |      | - payload: Float    |
    | - trunk: Float     |      | - axles: Integer    |
    +--------------------+      +---------------------+
    | + fuelType(): Str  |      | + fuelType(): Str   |
    | + openTrunk(): VOID|      | + loadCargo(): VOID |
    +--------------------+      +---------------------+
              |
    +---------+----------+
    |   ElectricCar      |
    +--------------------+
    | - battery: Float   |
    +--------------------+
    | + fuelType(): Str  |
    | + charge(): VOID   |
    +--------------------+
```

## Interfaces and Contracts

Interfaces define a set of methods that a class must implement, without specifying how.
They enable polymorphism across unrelated class hierarchies.

```
DEFINE INTERFACE Serializable:
    METHOD toText() -> String
    METHOD fromText(data: String) -> VOID
END INTERFACE

DEFINE INTERFACE Comparable:
    METHOD compareTo(other: ANY) -> Integer
END INTERFACE

DEFINE CLASS Student IMPLEMENTS Serializable, Comparable:
    name: String
    gpa: Float

    METHOD toText() -> String:
        RETURN this.name + "," + TO_STRING(this.gpa)
    END METHOD

    METHOD fromText(data: String) -> VOID:
        SET parts = SPLIT(data, ",")
        SET this.name = parts[0]
        SET this.gpa = TO_FLOAT(parts[1])
    END METHOD

    METHOD compareTo(other: Student) -> Integer:
        IF this.gpa > other.gpa THEN RETURN 1 END IF
        IF this.gpa < other.gpa THEN RETURN -1 END IF
        RETURN 0
    END METHOD
END CLASS
```

## Composition vs Inheritance

```
INHERITANCE (IS-A)                    COMPOSITION (HAS-A)
+-------------------+                +-------------------+
|     Bird          |                |     Robot         |
+-------------------+                +-------------------+
| + fly()           |                | - engine: Engine  |
+--------+----------+                | - sensor: Sensor  |
         |                           | - arm: Arm        |
    +----+----+                      +-------------------+
    | Penguin |                      | + move(): VOID    |
    +---------+                      | + sense(): Data   |
    | + fly() | ??? Penguins         | + grab(): VOID    |
    |         | cannot fly!          +-------------------+
    +---------+
                                     Robot delegates behavior to
Inheritance can create               its components. Components
awkward hierarchies.                  can be swapped at runtime.
```

**Prefer composition over inheritance** when:
- The relationship is "has-a" rather than "is-a"
- You need to combine behaviors from multiple sources
- The hierarchy would be awkward or force unwanted behavior
- You want runtime flexibility to swap implementations

```
// Composition example
DEFINE CLASS Logger:
    METHOD log(message: String) -> VOID:
        PRINT "[LOG] " + message
    END METHOD
END CLASS

DEFINE CLASS EmailService:
    PRIVATE logger: Logger

    CONSTRUCTOR():
        SET this.logger = NEW Logger()
    END CONSTRUCTOR

    METHOD sendEmail(to: String, body: String) -> VOID:
        // ... send email logic ...
        this.logger.log("Email sent to " + to)
    END METHOD
END CLASS
```

## Method Dispatch

When a method is called on an object, the runtime determines which implementation to
execute based on the actual type of the object (dynamic dispatch).

```
SET animals: List<Animal> = [NEW Cat("Luna", TRUE), NEW Dog("Rex", "Lab", 3)]

FOR EACH animal IN animals DO
    PRINT animal.speak()
END FOR

// The runtime checks the actual type:
//   animals[0] is Cat  -> calls Cat.speak()
//   animals[1] is Dog  -> calls Dog.speak()
```

```
DISPATCH TABLE
+-----------+----------+---------+
| Type      | speak()  | eat()   |
+-----------+----------+---------+
| Animal    | Animal   | Animal  |   <- base implementations
| Cat       | Cat*     | Animal  |   <- Cat overrides speak()
| Dog       | Dog*     | Animal  |   <- Dog overrides speak()
+-----------+----------+---------+
  * = overridden method
```

## Connection to SOLID Principles

OOP is most effective when guided by the SOLID principles:

| Principle                  | Description                                    |
|----------------------------|------------------------------------------------|
| **S**ingle Responsibility  | A class should have only one reason to change   |
| **O**pen/Closed            | Open for extension, closed for modification     |
| **L**iskov Substitution    | Subtypes must be usable in place of their base  |
| **I**nterface Segregation  | Prefer small, specific interfaces over large ones|
| **D**ependency Inversion   | Depend on abstractions, not concrete classes     |

```
// Dependency Inversion example
// BAD: High-level module depends on concrete class
DEFINE CLASS OrderService:
    PRIVATE db = NEW MySQLDatabase()    // tightly coupled
END CLASS

// GOOD: High-level module depends on abstraction
DEFINE INTERFACE Database:
    METHOD save(data: Record) -> VOID
    METHOD find(id: String) -> Record
END INTERFACE

DEFINE CLASS OrderService:
    PRIVATE db: Database                // depends on interface

    CONSTRUCTOR(database: Database):
        SET this.db = database          // injected dependency
    END CONSTRUCTOR
END CLASS
```

## Design Patterns Preview

OOP enables well-known design patterns that solve recurring problems:

| Pattern    | Purpose                                          | Structure               |
|------------|--------------------------------------------------|-------------------------|
| Factory    | Create objects without specifying exact class     | Creator -> Product      |
| Observer   | Notify multiple objects of state changes          | Subject -> Observer     |
| Strategy   | Swap algorithms at runtime                        | Context -> Strategy     |
| Decorator  | Add behavior to objects dynamically               | Component -> Decorator  |
| Adapter    | Make incompatible interfaces work together        | Target <- Adapter       |

```
// Strategy pattern example
DEFINE INTERFACE SortStrategy:
    METHOD sort(data: List<Integer>) -> List<Integer>
END INTERFACE

DEFINE CLASS QuickSort IMPLEMENTS SortStrategy:
    METHOD sort(data: List<Integer>) -> List<Integer>:
        // quick sort implementation
    END METHOD
END CLASS

DEFINE CLASS MergeSort IMPLEMENTS SortStrategy:
    METHOD sort(data: List<Integer>) -> List<Integer>:
        // merge sort implementation
    END METHOD
END CLASS

DEFINE CLASS DataProcessor:
    PRIVATE strategy: SortStrategy

    METHOD setStrategy(s: SortStrategy) -> VOID:
        SET this.strategy = s
    END METHOD

    METHOD process(data: List<Integer>) -> List<Integer>:
        RETURN this.strategy.sort(data)
    END METHOD
END CLASS
```

## Common Pitfalls and Best Practices

### Pitfalls

1. **God objects**: Classes that know too much or do too much. Split them by
   responsibility.

2. **Deep inheritance hierarchies**: More than 2-3 levels deep becomes fragile and
   hard to reason about. Prefer composition.

3. **Leaky abstractions**: Exposing internal implementation details through the public
   interface defeats the purpose of encapsulation.

4. **Inheritance for code reuse only**: If there is no true "is-a" relationship, use
   composition instead.

5. **Anemic domain model**: Objects that are just data holders with no behavior,
   where all logic lives in external service classes.

6. **Violating Liskov Substitution**: A subclass that throws exceptions for inherited
   methods it does not support (like Penguin.fly()) is a design flaw.

### Best Practices

1. **Program to interfaces, not implementations**: Depend on abstract contracts.
2. **Favor composition over inheritance**: Build complex behavior by combining simple
   objects.
3. **Keep classes focused**: Each class should have a single, clear responsibility.
4. **Use access modifiers deliberately**: Make fields private by default; expose only
   what is necessary.
5. **Design for extension**: Use the Open/Closed Principle to allow adding features
   without modifying existing code.
6. **Encapsulate what varies**: Identify the parts of your design that change and
   separate them from what stays the same.

## Real-World Applications

- **GUI frameworks**: Widgets inherit from a base component; event handling uses
  polymorphism.
- **Game engines**: Entities with components (composition), inheritance for entity types,
  polymorphic update/render loops.
- **Web frameworks**: Controllers, models, and views structured as classes with clear
  interfaces.
- **Plugin systems**: Interfaces define plugin contracts; polymorphism loads and
  executes plugins at runtime.
- **Domain-Driven Design**: Entities, value objects, and aggregates model business
  concepts as objects.

## Related Topics

- [Variables and Data Types](variables_and_data_types.md) -- Fields are typed variables inside objects
- [Functions and Scope](functions_and_scope.md) -- Methods are functions bound to a class
- [Functional Programming](functional_programming.md) -- Alternative paradigm with different trade-offs
- [Error Handling](error_handling.md) -- Exception hierarchies use inheritance

## Practice Problems

1. **Shape hierarchy**: Design a class hierarchy for shapes (Circle, Rectangle, Triangle)
   with an abstract `Shape` base class. Include `area()` and `perimeter()` methods.

2. **Composition refactor**: Take a class that uses inheritance to share logging behavior
   and refactor it to use composition instead.

3. **Interface design**: Create a `Cacheable` interface and implement it for at least two
   unrelated classes (e.g., `UserProfile` and `SearchResult`).

4. **Strategy pattern**: Implement a text formatter that can switch between plain text,
   uppercase, and title case formatting strategies.

5. **Liskov violation**: Identify and fix a Liskov Substitution Principle violation in a
   given class hierarchy where a subclass breaks parent expectations.

6. **Observer pattern**: Design a simple event system where multiple listeners can
   subscribe to and receive notifications from an event emitter.

7. **SOLID audit**: Review a given class and identify which SOLID principles it violates,
   then refactor it to comply.
