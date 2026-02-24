# KISS - Keep It Simple, Stupid

## Definition

The KISS principle states that most systems work best when kept simple rather
than made complicated. Simplicity should be a key goal in design, and
unnecessary complexity should be avoided. Attributed to Kelly Johnson, lead
engineer at Lockheed Skunk Works, circa 1960.

In software, KISS means: prefer the straightforward solution. If two designs
solve the same problem, choose the one that is easier to understand, test,
and maintain.

---

## Measuring Complexity

### Cyclomatic Complexity

Cyclomatic complexity counts the number of independent paths through a piece
of code. Each decision point (IF, ELSE, WHILE, FOR, CASE) adds a path.

```
Complexity Rating Scale:

  1 - 5    | Simple       | Low risk, easy to test
  6 - 10   | Moderate     | Manageable, some care needed
  11 - 20  | Complex      | Difficult to test, refactor advised
  21+      | Very Complex | High risk, must be simplified
```

**Example -- counting complexity:**

```
FUNCTION processOrder(order: Order) -> Result:       // +1 base path
    IF order IS NULL THEN                             // +1
        RETURN Error("Null order")
    END IF
    IF order.items IS EMPTY THEN                      // +1
        RETURN Error("No items")
    END IF
    FOR EACH item IN order.items DO                   // +1
        IF item.quantity <= 0 THEN                    // +1
            RETURN Error("Bad quantity")
        END IF
        IF item.price < 0 THEN                        // +1
            RETURN Error("Negative price")
        END IF
    END FOR
    IF order.customer.isBlacklisted THEN              // +1
        RETURN Error("Blacklisted")
    END IF
    RETURN Success(order)
END FUNCTION

// Cyclomatic complexity = 7 (moderate)
```

### Other Complexity Indicators

```
+----------------------------------------------+
| Indicator              | Simplicity Target   |
+----------------------------------------------+
| Lines per function     | < 20                |
| Parameters per func    | < 4                 |
| Nesting depth          | < 3 levels          |
| Number of branches     | < 7                 |
| Dependencies per module| < 5                 |
+----------------------------------------------+
```

---

## Complex vs Simple Solutions

### Example 1: Configuration Parsing

**Complex (over-engineered):**

```
MODULE ConfigParser:
    SET plugins: List<ParserPlugin>
    SET middleware: List<Middleware>
    SET cache: CacheLayer
    SET validator: SchemaValidator
    SET transformer: DataTransformer

    FUNCTION parse(input: Text) -> Config:
        SET rawData = CALL tokenizer.tokenize(input)
        FOR EACH plugin IN plugins DO
            SET rawData = CALL plugin.preProcess(rawData)
        END FOR
        SET ast = CALL astBuilder.build(rawData)
        SET validated = CALL validator.validate(ast, schema)
        FOR EACH mw IN middleware DO
            SET validated = CALL mw.process(validated)
        END FOR
        SET result = CALL transformer.transform(validated)
        CALL cache.store(hash(input), result)
        RETURN result
    END FUNCTION
END MODULE
```

**Simple (sufficient for most cases):**

```
FUNCTION parseConfig(input: Text) -> Config:
    SET lines = SPLIT(input, NEWLINE)
    SET config = NEW EmptyMap()
    FOR EACH line IN lines DO
        IF line IS EMPTY OR STARTS_WITH(line, "#") THEN
            CONTINUE
        END IF
        SET parts = SPLIT(line, "=", limit: 2)
        SET config[TRIM(parts[0])] = TRIM(parts[1])
    END FOR
    RETURN config
END FUNCTION
```

The simple version handles the actual requirement. The complex version handles
hypothetical future requirements that may never arise.

### Example 2: User Lookup

**Complex (unnecessary abstraction layers):**

```
MODULE UserLookupServiceFactoryProvider:
    FUNCTION createLookupService(config: Config) -> UserLookupService:
        SET strategy = CALL StrategyResolver.resolve(config)
        SET adapter = CALL AdapterFactory.create(strategy)
        SET proxy = NEW CachingProxy(adapter)
        SET decorator = NEW LoggingDecorator(proxy)
        RETURN NEW UserLookupService(decorator)
    END FUNCTION
END MODULE
```

**Simple (direct and clear):**

```
FUNCTION findUserById(id: Text) -> User:
    SET user = CALL userRepository.getById(id)
    IF user IS NULL THEN
        RAISE UserNotFoundError(id)
    END IF
    RETURN user
END FUNCTION
```

---

## Decision Tree for Simplicity

```
                    Is this solution correct?
                           |
                    YES    |    NO
                    |            |
             Is it simple?    Fix correctness first.
                |
         YES   |   NO
         |          |
    Ship it.   Can it be simplified
               without losing correctness?
                    |
              YES   |   NO
              |          |
        Simplify.   Document the
                    necessary complexity
                    and move on.
```

---

## Common Complexity Traps

### 1. Speculative Generality

Building abstractions for use cases that do not yet exist.

```
BAD:  GenericDataProcessorWithPluggableStrategyAndMiddleware
GOOD: processOrders(orders: List<Order>) -> List<Result>
```

### 2. Premature Optimization

Optimizing before measuring, adding complexity for negligible gain.

```
BAD:  Hand-rolled hash map with custom collision resolution
      for a collection that will never exceed 20 items
GOOD: Use a standard list and linear search for 20 items
```

### 3. Gold Plating

Adding features or polish beyond what was requested.

```
BAD:  Adding 5 export formats when only CSV was requested
GOOD: Implement CSV export; add others when requested
```

### 4. Cargo Cult Architecture

Adopting patterns because "big companies use them" without understanding
whether the problem warrants them.

```
BAD:  Microservices for a single-developer project with 3 endpoints
GOOD: Monolith with clean module boundaries; split later if needed
```

---

## Simplicity vs Flexibility Trade-off

```
                        Flexibility
                            ^
                            |
                    +-------+-------+
                    |       |       |
                    | Over- | Sweet |
                    | eng.  | Spot  |
                    |       |       |
                    +-------+-------+
                    |       |       |
                    | Too   | Under-|
                    | rigid | eng.  |
                    |       |       |
                    +-------+-------+
                            |
                            +-----------> Simplicity

  The goal is the "Sweet Spot": simple enough to understand,
  flexible enough to evolve.
```

### Guidelines for the Sweet Spot

1. Start simple. Add complexity only when required by real needs.
2. Refactor when complexity accumulates, not preemptively.
3. Measure before optimizing. Profile before restructuring.
4. Ask: "Will someone reading this in 6 months understand it quickly?"

---

## Occam's Razor in Software

Occam's Razor: "Entities should not be multiplied beyond necessity."

In software terms: do not add modules, abstractions, patterns, layers,
or services beyond what the problem actually requires.

```
+----------------------------------------------------+
| Question                        | KISS Answer      |
+----------------------------------------------------+
| How many layers of abstraction? | As few as needed |
| How many design patterns?       | Only proven ones |
| How many configuration options? | Only requested   |
| How many services?              | Start with one   |
| How many parameters?            | < 4 per function |
+----------------------------------------------------+
```

---

## KISS in Practice

### Code Review Checklist

```
+-----------------------------------------------------------+
| KISS Review Checklist                                     |
+-----------------------------------------------------------+
| [ ] Can I explain this function in one sentence?          |
| [ ] Does this function do exactly one thing?              |
| [ ] Are there fewer than 4 parameters?                    |
| [ ] Is nesting depth 3 or fewer?                          |
| [ ] Are there no unnecessary abstractions?                |
| [ ] Would a junior developer understand this?             |
| [ ] Is the solution proportional to the problem?          |
| [ ] Have I avoided speculative generality?                |
+-----------------------------------------------------------+
```

### When Complexity is Justified

Not all complexity is bad. Some problems are inherently complex. KISS does not
mean oversimplifying -- it means not adding unnecessary complexity.

```
Justified Complexity Examples:

  - Distributed consensus algorithms (inherently complex problem)
  - Cryptographic implementations (security demands precision)
  - Real-time systems (timing constraints add necessary complexity)
  - Regulatory compliance (legal requirements dictate structure)
```

The key distinction: essential complexity (demanded by the problem) versus
accidental complexity (introduced by the solution). KISS targets only
accidental complexity.

---

## Summary

KISS is not about writing clever one-liners or avoiding all abstraction. It is
about choosing the simplest design that correctly solves the problem at hand.
Complexity should be a conscious, justified choice -- never the default.

"Simplicity is the ultimate sophistication." -- Leonardo da Vinci
