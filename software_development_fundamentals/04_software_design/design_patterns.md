# Design Patterns (Gang of Four)

## Overview

Design patterns are reusable solutions to common problems in software design.
The 23 patterns cataloged by Gamma, Helm, Johnson, and Vlissides (the "Gang of
Four" or GoF) in 1994 are organized into three categories.

```
+------------------------------------------------------------------+
|                        DESIGN PATTERNS                           |
+------------------------------------------------------------------+
| CREATIONAL (5)    | STRUCTURAL (7)     | BEHAVIORAL (11)         |
|-------------------|--------------------+-------------------------|
| Singleton         | Adapter            | Chain of Responsibility |
| Factory Method    | Bridge             | Command                 |
| Abstract Factory  | Composite          | Iterator                |
| Builder           | Decorator          | Mediator                |
| Prototype         | Facade             | Memento                 |
|                   | Flyweight          | Observer                |
|                   | Proxy              | State                   |
|                   |                    | Strategy                |
|                   |                    | Template Method         |
|                   |                    | Visitor                 |
|                   |                    | Interpreter             |
+------------------------------------------------------------------+
```

---

# Creational Patterns

## 1. Singleton

**Intent:** Ensure a module has only one instance and provide a global point
of access to it.

```
  [Client A] --+
               +--> [Singleton Instance]
  [Client B] --+
```

```
MODULE DatabasePool:
    SET instance: DatabasePool = NULL

    FUNCTION getInstance() -> DatabasePool:
        IF instance IS NULL THEN
            SET instance = NEW DatabasePool()
        END IF
        RETURN instance
    END FUNCTION
END MODULE
```

**Use when:** Exactly one instance must coordinate actions (connection pools,
configuration managers, registries).

---

## 2. Factory Method

**Intent:** Define an interface for creating objects, but let submodules decide
which type to instantiate.

```
  [Creator] ---------> [Product]
      ^                    ^
      |                    |
  [ConcreteCreator] -> [ConcreteProduct]
```

```
INTERFACE Transport:
    FUNCTION deliver() -> Void
END INTERFACE

MODULE Logistics:
    FUNCTION createTransport() -> Transport:
        // Submodules override this
    END FUNCTION
END MODULE

MODULE RoadLogistics EXTENDS Logistics:
    FUNCTION createTransport() -> Transport:
        RETURN NEW Truck()
    END FUNCTION
END MODULE
```

**Use when:** A module cannot anticipate the type of objects it must create.

---

## 3. Abstract Factory

**Intent:** Provide an interface for creating families of related objects
without specifying their concrete types.

```
  [AbstractFactory] ----> [ProductA]
       |                  [ProductB]
       v
  [ConcreteFactory1] --> [ProductA1, ProductB1]
  [ConcreteFactory2] --> [ProductA2, ProductB2]
```

```
INTERFACE UIFactory:
    FUNCTION createButton() -> Button
    FUNCTION createCheckbox() -> Checkbox
END INTERFACE

MODULE DarkThemeFactory IMPLEMENTS UIFactory:
    FUNCTION createButton() -> Button: RETURN NEW DarkButton() END FUNCTION
    FUNCTION createCheckbox() -> Checkbox: RETURN NEW DarkCheckbox() END FUNCTION
END MODULE
```

**Use when:** The system must work with multiple families of related products.

---

## 4. Builder

**Intent:** Separate the construction of a complex object from its
representation, allowing the same construction process to create different
representations.

```
  [Director] ---uses---> [Builder]
                            ^
                            |
                    [ConcreteBuilder] --builds--> [Product]
```

```
MODULE QueryBuilder:
    SET table, conditions, sortField: Text

    FUNCTION from(t: Text) -> QueryBuilder: SET table = t; RETURN this END FUNCTION
    FUNCTION where(c: Text) -> QueryBuilder: SET conditions = c; RETURN this END FUNCTION
    FUNCTION orderBy(f: Text) -> QueryBuilder: SET sortField = f; RETURN this END FUNCTION
    FUNCTION build() -> Query: RETURN NEW Query(table, conditions, sortField) END FUNCTION
END MODULE
```

**Use when:** An object requires many optional configuration steps.

---

## 5. Prototype

**Intent:** Create new objects by copying an existing instance (prototype)
rather than constructing from scratch.

```
  [Prototype] --clone()--> [Copy 1]
      |
      +------clone()--> [Copy 2]
```

```
MODULE ShapePrototype:
    SET color, size, position: Any

    FUNCTION clone() -> ShapePrototype:
        SET copy = NEW ShapePrototype()
        SET copy.color = this.color
        SET copy.size = this.size
        SET copy.position = this.position
        RETURN copy
    END FUNCTION
END MODULE
```

**Use when:** Creating an object is expensive and a similar one already exists.

---

# Structural Patterns

## 6. Adapter

**Intent:** Convert the interface of a module into another interface that
clients expect, enabling incompatible interfaces to work together.

```
  [Client] ---> [Target Interface] <--- [Adapter] ---> [Adaptee]
```

```
INTERFACE ModernLogger:
    FUNCTION log(level: Text, message: Text) -> Void
END INTERFACE

MODULE LegacyLoggerAdapter IMPLEMENTS ModernLogger:
    SET legacyLogger: OldLogger

    FUNCTION log(level: Text, message: Text) -> Void:
        CALL legacyLogger.writeToFile(level + ": " + message)
    END FUNCTION
END MODULE
```

**Use when:** You need to use an existing module whose interface does not match
what your code expects.

---

## 7. Bridge

**Intent:** Decouple an abstraction from its implementation so that the two can
vary independently.

```
  [Abstraction] ----has----> [Implementation]
       ^                          ^
       |                          |
  [RefinedAbstr.]        [ConcreteImpl A]
                         [ConcreteImpl B]
```

```
INTERFACE Renderer:
    FUNCTION renderShape(shape: Text, x: Number, y: Number) -> Void
END INTERFACE

MODULE Circle:
    SET renderer: Renderer
    SET x, y, radius: Number

    FUNCTION draw() -> Void:
        CALL renderer.renderShape("circle", x, y)
    END FUNCTION
END MODULE
```

**Use when:** You want to avoid a permanent binding between abstraction and
implementation (e.g., supporting multiple platforms).

---

## 8. Composite

**Intent:** Compose objects into tree structures to represent part-whole
hierarchies. Let clients treat individual objects and compositions uniformly.

```
       [Component]
       /         \
  [Leaf]     [Composite]
              /    |    \
          [Leaf] [Leaf] [Composite]
                          |
                        [Leaf]
```

```
INTERFACE FileSystemNode:
    FUNCTION getSize() -> Number
END INTERFACE

MODULE File IMPLEMENTS FileSystemNode:
    SET size: Number
    FUNCTION getSize() -> Number: RETURN size END FUNCTION
END MODULE

MODULE Directory IMPLEMENTS FileSystemNode:
    SET children: List<FileSystemNode>
    FUNCTION getSize() -> Number:
        SET total = 0
        FOR EACH child IN children DO
            SET total = total + child.getSize()
        END FOR
        RETURN total
    END FUNCTION
END MODULE
```

**Use when:** You need to represent hierarchies where containers and
contents share the same interface.

---

## 9. Decorator

**Intent:** Attach additional responsibilities to an object dynamically.
Decorators provide a flexible alternative to subtyping for extending behavior.

```
  [Component]
       ^
       |
  [Decorator] ------has------> [Component]
       ^
       |
  [ConcreteDecoratorA]
  [ConcreteDecoratorB]
```

```
INTERFACE DataSource:
    FUNCTION read() -> Text
    FUNCTION write(data: Text) -> Void
END INTERFACE

MODULE EncryptionDecorator IMPLEMENTS DataSource:
    SET wrapped: DataSource
    FUNCTION read() -> Text: RETURN decrypt(wrapped.read()) END FUNCTION
    FUNCTION write(data: Text) -> Void: CALL wrapped.write(encrypt(data)) END FUNCTION
END MODULE

// Usage: stack decorators
SET source = NEW EncryptionDecorator(NEW CompressionDecorator(NEW FileSource("data.txt")))
```

**Use when:** You need to add behavior to individual objects without affecting
other objects of the same type.

---

## 10. Facade

**Intent:** Provide a unified, simplified interface to a set of interfaces in a
subsystem, making the subsystem easier to use.

```
  [Client] ---> [Facade] --+--> [Subsystem A]
                            +--> [Subsystem B]
                            +--> [Subsystem C]
```

```
MODULE VideoConverter:
    FUNCTION convert(filename: Text, targetFormat: Text) -> File:
        SET source = CALL CodecFactory.extract(filename)
        SET audio = CALL AudioMixer.process(source)
        SET video = CALL BitrateReader.read(source)
        SET result = CALL Encoder.encode(video, audio, targetFormat)
        RETURN result
    END FUNCTION
END MODULE
```

**Use when:** You want to provide a simple interface to a complex subsystem.

---

## 11. Flyweight

**Intent:** Use sharing to support large numbers of fine-grained objects
efficiently by separating intrinsic (shared) from extrinsic (unique) state.

```
  [FlyweightFactory] --manages--> [SharedFlyweight Pool]
                                        |
  [Client] passes extrinsic state --> [Flyweight].draw(extrinsicState)
```

```
MODULE TreeFactory:
    SET treeTypes: Map<Text, TreeType>

    FUNCTION getType(species: Text, color: Text, texture: Text) -> TreeType:
        SET key = species + color + texture
        IF key NOT IN treeTypes THEN
            SET treeTypes[key] = NEW TreeType(species, color, texture)
        END IF
        RETURN treeTypes[key]
    END FUNCTION
END MODULE
```

**Use when:** An application uses a huge number of similar objects and memory
is a concern.

---

## 12. Proxy

**Intent:** Provide a surrogate or placeholder for another object to control
access to it.

```
  [Client] ---> [Proxy] --controls-access--> [RealSubject]
```

```
MODULE CachedWeatherService IMPLEMENTS WeatherService:
    SET realService: WeatherService
    SET cache: Map<Text, WeatherData>

    FUNCTION getWeather(city: Text) -> WeatherData:
        IF city IN cache AND cache[city].age < 3600 THEN
            RETURN cache[city]
        END IF
        SET data = CALL realService.getWeather(city)
        SET cache[city] = data
        RETURN data
    END FUNCTION
END MODULE
```

**Use when:** You need lazy initialization, access control, logging, or
caching around an object.

---

# Behavioral Patterns

## 13. Chain of Responsibility

**Intent:** Pass a request along a chain of handlers. Each handler decides
whether to process the request or pass it to the next handler.

```
  [Request] --> [Handler A] --> [Handler B] --> [Handler C] --> ...
```

```
INTERFACE Handler:
    FUNCTION setNext(h: Handler) -> Handler
    FUNCTION handle(request: Request) -> Response
END INTERFACE

MODULE AuthHandler IMPLEMENTS Handler:
    SET next: Handler
    FUNCTION handle(request: Request) -> Response:
        IF NOT CALL isAuthenticated(request) THEN RETURN Error(401) END IF
        IF next IS NOT NULL THEN RETURN next.handle(request) END IF
    END FUNCTION
END MODULE
```

**Use when:** Multiple objects may handle a request and the handler is not
known in advance.

---

## 14. Command

**Intent:** Encapsulate a request as an object, allowing parameterization of
clients, queuing, logging, and undo operations.

```
  [Invoker] --has--> [Command] --executes--> [Receiver]
```

```
INTERFACE Command:
    FUNCTION execute() -> Void
    FUNCTION undo() -> Void
END INTERFACE

MODULE PasteCommand IMPLEMENTS Command:
    SET editor: TextEditor
    SET backup: Text
    FUNCTION execute() -> Void: SET backup = editor.text; CALL editor.paste() END FUNCTION
    FUNCTION undo() -> Void: SET editor.text = backup END FUNCTION
END MODULE
```

**Use when:** You need undo/redo, queued execution, or operation logging.

---

## 15. Iterator

**Intent:** Provide a way to access elements of a collection sequentially
without exposing its underlying representation.

```
  [Client] --uses--> [Iterator] --traverses--> [Collection]
```

```
INTERFACE Iterator:
    FUNCTION hasNext() -> Boolean
    FUNCTION next() -> Element
END INTERFACE

MODULE TreeIterator IMPLEMENTS Iterator:
    SET stack: Stack<Node>
    FUNCTION hasNext() -> Boolean: RETURN stack IS NOT EMPTY END FUNCTION
    FUNCTION next() -> Element:
        SET node = CALL stack.pop()
        IF node.right IS NOT NULL THEN CALL stack.push(node.right) END IF
        IF node.left IS NOT NULL THEN CALL stack.push(node.left) END IF
        RETURN node.value
    END FUNCTION
END MODULE
```

**Use when:** You need to traverse a collection without exposing its structure.

---

## 16. Mediator

**Intent:** Define an object that encapsulates how a set of objects interact,
promoting loose coupling by preventing direct references.

```
  [ColleagueA] <--+--> [Mediator] <--+--> [ColleagueB]
                                     +--> [ColleagueC]
```

```
MODULE ChatRoom:
    SET participants: List<User>
    FUNCTION send(message: Text, sender: User) -> Void:
        FOR EACH user IN participants DO
            IF user <> sender THEN CALL user.receive(message) END IF
        END FOR
    END FUNCTION
END MODULE
```

**Use when:** Communication between objects is complex and tangled.

---

## 17. Memento

**Intent:** Capture and externalize an object's internal state so it can be
restored later, without violating encapsulation.

```
  [Originator] --creates--> [Memento] --stored-in--> [Caretaker]
```

```
MODULE EditorMemento:
    SET content: Text
    SET cursorPosition: Number
END MODULE

MODULE Editor:
    SET content: Text
    SET cursor: Number
    FUNCTION save() -> EditorMemento: RETURN NEW EditorMemento(content, cursor) END FUNCTION
    FUNCTION restore(m: EditorMemento) -> Void: SET content = m.content; SET cursor = m.cursorPosition END FUNCTION
END MODULE
```

**Use when:** You need to implement undo or snapshots of object state.

---

## 18. Observer

**Intent:** Define a one-to-many dependency so that when one object changes
state, all dependents are notified and updated automatically.

```
  [Subject] --notifies--> [Observer A]
      |                   [Observer B]
      +--notifies-------> [Observer C]
```

```
MODULE EventEmitter:
    SET listeners: Map<Text, List<Callable>>

    FUNCTION on(event: Text, callback: Callable) -> Void:
        CALL listeners[event].add(callback)
    END FUNCTION

    FUNCTION emit(event: Text, data: Any) -> Void:
        FOR EACH callback IN listeners[event] DO
            CALL callback(data)
        END FOR
    END FUNCTION
END MODULE
```

**Use when:** Changes in one object require updating others, and you do not
know how many objects need to be updated.

---

## 19. State

**Intent:** Allow an object to alter its behavior when its internal state
changes. The object will appear to change its type.

```
  [Context] --delegates--> [State]
                              ^
                          [StateA] [StateB] [StateC]
```

```
INTERFACE OrderState:
    FUNCTION next(order: Order) -> Void
    FUNCTION cancel(order: Order) -> Void
END INTERFACE

MODULE PendingState IMPLEMENTS OrderState:
    FUNCTION next(order: Order) -> Void: SET order.state = NEW ProcessingState() END FUNCTION
    FUNCTION cancel(order: Order) -> Void: SET order.state = NEW CancelledState() END FUNCTION
END MODULE
```

**Use when:** An object's behavior depends on its state and must change at
runtime based on that state.

---

## 20. Strategy

**Intent:** Define a family of algorithms, encapsulate each one, and make them
interchangeable. Strategy lets the algorithm vary independently from clients.

```
  [Context] --uses--> [Strategy]
                         ^
                     [StratA] [StratB] [StratC]
```

```
INTERFACE SortStrategy:
    FUNCTION sort(data: List<Any>) -> List<Any>
END INTERFACE

MODULE Sorter:
    SET strategy: SortStrategy
    FUNCTION execute(data: List<Any>) -> List<Any>:
        RETURN strategy.sort(data)
    END FUNCTION
END MODULE
```

**Use when:** You need multiple algorithms for a task and want to switch
between them at runtime.

---

## 21. Template Method

**Intent:** Define the skeleton of an algorithm in a base module, deferring
some steps to submodules. Submodules redefine certain steps without changing
the algorithm's structure.

```
  [AbstractModule]
    templateMethod() calls:  step1() -> step2() -> step3()
       ^
       |
  [ConcreteModule]
    overrides: step2()
```

```
MODULE DataMiner:
    FUNCTION mine(path: Text) -> Report:
        SET raw = CALL extractData(path)       // abstract
        SET parsed = CALL parseData(raw)       // abstract
        SET analysis = CALL analyze(parsed)    // concrete (shared)
        RETURN CALL generateReport(analysis)   // concrete (shared)
    END FUNCTION
END MODULE
```

**Use when:** Multiple modules share the same algorithm structure but differ
in specific steps.

---

## 22. Visitor

**Intent:** Represent an operation to be performed on elements of a structure.
Visitor lets you define a new operation without changing the element types.

```
  [Element] --accepts--> [Visitor]
     ^                      ^
  [ElementA]           [VisitorX]
  [ElementB]           [VisitorY]
```

```
INTERFACE Visitor:
    FUNCTION visitCircle(c: Circle) -> Void
    FUNCTION visitSquare(s: Square) -> Void
END INTERFACE

MODULE AreaCalculator IMPLEMENTS Visitor:
    SET totalArea: Number = 0
    FUNCTION visitCircle(c: Circle) -> Void: SET totalArea = totalArea + PI * c.radius ^ 2 END FUNCTION
    FUNCTION visitSquare(s: Square) -> Void: SET totalArea = totalArea + s.side ^ 2 END FUNCTION
END MODULE
```

**Use when:** You need to perform many unrelated operations on a structure
of objects and want to avoid polluting their interfaces.

---

## 23. Interpreter

**Intent:** Given a language, define a representation for its grammar along
with an interpreter that uses the representation to interpret sentences.

```
  [AbstractExpression]
       ^         ^
  [Terminal]  [NonTerminal] --has--> [AbstractExpression]
```

```
INTERFACE Expression:
    FUNCTION interpret(context: Map<Text, Number>) -> Number
END INTERFACE

MODULE NumberLiteral IMPLEMENTS Expression:
    SET value: Number
    FUNCTION interpret(ctx: Map<Text, Number>) -> Number: RETURN value END FUNCTION
END MODULE

MODULE AddExpression IMPLEMENTS Expression:
    SET left, right: Expression
    FUNCTION interpret(ctx: Map<Text, Number>) -> Number:
        RETURN left.interpret(ctx) + right.interpret(ctx)
    END FUNCTION
END MODULE
```

**Use when:** You need to interpret or evaluate expressions in a simple
domain-specific language.

---

## Pattern Selection Guide

```
+-------------------------------+----------------------------------+
| Problem                       | Consider These Patterns          |
+-------------------------------+----------------------------------+
| Creating objects flexibly     | Factory, Abstract Factory,       |
|                               | Builder, Prototype               |
+-------------------------------+----------------------------------+
| Ensuring one instance         | Singleton                        |
+-------------------------------+----------------------------------+
| Making incompatible things    | Adapter, Bridge, Facade          |
| work together                 |                                  |
+-------------------------------+----------------------------------+
| Adding behavior dynamically   | Decorator, Proxy, Chain of Resp. |
+-------------------------------+----------------------------------+
| Simplifying complex systems   | Facade, Mediator                 |
+-------------------------------+----------------------------------+
| Supporting undo               | Command, Memento                 |
+-------------------------------+----------------------------------+
| Varying algorithms            | Strategy, Template Method        |
+-------------------------------+----------------------------------+
| Reacting to state changes     | Observer, State                  |
+-------------------------------+----------------------------------+
| Traversing structures         | Iterator, Visitor, Composite     |
+-------------------------------+----------------------------------+
```

---

## Anti-Pattern Warning

Patterns are solutions to specific problems. Applying them without a
corresponding problem is itself an anti-pattern called "pattern-itis."

```
+----------------------------------------------------+
| BEFORE applying a pattern, verify:                 |
|                                                    |
| 1. The problem exists (not hypothetical)           |
| 2. The pattern solves THIS problem                 |
| 3. The added complexity is justified               |
| 4. A simpler solution does not suffice             |
+----------------------------------------------------+
```

Design patterns are tools in a toolbox, not goals in themselves. Use them when
they reduce complexity, not when they demonstrate cleverness.
