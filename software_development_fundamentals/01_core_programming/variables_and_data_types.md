# Variables and Data Types

## Overview

Variables are named storage locations that hold data during program execution. They serve as
the fundamental building blocks for all computation, allowing programs to store, retrieve,
and manipulate information. Every variable has a name (identifier), a value, and a type that
determines what operations can be performed on it.

Data types classify values into categories that define their behavior, memory requirements,
and the operations that can be applied to them. Understanding the type system of any
programming environment is essential because it governs how data flows through a program,
how errors are detected, and how memory is allocated and managed.

The relationship between variables and types varies across languages and paradigms. Some
environments bind types rigidly at compile time, while others allow types to shift
dynamically at runtime. Mastering these concepts enables developers to write correct,
efficient, and maintainable code regardless of which language they work in.

## Key Concepts

### Primitive Types

Primitive types represent the simplest, indivisible units of data. They are directly
supported by the underlying hardware and typically stored inline in memory.

| Type      | Description                        | Example Values         |
|-----------|------------------------------------|------------------------|
| Integer   | Whole numbers                      | -42, 0, 1, 2048       |
| Float     | Decimal / floating-point numbers   | 3.14, -0.001, 1.0e10  |
| Boolean   | Logical true or false              | TRUE, FALSE            |
| Character | Single text symbol                 | 'A', '7', '$'         |
| String    | Sequence of characters             | "Hello", "42"          |
| Null/Nil  | Absence of a value                 | NULL, NIL, NOTHING     |

### Composite Types

Composite types combine multiple values into a single structure.

- **Array / List**: Ordered collection of elements accessed by index.
- **Record / Struct**: Named group of fields, each with its own type.
- **Map / Dictionary**: Collection of key-value pairs.
- **Tuple**: Fixed-size, ordered group of heterogeneous values.
- **Set**: Unordered collection of unique elements.
- **Enum**: Named set of distinct constant values.

### Type Systems: Static vs Dynamic

```
+-----------------------------+-------------------------------+
|       STATIC TYPING         |        DYNAMIC TYPING         |
+-----------------------------+-------------------------------+
| Types checked at COMPILE    | Types checked at RUNTIME      |
| time (before execution).    | (during execution).           |
|                             |                               |
| Variable types declared     | Variable types inferred from  |
| explicitly or inferred      | the value currently held.     |
| once at definition.         |                               |
|                             |                               |
| Catches type errors early.  | More flexible but errors      |
|                             | surface later.                |
+-----------------------------+-------------------------------+
```

In a statically typed system, a variable's type is fixed:

```
DECLARE age: Integer = 30
SET age = "thirty"          // ERROR at compile time
```

In a dynamically typed system, a variable can change type:

```
SET age = 30                // age is Integer
SET age = "thirty"          // age is now String -- no error
```

### Type Systems: Strong vs Weak

Strong and weak typing describe how strictly a language enforces type rules.

- **Strong typing**: Operations between incompatible types are rejected.
- **Weak typing**: Implicit conversions happen automatically, sometimes with surprising results.

```
// STRONG typing behavior
SET result = "5" + 3        // ERROR: cannot add String and Integer

// WEAK typing behavior
SET result = "5" + 3        // result = "53" (implicit coercion to String)
                            // OR result = 8 (implicit coercion to Integer)
```

### Type Coercion

Type coercion is the automatic or explicit conversion of a value from one type to another.

```
// Implicit coercion (automatic)
SET x: Float = 5            // Integer 5 coerced to Float 5.0

// Explicit coercion (manual cast)
SET y: Integer = CAST(3.7 AS Integer)   // y = 3 (truncated)
SET s: String = TO_STRING(42)           // s = "42"
SET n: Integer = TO_INTEGER("99")       // n = 99
```

**Widening** (safe): Integer -> Float, Character -> String
**Narrowing** (risky): Float -> Integer, String -> Integer

### Value Semantics vs Reference Semantics

```
VALUE SEMANTICS                     REFERENCE SEMANTICS
+----------+                        +----------+
| x =  42  |  (independent copy)   | x = ----+---> [ 42 ]
+----------+                        +----------+       ^
| y =  42  |  (own copy of 42)     | y = ----+--------+
+----------+                        +----------+  (shared reference)

Changing y does NOT affect x.       Changing y's target DOES affect x.
```

```
// Value semantics
SET a = 10
SET b = a          // b gets a COPY of a's value
SET b = 20         // a is still 10

// Reference semantics
SET list1 = [1, 2, 3]
SET list2 = list1  // list2 points to the SAME data
APPEND list2, 4    // list1 is now [1, 2, 3, 4] too!
```

## Pseudocode Examples

### Variable Declaration and Assignment

```
// Declaration with explicit type
DECLARE name: String
DECLARE count: Integer
DECLARE price: Float
DECLARE active: Boolean

// Declaration with initialization
SET name = "Alice"
SET count = 0
SET price = 19.99
SET active = TRUE

// Multiple assignment
SET x = 0
SET y = 0
SET z = 0
```

### Working with Composite Types

```
// Array / List
SET scores: List<Integer> = [85, 92, 78, 95, 88]
SET firstScore = scores[0]          // 85
SET length = LENGTH(scores)         // 5

// Record / Struct
DEFINE RECORD Point:
    x: Float
    y: Float
END RECORD

SET origin: Point = Point(0.0, 0.0)
SET p: Point = Point(3.5, -2.1)
PRINT p.x                          // 3.5

// Map / Dictionary
SET ages: Map<String, Integer> = {
    "Alice" -> 30,
    "Bob"   -> 25,
    "Carol" -> 28
}
SET bobAge = ages["Bob"]            // 25

// Enum
DEFINE ENUM Direction:
    NORTH, SOUTH, EAST, WEST
END ENUM

SET heading: Direction = Direction.NORTH
```

### Constants and Immutability

```
// Constant -- value cannot change after initialization
CONSTANT PI: Float = 3.14159265
CONSTANT MAX_RETRIES: Integer = 3

SET PI = 3.0    // ERROR: cannot reassign a constant

// Immutable data structure
SET frozenList = FREEZE([1, 2, 3])
APPEND frozenList, 4   // ERROR: structure is immutable
```

### Type Inference

```
// The system infers the type from the assigned value
SET message = "hello"       // inferred as String
SET count = 42              // inferred as Integer
SET ratio = 0.75            // inferred as Float
SET flag = TRUE             // inferred as Boolean
```

### Generics Concept

Generics allow writing type-safe code that works with any data type.

```
FUNCTION first<T>(collection: List<T>) -> T:
    RETURN collection[0]
END FUNCTION

SET names: List<String> = ["Alice", "Bob"]
SET firstName: String = first(names)      // "Alice"

SET numbers: List<Integer> = [10, 20, 30]
SET firstNum: Integer = first(numbers)    // 10
```

## Visual Representation

### Memory Layout of Common Types

```
MEMORY ADDRESS    CONTENT             DESCRIPTION
+-----------+---+------------------+---------------------------+
| 0x1000    |   | 00 00 00 2A     | Integer: 42               |
+-----------+---+------------------+---------------------------+
| 0x1004    |   | 40 49 0F DB     | Float: 3.14159            |
+-----------+---+------------------+---------------------------+
| 0x1008    |   | 01              | Boolean: TRUE             |
+-----------+---+------------------+---------------------------+
| 0x1009    |   | 41              | Character: 'A'            |
+-----------+---+------------------+---------------------------+
| 0x100A    |   | PTR -> 0x2000   | String reference ------+  |
+-----------+---+------------------+-------------------------|-+
                                                             |
| 0x2000    |   | 05 48 65 6C 6C  | String data: "Hello"   <-+
|           |   | 6F              | (length + chars)          |
+-----------+---+------------------+---------------------------+
```

### Type Hierarchy

```
            +-----------+
            |   ANY     |
            +-----+-----+
                  |
       +----------+----------+
       |                     |
  +----+----+          +-----+-----+
  |PRIMITIVE|          | COMPOSITE |
  +----+----+          +-----+-----+
       |                     |
  +----+----+----+     +-----+-----+-----+
  |    |    |    |     |     |     |     |
 Int Float Bool Char  Array Record Map  Enum
```

## Common Pitfalls and Best Practices

### Pitfalls

1. **Uninitialized variables**: Using a variable before assigning it a value can produce
   garbage data or runtime errors. Always initialize variables.

2. **Floating-point precision**: `0.1 + 0.2` may not equal `0.3` exactly due to binary
   representation. Never compare floats for exact equality.

3. **Null reference errors**: Accessing a member of a null/nil variable causes crashes.
   Check for null before dereferencing.

4. **Unintended aliasing**: With reference semantics, two variables may share the same
   data. Modifying one unintentionally modifies the other.

5. **Implicit coercion surprises**: Relying on automatic type conversion can produce
   unexpected results, especially with string-number conversions.

### Best Practices

1. **Use meaningful names**: `SET customerAge = 34` not `SET x = 34`.
2. **Prefer constants**: If a value should not change, declare it as constant.
3. **Minimize scope**: Declare variables as close to their first use as possible.
4. **Be explicit about types**: Even in dynamically typed environments, clear types
   improve readability.
5. **Copy when needed**: When passing composite data, make explicit copies if you do
   not want shared mutation.

## Real-World Applications

- **Configuration management**: Constants and enums define application settings.
- **Data validation**: Type checks ensure user input matches expected formats.
- **Serialization**: Converting structured data (records, maps) to text for storage
  or network transmission.
- **Database mapping**: Records model database rows; type systems enforce schema rules.
- **API contracts**: Generics and strong typing ensure type safety across service
  boundaries.

## Related Topics

- [Control Flow](control_flow.md) -- Using variables in conditional and loop logic
- [Functions and Scope](functions_and_scope.md) -- Variable lifetime and visibility rules
- [Error Handling](error_handling.md) -- Null/nil safety and type-related errors
- [Object-Oriented Programming](object_oriented_programming.md) -- Records evolve into classes

## Practice Problems

1. **Type identification**: Given the values `42`, `"42"`, `42.0`, `TRUE`, and `'4'`,
   identify the type of each.

2. **Value vs reference**: Write pseudocode that demonstrates the difference between
   copying a list by value and by reference. Show what happens when the copy is modified.

3. **Safe coercion**: Write a function `SAFE_TO_INTEGER(value: String) -> Integer` that
   converts a string to an integer, returning `0` if the conversion fails.

4. **Generic container**: Design a generic `Stack<T>` with `PUSH`, `POP`, and `PEEK`
   operations using pseudocode.

5. **Enum state machine**: Define an enum `TrafficLight` with states RED, YELLOW, GREEN
   and write a function that returns the next state in the cycle.

6. **Immutability exercise**: Rewrite a block of code that mutates a list in place so
   that it instead creates new lists at each step, preserving the original.

7. **Precision trap**: Write pseudocode that demonstrates the floating-point precision
   problem and propose a workaround using integer arithmetic or an epsilon comparison.
